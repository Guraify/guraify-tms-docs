12 La operación en vivo
=======================

Los capítulos anteriores cuentan cada pieza del TMS en su sitio: el despacho en el 5, el
recálculo del ETA en el 1 y el 4, los avisos en el 11, la telemática en el 7. Este capítulo las
pone **en orden de tiempo**: qué pasa, de principio a fin, con un Viaje desde que sale a la
calle hasta que vuelve, qué entra en Odoo, qué calcula, qué guarda, qué decide, qué comunica y
qué deja preparado para la **torre de control**. Es la lectura previa para quien vaya a
construir sobre la operación en vivo —la torre, un cuadro de mando, una integración— y para el
consultor que tenga que explicar por qué el ETA cambió o por qué saltó un aviso.

No repite lo que ya está escrito: cada apartado dice lo justo para seguir el hilo y enlaza a la
sección que lo detalla. Lo que **todavía no existe en Odoo** (las alarmas y la propia torre) va
marcado como evolución prevista, con la especificación como fuente.

.. mermaid::

   flowchart LR
       D["Despacho<br/>5.4.1"] --> A["Eventos de la app<br/>y señales de telemática<br/>12.2"]
       A --> T["Trazabilidad<br/>y estados<br/>3.2.7"]
       T --> E["Recálculo del ETA<br/>12.3"]
       T --> N["Avisos al Destinatario<br/>12.4"]
       E --> N
       E --> AL["Alarmas<br/>12.5"]
       T --> AL
       AL --> TC["Torre de control<br/>12.6"]
       E --> TC
       N --> TC

.. toctree::
   :maxdepth: 1

   12_1_journey
   12_2_inputs
   12_3_eta
   12_4_notifications
   12_5_alarms
   12_6_tower
