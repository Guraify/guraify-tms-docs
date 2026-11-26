8.9 Canon de combustible
------------------------

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Administración › Opciones de Tarifa › **Canon de combustible** (Índices de precios,
   Precios, Tablas de coeficientes, Políticas, Periodos, Importar precios) · TMS › Configuración ›
   Ajustes › bloque **Canon de combustible** · pestaña **Canon de combustible** de la Orden, del
   Viaje y de la factura

El :term:`canon de combustible <Canon de combustible>` es el recargo (o descuento) que la ley
española obliga a repercutir en el precio del transporte cuando el gasóleo sube o baja entre la
fecha en que se pactó el precio y la fecha del transporte (artículo 38 de la Ley 15/2009, con la
fórmula de la condición 3.4 de la Orden FOM/1882/2012 y sus modificaciones de 2022 y 2026). El
TMS lo calcula por sí solo, con la fórmula legal o con lo que cada cliente tenga pactado, y lo
pone como **una línea más** de la Orden de venta, de la Orden de compra del Viaje o de la factura.
Cuando lo pactado no es la fórmula legal, calcula también el valor legal y avisa si difieren.

Es un módulo aparte (``tms_fuel_surcharge``) y nace **apagado**: hasta que un responsable activa
el interruptor de la compañía, ninguna Orden ni factura recibe una línea de canon.

.. figure:: /_static/img/8_economic-administration/8_9_fuel-surcharge_01_politica.png
   :alt: Ficha de una política del canon de combustible con la fórmula legal

   Una política del canon: lado, método y aplicación; dónde se toma la fecha de referencia y
   la clase de vehículo; y los periodos calculados, mes a mes.

8.9.1 Cómo se calcula
~~~~~~~~~~~~~~~~~~~~~

La fórmula legal dice que el precio del transporte varía un porcentaje igual a **G × C**:

- **G** es la variación, en porcentaje, del precio del gasóleo entre la **fecha de referencia**
  (cuando se pactó el precio) y la **fecha del transporte**. El TMS la mide con el precio
  semanal que publica el Ministerio de Transportes.
- **C** es el **coeficiente** que la ley fija según la clase de vehículo (su MMA, y si es de
  obras) y, desde 2026, según el tramo de precio en que esté el gasóleo. Los coeficientes vienen
  de serie, versionados por fecha: la tabla que vale es la vigente **el día del transporte**.

Dos detalles de la ley que el TMS aplica solos. El precio con el que se mide G no es siempre el
mismo: con las tablas de 2026 es el precio antes de impuestos más el impuesto de hidrocarburos
(PAI + IEH) y, si el vehículo tiene **7.500 kg o más** y quien lo lleva está acogido a la
**devolución del gasóleo profesional**, se descuenta esa devolución (Pmed). Y la revisión solo
procede si la variación supera un **umbral** (el 5 % como máximo; se puede pactar uno menor), salvo
en los **contratos continuados**, donde se revisa en cada periodo de facturación sea cual sea la
variación. Las políticas nacen como contrato continuado.

El resultado es un porcentaje, con signo: si el gasóleo ha bajado desde la referencia, el canon
es negativo y la línea descuenta. Se aplica sobre la **base**: la suma de las líneas de la Orden
cuyo producto está marcado como **Base del canon de combustible** (los portes y sus suplementos,
nunca los reembolsos, la manipulación ni el propio canon; :doc:`4.4.5
</17.0/4_parametrization/4_4_economic-configuration>`).

8.9.2 Precios del gasóleo y coeficientes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Los precios viven en **Índices de precios**: una serie por fuente, con un punto por semana
(**Precios**) que guarda el precio en surtidor, el precio sin IVA, el precio antes de impuestos,
el impuesto de hidrocarburos y la devolución del gasóleo profesional. Vienen dos índices de
serie: el del **Ministerio de Transportes** (el que usa el canon) y el **Oil Bulletin semanal de
la UE**, que sirve de contraste. Un precio que no está publicado no existe: el TMS **nunca
estima ni interpola**, y una semana sin precio deja el canon sin calcular, con su diagnóstico.

Los precios entran de dos maneras:

- **Descarga.** Una tarea programada semanal (los martes de madrugada) descarga el fichero del
  Ministerio y el del Oil Bulletin y añade las semanas nuevas; **Descargar ahora**, en la ficha
  del índice, lo hace al momento. Cada intento queda en el registro de actividad de las APIs
  (:doc:`/17.0/7_edi-integrations/7_7_integration-best-practices`), y si la serie se queda más de
  catorce días atrasada, el **Responsable** del índice recibe una actividad. Un servidor sin
  salida a internet no puede descargar: el índice guarda el error y queda la importación.
- **Importar precios.** El asistente lee el xlsx semanal del Ministerio tal como se descarga de
  su web, o un xlsx o csv propio con columnas por cabecera o por letra, y dice cuántas filas ha
  leído, creado, actualizado y descartado, con el motivo de cada descarte.

Una semana puntual que el Ministerio no publica («ND») se da de alta a mano como PVP sin IVA, y
el TMS la lee como PAI + IEH.

Las **Tablas de coeficientes** traen los coeficientes C de cada periodo legal, por país y con
sus fechas de transporte: la Orden FOM/1882/2012, los reales decretos-ley de 2022 y 2023, y los
tres de 2026 (RDL 7/2026, 9/2026 y 18/2026), que introducen los tramos de precio. Son datos, no
código: cuando cambie la ley, un responsable cierra la tabla vigente con su fecha de fin y crea la
siguiente. Una celda que no está en la tabla no se inventa: el canon queda sin calcular con el
diagnóstico «coeficiente no verificado».

8.9.3 Las políticas
~~~~~~~~~~~~~~~~~~~

Una :term:`política <Política del canon>` dice cómo se calcula el canon para alguien. Lo primero
que decide es el **método**:

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Método
     - Qué hace
   * - **Fórmula legal (España)**
     - La de 8.9.1. Es el método por defecto y el que se calcula siempre en paralelo, se use el
       que se use.
   * - **Mes de referencia y % de repercusión**
     - Lo pactado en muchos contratos: la variación del precio entre el mes de referencia y el
       del transporte, multiplicada por un **% de repercusión** (la parte del coste que es
       combustible), con **puntos fijos** que se suman si el resultado es positivo y, si se
       quiere, **nunca negativo**. Elige con qué precio se mide (**Base de precio**), si con la
       semana o con la **media del mes** (**Muestreo**) y con cuántos meses de **decalaje**.
   * - **Tabla de precios**
     - Un porcentaje por tramo de precio del gasóleo (pestaña **Tramos de precio**).
   * - **% fijo**
     - El porcentaje pactado, sin mirar el precio.
   * - **Incluido en la tarifa**
     - No se añade línea: el precio ya lo lleva.
   * - **Fijado a mano por periodo**
     - Lo que diga cada **periodo** escrito a mano (abajo).

Después, **para quién** y **dónde** se aplica: el **lado** (ventas, compras o ambos) y la
**aplicación**, **orden a orden** (la línea va en cada Orden de venta y en cada Orden de compra)
o **una vez por factura** (una sola línea en la factura agrupada, 8.9.5). Y los datos del
cálculo:

- **Fecha de referencia.** Por defecto, el **inicio de la versión de tarifa** vigente en la fecha
  del transporte (:doc:`4.4.3 </17.0/4_parametrization/4_4_economic-configuration>`): cada
  revisión de precios reinicia la referencia, y una sola política vale para todos los clientes
  con tarifa versionada. Quien no tiene tarifa versionada usa una **fecha fija**.
- **Fecha del transporte.** En ventas, la de la Orden o la de su Viaje; en compras, siempre la
  del Viaje.
- **Clase de vehículo.** **Del vehículo** del Viaje (si no tiene, de su categoría, por la MMA de
  su modelo; y si tampoco, la **clase por defecto** de la compañía) o una **clase fija**. Las
  clases son las de la ley (hasta 3.500 kg, de 3.500 a 20.000, 20.000 o más, y vehículo de
  obras), partidas en 7.500 kg para saber si procede la devolución del gasóleo profesional.
- **Umbral** y **contrato continuado**, para la fórmula legal y la de repercusión.
- **Base y línea**: todas las líneas de productos base o solo las que vienen de la tarifa, unos
  productos base propios si los de la compañía no valen, los decimales del porcentaje, el
  producto de la línea del canon (sin él, el de la compañía) y si se avisa cuando difiere del
  valor legal.

La ficha avisa, sin impedir guardar, cuando una política va contra la ley: un umbral por encima
del 5 %, puntos fijos o un suelo en cero.

**Quién usa cada política.** El TMS busca la política en este orden y se queda con la primera
que encuentra. En **ventas**: la Orden (un responsable puede fijarla a mano en la ficha), su
Proyecto, su cliente (o la empresa del cliente), su Tarifa y, por último, la **política de ventas
por defecto** de la compañía. En **compras**: el Viaje, su Proyecto, su transportista, la Tarifa
del transportista y la **política de compras por defecto**. Sin ninguna, se aplica la fórmula
legal con el índice de la compañía. Las políticas están pensadas para ser **pocas y
compartidas** (una por método, porcentaje y fecha de referencia, no una por cliente), y se
asignan en bloque desde las listas de contactos, de Proyectos y de Tarifas: se muestra la columna
de la política, se seleccionan las filas y se edita una vez.

**Periodos.** Cada política lleva sus periodos (pestaña **Periodos**, y la lista **Periodos** del
menú). Los **calculados** los escribe el TMS, uno por mes, con el porcentaje, el legal, G y C del
último cálculo de ese mes: son informativos. Los **fijados a mano** mandan: en las fechas que
cubren, el canon es el porcentaje escrito, con una **nota** obligatoria que diga quién lo dijo, y
nacen bloqueados. Es la forma de recoger un acuerdo puntual («este mes el 7 %, lo dijo el
cliente») sin tocar la política.

8.9.4 La línea del canon, orden a orden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: /_static/img/8_economic-administration/8_9_fuel-surcharge_02_orden.png
   :alt: Pestaña Canon de combustible de una Orden

   La pestaña Canon de combustible de una Orden: el porcentaje y el importe aplicados, el
   valor legal, si difieren, y el botón Recalcular canon.

Con una política de aplicación **orden a orden**, el canon se escribe **justo después de
tarificar** (:doc:`/17.0/5_operational-flows/5_6_settlement`), en la venta y en la compra:

- En la **Orden de venta**, sobre la suma de sus líneas base, con la fecha de la Orden (o la de
  su Viaje) y el vehículo del Viaje; sin Viaje, con la categoría de vehículo de la Orden.
- En la **Orden de compra del Viaje**, sobre sus líneas base, con la fecha del Viaje y su
  vehículo. Para la devolución del gasóleo profesional cuenta la marca del transportista; en la
  flota propia, la de la compañía. Un Viaje de flota propia no tiene Orden de compra ni canon.

La línea lleva el producto del canon de la compañía (referencia interna **0022**), cantidad 1,
el importe como precio, los impuestos del producto, y el nombre «Canon de combustible
septiembre 2026 · 13,18 %». Hay **una sola** por Orden: cada tarificación la crea, la actualiza o
la borra, y un importe de cero la borra (una línea a cero acabaría en la factura). Las pasadas de
tarifa no la tocan, y si alguien la ha editado a mano, el TMS **no la vuelve a tocar**: lo dice en
el diagnóstico. Una Orden confirmada o una Orden de compra facturada no se recalculan.

La pestaña **Canon de combustible** de la Orden y del Viaje enseña el porcentaje y el importe
aplicados, el **porcentaje legal**, si **difieren** y, cuando no se ha podido calcular, el
**diagnóstico** (8.9.6). **Recalcular canon** vuelve a calcular solo el canon, sin retarificar.
En la ficha, un responsable puede fijar la **política** a mano, por delante de la del Proyecto y
el cliente.

8.9.5 Una vez por factura
~~~~~~~~~~~~~~~~~~~~~~~~~

Con una política de aplicación **una vez por factura**, las Órdenes no llevan línea (su
diagnóstico lo dice) y el canon se añade a la **factura agrupada** de cliente o de transportista
(:doc:`/17.0/5_operational-flows/5_7_invoicing`) como una sola línea: «Canon de combustible ·
9,77 % sobre 52,83 €». La factura estándar de Odoo, una por Orden, no lo lleva.

El cálculo sigue siendo **pedido a pedido**: cada Orden de la factura se calcula con su propia
fecha, su vehículo y su política, y los importes se suman; el porcentaje de la factura es el
importe entre la base. Una línea añadida a mano a la factura, sin Orden detrás, no entra en la
base. La pestaña **Canon de combustible** de la factura enseña la política, la base, el
porcentaje, el legal y los **periodos** que cubren sus Órdenes; **Recalcular canon** solo actúa
en borrador y, al **confirmar** la factura, esos periodos quedan bloqueados. El abono que el TMS
genera cuando una factura sale negativa no lleva canon.

8.9.6 Cuando el cálculo no puede hacerse
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si falta algo, no hay línea y el **diagnóstico** de la Orden, del Viaje o de la factura dice qué:

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Diagnóstico
     - Qué hacer
   * - No hay producto del canon en la compañía (referencia 0022)
     - Crear el producto y ponerlo en Ajustes. Sin él no se crea nada.
   * - No hay precio publicado para una de las fechas
     - Descargar o importar la semana que falta; si el Ministerio no la publica, darla de alta a
       mano como PVP sin IVA.
   * - No hay fecha de referencia
     - La Orden no tiene Tarifa con versión y la política no tiene fecha fija: darle versión a
       la Tarifa o una fecha fija a la política.
   * - Coeficiente no verificado para la clase y la fecha / No hay tabla de coeficientes
     - Revisar las Tablas de coeficientes: falta el periodo o la celda.
   * - El precio no cae en ningún tramo
     - Completar los Tramos de precio de la política.
   * - La variación no llega al umbral
     - No es un error: el contrato no es continuado y el precio se movió menos que el umbral.
   * - Incluido en la tarifa / Sin canon fijado para este periodo
     - Lo dice la política: incluido, o método a mano sin periodo que cubra la fecha.
   * - Aplicado una vez por factura
     - La Orden no lleva línea; la llevará su factura agrupada.
   * - La línea se editó a mano
     - El TMS la respeta. Para volver al cálculo, borrar la línea y Recalcular canon.

8.9.7 Puesta en marcha en un cliente
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El orden que evita sorpresas:

#. **Producto y ajustes.** Comprobar que la compañía tiene el producto del canon (0022) con su
   IVA, y en Ajustes › TMS › Canon de combustible darle el índice por defecto, la clase de
   vehículo por defecto, las políticas de ventas y compras por defecto y, si la flota propia
   está acogida a la devolución del gasóleo profesional, marcarlo. El interruptor, todavía no.
#. **Precios.** Descargar el índice del Ministerio (o importar su xlsx) y comprobar que llega
   hasta la semana en curso.
#. **Productos base.** En Ventas › Productos, con el filtro **Base del canon de combustible**,
   revisar qué productos entran en la base. Al instalar, el TMS marca los productos de transporte
   que usa alguna Tarifa; un descuento marcado como base hace que el canon baje con él.
#. **Vehículos.** Que los modelos tengan su MMA y, si procede, la marca **Vehículo de obras**; de
   ahí sale la clase. Marcar la devolución del gasóleo profesional en los transportistas acogidos.
#. **Políticas, pocas.** Una por método, porcentaje y fecha de referencia; con la fecha de
   referencia por versión de tarifa, una sola sirve a todos los clientes con tarifa versionada.
   Asignarlas en bloque desde las listas. Lo singular, con un periodo a mano.
#. **Contraste.** Antes de encender el interruptor, comparar el porcentaje que da la pestaña
   Canon de combustible de unas Órdenes de un mes cerrado con lo que se facturó ese mes.
#. **Encender** el interruptor de la compañía. Desde ese momento, cada tarificación añade su
   línea.

Campos en :doc:`/17.0/annexes/A_36_canon-combustible`.
