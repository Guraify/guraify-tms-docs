13.2 Configurar, paso a paso
----------------------------

El orden importa: cada paso se apoya en el anterior. Casi todo lo que el optimizador lee se
puede repasar, además, desde un único menú de listas editables (13.2.14). Al final hay una lista
de comprobación.

13.2.1 El token de PTV
~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** Ajustes (generales) › Integraciones › **Rutas del mapa (PTV)** › Token.

Es por compañía: se guarda en la compañía activa. Al escribirlo se comprueba contra PTV; si no
vale, el campo se vacía y sale un aviso. Sin token, cualquier cálculo se para con «No hay token
PTV configurado para la compañía…». Detalle en
:doc:`1.4.2 </17.0/1_introduction/1_4_technological-architecture>`.

13.2.2 Proveedores geo
~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Configuración › Ajustes › bloque **Proveedores geo** › Proveedor por proceso.

Quién hace cada proceso (Routing y ETA, Secuenciación, Optimización, Estilo del mapa), cuántas
**Secuenciaciones simultáneas** salen a la vez (6 de serie; más chocan con el límite de PTV), si
se pide la **Optimización premium** y la **Distancia máxima al hub** (300 km de serie: una Parada
más lejos de su Hub, o en otro país, se considera mal geolocalizada y no se envía; 0 lo
desactiva). OptiFlow solo sabe optimizar: elegirlo para rutas o secuenciación da error. Detalle en
:ref:`proveedores-geo` y en :ref:`17.0/1_introduction/1_4_technological-architecture:1.4.2.1 Dos motores de optimización`.

13.2.3 Hubs y su horario
~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Maestros › Operaciones › Hubs (crea el contacto ya marcado como Hub). El horario,
en la pestaña **Horarios TMS** del contacto: **Tiempo por Recogida** y **Tiempo por entrega** (los
minutos fijos de cada visita al Hub), el horario por defecto de **MAÑANA** y de **TARDE** y una
fila por día de la semana (:doc:`4.1 </17.0/4_parametrization/4_1_operational-configuration>`).
Cinco reglas que se malinterpretan:

#. **La casilla de un día no significa «cerrado».** Marcarla («Lunes Mañana Abierto») dice que
   ese día tiene un horario propio, el de su fila; sin marcar, vale el horario por defecto. Una
   casilla marcada con horas 0–0 cuenta como no marcada.
#. **Un Hub sin ningún horario está abierto todo el día**, sin aviso.
#. **Si solo se configura media jornada**, la secuenciación y la optimización suponen 08–14 y
   15–20 para la otra media; el cálculo de ruta solo usa lo configurado. Conviene rellenar
   siempre las dos.
#. Mañana y tarde no pueden solaparse y cada una dura al menos una hora.
#. **La vuelta al Hub al final de la ruta no se compara con el horario**: un vehículo que vuelve
   después del cierre ni espera ni queda marcado.

**Cuándo sale el vehículo del Hub.** Una sola regla, igual para los tres servicios; se usa lo
primero que exista: el **Inicio de carga** del Viaje (la ruta sale entre esa hora y quince
minutos después); la ventana de salida del vehículo (**Salida desde** – **Salida hasta**); la de
su categoría; la de la categoría de vehículo marcada por defecto en la compañía. Si nada de eso
existe, puede salir a cualquier hora. Si la salida no coincide con ninguna apertura del Hub, la
ejecución **avisa pero no corrige**: la ruta espera a que abra. Antes de salir se suma la carga:
el tiempo fijo del Hub más el Tiempo Recogida del Planning por cada Parada que se carga allí.

13.2.4 Áreas geográficas y Plan de transporte
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Rutas:** TMS › Configuración › Áreas Geográficas · Asistente de áreas OSM · Planes de Transporte.

Las zonas operativas se crean con el **Asistente de áreas OSM** (toma los límites de
OpenStreetMap) o a mano, con Tipo «Plan de Transporte», su **Hub**, su Agencia, su **Código** y su
**Tolerancia después**; varias se unen con **Fusionar áreas**. El Plan de transporte las reúne.
Cómo recibe una Parada su zona: el plan sale del **Planning** de la Parada; la zona es la que
contiene su dirección (solo polígono, no códigos postales); si no cae en ninguna y el Proyecto
tiene **Autoasignar zona operativa más cercana**, se le da la más cercana; de la zona se copian a
la Parada el Hub y la Agencia; se recalcula al cerrar o importar un Manifiesto y al tarificar,
solo en Paradas sin ejecutar y nunca en un Viaje facturado. Lo que hace cada dato está en
:doc:`4.3.2 y 4.3.3 </17.0/4_parametrization/4_3_planning-configuration>`. Dos avisos: los **días
disponibles** de la zona no limitan la optimización (solo la fecha de un Tramo nuevo y la
reprogramación), y **sin Hub no se puede optimizar**.

13.2.5 El Planning
~~~~~~~~~~~~~~~~~~

**Rutas:** TMS › Configuración › Ajustes › bloque **Datos auxiliares** › Plannings (para crear);
en lista, Parámetros del optimizador › Plannings. Lo que el optimizador lee: el Plan de
transporte, los tiempos fijos de recogida y de entrega, la **Tolerancia después**, el **Margen
sobre el fin**, las tres **Pasadas** y el **Proveedor de optimización**
(:doc:`4.3.1 </17.0/4_parametrization/4_3_planning-configuration>`). El Modo División Viaje no
influye en la optimización. Aviso para la puesta en marcha: con tolerancia 0 y margen 0, las
pasadas 1 y 2 piden lo mismo y la segunda se salta; las pasadas solo sirven si el Planning (o la
zona) tiene minutos (13.3).

13.2.6 El Proyecto
~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Configuración › Proyectos, pestaña TMS. Lo que cuenta para planificar: el
**Planning** (obligatorio), **Secuenciar Viaje** (la secuenciación automática al cerrar el
Manifiesto y al validar la Orden), **Horarios Informados** (la franja de la Parada es la que trae
la Orden; desactivado, la del Destinatario) y la hora predeterminada cuando ni la Orden ni el
Destinatario la dan, el esquema de **Tiempo Servicio**, **Autoasignar zona operativa más
cercana**, las **Categorías de carga** y, solo para OptiFlow, el **Coste de subcontratación por
defecto** (lo que cuesta dejar una Orden sin planificar; bajo, puede dejar fuera las lejanas o
caras; vacío, planificarlo todo). Detalle en
:doc:`4.5 </17.0/4_parametrization/4_5_project-configuration>`.

13.2.7 Tiempos de servicio
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Configuración › Tiempos de Servicio. El tiempo de servicio de una Parada es lo que
da el esquema del Proyecto (según la dificultad de aparcamiento, el ascensor y las plantas del
Destinatario) **más** el tiempo fijo del Planning, todo multiplicado por el **Factor de tiempo de
servicio** del conductor. En el Hub no se aplica el factor. Detalle en
:doc:`4.3.5 </17.0/4_parametrization/4_3_planning-configuration>`.

13.2.8 Categorías de vehículo y vehículos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Rutas:** TMS › Maestros › Flota › Categorías / Vehículos, y en lista en Parámetros del
optimizador. La categoría da los valores y el vehículo los hereda: al ponerle categoría, solo se
rellena lo que tiene vacío o a 0; en los costes, vacío en el vehículo significa «el de la
categoría», y las columnas «efectivo» enseñan lo que se usará. Lo que decide cada grupo (perfil,
capacidades, límites de ruta, salida, inicio y fin, costes, conducción, forma de las rutas,
restricciones, jornada) está en :doc:`4.2.5 </17.0/4_parametrization/4_2_logistic-configuration>`.
Tres avisos: **sin Perfil Vehículo, la optimización supone tráiler y la secuenciación
furgoneta**, así que hay que rellenarlo; una capacidad a **0** es «no cabe» en Developer y en
secuenciación y «sin límite» en OptiFlow; y la conducción máxima es siempre un límite duro.

13.2.9 Conductores y Perfil de jornada
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Conductores** (Parámetros del optimizador › Conductores): el **Factor de tiempo de servicio**
(un conductor nuevo tarda más) y sus **Horas de Conducción** propias. El :term:`Perfil de
jornada` (en la interfaz, «Horas de Conducción») vive en Ajustes › bloque Optimización de ruta;
cada perfil lleva el reglamento que se envía a PTV, una duración máxima de ruta y las pausas que
planifica OptiFlow; de serie, EU 561/2006. Cuál se aplica, lo primero que exista: el del
conductor › el de la categoría del vehículo › el de la franja del Plan de disponibilidad › el por
defecto de la compañía › EU 561/2006. Se aplica **siempre**, salvo en el nivel Fin abierto sin
límites. Detalle en :ref:`horas-conduccion`.

13.2.10 Categorías de carga y equipamientos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Equipos requeridos** (la Orden solo va en vehículos que los tengan todos; si ninguno los tiene,
queda sin planificar), **Equipos incompatibles** y **Incompatible con** otra carga (solo OptiFlow;
se rellena en un lado). Detalle en :doc:`4.2.1, 4.2.2 y 4.2.6
</17.0/4_parametrization/4_2_logistic-configuration>`.

13.2.11 Destinatarios
~~~~~~~~~~~~~~~~~~~~~

En lista, Parámetros del optimizador › Destinatarios (filtro «Con envíos»): la **Prioridad** (qué
se planifica antes cuando no cabe todo; no cambia el orden de visita; OptiFlow no la mira), las
horas de mañana y tarde (la franja de sus Paradas nuevas cuando la Orden no trae una y el Proyecto
no usa Horarios Informados; con mañana y tarde, la franja va del inicio de la mañana al fin de la
tarde, sin hueco al mediodía), la **Precisión** de la geolocalización (por debajo de 80 conviene
normalizar la dirección) y la dificultad de aparcamiento, el ascensor y las plantas, que entran
en el tiempo de servicio. Las filas por día de la semana de un Destinatario no se leen: solo
cuentan en los Hubs; no existe «día cerrado» del cliente.

13.2.12 Presets de optimización
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Un :term:`Preset de optimización` dice **para qué** se optimiza y se elige en cada lanzamiento;
no fija la tolerancia ni las franjas, que son de las pasadas. Los cuatro de serie y lo que un
administrador puede tocar, en :ref:`presets-optimizacion`.

13.2.13 Plan de disponibilidad de conductores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Operaciones › Planning › Plan Disponibilidad Conductores. Cada franja del plan es,
para el optimizador, un vehículo disponible con su horario; para optimizar «Por Turno», cada
franja necesita **vehículo y tarifa de transportista**, y el fin de la franja es además la hora
límite del nivel Fin abierto. Detalle en :doc:`4.3.4 </17.0/4_parametrization/4_3_planning-configuration>`.

13.2.14 El repaso: Parámetros del optimizador
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Operaciones › Planning › **Parámetros del optimizador**. Nueve listas editables,
con edición en bloque, que reúnen lo que el optimizador lee: Categoría Vehículo, Vehículos,
Conductores, Categorías de Carga, Destinatarios, Hubs, Plannings, Proyectos y Zonas operativas.
No dejan crear ni borrar; la ayuda de cada columna sale al pasar el ratón por la cabecera. Es la
pantalla del repaso antes de la primera optimización y de la revisión periódica.

.. CAPTURA: 13_2_01 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/13_optimization-course/13_2_setup_01_plannings.png
      :alt: Lista de Plannings de Parámetros del optimizador con las columnas de pasadas

      Parámetros del optimizador › Plannings: tolerancia, margen y las tres pasadas de cada uno.

13.2.15 Política operativa (ETA)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No afecta a la optimización ni a la secuenciación: decide cuándo se recalcula el ETA de un Viaje
en marcha y si el cálculo de ruta pide peajes y emisiones (:doc:`4.3.9
</17.0/4_parametrization/4_3_planning-configuration>`, :doc:`12.3 </17.0/12_live-operations/12_3_eta>`).

13.2.16 Lista de comprobación antes de optimizar
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Token de PTV puesto en la compañía.
#. Hubs con horario de mañana **y** tarde, y con sus tiempos por visita.
#. Zonas operativas con polígono, Hub y código; Plan de transporte que las reúne.
#. Planning con su Plan de transporte, tolerancia, margen y niveles de pasada.
#. Proyecto con su Planning y la decisión sobre Secuenciar Viaje y Horarios Informados.
#. Categorías de vehículo con perfil, capacidades, ventana de salida y, para OptiFlow, costes.
#. Vehículos con categoría; conductores con su factor.
#. Plan de disponibilidad del día, si se optimiza «Por Turno».
#. Paradas geolocalizadas y con Hub.
