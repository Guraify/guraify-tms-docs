13.7 Encontrar ruta
-------------------

Sirve para meter Órdenes sueltas en Viajes que ya existen, o para **repartir** un Viaje entre los
demás (un vehículo averiado, un conductor que falta). El detalle de los dos modos, las candidatas
y la reversión está en :ref:`encontrar-ruta`.

**Por dónde se entra.** Lista de Órdenes › Acción › **Encontrar ruta**; Viaje › **Repartir este
viaje**; Asignar, modo Existente › **Buscar la mejor ruta**.

**El asistente.** Qué (Viaje a repartir u Órdenes a enrutar); las rutas: fecha, Planning y
**Rutas candidatas** (por defecto, los Viajes de ese Planning y día que no han empezado; se
pueden añadir Viajes ya empezados); cómo (**Mantener estructura**, la Orden se intercala en un
Viaje sin tocar los demás, con **Permitir reordenar dentro de la ruta**; o **Romper estructura**,
se rehacen los Viajes candidatos); y qué se puede violar: Preset, **Pasada 1** (y 2 y 3 solo en
Mantener estructura), Tolerancia después y Respetar zonas operativas.

**Mantener estructura, paso a paso.** Se descartan los Viajes que no valen por equipo, zona, bajas
emisiones o capacidad y se evalúan los tres con menos desvío; cada candidata se prueba de verdad
con una secuenciación, por pasadas, sin escribir nada; la mejor es, por este orden, la que cabe
al **nivel más estricto**, la de menor coste añadido, la que añade menos Paradas fuera de
ventana, menos en tolerancia y menos km. La ejecución enseña, por Orden, la ruta propuesta, el
**Nivel**, los km y minutos que añade y las Paradas que pasan a tolerancia o fuera de ventana;
en **Candidatas** están todas, con «Por qué no» en las descartadas y **Elegir** para cambiar la
propuesta. **Aplicar** mueve las Órdenes; **Revertir** las devuelve.

.. figure:: /_static/img/13_optimization-course/13_7_find-route_01_nivel.png
   :alt: Ejecución de Encontrar ruta con la columna Nivel en la propuesta de cada Orden

   La propuesta de Encontrar ruta: por Orden, la ruta elegida y el nivel de pasada al que cabe.

**Repartir un Viaje.** Si el Viaje se vacía, se conserva hasta pulsar **Borrar el viaje
vaciado**; si ya había empezado, queda como repartido y sin cerrar hasta **Cerrar viaje
repartido**. El historial está en TMS › Operaciones › Planning › Encontrar ruta.
