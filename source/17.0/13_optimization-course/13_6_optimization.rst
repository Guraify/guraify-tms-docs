13.6 Optimización completa
--------------------------

El recorrido completo del asistente, sus pasos y la ficha de la ejecución están en
:doc:`5.2.4 y 5.2.4.1 </17.0/5_operational-flows/5_2_trip-generation>`; aquí, lo que un
planificador tiene que saber para usarlo bien.

13.6.1 Por dónde se entra
~~~~~~~~~~~~~~~~~~~~~~~~~

- TMS › Operaciones › Planning › **Optimizador de Paradas**: el mapa, filtrado en Sin Planificar
  y Mañana; se seleccionan Paradas y **Optimizar rutas** abre el asistente en modo Optimización.
- Lista de Paradas (Operaciones › Maestros › Paradas) › Acción › **Manage**: modo Optimización.
- Lista de Órdenes › botón **Manage**: abre en modo Nuevo; hay que pulsar **Optimización**.

El asistente Asignar tiene tres modos: **Nuevo** (un Viaje a mano), **Existente** (añadir a un
Viaje) y **Optimización**.

13.6.2 Qué Paradas valen
~~~~~~~~~~~~~~~~~~~~~~~~

Solo Paradas de ruta y de recogida o entrega directa. Sin nada reportado, una Parada ya asignada,
incluso de un Viaje despachado sin empezar, se puede volver a optimizar: saldrá de su Viaje. Todas
con Hub: si falta alguno, se abre **Paradas sin hub** para ponerlo y continuar. Antes de lanzar,
un recuadro amarillo lista las **Paradas no enviadas** y el motivo, y se pide confirmación.

13.6.3 El bloque Optimización
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 28 72

   * - Campo
     - Qué es
   * - Planning
     - Trae sus niveles y su tolerancia y decide el motor (13.1.2).
   * - Fecha
     - Se propone si todas las Paradas comparten día.
   * - Tipo de Asignación
     - **Por Turno**: los vehículos son las franjas del Plan de disponibilidad de ese Planning y
       ese día; cada turno necesita vehículo y tarifa. **Por Categoría**: se dice cuántos
       vehículos de cada categoría hay, con su salida; con OptiFlow, además Hub de inicio, horario
       de conductor, ruta y conducción máximas.
   * - Preset
     - Para qué se optimiza (13.2.12). Obligatorio.
   * - Pasada 1 / 2 / 3
     - Vienen del Planning; se pueden cambiar para este lanzamiento (13.3).
   * - Permitir violar jornada, km, paradas y carga
     - Lo marca el Preset si lleva límites blandos.
   * - Permitir violar restricciones de vía
     - Solo OptiFlow.
   * - Respetar zonas operativas
     - No / Estricto / Preferido (este, solo OptiFlow).
   * - Avanzado
     - **Tolerancia después (min)** (0 = la de cada zona) y **Evitar cruces** (solo Developer).

Se lanza con **Ejecutar**.

.. figure:: /_static/img/13_optimization-course/13_6_optimization_01_bloque.png
   :alt: Bloque Optimización del asistente Asignar con preset, pasadas y lo que se puede violar

   El bloque Optimización del asistente: Planning, tipo de asignación, preset, las tres pasadas
   y lo que se puede violar.

13.6.4 Mientras calcula, el plan y las pasadas 2 y 3
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se abre el diálogo de progreso con el avance y la fase; se puede cancelar, y si se cierra, la
optimización sigue y se recupera desde Ejecuciones de optimización. Al terminar, el diálogo
enseña el plan **sin aplicarlo**: rutas, Órdenes programadas y sin programar, km y horas, cuántas
Paradas quedan en cada estado de ventana, las Paradas sin planificar en dos grupos (excluidas por
nosotros; enviadas y no planificadas por el optimizador), avisos y violaciones. No enseña
vehículo por vehículo: eso se ve en la ejecución o al abrir los Viajes.

**Aplicar plan** crea los Viajes. Las Paradas que estaban en otros Viajes salen de ellos; un
Viaje que se queda vacío se borra y el que conserva Paradas cierra el hueco y recalcula su ruta.
No se aplica si entretanto alguna Parada se ha reportado.

**Las pasadas 2 y 3** son el segundo paso. La pasada 1 es la optimización; si el motor dejó
Paradas sin planificar, tras Aplicar plan el diálogo sigue abierto, «Pasadas 2 y 3: Evaluando»,
y para cada Parada sobrante busca hueco en los Viajes recién creados, sin cambiar su estructura,
a los niveles de las pasadas 2 y 3. Sale una tabla Parada / Viaje / Pasada / Nivel / Franja
real; la que no encuentra hueco dice **Sin ruta**, con el motivo. **Nada se mueve hasta pulsar
Aplicar pasadas**; si se cierra sin pulsarlo, esas Paradas siguen sin Viaje y la propuesta
espera en la ejecución. Las «excluidas por nosotros» no entran: hay que corregir su dato. Límite
conocido: las sobrantes se evalúan una a una; si dos eligen el mismo Viaje y no caben juntas,
las dos se quedan sin ruta, y se resuelven con Encontrar ruta (13.7).

.. figure:: /_static/img/13_optimization-course/13_6_optimization_02_plan.png
   :alt: Diálogo del plan listo con Paradas por estado de ventana y el botón Aplicar plan

   El plan listo: rutas, Paradas por estado de ventana, las sin planificar por origen y el
   botón Aplicar plan.

.. CAPTURA: 13_6_03 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/13_optimization-course/13_6_optimization_03_pasadas.png
      :alt: Tabla de las pasadas 2 y 3 con Parada, Viaje, Pasada, Nivel y Franja real, y el botón Aplicar pasadas

      Las pasadas 2 y 3: dónde cabe cada Parada sobrante, a qué nivel, y el botón Aplicar pasadas.

13.6.5 Ejecuciones de optimización
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Operaciones › Planning › **Ejecuciones de optimización**. Cada lanzamiento queda
con su estado, proveedor, duración y quién lo pidió; en el formulario, **Mostrar progreso**,
**Aplicar plan**, **Cancelar** y **Revertir**, que devuelve las Paradas a donde estaban y borra
los Viajes que creó el plan, deshaciendo primero las pasadas 2 y 3 (no se puede si una Parada se
movió o se reportó después, o si aplicar el plan borró un Viaje de origen). Las pestañas llevan
las Paradas, los Viajes, las métricas, las violaciones, **Pasadas 2 y 3** y las Paradas sin
planificar, con el motivo de cada una y qué arreglar (5.2.4.1).

13.6.6 Qué cuenta el optimizador
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Capacidades (seis medidas), límites de ruta, equipamientos, zonas operativas y de bajas
emisiones, franjas al nivel de la pasada, tiempos de servicio, Perfil de jornada, horario del
Hub, ventana de salida, punto de inicio y fin, costes y, con OptiFlow, el coste de no planificar
y el ingreso de la Orden. Qué restringe a qué está en
:ref:`que-restringe`.
