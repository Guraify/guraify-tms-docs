A.7 Planning
============

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Configuración › Ajustes › Datos auxiliares › Plannings

Modelo Odoo: ``tms.planning``. La segmentación operativa: última milla, recogidas, distribución
urbana, larga distancia. Se escribe siempre «Planning», sin traducir, también en el menú de la
interfaz.

Además de los campos de la tabla, el maestro lleva los cuatro campos comunes a todos los catálogos
del TMS: **Secuencia** (orden en las listas), **Color** (etiqueta visual), **Compañía** (a qué
compañía pertenece; vacío es compartido) y **Defecto** (el registro que el sistema propone cuando
hay que elegir uno y nadie ha elegido).

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Campo
     - Tipo
     - Qué es
   * - Nombre
     - Texto
     - Nombre del Planning. Obligatorio.
   * - Descripción
     - Texto
     - Explicación breve. Obligatoria.
   * - Información
     - Texto
     - Notas internas.
   * - Plan de Transporte
     - Relación con Plan de transporte
     - La red territorial sobre la que opera este Planning. Obligatorio: sin él no se resuelven las
       zonas operativas.
   * - Tiempo Recogida (seg) / Tiempo de Servicio (seg)
     - Número
     - Segundos que se añaden a cada recogida y a cada entrega cuando se envía el Planning al
       optimizador, además del tiempo de servicio propio de la Parada.
   * - Tolerancia después (min)
     - Número
     - Minutos que se admite llegar después del fin de la franja de una Parada. A 0, los de la zona
       operativa de cada Parada (``tms_tw_tolerance_after``).
   * - Margen sobre el fin (min)
     - Número
     - Minutos que se quitan al fin de la franja en el nivel Con margen; nunca deja una franja de
       menos de 30 minutos (``tms_tw_end_margin_min``).
   * - Pasada 1 / Pasada 2 / Pasada 3
     - Selección
     - El nivel de cada pasada: Con margen, Real, Con tolerancia, Con tolerancia × 2, Con tolerancia
       × 4, Fin abierto, Fin abierto sin límites; la 2 y la 3 admiten Sin pasada. De serie, Con
       margen / Con tolerancia / Fin abierto sin límites (``tms_pass_1_level`` a
       ``tms_pass_3_level``). Grupo **Margen y pasadas**.
   * - Modo División Viaje
     - Selección
     - Cómo se reparten los importes de un Viaje entre sus Paradas: Volumen, Lineal, Metros, Bulto,
       Pallets, Cantidad, Peso, o cualquiera de ellos combinado con los kilómetros (Km y Peso, Km y
       Bulto...). Ver :doc:`/17.0/8_economic-administration/8_3_cost-split`.
   * - Remitente
     - Relación con Contacto
     - Remitente asociado al Planning, cuando toda la operativa es de un mismo origen.
   * - Proveedor de optimización
     - Selección
     - PTV Developer o PTV OptiFlow para las optimizaciones de este Planning. Vacío, el de la
       compañía (``geo_provider_optimize``).

Cómo se usa y qué decide el cliente: :doc:`/17.0/4_parametrization/4_3_planning-configuration`.
