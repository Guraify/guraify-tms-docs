13.5 Secuenciación
------------------

13.5.1 Lanzarla y el diálogo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En el Viaje, botón **Secuenciar viaje** (pestaña Ruta o Paradas); desde la lista de Viajes,
seleccionando varios, Acción › Secuenciar Viaje. Se abre el diálogo **Secuenciar viajes**: los
Viajes elegidos, **Mantener orden relativo** (las Paradas que ya tienen orden lo conservan entre
sí; las demás se intercalan), el **Inicio de carga** (solo con un Viaje: la hora a la que empieza
a cargar, y la ruta sale entre esa hora y quince minutos después; si se cambia, se guarda en el
Viaje) y las tres **Pasadas**, que vienen del Planning del Viaje y se pueden cambiar para este
lanzamiento (un Viaje sin Planning se secuencia una vez a nivel Real). Se confirma con
**Secuencia**. Lo que cada botón hace sobre el Viaje está en
:doc:`5.2.3 </17.0/5_operational-flows/5_2_trip-generation>`.

.. CAPTURA: 13_5_01 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/13_optimization-course/13_5_sequencing_01_dialogo.png
      :alt: Diálogo Secuenciar viajes con Mantener orden relativo, Inicio de carga y las tres pasadas

      El diálogo Secuenciar viajes: Mantener orden relativo, Inicio de carga y las tres pasadas
      con su nivel.

13.5.2 Qué pasa después
~~~~~~~~~~~~~~~~~~~~~~~

#. El Viaje entra en cola («En cola para secuenciar») y se abre el diálogo de progreso, que
   informa: «En espera: N secuenciación(es) por delante», «PTV Sequence está ordenando las
   paradas», «Viaje secuenciado».
#. Se puede cerrar el diálogo: el trabajo sigue en segundo plano y al acabar llega una
   notificación. Mientras dura, el Viaje avisa de que se está secuenciando.
#. Al terminar se escriben el orden y las horas, se calcula la ruta (13.4) y se marca cada
   Parada con su Estado de la ventana (13.3.4).
#. Un Viaje que ya tiene una secuenciación pendiente no se secuencia dos veces.

El diálogo final resume: Órdenes planificadas y sin planificar, km, horas, cuántas Paradas hay en
cada estado de ventana, las violaciones y lo que hizo cada pasada.

13.5.3 Paradas que no caben
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Con Sin pasada en la 3, o si ni el nivel más abierto las coloca, las Paradas siguen en el Viaje
pero marcadas **Sin planificar**: sin orden, sin horas y en ámbar en la lista. Las que tienen
coordenadas sospechosas (13.2.2) ni se envían y aparecen en **Paradas excluidas**. Las que entran
con Fin abierto sin límites llevan la marca de lo que incumplen: distancia, capacidad o jornada
(:doc:`1.4.2 </17.0/1_introduction/1_4_technological-architecture>`).

13.5.4 El historial: Secuenciaciones
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Ruta:** TMS › Operaciones › Planning › **Secuenciaciones**. Cada lanzamiento queda registrado con
su estado (En cola, Secuenciando, Terminado, Fallido, Anulado), quién lo pidió, cuánto esperó y
cuánto tardó. La pestaña **Pasadas** lleva **Qué hizo cada pasada**, con frases como «Pasada 1
(Con margen): 18 transporte(s) enviados, 3 fuera», «Pasada 2 (Con tolerancia): 2 de 3 parada(s)
planificadas», «Pasada 2 (Con tolerancia) no lanzada: la misma petición que la pasada anterior» o
«La pasada 3 dejó fuera 4 parada(s), más que las 1 de la pasada 2: se aplicó el orden de la
pasada 2». Los avisos en ámbar son informativos (por ejemplo, la salida fuera del horario del
Hub).

**Deshacer:** **Restaurar orden anterior**, en el Viaje o en la ejecución, devuelve orden, horas
y distancias. Solo deshace la **última** secuenciación terminada del Viaje.

.. CAPTURA: 13_5_02 — descomentar el figure cuando esté la imagen
   .. figure:: /_static/img/13_optimization-course/13_5_sequencing_02_pasadas.png
      :alt: Una Secuenciación con la pestaña Pasadas y el campo Qué hizo cada pasada

      Una Secuenciación terminada: la pestaña Pasadas dice qué envió, qué colocó y qué saltó cada
      una.

13.5.5 Secuenciación automática
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Con **Secuenciar Viaje** activo en el Proyecto, el Viaje se secuencia solo al **cerrar el
Manifiesto** y al **validar la Orden**. No bloquea al usuario: el Viaje entra en la cola y se
ordena a los pocos minutos, con los niveles de su Planning; no se abre ningún diálogo y el
resultado queda en Secuenciaciones. Un Viaje con Tramos sin Parada de carga o de descarga se
salta. La cola avanza con una tarea programada cada minuto y con el tope de Secuenciaciones
simultáneas (13.2.2); si esa tarea está desactivada, la secuenciación se hace en el momento y el
cierre tarda más.

13.5.6 Cuando un Viaje pierde una Parada
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No se vuelve a secuenciar. Las Paradas que quedan se renumeran en su mismo orden y **solo se
recalcula la ruta**. Si el Viaje ya había empezado, se recalcula el ETA.
