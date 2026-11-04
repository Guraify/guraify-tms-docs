12.4 Lo que Odoo comunica: los avisos
-------------------------------------

.. admonition:: Ruta en Odoo
   :class: tip

   TMS › Configuración › **Avisos**: Workflows, Mensajes enviados, Enlaces de seguimiento ·
   Orden › pestaña **Avisos** · contacto › Propiedades TMS › grupo **AVISOS**

La comunicación proactiva del TMS es el capítulo :doc:`11 </17.0/11_notifications/index>`. Este
apartado la resume desde el punto de vista de quien construye encima: qué dispara un aviso, qué
decide si sale, por dónde sale y qué queda escrito.

**Qué lo dispara.** Quince momentos, y todos nacen de lo que cuenta 12.2 o del reloj: la Parada
existe (*Orden recibida*), entra en un Viaje (*Planificado*), el Viaje se despacha (*Viaje
iniciado*, en lote a una hora), el conductor cierra la anterior (*Eres el siguiente*), el ETA
vigente está a N minutos (*ETA dentro de N minutos*), el conductor llega (*Vehículo llegado*),
la Parada se cierra con su resultado (*Entregado*, *Con reservas*, *Fallida*, *Reprogramado*,
*Devuelto*), cambia la dirección, es el día anterior, hay un Reembolso sin cobrar o sube una
prueba de entrega. Los disparadores de hora (*El día anterior*, *Viaje iniciado*, *Reembolso
pendiente*) los lanza una tarea programada; los demás, el propio evento.

**Qué decide si sale.** Un :term:`Flujo de aviso` por cada cosa que se quiere decir: el
disparador, una condición sobre la Parada, el rol del destinatario (el Destinatario de la Parada,
el cliente que encargó la Orden o los seguidores internos), los Proyectos a los que aplica, la
política de canal, las plantillas y las restricciones: no repetir el mismo aviso en N minutos,
el horario de silencio en la zona horaria del Destinatario. Y antes de todo eso, el
**consentimiento**: sin móvil o correo, sin la marca de WhatsApp o en la lista negra, el aviso se
**omite con su motivo**, no se envía (:doc:`11.4 </17.0/11_notifications/11_4_recipient>`).

**Por dónde sale.** Correo, SMS (Esendex) y WhatsApp (la cuenta de Meta de la compañía, con
plantillas aprobadas), por los canales de Odoo. El flujo elige el preferido del contacto con
alternativa, uno fijo o todos. El envío lo hace una **tarea programada cada minuto** sobre la
cola; el evento solo encola. Los acuses vuelven: en WhatsApp, entregado, leído y **respondido**
(la respuesta abre un canal de chat de Odoo enlazado a la Parada, que es lo que la torre usará
para el chat operador ↔ destinatario); en SMS, la entrega la consulta la tarea de Esendex.

**Qué queda escrito.** Cada aviso, enviado u omitido, es una fila de **Mensajes enviados**
ligada a la Orden y a la Parada: disparador, canal, contacto, número o correo, estado (en cola,
retenido, enviado, entregado, leído, respondido, fallido, omitido) y motivo. Es la pestaña
**Avisos** de la Orden y una de las fuentes de la línea de tiempo de la torre (12.6).

**El enlace de seguimiento.** Cada aviso puede llevar un enlace personal
(``https://<dominio>/t/<código>``) que apunta a la **página del Destinatario**. Odoo genera el
código, lo guarda con su audiencia, su caducidad y su contador de accesos, y lo revoca; **la
página que abre es de la torre de control y todavía no existe**. Mientras tanto, los flujos van
con el enlace desmarcado. El dominio del enlace es el **dominio de seguimiento** de la marca de la
compañía (:doc:`4.1 </17.0/4_parametrization/4_1_operational-configuration>`, Marca de la
compañía).

**Para probar sin avisar a nadie.** En una copia de producción hay miles de móviles reales. La
**lista blanca de pruebas** (parámetro ``tms_notify.allowlist``) hace que solo se avise a los
móviles y correos que contenga; el resto se omite con su motivo. Se pone **antes** de crear el
primer flujo (:doc:`11.6 </17.0/11_notifications/11_6_go-live>`).

Lo que la comunicación proactiva **no hace todavía**: la página del enlace de seguimiento, la
cuenta real de WhatsApp en producción y el chat desde la página de seguimiento para quien no tiene
WhatsApp. Las tres son de la torre.
