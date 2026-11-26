13.8 Escenarios
---------------

**Ruta:** TMS › Operaciones › Planning › **Escenarios**. Un :term:`Escenario` calcula varios
planes del mismo día (cada uno, una :term:`ventana <Ventana de escenario>`), los compara y aplica
uno; nada se escribe en los Viajes hasta aplicar. El detalle (origen y universo, caducidad,
comparativa, aplicar por diferencias, revertir) está en :ref:`escenarios`. El recorrido:

#. **Nuevo**: nombre, Planning, fecha y origen. **Paradas pendientes** toma las de ese Planning y
   día sin Viaje; **Viajes planificados**, las de los Viajes que se elijan.
#. **Cargar las Paradas** (el botón se lee «Paradas de carga»).
#. **Añadir ventana**: motor (Manual, Developer u OptiFlow), tipo de asignación, categorías o
   turnos, **Preset** y **Nivel**. Una ventana se calcula a un solo nivel: no encadena pasadas
   (13.3).
#. **Optimizar** cada ventana. Para tener la referencia de lo que hay hoy, una ventana Manual con
   **Tomar foto del plan actual**.
#. **Comparar** en la pestaña Comparativa: rutas, Paradas planificadas, km, horas, coste y CO₂e.
   Para comparar retrasos y esperas, antes **Rutear todas (en seco)**.
#. **Aplicar** la elegida: se aplica por diferencias, el Viaje que ya existía se conserva y solo
   cambian sus Paradas y su orden; lo nuevo se crea y lo que sobra se borra. Las demás ventanas
   quedan descartadas.

Si algún Viaje de origen ya ha empezado, el Escenario es **Solo evaluación**: se compara, no se
aplica. Si las Paradas han cambiado desde que se cargaron, queda caducado hasta **Actualizar
paradas**.

.. figure:: /_static/img/13_optimization-course/13_8_scenarios_01_ventana.png
   :alt: Ventana de un Escenario con motor, preset y nivel

   Una ventana de Escenario: motor, tipo de asignación, preset y el nivel de franja.
