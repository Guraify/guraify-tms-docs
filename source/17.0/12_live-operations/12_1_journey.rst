12.1 El recorrido de un Viaje en la calle
-----------------------------------------

Un Viaje pasa por la calle en seis momentos. Para cada uno, la tabla dice qué ocurre, quién lo
provoca, qué hace Odoo, dónde se configura y dónde se ve. Es el mapa del capítulo: cada fila
tiene su sección.

.. list-table::
   :header-rows: 1
   :widths: 14 22 28 18 18

   * - Momento
     - Qué ocurre
     - Qué hace Odoo
     - Dónde se configura
     - Dónde se ve
   * - **1. Despacho**
     - El planificador pulsa **Despachar** (o la agencia recibe el Viaje por API).
     - Viaje, Paradas, Tramos y Órdenes pasan a Despachado; el Viaje guarda fecha, canal y
       resultado; la app del conductor lo carga; sale el lote de avisos «Viaje iniciado».
     - Tipo de Viaje y endpoints de despacho del Proyecto (7.4); hora del flujo de aviso (11.3).
     - Ficha del Viaje, grupo Despacho (:doc:`5.4.1 </17.0/5_operational-flows/5_4_app-execution>`).
   * - **2. En ruta**
     - El conductor avanza; el vehículo se mueve.
     - Cada evento de la app (en camino, llegada, resultado) es un apunte de trazabilidad que
       mueve los estados. Cada posición de telemática actualiza la última posición del Viaje y
       detecta la llegada por geocerca.
     - Perfiles de la app del Proyecto (4.5.5); proveedores y dispositivos de telemática (7.8.6);
       geocerca de la política operativa (4.3.9).
     - Pestaña Puntualidad y trazabilidad de la Parada; ETA y ruta del Viaje (12.2).
   * - **3. El ETA se mueve**
     - Se cierra una Parada, pasa el intervalo, el vehículo se desvía o alguien lo pide.
     - Odoo marca el Viaje; una tarea, cada minuto, pide a PTV la llegada a lo pendiente desde
       la última Parada cerrada o la posición. El prometido no cambia; el vigente sí, y queda
       en el historial de cada Parada.
     - Política operativa: modo, intervalos, tope diario (4.3.9).
     - Pestaña ETA y ruta del Viaje; Puntualidad e Histórico de ETA de la Parada (12.3).
   * - **4. Se avisa**
     - Un evento de la Parada o una hora cumplen un disparador.
     - El flujo de aviso decide a quién y por qué canal; la cola lo envía; cada aviso queda en
       la Orden con su estado o su motivo de omisión.
     - Flujos de aviso, plantillas y canales (11.2, 11.3); consentimiento del Destinatario
       (11.4).
     - Pestaña Avisos de la Orden; Mensajes enviados (12.4).
   * - **5. Algo va mal**
     - Pasa el ETA sin evento, el vehículo lleva parado demasiado, el Viaje no arranca, se sale
       de la ruta, la temperatura se dispara.
     - Hoy: la telemática deja el evento y, en la temperatura, una nota en las Órdenes. Las
       **alarmas** con umbral son de la torre: Odoo guarda los umbrales y todavía no las evalúa.
     - Umbrales de alarma de la política operativa (4.3.9).
     - Eventos de telemática (7.8); la torre, cuando exista (12.5).
   * - **6. Cierre**
     - La última Parada se cierra.
     - El Viaje pasa a Completado (o Por validar, o Fallido); se tarifica el pasivo; dejan de
       recalcularse ETA; los avisos de resultado ya han salido. Las Órdenes con una entrega
       fallida quedan señaladas hasta que se decide reprogramar, devolver o destruir.
     - Validación del Proyecto (4.5.3).
     - :doc:`5.5 </17.0/5_operational-flows/5_5_trip-closing>`; ciclo de estados (3.2.7).

Tres cosas que conviene tener claras antes de seguir:

- **Nada llama a PTV ni envía un aviso dentro de la petición que trae el evento.** La app del
  conductor tiene que recibir respuesta en milisegundos, así que el evento solo deja una marca:
  «este Viaje necesita recálculo», «esta Parada tiene un aviso pendiente». Dos tareas
  programadas, cada minuto, hacen el trabajo: una recalcula el ETA y otra envía la cola de avisos.
  Si los crons están parados, el evento se registra y los estados se mueven, pero el ETA no se
  actualiza y nadie recibe nada.
- **La posición del vehículo todavía no llega.** El conector de telemática está hecho y la app del
  conductor es su primer proveedor, pero falta el servicio intermedio y la app no envía su
  posición (:ref:`17.0/1_introduction/1_4_technological-architecture:1.4.6 Evolución prevista`).
  Mientras tanto, todo lo que depende de la posición —la geocerca de llegada, el recálculo al
  desviarse, la alarma de vehículo parado— existe en la configuración pero no se dispara. El ETA
  se recalcula desde la última Parada cerrada.
- **Los estados no se escriben: se leen de la trazabilidad.** El estado de una Parada, de un
  Tramo y de una Orden es el de su último apunte; el del Viaje se calcula de sus Paradas. Por eso
  cualquier sistema que lea Odoo puede reconstruir la historia completa de un Viaje a partir de
  los apuntes, con su hora, su conductor y sus coordenadas (:doc:`3.2.7
  </17.0/3_functional-architecture/3_2_7_state-cycle>`).
