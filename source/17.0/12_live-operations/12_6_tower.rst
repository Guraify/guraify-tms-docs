12.6 Lo que la torre lee de Odoo
--------------------------------

.. note::

   La torre de control (**e-transport connect**) es una aplicación aparte, en construcción, que
   lee y escribe en Odoo a través de un servicio intermedio. Este apartado no es su contrato
   —eso es la especificación `02_SPEC_api_connect.md`— sino el inventario de **lo que Odoo ya
   tiene preparado** para ella y dónde está, para que quien la construya o la configure sepa
   qué hay y qué falta.

12.6.1 Lo que está
~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 24 46 30

   * - Qué
     - Qué hay en Odoo
     - Dónde se cuenta
   * - **Estados**
     - El ciclo de estados de Parada, Tramo, Orden y Viaje, con un vocabulario único (sin
       planificar, planificado, despachado, en curso, por validar, completado, fallido) y la
       trazabilidad de la que salen.
     - :doc:`3.2.7 </17.0/3_functional-architecture/3_2_7_state-cycle>`
   * - **Despacho**
     - Fecha, canal, resultado y error del despacho en el Viaje; Despachar, Reenviar, Revertir
       despacho, Validar, Rechazar y Cancelar como acciones del Viaje.
     - :doc:`5.4.1 </17.0/5_operational-flows/5_4_app-execution>`, 5.5
   * - **ETA**
     - El prometido y el vigente de cada Parada, la desviación, el origen, las marcas de caducado
       y sin ruta, el historial por Parada (180 días) y, en el Viaje, cuándo y por qué se
       recalculó y el trazado restante.
     - 12.3, :ref:`17.0/1_introduction/1_4_technological-architecture:1.4.2.3 Referencia técnica`
   * - **Posición y señales**
     - La última posición del Viaje (fecha, origen, coordenadas), los eventos de telemática que
       no se purgan y las lecturas que la política de persistencia conserve. Las puertas por las
       que el servicio intermedio entrega posiciones, lecturas y eventos, y la de configuración
       por la que lee proveedores, dispositivos, mapeos y las geocercas de los Viajes del día.
     - 12.2, :doc:`7.8 </17.0/7_edi-integrations/7_8_telematics>`
   * - **Enriquecimiento del Viaje**
     - Peajes, coste de ruta, emisiones y consumo, violaciones, zonas, el informe de PTV y, en
       bruto, los bloques de resultado que la política pida. El trazado planificado y el restante.
     - :doc:`5.4.4 </17.0/5_operational-flows/5_4_app-execution>`, 4.3.9
   * - **Política operativa**
     - Cadencia del ETA, geocerca, umbrales de alarma y **privacidad del Destinatario** (qué ve de
       la posición: siempre, solo cuando es la siguiente, solo una cuenta atrás; si puede
       reprogramar o cambiar la dirección). Lo de privacidad y alarmas lo lee solo la torre.
     - 4.3.9, :doc:`/17.0/annexes/A_31_politica-operativa`
   * - **Mensajería**
     - Los avisos enviados y omitidos con su estado, el canal de WhatsApp de cada Parada con
       respuesta, y los enlaces de seguimiento con su código, caducidad y accesos.
     - 12.4, :doc:`11 </17.0/11_notifications/index>`
   * - **Audiencias y aislamiento**
     - La audiencia de cada contacto (cliente, transportista, ambas, ninguna), las reglas que
       limitan lo que cada una lee de Órdenes, Tramos, Paradas, Viajes, Bultos y trazabilidad, y
       la **lista blanca de campos** que se le pueden enseñar. Las lee cualquier camino que
       entre en Odoo como ese usuario, también el servicio intermedio.
     - :doc:`7.9 </17.0/7_edi-integrations/7_9_external-access>`
   * - **Marca**
     - Logos, paleta de día y de noche, tipografía, dominio de seguimiento y nombre del remitente
       por compañía, servidos en un solo punto.
     - :doc:`4.1 </17.0/4_parametrization/4_1_operational-configuration>`, Marca de la compañía
   * - **Indicadores**
     - Los indicadores configurables, con su audiencia, y los tableros.
     - :doc:`8.8 </17.0/8_economic-administration/8_8_control-reporting>`, 7.10
   * - **Escenarios**
     - El gestor de escenarios como motor: el plan de cada ventana, su versión, las métricas, el
       ruteo en seco y la escritura del plan con control de versión. El mapa es de la torre.
     - :doc:`5.2.5 </17.0/5_operational-flows/5_2_trip-generation>`
   * - **Acciones**
     - Todo lo que el operador hace en Odoo (despachar, secuenciar, asignar, reportar un evento
       como la app, recalcular el ETA) existe como método del modelo y deja su nota en el
       historial con el autor real.
     - Cada sección

12.6.2 Lo que falta
~~~~~~~~~~~~~~~~~~~

- **El módulo ``tms_connect``**: el estado operativo único (el semáforo que combina estado, ETA,
  ventana y alarmas en un solo color), las alarmas (12.5), la línea de tiempo unificada (un solo
  hilo con trazabilidad, historial de ETA, avisos, eventos de telemática, alarmas y acciones), las
  preferencias de usuario y las lentes por audiencia de cada lectura.
- **El servicio intermedio** (el BFF y el hub de telemática): la autenticación de las cuatro
  audiencias, el tiempo real, las posiciones en memoria y los adaptadores de proveedores.
- **Las posiciones**: la app del conductor no envía la suya y no hay otro proveedor conectado.
- **La página del Destinatario**, que abren los enlaces de seguimiento, y la cuenta real de
  WhatsApp.

12.6.3 Cómo leer el estado de un Viaje hoy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hasta que exista el estado operativo único, lo que la torre tendrá que combinar está repartido en
cuatro datos de cada Parada, y conviene conocerlos para no leerlos mal:

#. **El estado** (3.2.7): dice en qué punto del ciclo está. «Despachada» y «en curso» son
   Paradas vivas; las seis de cierre, terminadas.
#. **La ventana** (hora de inicio y fin de la Parada) y **el ETA vigente**: comparados dan «va a
   tiempo», «va tarde» o «fuera de ventana». Con la marca **Caducado**, el ETA es viejo y no
   debe pintarse como fiable.
#. **La puntualidad** (pestaña Puntualidad): cuando la Parada se ha cerrado, lo que cuenta es la
   hora ejecutada frente a la ventana, no el ETA.
#. **Los eventos**: una Parada despachada cuyo ETA pasó y sin evento de llegada es el caso que la
   alarma «sin evento tras el ETA» cubrirá; hoy se detecta leyendo la trazabilidad.

El semáforo de la torre está definido regla a regla en el sistema de diseño de la torre (quince
reglas para la Parada, nueve para el Viaje): cuando ``tms_connect`` exista, Odoo lo calculará y la
torre solo lo pintará.
