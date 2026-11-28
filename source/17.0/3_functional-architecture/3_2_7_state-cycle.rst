.. _ciclo-estados:

3.2.7 El ciclo de estados en todos los niveles
----------------------------------------------

Cada entidad del modelo lleva su estado, y cada sección lo describe para la suya: la Orden en
:doc:`3_2_1_orders`, el Viaje en :doc:`3_2_2_trips`, el Tramo y la Parada en
:doc:`3_2_6_legs-and-stops`. Esta sección los pone **uno al lado del otro**: qué momento de la
operación mueve cada nivel y hasta dónde. Es la que hay que tener delante cuando una Orden dice
una cosa y su Viaje otra.

3.2.7.1 Tres reglas
~~~~~~~~~~~~~~~~~~~

Los estados no se escriben a mano: el sistema los deriva de la última anotación de
:term:`Trazabilidad` de cada entidad (Tramo y Parada), del tramo activo (Orden) y de las Paradas
(Viaje). Tres reglas gobiernan el ciclo:

#. **Planificar mueve solo la Parada.** Asignarla a un Viaje la deja **Planificada**; quitarla
   del Viaje la devuelve a **Sin planificar**. La Orden y el Tramo no se enteran: siguen en
   **Validado** desde que se validó la Orden o se cerró el Manifiesto. Por eso borrar un Viaje
   nunca deja una Orden sin validar.
#. **Despachado solo lo escribe el despacho.** El botón **Despachar** del Viaje (ver
   :doc:`/17.0/5_operational-flows/5_4_app-execution`) es el único que lleva a **Despachado**, y
   lo hace en los cuatro niveles a la vez: Viaje, Paradas, Tramos y Órdenes. Una Parada que
   entra en un Viaje ya despachado se despacha con él.
#. **«¿Está planificada?» se mira en el Viaje, no en el estado.** Una Parada está planificada si
   tiene Viaje. El estado la acompaña, pero la pregunta que hacen el asistente de asignación, el
   Optimizador de Paradas y las guardas de borrado es si la Parada tiene Viaje y si ese Viaje
   está aún sin despachar.

3.2.7.2 La matriz: qué mueve cada momento
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cada fila es un momento de la operación; cada columna, un nivel. Una celda vacía significa que
ese momento no toca ese nivel. Los nombres son los de esta documentación; la etiqueta que muestra
Odoo está en la referencia técnica de cada sección.

.. list-table::
   :header-rows: 1
   :widths: 17 11 15 15 14 14 14

   * - Momento
     - Manifiesto
     - Pedido de venta
     - Orden y Tramo
     - Parada
     - Viaje
     - Orden de compra
   * - **Crear la Orden** (a mano, por fichero o por API)
     - Abierto
     - Presupuesto
     - Borrador
     -
     -
     -
   * - **Validar la Orden / Cerrar el Manifiesto** (nacen las Paradas)
     - Cerrado
     - Presupuesto enviado
     - **Validado**
     - Sin planificar
     -
     -
   * - **Planificar** (la Parada entra en un Viaje)
     -
     -
     - Validado
     - **Planificada**
     - Planificado
     - Solicitud de presupuesto
   * - **Despachar** (el Viaje sale al conductor o a la agencia)
     -
     -
     - **Despachado**
     - **Despachada**
     - **Despachado**
     - Solicitud enviada
   * - **El conductor actúa** (en camino, en el punto, carga)
     -
     -
     - En curso · Cargado
     - En curso · Cargada
     - **En curso**
     -
   * - **Resultado** (todas las Paradas cerradas)
     -
     - Pedido de venta
     - Completado · Con reservas · Fallido · Devuelto · Destruido · Cancelado
     - los mismos
     - **Por validar** (si el Proyecto pide revisión) · Completado · Fallido
     - Pedido de compra, bloqueada · Cancelada si el Viaje falla

Tres lecturas de la matriz:

- **La Orden y el Tramo se quedan en Validado mientras se planifica.** Lo que cambia entre
  «validar» y «despachar» es solo la Parada (y el Viaje, que existe desde que tiene Paradas).
- **El pedido de venta va por detrás.** Validar la Orden la deja en *Presupuesto enviado* (sin
  enviar ningún correo: es solo el estado); reabrir el Manifiesto la devuelve a *Presupuesto*.
  Pasa a *Pedido de venta* cuando todos sus Tramos están cerrados **y el Tramo actual terminó en
  Completado, Con reservas, Fallido, Devuelto o Destruido**. Si la entrega fallida se reprograma,
  el Tramo actual pasa a ser el del nuevo intento, que está abierto, y la Orden vuelve a
  *Presupuesto enviado* hasta que ese intento termine (3.2.7.4). Lo hace la tarea programada de
  tarificación, no el evento de la app, por lo que hay unos minutos de desfase
  (:doc:`/17.0/5_operational-flows/5_6_settlement`).
- **El Viaje tiene dos estados que los demás no tienen.** *En curso* aparece en cuanto el
  conductor (o la API de la agencia) toca una Parada, aunque el Viaje no se hubiera despachado.
  *Por validar* solo existe en los Proyectos que exigen revisar el Viaje antes de facturar; el
  resto pasa directamente a *Completado* (ver :doc:`/17.0/5_operational-flows/5_5_trip-closing`).

3.2.7.3 La escalera del Viaje y sus vueltas atrás
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El Viaje sube por una escalera fija, y cada peldaño que se sube a mano tiene su botón para
bajarlo. La cabecera del Viaje solo muestra el botón que se puede pulsar en ese estado.

.. mermaid::

   flowchart LR
       D["Sin Paradas"] --> P["Planificado"]
       P -->|"Despachar"| S["Despachado"]
       S -->|"el conductor actúa"| R["En curso"]
       R -->|"todas las Paradas cerradas"| V["Por validar"]
       V -->|"Validar"| C["Completado"]
       R -->|"todas las Paradas cerradas,<br/>sin revisión"| C
       R -->|"todas fallidas"| F["Fallido"]
       S -. "Revertir despacho" .-> P
       C -. "Revertir validación" .-> V
       S -. "Cancelar viaje" .-> P
       R -. "Cancelar viaje<br/>(salen las Paradas no servidas)" .-> C

- Un Viaje **sin Paradas de ruta no existe**: cuando se le quita la última, se borra, salvo que
  su Orden de compra esté confirmada o facturada.
- **Revertir despacho** solo se admite en *Despachado*: en *En curso* el conductor ya ha actuado.
  Devuelve las Paradas a Planificada y los Tramos a Validado y, si el Viaje era de agencia, avisa
  a la agencia; si la agencia no lo acepta, no se revierte.
- **Cancelar viaje** no es un estado: las Paradas que nadie ha servido salen del Viaje y vuelven
  a Sin planificar. Si no se sirvió ninguna, el Viaje desaparece con su Orden de compra; si
  alguna se sirvió, el Viaje se queda con ellas y cierra con ellas, porque eso se paga.
- **Revertir validación** devuelve un Viaje completado a *Por validar* (o lo reabre, si el
  Proyecto no pide revisión) y su Orden de compra a solicitud de presupuesto. No es posible con
  la Orden de compra facturada.
- Si el conductor **reabre una Parada** cerrada, el Viaje vuelve a *En curso* por sí solo y la
  Orden de compra, a solicitud; es la reapertura automática de
  :doc:`/17.0/5_operational-flows/5_5_trip-closing`.
- Un Viaje cuya Orden de compra está **facturada** no se recalcula ni se borra.

.. _ciclo-fallo:

3.2.7.4 Cuando una entrega falla: el fallo permanece
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Un fallo no se borra con lo que venga después. El Tramo y la Parada que fallaron **se quedan
Fallidos para siempre**, y lo que se decida hacer con la mercancía es **otro Tramo**, encadenado
con el anterior. Así el historial de la Orden cuenta lo que pasó de verdad: cuántas veces se
intentó, por qué falló cada una y cómo acabó.

.. mermaid::

   flowchart LR
       I1["Tramo · intento 1<br/>Fallido"] --> D{"Decisión<br/>(Gestión del fallo)"}
       D -->|"Reprogramar"| I2["Tramo · intento 2<br/>Entrega<br/>Validado → … → resultado"]
       D -->|"Devolver al cliente"| R["Tramo de Devolución<br/>→ Devuelto"]
       D -->|"Destruir"| X["Tramo de Destrucción<br/>→ Destruido"]
       D -->|"Cerrar sin más"| N["La Orden se queda<br/>Fallida"]

- **Cada entrega es un** :term:`Intento` **numerado.** Reprogramar crea el intento siguiente: un
  Tramo y una Parada nuevos, sin planificar, que recorren el ciclo desde Validado. El Tramo
  anterior deja de ser el :term:`tramo activo <Tramo activo>` y conserva su estado Fallido. La
  constancia de que se reprogramó es una :term:`traza informativa <Traza informativa>` en el
  Tramo y la Parada viejos: se ve en la Trazabilidad y en los avisos, y no cambia su estado.
  **Reprogramado** como estado ya no lo escribe nadie; aparece solo en operaciones antiguas.
- **Devolver y destruir crean un Tramo con otro** :term:`propósito <Propósito del Tramo>`. No es
  una entrega ni cuenta como intento. El de **Devolución** se planifica, se despacha y se reporta
  como cualquier Parada; cuando el conductor la cierra, la Parada queda Completada y **el Tramo
  y la Orden leen Devuelto**. El de **Destrucción** acaba en **Destruido** solo cuando alguien
  cierra la destrucción con el documento que la acredita; hasta entonces, entregada en el sitio,
  sigue *En curso*. Si la mercancía se destruye en el mismo Hub donde está, el Tramo no tiene
  Parada ni Viaje.
- **La Orden lee siempre su Tramo actual.** Con un intento nuevo abierto, vuelve a estar abierta;
  con la devolución entregada, Devuelta; con la destrucción cerrada, Destruida. Mientras tanto
  lleva el :term:`Seguimiento del fallo`, que dice en qué punto está la gestión.
- **Lo facturado no se recalcula.** Una Orden facturada conserva su pedido de venta y sus
  líneas; de la gestión del fallo solo recibe el estado del resultado.

El flujo de trabajo (quién decide, con qué asistente y qué pasa con la mercancía) está en
:ref:`gestion-del-fallo`.

3.2.7.5 Referencia técnica
~~~~~~~~~~~~~~~~~~~~~~~~~~

Los códigos son los mismos en los cuatro niveles hasta el despacho; la etiqueta cambia según
el nivel. Un mismo código, ``in_process``, se lee *Validado* en la Orden y el Tramo y
*Planificado* en la Parada y el Viaje.

.. list-table::
   :header-rows: 1
   :widths: 16 22 22 22 18

   * - Código
     - Orden (``tms_status``) y Tramo (``state``)
     - Parada (``state``)
     - Viaje (``state``)
     - Nombre en la documentación
   * - ``draft``
     - Borrador
     - Borrador
     - Borrador
     - Borrador · Sin planificar (Parada) · Sin Paradas (Viaje)
   * - ``in_process``
     - Validado
     - Planificado
     - Planificado
     - Validado · Planificada · Planificado
   * - ``processed``
     - Despachado
     - Despachado
     - Despachado
     - Despachado · Despachada
   * - ``running``
     - En curso
     - En curso
     - En curso
     - En curso
   * - ``loaded``
     - Cargado
     - Cargado
     - —
     - Cargado · Cargada
   * - ``to_validate``
     - —
     - —
     - Por validar
     - Por validar
   * - ``completed``, ``reserves``, ``failed``, ``returned``, ``destroyed``, ``canceled``
     - Completado, Con reservas, Fallido, Devuelto, Destruido, Cancelado
     - los mismos
     - ``completed``, ``failed``
     - los seis :term:`estados de cierre <Estados de la Parada>`
   * - ``rescheduled``
     - Reprogramado (solo histórico)
     - Reprogramado (solo histórico)
     - —
     - no se escribe desde la gestión del fallo

Los momentos de la matriz corresponden a estos puntos del sistema: validar la Orden y cerrar el
Manifiesto escriben ``in_process`` en Tramo y Orden y dejan el pedido en ``sent``; asignar o
quitar una Parada de un Viaje escribe solo en la Parada; ``tms.trip.send_trip`` es el único punto
que escribe ``processed`` (en los cuatro niveles) y anota ``dispatch_date``; el estado del Viaje
lo calcula ``_compute_state_from_stops`` a partir de sus Paradas de ruta, de ``dispatch_date``,
de ``validation_date`` y de la marca de Viaje repartido; el pedido pasa a ``sale`` por la tarea
programada de tarificación cuando ``_tms_ready_to_confirm`` lo permite.

De la gestión del fallo: el Tramo lleva ``attempt_number``, ``leg_purpose`` (``delivery``,
``return``, ``destruction``), ``previous_leg_id`` y ``next_leg_id``; la traza informativa es
``tms.traceability`` con ``tms_informative``; el Tramo de devolución cerrado lee ``returned``
aunque su traza y su Parada digan ``completed``; ``destroyed`` lo escribe solo el cierre de la
destrucción; la señal de la Orden es ``tms_followup`` (``goods_pending``, ``to_decide``,
``in_progress``).
