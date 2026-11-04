12.5 Las alarmas
----------------

.. note::

   Las alarmas **no existen todavía en Odoo** (noviembre de 2026). Lo que existe son sus
   **umbrales**, en la política operativa, y los **eventos** de telemática y de la app que las
   dispararán. El modelo de alarma, su catálogo y su evaluación están diseñados en la
   especificación de la torre de control (`02_SPEC_api_connect.md`, §6, y el sistema de diseño,
   §4.3) y es lo que aquí se describe, **como diseño**. Cuando el módulo ``tms_connect`` exista,
   este apartado pasará a contar lo que hay.

Una :term:`Alarma` es la forma en que el sistema dice «esto necesita a alguien»: un Viaje que no
ha arrancado, un ETA que ya pasó sin que el conductor haya reportado nada, un vehículo parado
donde no toca, una temperatura fuera de rango. No es un aviso al Destinatario (eso es 12.4) ni
un estado (eso es 3.2.7): es una **excepción para el operador**, que se ve, se silencia o se
resuelve.

12.5.1 Lo que ya está en Odoo: los umbrales
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La :term:`Política operativa` (:doc:`4.3.9 </17.0/4_parametrization/4_3_planning-configuration>`)
lleva desde el principio el grupo **Alarmas**, por compañía y por Proyecto. Hoy ningún proceso
de Odoo los lee, salvo el de desvío de ruta, que además es el umbral del modo de recálculo «Al
desviarse del plan». Esto es lo que cada uno significará:

.. list-table::
   :header-rows: 1
   :widths: 30 14 56

   * - Umbral
     - Por defecto
     - Qué detectará
   * - **Sin evento tras el ETA (min)**
     - 30
     - El ETA vigente de una Parada pasó hace más de N minutos y el conductor no ha reportado
       ni la llegada ni el resultado. Es la alarma de «¿qué pasa ahí?», una fila por Parada.
   * - **Vehículo parado (min)**
     - 20
     - El vehículo lleva más de N minutos sin moverse **fuera** de la geocerca de una Parada. Lo
       detecta el servicio intermedio de telemática con las posiciones y lo entrega como el
       evento *Parado demasiado tiempo*. Sin posiciones, no se dispara.
   * - **Desviación de ruta (km)**
     - 2
     - El vehículo está a más de N km del trazado planificado (evento *Fuera de la ruta
       planificada*, también del servicio intermedio). Es, además, el umbral con el que Odoo pide
       hoy un recálculo del ETA en el modo «Al desviarse del plan» (12.3.1).
   * - **Viaje sin empezar (min)**
     - 15
     - La hora de salida del Viaje pasó hace más de N minutos y no está despachado (crítica) o,
       despachado, el conductor no ha dado señales de vida (aviso).
   * - **Geocerca de llegada (m)**
     - 150
     - No es una alarma, pero las alimenta: el radio dentro del cual una posición cuenta como
       «está en la Parada». Un Tipo de Parada puede tener el suyo.

Dos umbrales más que la especificación añade a la política y que todavía no están en ella:
**margen de riesgo del ETA** (15 min: el ETA se acerca al fin de la ventana) y **antelación de
una Parada sin planificar** (120 min: una Parada de hoy sin Viaje a menos de ese tiempo de su
ventana).

12.5.2 El diseño: catálogo, evaluación y gestión
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Una alarma es un registro en Odoo**, no un cálculo de la torre: con su código, su severidad
(información, aviso, crítica), el afectado (Viaje, Parada u Orden), su estado (**nueva** →
**vista** → **resuelta**, o **silenciada** hasta una hora), cuándo saltó, quién la vio y quién la
resolvió, y el dato que la disparó. Un **catálogo** dice qué alarmas existen, con qué severidad,
si están activas y quién las ve (el equipo interno, el cliente, el transportista): cambiar la
severidad o apagar una alarma es una fila de configuración, no código.

El catálogo previsto, en cinco familias:

- **ETA**: fuera de ventana, en riesgo, sin evento tras el ETA, ETA caducado, plan que incumple
  la ventana ya antes de arrancar.
- **Viaje**: sin despachar, sin empezar, error de despacho, por validar.
- **Vehículo** (de la telemática): parado demasiado tiempo, fuera de ruta, temperatura fuera de
  rango, avería.
- **Entrega**: entrega fallida, Reembolso sin cobrar, incidencia de bulto (no cargado, con
  reservas, fallido), Destinatario que escribe y no recibe respuesta, aviso fallido.
- **Planificación e integración**: Orden sin geocodificar, Orden de hoy sin planificar a menos
  de N minutos, líneas no válidas en la Bandeja de entrada API, Manifiesto sin cerrar a T horas,
  y dos de sistema, cola de avisos atascada y recálculo de ETA parado.

**Cómo se evalúan.** Una tarea, cada minuto, recorre los Viajes del día que no están cerrados y
las Paradas del día sin Viaje, y es idempotente: crea la alarma que aparece, **resuelve sola** la
que desaparece (el conductor reporta, el Viaje arranca) y no toca las que alguien ya trató.
Los hitos de telemática (parado, desvío, temperatura, avería) y un error de despacho crean la
suya **en el acto**, sin esperar al minuto. Las de ETA se **agrupan por Viaje**: una fila por
Viaje y código con sus Paradas, porque en un día real de treinta Viajes salen más de doscientas
Paradas tarde y seis alarmas se pueden gestionar, doscientas no.

**Cómo se gestionan.** En el Panel de la torre, agrupadas por familia y ordenadas por severidad
y antigüedad, con tres acciones: **Vista**, **Silenciar** (hasta la hora que se elija) y **Resolver**, que
dejan nota en el historial del afectado. Una alarma resuelta a mano no vuelve a saltar hasta que
el dato cambie. Cada audiencia ve las suyas: el cliente, las de sus Paradas y Proyectos cuyo tipo
las permita; el transportista, las de sus Viajes; el equipo interno, todas.

12.5.3 Lo que hay que preparar mientras tanto
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Revisar los **umbrales** de la política de la compañía y de los Proyectos que lo necesiten: son
  los que la evaluación usará el primer día.
- Dar a los **Tipos de Parada** su geocerca cuando 150 m no valgan (un polígono industrial, un
  centro comercial).
- Poner los **rangos de temperatura** en las clases de temperatura, o la alarma de cadena de frío
  no tiene con qué comparar (:doc:`7.8 </17.0/7_edi-integrations/7_8_telematics>`).
- Conectar la **telemática**: sin posiciones, de las cuatro alarmas de vehículo solo funcionan la
  de temperatura y la de avería.
