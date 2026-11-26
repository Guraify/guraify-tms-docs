13.3 La franja, la tolerancia, el margen y las pasadas
------------------------------------------------------

Es la pieza que explica casi todos los resultados que sorprenden: por qué una Parada entró
tarde, por qué otra se quedó fuera, por qué la pasada 2 «no se lanzó». Vale igual para la
secuenciación, la optimización completa y Encontrar ruta.

13.3.1 La franja de una Parada
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La Parada tiene una sola franja, **Hora Inicio** – fin, que se ve y se edita en su formulario
(bloque Parada y pestaña **Puntualidad** › Ventana cliente). De dónde sale, por este orden: de la
Orden, si el Proyecto tiene **Horarios Informados**; si no, del horario por defecto del
Destinatario; si no, de la hora predeterminada del Proyecto. Elegir una Franja horaria en el Tramo
copia su inicio y su fin. En una Parada de Hub, la franja es el horario del Hub ese día.

13.3.2 Los niveles
~~~~~~~~~~~~~~~~~~

Una :term:`Pasada` es una llamada al motor con la franja abierta hasta cierto
:term:`nivel <Nivel de pasada>`. **El inicio de la franja no se toca nunca**: llegar antes
siempre es esperar.

.. list-table::
   :header-rows: 1
   :widths: 26 48 26

   * - Nivel
     - Fin de franja que se pide
     - Jornada y distancia máxima
   * - **Con margen**
     - El fin real **menos** el :term:`Margen sobre el fin` del Planning (sin dejar la franja en
       menos de 30 minutos)
     - Se respetan
   * - **Real**
     - El fin real
     - Se respetan
   * - **Con tolerancia**
     - El fin real más la tolerancia
     - Se respetan
   * - **Con tolerancia × 2** / **× 4**
     - El fin real más dos, o cuatro, veces la tolerancia
     - Se respetan
   * - **Fin abierto**
     - Sin fin: hasta que acabe la franja del conductor o las 23:59
     - Se respetan
   * - **Fin abierto sin límites**
     - Sin fin
     - **No se aplican** (tampoco el perfil de jornada)
   * - **Sin pasada**
     - La pasada no se lanza (solo en la 2 y la 3)
     - —

En ningún nivel se relajan los equipamientos, las zonas de bajas emisiones ni las zonas
operativas. Los minutos de tolerancia son, lo primero que exista: los del lanzamiento › los del
Planning › los de la zona de la Parada › 0.

13.3.3 Cómo encadenan
~~~~~~~~~~~~~~~~~~~~~

#. La pasada 1 envía todas las Paradas a su nivel.
#. Si alguna queda fuera, se lanza la siguiente pasada que valga la pena: se salta la que esté
   en Sin pasada, la que no sea más holgada que la anterior y la que pediría exactamente lo mismo.
#. En la pasada siguiente, las Paradas que quedaron fuera reciben el nivel nuevo y **las demás
   conservan el nivel con el que entraron**.
#. Si una pasada deja fuera más Paradas que la anterior, se descarta y se aplica la anterior.
#. Si una pasada falla, se aplica la anterior.

Con los valores de serie (Con margen › Con tolerancia › Fin abierto sin límites) el resultado
normal es que **ninguna Parada quede fuera**: las que no caben a su hora entran tarde y marcadas.
Cada ejecución guarda **Qué hizo cada pasada** (13.5.4).

13.3.4 El veredicto: Estado de la ventana
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El :term:`Estado de ventana` se calcula siempre igual, sea cual sea la pasada, comparando el
inicio del servicio con la **franja real**: **En ventana** (dentro; llegar pronto y esperar cuenta
como dentro), **En tolerancia** (después del fin, pero como mucho los minutos de tolerancia, una
sola vez), **Fuera de ventana** (más tarde) o **Sin ventana** (la Parada no tiene franja).

La consecuencia que hay que enseñar: una Parada que entró gracias a Con tolerancia × 2, × 4 o Fin
abierto sale **Fuera de ventana**. La pasada la deja entrar; el estado dice la verdad. Se ve en la
pestaña **Puntualidad** de la Parada, con la **Desviación de la ventana (min)**, y como columna
opcional con colores en las listas de Paradas.

.. figure:: /_static/img/13_optimization-course/13_3_passes_01_puntualidad.png
   :alt: Pestaña Puntualidad de una Parada con su franja, su estado de ventana y la desviación

   La pestaña Puntualidad de una Parada: la franja real, el estado de la ventana y la
   desviación en minutos.

13.3.5 Cómo elegir niveles
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 30 36 34

   * - Operativa
     - Pasadas sugeridas
     - Por qué
   * - Última milla con muchas Paradas
     - Con margen / Con tolerancia / Fin abierto sin límites
     - Colchón primero y que salga todo
   * - Citas estrictas (carga completa, convocatoria)
     - Real / Sin pasada / Sin pasada
     - Lo que no cabe a su hora debe verse como no planificado
   * - Franjas anchas y pocas incidencias
     - Real / Con tolerancia / Fin abierto
     - Sin recortar; se respeta la jornada hasta el final

Regla práctica: poner **Sin pasada** en la 3 es la única forma de que una Parada pueda quedar sin
planificar por horario.
