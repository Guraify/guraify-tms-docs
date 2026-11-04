12.2 Lo que entra: eventos y señales
------------------------------------

Dos cosas distintas llegan a Odoo mientras el Viaje está en la calle: lo que **el conductor
reporta** desde la app, que son hechos de la operación (he llegado, he entregado, ha fallado), y
lo que **el vehículo emite**, que son medidas (dónde está, a qué temperatura va, cuánto le queda
de conducción). Entran por dos puertas, se guardan en dos sitios y mueven cosas distintas.

12.2.1 Los eventos de la app
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. admonition:: Ruta en Odoo
   :class: tip

   Parada › pestaña **Trazabilidad** · TMS › Configuración › App › **App Events** (lo que llegó
   de la app, procesado o pendiente)

Cada evento de la app es un **apunte de trazabilidad** sobre la Parada, el Tramo, el Bulto o el
Reembolso, con fecha y hora, conductor, coordenadas y precisión del GPS, dispositivo, batería,
tipo de red, motivo, comentario, importe y adjuntos. Los eventos y lo que mueven están en
:doc:`5.4.2 y 5.4.3 </17.0/5_operational-flows/5_4_app-execution>`; lo que importa aquí es qué
desencadena cada uno fuera de los estados:

.. list-table::
   :header-rows: 1
   :widths: 24 76

   * - Evento
     - Además de mover el estado…
   * - En camino
     - Nada más. Es el único evento que no dispara ningún aviso ni recálculo.
   * - Llegada (En curso)
     - Dispara el aviso **Vehículo llegado** al Destinatario de la Parada.
   * - Resultado de una entrega (Completada, Con reservas, Fallida, Reprogramada, Devuelta)
     - Cierra la Parada: pide el **recálculo del ETA** de las Paradas pendientes del Viaje;
       dispara el aviso del resultado al Destinatario y el aviso **Eres el siguiente** al de la
       Parada siguiente; si era la última, cierra el Viaje.
   * - Resultado de una carga (Cargada, con reservas, No cargada)
     - Cierra la Parada igual: recálculo del ETA y cierre del Viaje si era la última. Los avisos
       de resultado son de entregas; una carga no avisa al Destinatario.
   * - Prueba de entrega (firma, foto, albarán escaneado)
     - Dispara el aviso **Prueba de entrega disponible** si el flujo acepta ese tipo; la firma y
       el albarán cuentan para la validación del Viaje, la foto no.
   * - Escaneo de un Bulto
     - Mueve el Bulto y deja su traza; no toca el ETA ni los avisos.
   * - Cambio de dirección de la Parada (desde la oficina)
     - Dispara el aviso **Dirección cambiada**.

**Un evento reportado no se pierde.** Antes de procesarlo, Odoo lo guarda tal como llegó
(**App Events**) y lo confirma; si el proceso falla, el evento queda pendiente y una tarea lo
reintenta. El mismo evento dos veces se reconoce como duplicado. Y cuando dos eventos del mismo
Viaje llegan a la vez, el segundo espera al primero. Para la torre esto significa que lo que la
app reportó está en Odoo aunque el estado tarde unos segundos en moverse.

12.2.2 Las señales de telemática
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Configuración › **Telemática** (Proveedores, Dispositivos, Señales, Lecturas, Eventos) ·
   Viaje › pestaña **ETA y ruta** › Última posición conocida

Las señales entran por el conector de telemática (:doc:`7.8
</17.0/7_edi-integrations/7_8_telematics>`): un **proveedor** (la caja negra del vehículo, un
servicio de flota, la app del conductor) con sus **dispositivos**, cada uno atado a un vehículo
o a un conductor, y un catálogo de veinte **señales** con su unidad y su efecto. La señal entra
siempre por el mismo punto, venga de donde venga, se asocia al Viaje en curso del vehículo y
produce, según su tipo, una **lectura** (un valor que se guarda y se purga) o un **evento** (un
hito que no se purga y que la torre leerá).

Lo que cada señal mueve en Odoo está en 7.8.5. Visto desde la operación en vivo:

- **La posición** no crea filas: sobrescribe la última posición del Viaje (fecha, origen,
  coordenadas, en la pestaña ETA y ruta). Con ella Odoo hace dos cosas. Si el vehículo entra en la
  **geocerca de llegada** de una Parada pendiente (150 m, o el radio del Tipo de Parada o de la
  política operativa), registra el evento *Entró en la geocerca* y, si el conductor no ha
  reportado todavía la llegada, anota la hora; el reporte del conductor manda sobre el GPS. Y con
  la política en modo **Al desviarse del plan**, si el vehículo está a más de la **desviación de
  ruta** (en km) de la siguiente Parada, pide un recálculo del ETA. Un acelerador descarta las
  posiciones que llegan demasiado seguidas (dos minutos por defecto).
- **Odómetro y horas de motor** entran en el odómetro único del vehículo con origen Telemetría
  (:doc:`6.2 </17.0/6_resources/6_2_odometer>`).
- **Temperatura** se contrasta con el rango del compartimento: fuera de rango, evento *Temperatura
  fuera de rango* y una nota en las Órdenes del Viaje.
- **Tacógrafo** rellena el grupo del conductor, que el cálculo de ruta usa para colocar las pausas.
- **Avería** crea el evento y una orden de mantenimiento, una por avería.
- **Frenada brusca, aceleración, exceso de velocidad, contacto, descansos** dejan solo el evento.

Los **eventos de telemática** son el registro que la torre leerá: entrada y salida de geocerca,
contacto, temperatura fuera de rango, avería, inicio y fin de descanso, *parado demasiado tiempo*,
*fuera de la ruta planificada*, conducción brusca y exceso de velocidad. Hoy Odoo produce los de
geocerca, temperatura y avería; los de parada prolongada y desvío de ruta los producirá el
servicio intermedio, que compara la posición con el trazado del Viaje, y Odoo los recibirá hechos.

12.2.3 Lo que no entra todavía
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mientras la app del conductor no envíe su posición y no haya otro proveedor conectado, en Odoo
entran los eventos de la app y nada más. La consecuencia, apartado por apartado:

- La geocerca de llegada no registra nada; la llegada es la que reporta el conductor.
- El modo «Al desviarse del plan» de la política equivale a no recalcular.
- La última posición del Viaje está vacía y el ETA se calcula desde la última Parada cerrada.
- Las alarmas de vehículo parado y de desvío de ruta no tienen con qué dispararse.
