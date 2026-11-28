5.5 Cierre de Viaje
-------------------

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Operaciones › Tráfico › Viajes · TMS › Operaciones › Tráfico › Viajes por validar ·
   TMS › Operaciones › Tráfico › Órdenes · botón «Manage» de las listas de Órdenes y de Paradas

.. figure:: /_static/img/5_operational-flows/5_5_trip-closing_01_cierre.png
   :alt: Cierre de un viaje

   Viaje cerrado tras completarse sus paradas.

Nadie cierra un Viaje (``tms.trip``) a mano. El cierre es la consecuencia de que su última
Parada haya terminado, y arrastra en cadena al Viaje, a su :term:`Orden de compra` (OC) y a las
Órdenes que transportaba. Esta sección cuenta esa cadena, con el paso de revisión que un
Proyecto puede exigir antes de facturar, y lo que viene después cuando alguna entrega ha
fallado (5.5.5). Qué hace cada botón de la cabecera del Viaje está en :ref:`viaje-validacion`, y
cómo encajan los estados de los cuatro niveles, en :ref:`ciclo-estados`.

5.5.1 Qué dispara el cierre
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Una Parada está cerrada cuando tiene uno de los seis :term:`estados de cierre <Estados de la
Parada>`. El cierre operativo no exige que todo haya salido bien: una Parada fallida está tan
cerrada como una completada. Lo que indica es que ya no queda nada por ejecutar.

Cuando **todas** las Paradas de ruta del Viaje están cerradas, el sistema recalcula el Viaje. Lo
hace cada vez que cambia el estado de una Parada, así que el cierre ocurre en el momento en que
la app registra el último resultado. Si todas fallaron, el Viaje queda **Fallido** y su OC se
cancela; en cualquier otra combinación, el Viaje cierra.

5.5.2 La cadena de efectos
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. mermaid::

   flowchart LR
       P["Última Parada<br/>cerrada"] --> V{"¿El Proyecto<br/>exige revisión?"}
       V -->|"no"| T["Viaje<br/>tarifica el coste · bloquea la OC<br/>→ Completado"]
       V -->|"sí"| W["Viaje<br/>tarifica la venta<br/>→ Por validar"]
       W -->|"Validar"| T
       P --> O["Orden<br/>marcada pendiente de confirmar"]
       O -->|"tarificación"| C["Orden<br/>tarificada · confirmada<br/>→ Pedido de venta"]
       W -. "la confirmación<br/>espera" .-> C
       T --> L["Liquidación al transportista<br/>(5.6)"]
       C --> F["Facturación al cliente<br/>(5.7)"]

**En el Viaje, sin revisión.** Si su OC no está confirmada ni facturada, el sistema tarifica el
coste con la tarifa del transportista (o el precio cerrado), fija en la OC las cantidades
recibidas y a facturar, la confirma y bloquea, y deja el Viaje en **Completado**. También anota
el Viaje en el historial del conductor con el vehículo (:doc:`5_3_resource-assignment`). Si el
transportista es de flota propia no hay OC (ver
:doc:`/17.0/8_economic-administration/8_1_active-passive-margin`) y el Viaje pasa a Completado
sin más.

**En el Viaje, con revisión.** Si el Proyecto del Viaje tiene activado **Los viajes requieren
validación** (:doc:`/17.0/4_parametrization/4_5_project-configuration`), el Viaje no cierra: se
queda en **Por validar**. En ese momento se tarifica el lado de la venta, para que el revisor vea
los importes, pero el coste, la OC y la confirmación de las Órdenes esperan. Un usuario con el
permiso **Validador de viajes** lo revisa desde **Tráfico › Viajes por validar** (la lista los
ordena del más antiguo al más reciente y los marca en rojo a partir de tres días) y pulsa
**Validar**, que ejecuta entonces la cadena de arriba, o **Rechazar**, que lo deja marcado con el
motivo hasta que se corrija lo que haga falta y se valide. Si el Proyecto exige además **prueba
de entrega**, la validación se niega mientras alguna Orden no tenga firma ni albarán escaneado;
solo un responsable puede validar en ese caso, dejando el motivo. Las Órdenes sin prueba se
listan en la pestaña **Revisión** del Viaje.

**En la Orden.** Cuando todos sus Tramos están cerrados y el tramo actual terminó en Completada,
Con reservas, Fallida, Devuelta o Destruida, la Orden queda marcada como **pendiente de
confirmar** (ver :doc:`/17.0/3_functional-architecture/3_2_1_orders`).
La tarea programada de tarificación (ver :doc:`5_6_settlement`) la recoge, vuelve a comprobar
que está lista, calcula sus líneas de venta con la tarifa del Proyecto y la **confirma**: pasa de
*Presupuesto enviado* a *Pedido de venta*, el estado nativo de Odoo, y, si la compañía bloquea
los pedidos confirmados, queda bloqueada. El desfase de unos minutos entre el cierre y la
confirmación es normal. En un Proyecto con revisión, la Orden espera a que el Viaje se valide.

Los efectos económicos del cierre no siempre corren en el instante del último evento: cuando el
evento llega desde la app, el estado del Viaje cambia en el acto y el dinero (tarifa de compra,
OC, historial) lo ejecuta la misma tarea programada de tarificación unos minutos después, para
que un cierre con muchas Paradas a la vez no retrase a la app. Un Viaje cuyo dinero falla (una
línea a cero, por ejemplo) conserva la marca y se reintenta en la pasada siguiente sin parar a
los demás.

A partir de aquí el flujo es económico: la OC confirmada es la base de la liquidación al
transportista (:doc:`5_6_settlement`) y la Orden confirmada, la base de la factura al cliente
(:doc:`5_7_invoicing`).

5.5.3 Reapertura
~~~~~~~~~~~~~~~~

La cadena funciona también hacia atrás. Si el conductor vuelve a abrir una Parada cerrada, el
Viaje regresa a **En curso** y, si su OC estaba confirmada, el sistema la devuelve a solicitud
de presupuesto con las cantidades a cero; la Orden, si estaba confirmada, vuelve a *Presupuesto
enviado*. Es el equivalente automático de **Revertir validación**, que hace lo mismo a mano
sobre un Viaje completado cuya OC no esté facturada (:ref:`viaje-validacion`).

Un caso frecuente de reapertura de la Orden es **reprogramar una entrega fallida**. La Parada y
el Tramo que fallaron se quedan Fallidos y su Viaje cierra con ellos; lo que se reabre es la
Orden, porque recibe un Tramo nuevo, el siguiente intento, con una Parada sin planificar. No se
confirma hasta que ese intento termine. Es una de las salidas de la gestión del fallo (5.5.5).

Otro caso es el **Viaje repartido**: cuando una ruta se cae a media jornada y sus Paradas
pendientes se llevan a otras con *Repartir este viaje* (:doc:`5_2_trip-generation`), el Viaje
original se queda solo con lo que ya entregó. Como todas sus Paradas están cerradas, cerraría en
el acto; para que el reparto se pueda deshacer, el Viaje se mantiene **En curso** con la marca
«repartido» hasta que alguien pulsa **Cerrar viaje repartido**, se revierte el reparto o termina
su día, momento en que una tarea programada diaria lo cierra con la cadena normal.

5.5.4 Lo que el cierre protege
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Una vez **facturada** la OC, el Viaje no se reabre ni por la app ni a mano; una vez
**facturada** la Orden, tampoco ella. El sistema deja de recalcular sus estados y trata la
operación como definitiva: costes, importes, recursos y estados quedan como estaban en el
momento de facturar. Si algo tiene que cambiar después, el camino es el contable (abono y nueva
factura, ver :doc:`5_7_invoicing`), no el operativo.

.. _gestion-del-fallo:

5.5.5 Cuando una entrega falla: la gestión del fallo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Que el Viaje cierre no significa que todo esté resuelto: una Parada Fallida cierra el Viaje, pero
la mercancía sigue sin entregar y alguien tiene que decidir qué se hace con ella. Eso es la
:term:`Gestión del fallo`. Se abre sola en cuanto se registra el fallo, venga de la app del
conductor, de una agencia, de una integración o de un asistente de la oficina, y deja la Orden
señalada hasta que se resuelve. El fallo en sí no se toca: el Tramo y la Parada se quedan
Fallidos y lo que se decida es un Tramo nuevo (:ref:`ciclo-fallo`).

.. mermaid::

   flowchart LR
       F["Entrega<br/>Fallida"] --> G["Mercancía<br/>por volver"]
       G -->|"entrada en almacén"| D["Por decidir"]
       D -->|"Reprogramar"| R["Intento siguiente<br/>(Parada sin planificar)"]
       D -->|"Devolver al cliente"| V["Tramo de Devolución"]
       D -->|"Destruir"| X["Tramo de Destrucción"]
       D -->|"Cerrar sin más"| N["Fin: Orden Fallida"]
       R --> E["En curso"]
       V --> E
       X --> E
       E -->|"se entrega · se devuelve ·<br/>se cierra la destrucción"| Z["Fin"]

.. note::

   Parte de las etiquetas de estas pantallas está todavía en inglés en la interfaz en español.
   Aquí se citan entre comillas tal como salen («Manage», «Manage failure», «Fail and
   reschedule»…), con su significado al lado.

5.5.5.1 La señal de la Orden
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

La Orden con un fallo abierto lleva el :term:`Seguimiento del fallo`, que se ve como un aviso
amarillo en su ficha y como columna y filtros en la lista de Órdenes (grupo «Failure
Follow-up»):

.. list-table::
   :header-rows: 1
   :widths: 24 46 30

   * - Valor
     - Qué significa
     - Filtro
   * - **Mercancía por volver**
     - La entrega falló y la mercancía no consta de vuelta en el almacén: sigue con el conductor.
     - «Goods to come back»
   * - **Por decidir**
     - Falta elegir salida: reprogramar, devolver, destruir o cerrar.
     - «To decide»
   * - **En curso**
     - Decidido; el reintento, la devolución o la destrucción todavía no ha terminado.
     - «Follow-up in progress»

El aviso de la ficha añade, cuando los hay, la **propuesta** de las reglas (5.5.5.6) y la marca
de **plazo de almacén pasado**. La señal se apaga sola cuando el reintento se entrega, la
devolución llega o la destrucción se cierra.

No toda anotación de fallo abre la gestión:

- Una incidencia de **Bulto** no la abre: es de la mercancía, no de la entrega.
- En una Orden **facturada** no se abre sola. Se puede abrir a mano, seleccionándola en la lista
  y lanzando Acción › «Open failure management», que sirve también para los fallos anteriores a
  esta función.
- Si el conductor vuelve y **acaba entregando** la Parada que había fallado, la gestión se cierra
  y la señal se apaga sin que nadie haga nada.

.. CAPTURA: 5_5_02 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/5_operational-flows/5_5_trip-closing_02_seguimiento.png
      :alt: Ficha de una Orden con el aviso de seguimiento del fallo y su propuesta

      El aviso de seguimiento del fallo en la ficha de la Orden.

5.5.5.2 La mercancía tiene que volver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Antes de reintentar, el almacén confirma que la mercancía ha vuelto. Se hace en bloque:
seleccionar las Órdenes en la lista (el filtro «Goods to come back» las reúne) y lanzar Acción ›
«Confirm goods in warehouse». La Orden pasa a **Por decidir**, o a **En curso** si ya se había
decidido.

Que la mercancía «vuelve» lo dice el **motivo** del fallo, con una casilla
(:doc:`/17.0/4_parametrization/4_1_operational-configuration`): un «Destinatario ausente» la
trae de vuelta; un «La mercancía no ha entrado en el Hub» no, porque nunca salió. Si la entrega
falló en manos de una **agencia**, la mercancía tampoco se espera: está en el almacén de la
agencia, que el sistema no ve.

Si alguien despacha un reintento sin que conste la entrada, el despacho se detiene con un diálogo
(«Goods not back in the warehouse») que lista las Órdenes afectadas, con su intento, la fecha y
el motivo del fallo, y ofrece dos salidas:

- **«Confirm entry and dispatch»**: confirma la entrada y despacha.
- **«Dispatch without these orders»**: saca esas Paradas del Viaje, que conservan su fecha, y
  despacha el resto.

El despacho automático de un Viaje de agencia no pregunta: no lo despacha y lo anota en el
historial del Viaje, que se queda Planificado.

5.5.5.3 Decidir: «Manage»
^^^^^^^^^^^^^^^^^^^^^^^^^

El botón de la lista de Órdenes y la acción de la lista de Paradas se llaman **«Manage»**
(«Gestionar»), y deciden por sí solos qué abrir según lo seleccionado: si son Órdenes con un
fallo por decidir, el asistente **«Manage failure»**; si son Paradas por planificar, el asistente
Asignar de siempre (:doc:`5_2_trip-generation`). Con una mezcla de las dos cosas, se niega y dice
cuántas hay de cada tipo. En la ficha de la Orden, el botón «Manage failure» aparece mientras hay
un fallo por decidir.

El asistente muestra, por cada Orden, los intentos que lleva, el plazo de almacén y la propuesta
de las reglas, y pide una de cuatro salidas:

.. list-table::
   :header-rows: 1
   :widths: 22 78

   * - Salida
     - Qué hace
   * - **Reprogramar**
     - Crea el intento siguiente: un Tramo y una Parada nuevos, sin planificar, para la fecha
       indicada. Vacía, el siguiente día de servicio de la zona de cada Parada, a partir de
       mañana. Con una sola Orden se puede cambiar además la franja y la dirección de entrega.
       El Destinatario recibe el aviso de «reprogramado» si el flujo existe (capítulo 11).
   * - **Devolver al cliente**
     - Crea un Tramo de Devolución (5.5.5.4).
   * - **Destruir**
     - Crea un Tramo de Destrucción, en la dirección que se indique en **Destruir en** o en la
       de la política (5.5.5.5).
   * - **Cerrar sin más**
     - Cierra la gestión. La Orden se queda Fallida y la señal se apaga.

El asistente preselecciona Reprogramar solo cuando todas las Órdenes elegidas lo tienen como
propuesta y ninguna ha llegado al máximo de intentos. Sin regla que proponga no preselecciona
nada, y aplicar sin elegir se niega: la decisión es siempre de una persona.

Si la Orden ha alcanzado el :term:`Máximo de intentos`, Reprogramar se niega con un aviso. Solo
un **Responsable** del TMS puede marcar «Force another attempt» para forzar uno más, y queda
anotado en la Orden.

Reprogramar tiene tres efectos que conviene conocer:

- **Tarifa.** Una Orden por facturar se reabre y se tarifica de nuevo entera: el importe por
  entrega sale **una vez por intento**. Una Orden ya **facturada** no recibe línea nueva; una nota
  en su historial dice que ese intento se cobra a mano (:doc:`5_6_settlement`).
- **Arrastre de agencia.** Si la Parada fallida iba en un arrastre ya despachado, la nueva entra
  en ese Viaje, que queda pendiente de **Reenviar**; el asistente lo avisa y nombra el Viaje.
- **No se deshace.** Devolver y destruir sí (5.5.5.5); reprogramar, no.

.. CAPTURA: 5_5_03 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/5_operational-flows/5_5_trip-closing_03_gestionar-fallo.png
      :alt: Asistente Manage failure con las cuatro salidas y la propuesta de cada Orden

      El asistente «Manage failure»: intentos, plazo, propuesta y las cuatro salidas.

5.5.5.4 Devolver al cliente
^^^^^^^^^^^^^^^^^^^^^^^^^^^

La devolución es un Tramo nuevo de la misma Orden, con propósito **Devolución**. Carga donde
está la mercancía y descarga en la **Dirección de devolución** de la :term:`Política operativa`
o, si no la hay, en el punto de carga original de la Orden. Su Parada nace sin planificar, para
la fecha del asistente (vacía, mañana) y con el día entero de esa dirección como franja; varias
devoluciones del mismo lote a la misma dirección comparten Parada.

A partir de ahí es una Parada como cualquier otra: se asigna a un Viaje, se despacha y el
conductor la reporta en la app como una entrega. Cuando la cierra, la Parada queda Completada y
**el Tramo y la Orden quedan Devueltos**.

Tres límites:

- Un **Tramo directo** (sin paso por Hub) no se puede devolver con este flujo; destruirlo en el
  Hub, sí.
- Si la dirección de devolución que resulta es **donde ya está la mercancía**, el asistente se
  niega y pide que se fije una en la política.
- La devolución de una entrega que falló en una **agencia** carga en el Hub propio; nada avisa
  de que la mercancía está todavía en el almacén de la agencia.

5.5.5.5 Destruir
^^^^^^^^^^^^^^^^

La destrucción es un Tramo con propósito **Destrucción**, hacia la dirección elegida en el
asistente o la **Dirección de destrucción** de la política. Tiene dos formas:

- **En el mismo Hub donde está la mercancía.** No hay transporte: el Tramo no tiene Parada ni
  Viaje, ni genera línea de tarifa.
- **Con transporte.** Parada sin planificar hacia el sitio, que se planifica y se reporta como
  la de una devolución. Entregada allí, la Orden sigue **En curso**: falta acreditarla.

En los dos casos la destrucción se cierra con el botón **Cerrar destrucción**, en el aviso de la
Orden o en el Tramo. Pide **el documento que la acredita**, la fecha y un comentario; sin
documento no cierra, y con transporte tampoco mientras la Parada no esté cerrada. Al cerrarla, el
Tramo, y con él la Orden, quedan **Destruidos**, y el documento queda adjunto a la anotación de
Trazabilidad. Si la mercancía no constaba de vuelta en el almacén, el asistente ofrece
**Confirmar entrada y cerrar**.

Una destrucción entregada en el sitio y sin cerrar no la reclama nadie: se localiza con el
filtro «Follow-up in progress».

**Deshacer decisión.** Mientras el Tramo de devolución o de destrucción no esté despachado, el
aviso de la Orden ofrece **Deshacer decisión**: el Tramo nuevo se cancela, su Parada sale del
Viaje y la Orden vuelve a **Por decidir**. Una vez despachado, se niega.

Mientras una Orden tenga viva una devolución o una destrucción, **no se puede Validar** de nuevo:
validar reconstruiría sus Paradas. Antes hay que deshacer la decisión.

.. CAPTURA: 5_5_04 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/5_operational-flows/5_5_trip-closing_04_cerrar-destruccion.png
      :alt: Asistente Cerrar destrucción con el documento, la fecha y el comentario

      Cerrar destrucción: el documento es obligatorio.

5.5.5.6 Lo que propone el sistema: reglas, instrucción y plazo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

El sistema **propone y nunca aplica**. La propuesta sale, por este orden, de:

#. La :term:`Instrucción de la Orden`, si el cliente dejó dicho qué hacer con esa Orden.
#. Las :term:`reglas de fallo <Regla de fallo>` de la política: por motivo y por número de
   intentos. Por ejemplo, «ausente, hasta el segundo intento: reprogramar; desde el tercero:
   destruir» y «rechazado: devolver».

La propuesta se ve en el aviso de la Orden, en el asistente y en los filtros «Proposed:
reschedule / return / destroy», con los que se trabaja por tandas: filtrar las que proponen
reprogramar, seleccionarlas todas y aplicar.

El :term:`Plazo de almacén` cuenta los días desde el fallo. Si vence con la decisión pendiente,
una tarea diaria marca la Orden («Storage deadline passed», con su filtro), cambia la propuesta
a la que la política tenga para ese caso y crea **una** actividad para el responsable del
Proyecto. Decidir da la actividad por hecha. Un Proyecto sin responsable no recibe actividad: la
Orden queda solo en el filtro.

Todo esto se configura en la política
(:ref:`4.3.9 <politica-operativa>`).

5.5.5.7 Fallar y reprogramar en bloque
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cuando lo que falla no es una entrega sino una tanda (el camión no sale, el arrastre no llega,
se cae una ruta), el modo **«Fail and reschedule»** de «Manage» falla y reprograma muchas Paradas
de una vez. Se lanza desde las listas de Paradas y de Órdenes, desde el botón del mismo nombre
en un Viaje (sus Paradas pendientes) o sobre un Viaje de arrastre (las entregas de las Órdenes
que llevaba).

- **Motivo**: obligatorio y sin valor por defecto. Solo ofrece motivos de fallo y recuerda el
  último que usó cada usuario.
- **Fecha**: opcional. Vacía, el siguiente día de servicio de cada Parada. La franja no se
  pregunta: cada una conserva la suya.
- **Lo que se queda fuera**, listado con su causa: lo facturado, lo ya reportado, las Órdenes
  que han llegado al máximo de intentos (se gestionan de una en una) y las repetidas.
- **Avisos**: el Destinatario recibe **uno solo**, el de «reprogramado», no el de «fallido» y
  después otro.
- **Tarifa**: no se calcula en el momento. Las Órdenes quedan reabiertas y la tarea de
  tarificación las recoge en sus pasadas, así que una tanda grande tarda unos minutos en tener
  todas sus líneas.
- Las Paradas que **aún no se habían despachado** salen de su Viaje y fallan sin él: su
  mercancía no se espera de vuelta, porque nunca salió.

5.5.5.8 La promesa al Destinatario
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cuando al Destinatario se le ha dicho un día y una franja, la Parada lleva una :term:`Promesa`
(la anota el aviso al enviarse; ver :doc:`/17.0/11_notifications/11_3_workflows`). Desde ese
momento:

- **No se le cambia la fecha ni la franja a mano.** El sistema lo niega y recuerda lo
  prometido. El camino es «Manage» › «Fail and reschedule», que deja constancia y avisa al
  Destinatario.
- **Los procesos automáticos no fallan por ella.** El cierre de un Manifiesto o una tarea
  programada que quisieran moverla saltan esos campos, hacen el resto y dejan una nota en la
  Orden. Re-cerrar un Manifiesto con otro día para una Parada prometida la respeta, y el Viaje
  que la lleva conserva su fecha.
- **Sacarla de un Viaje ya despachado pregunta.** La X de la pestaña Paradas, la de la pestaña
  Ruta y la del mapa, «Desasignar y continuar» y **Cancelar viaje** abren una pregunta con la
  fecha prometida y tres salidas:

  .. list-table::
     :header-rows: 1
     :widths: 34 66

     * - Salida
       - Qué pasa
     * - «They go out today» (salen hoy en otro Viaje)
       - Abre Asignar. La Parada no sale de su Viaje hasta que otro la toma.
     * - «No trip, to replan today» (sin Viaje, para replanificar hoy)
       - Sale con su fecha y su promesa, y aparece en el filtro «Promised without trip».
     * - «They do not go out today» (hoy no salen)
       - Fallar y reprogramar en bloque. El Viaje no se cancela: se queda con lo servido y con
         las fallidas, y cierra con ellas.

Dos filtros de la lista de Paradas vigilan las promesas: **«Promised without trip»** (prometidas
que se han quedado sin Viaje) y **«Promised without outcome»** (prometidas cuyo día ha llegado o
pasado sin resultado).

5.5.5.9 Lo que no hacen una devolución ni una destrucción
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sus Paradas viajan con las demás, pero no son entregas:

- El **Destinatario no recibe avisos** de ellas (capítulo 11).
- **No cuentan en los indicadores** de Parada ni en la primera entrega
  (:doc:`5_8_kpi-indicators`).
- **No se envían a las agencias** ni a sistemas de seguimiento externos como entregas
  (:doc:`/17.0/7_edi-integrations/7_3_api-integrations`).
- El **DeCA** trata la destrucción en el Hub como una operación sin transporte
  (:doc:`/17.0/9_deca/9_3_obligation`).
- En la **app**, el conductor ve la devolución como una entrega más.
- La **torre de control** todavía no las distingue: una devolución cerrada cuenta en sus paneles
  como una entrega hecha.

5.5.5.10 Referencia técnica
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 34 66

   * - Concepto
     - Dónde está
   * - Señal de la Orden, decisión y propuesta
     - ``sale.order``: ``tms_followup``, ``tms_followup_resolution``, ``attempt_count``,
       ``tms_failure_instruction``, ``tms_failure_proposal``, ``tms_storage_overdue``
   * - Intento, propósito, mercancía y decisión
     - ``tms.shipment.leg``: ``attempt_number``, ``leg_purpose``, ``previous_leg_id``,
       ``next_leg_id``, ``goods_state`` (``not_applicable``, ``out``, ``in_warehouse``,
       ``gone``), ``resolution`` (``pending``, ``reschedule``, ``return``, ``destroy``,
       ``none``), ``resolution_deadline``, ``storage_overdue``
   * - Promesa
     - ``tms.stop``: ``promise_at``, ``promise_date``, ``promise_start``, ``promise_end``
   * - Casillas del motivo
     - ``tms.traceability.event.reason``: ``goods_come_back``, ``counts_as_attempt``
   * - Reglas, máximo, plazo y direcciones
     - ``tms.failure.rule``; ``tms.operations.policy``: ``failure_rule_ids``,
       ``max_delivery_attempts``, ``storage_max_days``, ``return_address_id``,
       ``destruction_address_id``
   * - Asistentes
     - ``tms.stop.action`` (modos «Manage failure» y «Fail and reschedule»),
       ``tms.stop.promise.question``, ``tms.trip.goods.check.wizard``,
       ``tms.destruction.close.wizard``
   * - Tarea programada
     - «TMS: failures past their storage deadline», diaria
   * - Documento de la destrucción
     - adjunto de tipo ``DestructionProof`` en la anotación ``destroyed``
