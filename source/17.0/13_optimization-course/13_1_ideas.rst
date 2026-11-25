13.1 Las ideas
--------------

13.1.1 Tres servicios, de menos a más
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Todo lo que el TMS pide a PTV cabe en tres servicios, y conviene tener claro cuál se está
usando en cada momento porque hacen cosas distintas con los mismos datos
(:doc:`1.4.2 </17.0/1_introduction/1_4_technological-architecture>`):

.. list-table::
   :header-rows: 1
   :widths: 18 32 22 28

   * - Servicio
     - Qué hace
     - Qué no hace
     - Cómo se lanza
   * - :term:`Cálculo de ruta`
     - Sobre un Viaje **ya ordenado** calcula trazado, kilómetros, horas y la hora de llegada a
       cada Parada.
     - No cambia el orden.
     - Botón **Enrutar viaje**; solo, tras cada secuenciación o retoque (13.4).
   * - :term:`Secuenciación`
     - Ordena las Paradas de **un** Viaje respetando franjas, jornada y horario del Hub.
     - No mueve Paradas entre Viajes ni cambia de vehículo.
     - Botón **Secuenciar viaje**; sola, al cerrar un Manifiesto o validar una Orden si el
       Proyecto lo pide (13.5).
   * - :term:`Optimización completa`
     - A partir de Paradas sueltas decide cuántos Viajes, con qué vehículo y en qué orden.
     - —
     - Asistente Asignar en modo **Optimización**; Escenarios (13.6, 13.8).

Alrededor de los tres hay dos herramientas más: :term:`Encontrar ruta` (colocar una Orden, o
repartir un Viaje entero, entre Viajes que ya existen; 13.7) y los :term:`Escenarios
<Escenario>` (calcular varios planes del mismo día, compararlos y aplicar uno; 13.8).

13.1.2 Dos motores de optimización
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La optimización completa puede hacerla **PTV Developer** o **PTV OptiFlow**; el cálculo de ruta
y la secuenciación los hace siempre Developer. Qué motor se usa no se elige al lanzar: lo decide
el **Proveedor de optimización** del Planning y, si está vacío, el de la compañía. Las
diferencias que un planificador nota (tiempo de respuesta, qué hace con los costes, qué
significa una capacidad a cero, cómo trata las zonas «Preferido») están en la tabla de
:ref:`17.0/1_introduction/1_4_technological-architecture:1.4.2.1 Dos motores de optimización`.

13.1.3 Las piezas que se configuran
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Seis palabras que salen en todo el curso:

- **Planning.** La unidad que organiza la operativa: cada Orden, Tramo, Parada y Viaje pertenece
  a uno, heredado del Proyecto. Guarda los tiempos fijos de servicio, la tolerancia, el margen y
  los niveles de las tres pasadas (:doc:`4.3.1 </17.0/4_parametrization/4_3_planning-configuration>`).
- **Plan de transporte.** Un conjunto de Áreas geográficas. Cada Planning apunta a uno (4.3.2).
- **Zona operativa.** Un Área geográfica de tipo «Plan de Transporte». La Parada cae en la zona
  cuyo polígono contiene su dirección, y de la zona recibe su **Hub** y su Agencia (4.3.3).
- **Hub.** El contacto del que salen y al que vuelven los vehículos. Su horario de apertura
  limita cuándo se puede cargar y salir (13.2.3).
- **Franja de la Parada.** Una sola ventana, inicio–fin, en la que el Destinatario quiere el
  servicio (13.3.1).
- **Pasadas.** Hasta tres intentos encadenados, cada uno con la franja un poco más abierta, para
  que ninguna Parada se quede fuera (13.3). Y el **Estado de la ventana**, el veredicto por
  Parada después de calcular: En ventana, En tolerancia, Fuera de ventana o Sin ventana.
