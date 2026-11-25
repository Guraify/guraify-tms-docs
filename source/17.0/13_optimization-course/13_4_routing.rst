13.4 Cálculo de ruta
--------------------

**Lanzarlo.** En el Viaje (TMS › Operaciones › Tráfico › Viajes), botón **Enrutar viaje** de la
pestaña **Ruta** o de la pestaña **Paradas**, o Acción › Enrutar Viaje. Se calcula en el momento,
con la pantalla esperando, sobre el orden que tienen las Paradas, sin restricción por estado del
Viaje. Lo que PTV recibe y devuelve está en
:doc:`1.4.2 </17.0/1_introduction/1_4_technological-architecture>`.

**Qué deja.** En la cabecera del Viaje, km, horas y número de Paradas; en Datos del viaje ›
Cifras de la ruta, km, tiempo de viaje, conducción, servicio, espera, pausa y descanso; en la
pestaña **Ruta**, el trazado en el mapa con sus capas (peajes, bajas emisiones, restricciones,
tráfico, puntualidad; :doc:`3.2.2.5 </17.0/3_functional-architecture/3_2_2_trips>`); y en cada
Parada, llegada, salida, ETA, km y conducción del tramo, espera y el Estado de la ventana. Los
peajes, las emisiones y el resto del enriquecimiento dependen de la Política operativa
(:doc:`12.3 </17.0/12_live-operations/12_3_eta>`).

**Promesa y ETA.** El **Prometido** («ETA (planificado)») es la hora que se le promete al
Destinatario: cada cálculo de ruta lo reescribe mientras el Viaje no está despachado, y desde el
despacho queda fijo (revertir el despacho lo libera). El **Actual** («ETA») es la mejor
estimación de ahora, que reescriben todos los cálculos y, en marcha, el recálculo de ETA; la
**Desviación del ETA (min)** es actual menos prometido, positivo si tarde (12.3).

**Cuándo se calcula solo.** Al terminar cualquier secuenciación; cuando el Viaje pierde una
Parada (13.5.6), al restaurar un orden y al retocar la pestaña Ruta (13.9). En un Viaje ya
empezado no se recalcula la ruta: se pide un recálculo de ETA.

**Dos avisos.** El botón enseña el mensaje de éxito aunque PTV haya fallado: si falla, se
conservan los valores anteriores y las Paradas quedan marcadas **Caducado**; hay que mirar que los
km y las horas hayan cambiado. Y el asa de la lista de la pestaña Paradas cambia el orden pero
**no recalcula**: hay que pulsar Enrutar viaje después. La pestaña Ruta sí recalcula sola.
