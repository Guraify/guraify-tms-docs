13.10 Diagnóstico
-----------------

Qué mirar cuando el resultado no convence. Cada fila enlaza al módulo que lo explica.

.. list-table::
   :header-rows: 1
   :widths: 46 54

   * - Lo que se ve
     - Dónde mirar
   * - «No hay token PTV configurado…»
     - 13.2.1.
   * - Muchas Paradas «excluidas por nosotros» por coordenadas
     - La Precisión de los Destinatarios (13.2.11) y la distancia máxima al Hub (13.2.2).
   * - El asistente pide Hub para algunas Paradas
     - Zonas sin Hub, o Paradas fuera de todo polígono (13.2.4).
   * - Con Developer no se planifica nada
     - Capacidades a 0 en la categoría (13.2.8).
   * - Las rutas salen demasiado largas o cortas en tiempo
     - Perfil de vehículo vacío, factor de velocidad, tiempos de servicio (13.2.7, 13.2.8).
   * - La pasada 2 nunca se lanza
     - Tolerancia y margen a 0 en el Planning (13.2.5).
   * - Muchas Paradas «Fuera de ventana» pero planificadas
     - Entraron en una pasada abierta (13.3.4). Revisar vehículos, franjas o niveles.
   * - Aviso «…fuera del horario del hub…»
     - La ventana de salida no cuadra con el horario del Hub (13.2.3).
   * - El Viaje no se secuencia al cerrar el Manifiesto
     - Secuenciar Viaje del Proyecto; Tramos sin Parada; mirar Secuenciaciones (13.5.5).
   * - El Viaje se queda «En cola para secuenciar»
     - La tarea programada de la cola; Secuenciaciones simultáneas (13.5.5, 13.2.2).
   * - «Enrutar viaje» dice que fue bien y nada cambia
     - PTV falló: Paradas Caducado (13.4).
   * - El plan no se puede revertir
     - Aplicarlo borró un Viaje de origen, o alguna Parada se movió o se reportó (13.6.5).
   * - Dos Paradas sobrantes se quedan «Sin ruta» tras las pasadas
     - Eligieron el mismo Viaje (13.6.4): Encontrar ruta (13.7).
