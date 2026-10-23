A.36 Canon de combustible
=========================

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Administración › Opciones de Tarifa › Canon de combustible · TMS › Configuración ›
   Ajustes › Canon de combustible · Orden, Viaje y factura › pestaña Canon de combustible

Módulo ``tms_fuel_surcharge``. Los catálogos los leen los usuarios del TMS y los cambian los
responsables. Cómo se calcula y se pone en marcha:
:doc:`/17.0/8_economic-administration/8_9_fuel-surcharge`.

A.36.1 Política del canon
-------------------------

Modelo Odoo: ``tms.fuel.surcharge.policy``. Sin compañía, la comparten todas.

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Campo
     - Tipo
     - Qué es
   * - Lado
     - Selección
     - Ventas, Compras o Ambos (``side``). Una política de ventas no se aplica a un Viaje, ni al
       revés.
   * - Método
     - Selección
     - Fórmula legal (España), Mes de referencia y % de repercusión, Tabla de precios, % fijo,
       Incluido en la tarifa o Fijado a mano por periodo (``method``).
   * - Aplicación
     - Selección
     - Orden a orden o Una vez por factura (``apply_level``).
   * - Índice de precios
     - Relación
     - La serie de precios; vacío, el índice por defecto de la compañía.
   * - Base de precio
     - Selección
     - Con qué precio se mide la variación en los métodos pactados: PVP, PVP sin IVA, PAI,
       PAI + IEH (por defecto) o Pmed. La fórmula legal lo decide sola.
   * - Muestreo del precio
     - Selección
     - Semana de la fecha o Media del mes.
   * - Decalaje (meses)
     - Número
     - 1 aplica el precio del mes anterior.
   * - Fecha de referencia
     - Selección
     - Inicio de la versión de tarifa (por defecto) o Fecha fija, con la **Fecha de referencia
       fija** (``reference_date_mode``, ``reference_date``).
   * - Fecha del transporte
     - Selección
     - Fecha de la orden (por defecto) o Fecha del viaje; en compras, siempre la del Viaje.
   * - Clase de vehículo desde
     - Selección
     - Del vehículo (Viaje › categoría › compañía) o Fija, con la **Clase de vehículo fija**.
   * - Umbral (%)
     - Número
     - Por debajo de esta variación no hay revisión; 5 por defecto y nunca más de 5.
   * - Contrato continuado
     - Sí/No
     - Se revisa en cada periodo, sea cual sea la variación. Marcado por defecto.
   * - Repercusión (%), Puntos fijos añadidos (%), Nunca negativo
     - Números y Sí/No
     - Los parámetros del método de mes de referencia.
   * - % pactado
     - Número
     - El del método % fijo.
   * - Tramos de precio
     - Pestaña
     - Precio mínimo (incluido), máximo (excluido; 0 = sin límite) y porcentaje, para el método
       Tabla de precios.
   * - Periodos
     - Pestaña
     - Los periodos de la política (A.36.2).
   * - Base
     - Selección
     - Todas las líneas de productos base, o solo las líneas de tarifa de productos base
       (``base_scope``).
   * - Productos base
     - Relación
     - Vacío, los productos marcados como base; con valores, solo esos.
   * - Decimales del %
     - Número
     - Redondeo del porcentaje; 2 por defecto.
   * - Producto del canon
     - Relación
     - Vacío, el de la compañía.
   * - Avisar si difiere del valor legal
     - Sí/No
     - Marcado por defecto.
   * - Aviso legal
     - Texto calculado
     - Lo que de la política va contra el artículo 38: umbral mayor del 5 %, puntos fijos o suelo
       en cero. No impide guardar.

A.36.2 Periodo del canon
------------------------

Modelo Odoo: ``tms.fuel.surcharge.period``. Dos periodos de una política no se solapan.

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Campo
     - Tipo
     - Qué es
   * - Desde / Hasta
     - Fechas
     - Las fechas del transporte que cubre.
   * - Modo
     - Selección
     - Fijado a mano (manda, nace bloqueado, exige nota) o Calculado (lo escribe el TMS, uno por
       mes, con el último cálculo).
   * - Canon (%), Legal (%)
     - Números
     - El porcentaje aplicado y el legal.
   * - Precio de referencia, Precio del transporte, G (%), Coeficiente C, Clase de vehículo
     - Solo lectura
     - La traza del cálculo.
   * - Bloqueado
     - Sí/No
     - Un periodo bloqueado no se refresca. Confirmar una factura bloquea los suyos.
   * - Nota
     - Texto
     - Obligatoria en un periodo a mano: quién lo dijo.

A.36.3 Índice de precios y precios
----------------------------------

Modelos Odoo: ``tms.fuel.price.index`` y ``tms.fuel.price`` (un punto por índice y fecha).

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Campo
     - Tipo
     - Qué es
   * - Nombre, Código, País, Moneda
     - Texto y relaciones
     - La serie. De serie, ``ES_MITMA_WEEKLY`` (Ministerio de Transportes) y ``EU_WOB_ES_DIESEL``
       (Oil Bulletin de la UE).
   * - Unidad
     - Selección
     - Por litro o Por 1.000 litros, la de la fuente; el índice responde siempre por litro.
   * - Fuente, URL de la fuente
     - Selección y texto
     - Manual, Ministerio de Transportes (xlsx semanal), Oil Bulletin semanal de la UE o CSV
       genérico (este no se descarga), y de dónde.
   * - Periodicidad
     - Selección
     - Semanal (los puntos van al lunes de su semana), Mensual (al día 1) o Diaria.
   * - Responsable
     - Relación
     - Recibe una actividad cuando la serie descargada lleva más de 14 días atrasada; vacío,
       quien creó el índice.
   * - Última descarga, Estado y Mensaje de la última descarga
     - Solo lectura
     - El resultado del último intento.
   * - Precios: Fecha, Precio en surtidor (PVP), PVP sin IVA, Precio antes de impuestos (PAI),
       Impuesto sobre hidrocarburos (IEH), Devolución (GP), PAI + IEH, Pmed, Fuente, Cargado el
     - El punto
     - Los valores publicados de la semana; PAI + IEH y Pmed se calculan. La fuente dice si el
       punto es manual, importado o descargado.

A.36.4 Tabla de coeficientes
----------------------------

Modelo Odoo: ``tms.fuel.coefficient.table`` y sus líneas. Las tablas de un país no se solapan.

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Campo
     - Tipo
     - Qué es
   * - Nombre, Referencia legal, País
     - Texto y relación
     - La norma: Orden FOM/1882/2012, RDL 11/2022 + 20/2022, RDL 7/2026, 9/2026 y 18/2026 de
       serie.
   * - Transportes desde / hasta
     - Fechas
     - Las fechas de transporte a las que aplica; sin fin, vigente.
   * - Base de los tramos
     - Selección
     - Fijo (sin tramos de precio), PAI o Precio de referencia (Pmed): con qué precio se elige
       el tramo.
   * - Base de G
     - Selección
     - Con qué precio se mide G: PVP, PAI + IEH, o Pmed con la devolución del gasóleo profesional
       y PAI + IEH sin ella.
   * - Coeficientes
     - Líneas
     - Clase de vehículo (las cuatro legales), precio mínimo y máximo del tramo y coeficiente C.

A.36.5 Dónde más aparece
------------------------

.. list-table::
   :header-rows: 1
   :widths: 28 22 50

   * - Dónde
     - Campo
     - Qué es
   * - Ajustes › TMS › Canon de combustible (``res.company``)
     - Canon de combustible activo, Producto del canon, Índice por defecto, Política del canon
       (ventas) y (compras), Clase de vehículo por defecto, Devolución del gasóleo profesional
     - El interruptor (apagado de serie), el producto 0022, el índice, las políticas de último
       recurso, la clase cuando ni el vehículo ni su categoría la dan (MMA de 20.000 kg o más de
       serie) y si la flota propia está acogida a la devolución.
   * - Contacto › Propiedades TMS (``res.partner``)
     - Política del canon (ventas), Política del canon (compras), Devolución del gasóleo
       profesional
     - La política del cliente y la del transportista, y si el transportista está acogido a la
       devolución.
   * - Proyecto › Administración (``project.project``)
     - Política del canon (ventas) y (compras)
     - Por delante de las del cliente y el transportista.
   * - Tarifa › Asignación (``tms.pricelist``)
     - Política del canon de combustible
     - Por detrás del cliente o transportista.
   * - Orden y Viaje
     - Política del canon de combustible (solo responsables); Canon de combustible (%), importe,
       Canon legal (%), Difiere del valor legal, Diagnóstico
     - La política fijada a mano y el resultado, en la pestaña Canon de combustible.
   * - Factura (``account.move``)
     - Política, Base del canon, Canon (%), Canon legal (%), Difiere, Diagnóstico, Periodos
     - El resultado de una política de nivel factura.
   * - Producto (``product.template``)
     - Base del canon de combustible
     - El producto entra en la base. Filtro del mismo nombre en Ventas › Productos.
   * - Modelo y vehículo (``fleet.vehicle.model``, ``fleet.vehicle``)
     - Vehículo de obras
     - Junto a la MMA; con ella decide la clase de vehículo.
   * - Líneas de venta y de compra
     - Aplicación del precio = Canon de combustible
     - Lo que distingue la línea del canon de las de tarifa; la factura lleva su propia marca.
