# Proceso de citas

**Estado:** aprobado como definición canónica del proceso de citas del MVP

Este documento define el comportamiento canónico de una cita. Aplica al canal
público para clientes invitados y al canal interno para personal autenticado.

## Participantes y permisos

| Participante | Responsabilidad en el MVP |
| --- | --- |
| Público | Consultar información publicada, profesionales y horarios libres. |
| Cliente invitado | Reservar después de demostrar acceso al correo informado y gestionar únicamente su cita mediante acceso seguro. |
| Administrador | Configurar el negocio y gestionar cualquier cita de su organización. |
| Profesional | Consultar citas asignadas e iniciar, finalizar o marcar inasistencia. |
| Sistema | Calcular horarios libres, proteger la concurrencia, notificar y auditar. |

Solo el administrador puede crear o reprogramar citas desde el canal interno. Una
persona que atiende su propia agenda puede tener simultáneamente los roles de
administrador y profesional.

Cada solicitud vuelve a comprobar que la organización esté `active` y que la
membresía o capacidad segura continúe autorizada. Suspender la organización
bloquea los canales interno y público, incluidos los accesos de invitados.

## Condiciones previas

Para reservar deben cumplirse en el mismo tenant:

- Una organización `active` con una sede y zona horaria.
- Un servicio que no esté `archived`, usando su duración vigente.
- Un profesional operativo y una asignación activa para ese servicio.
- Un horario libre suficiente para cubrir todo el intervalo.
- Un contacto existente o datos válidos para crearlo atómicamente con la cita.

El canal público exige además página, servicio y profesional `published`, junto
con un correo verificado. El canal interno admite elementos `private` y exige
nombre y al menos un correo o teléfono para un contacto nuevo.

La cita guardará copias del nombre, duración, precio y moneda informativos del
servicio, además del nombre, dirección y zona horaria de la sede usados al
reservar. Los cambios posteriores no modificarán reservas existentes. Los datos
del cliente seguirán las reglas de instantánea de
[Configuración operativa](operational-configuration.md).

## Estados

La operación y la confirmación se modelan por separado:

| Estado operativo | Significado | Bloquea horario |
| --- | --- | --- |
| `scheduled` | Reserva vigente que todavía no ha iniciado. | Sí |
| `in_progress` | El profesional comenzó la atención. | Sí |
| `completed` | El servicio terminó. | No |
| `cancelled` | La cita terminó sin completarse por cancelación o interrupción. | No |
| `no_show` | El cliente no se presentó. | No |

| Confirmación | Significado |
| --- | --- |
| `pending` | El cliente todavía no declaró que asistirá. |
| `confirmed` | El cliente confirmó mediante su acceso seguro. |

Toda cita nace `scheduled` y `pending`. La falta de confirmación no cancela la
cita ni libera el horario. `completed`, `cancelled` y `no_show` son estados
terminales. Una cita `scheduled` cuyo fin previsto ya pasó se considera
`overdue` de forma derivada; no es otro estado ni cambia automáticamente.

## Transiciones

| Acción | Cambio | Actor autorizado | Regla principal |
| --- | --- | --- | --- |
| Reservar | creación → `scheduled` | Cliente invitado con correo verificado o administrador | El intervalo se reclama atómicamente. |
| Confirmar asistencia | `pending` → `confirmed` | Cliente | Solo mientras siga `scheduled` y antes del inicio previsto. |
| Reprogramar | `scheduled` → `scheduled` | Cliente o administrador | Solo antes del inicio previsto; cambia el intervalo y vuelve a `pending`. |
| Cancelar | `scheduled` → `cancelled` | Cliente o administrador | El cliente solo antes del inicio; el administrador exige motivo si ya pasó. |
| Iniciar | `scheduled` → `in_progress` | Profesional asignado o administrador | Desde el inicio previsto; reclama atómicamente la atención del profesional. Tras el fin, solo el administrador con motivo. |
| Finalizar | `in_progress` → `completed` | Profesional asignado o administrador | Registra una hora final posterior al inicio. |
| Interrumpir | `in_progress` → `cancelled` | Administrador | Exige motivo y hora real de finalización. |
| Marcar inasistencia | `scheduled` → `no_show` | Profesional asignado o administrador | Desde quince minutos después del inicio previsto. |
| Registrar atención realizada | `scheduled` → `completed` | Administrador | Solo después del fin previsto; exige horas reales y motivo. |
| Corregir resultado | `no_show` ↔ `completed` | Administrador | Solo después del fin previsto, con motivo y sin recalcular disponibilidad. |

Confirmar repetidamente tendrá el mismo resultado sin duplicar eventos. La
reprogramación conservará la cita y registrará los intervalos anterior y nuevo;
no será un estado. Las acciones sobre estados terminales se rechazarán, salvo la
corrección de resultado definida en la tabla. Una cancelación no se restaurará:
si fue errónea, deberá crearse otra cita.

## Horarios libres y concurrencia

Un horario libre es un intervalo calculado a partir de la disponibilidad
recurrente, los bloqueos, las citas vigentes y la duración del servicio. No se
almacena ni garantiza que continúe libre hasta reservarlo.

- Los cálculos se realizan en la zona horaria IANA de la organización y los
  instantes se almacenan en UTC.
- Los horarios locales inexistentes o ambiguos por cambios de horario estacional
  no se ofrecerán automáticamente.
- Los intervalos son semiabiertos: `[inicio, fin)`. Dos citas adyacentes no se
  superponen.
- Toda creación o reprogramación, pública o interna, debe iniciar en una
  cuadrícula de quince minutos, nunca en el pasado y como máximo noventa días
  hacia el futuro.
- El intervalo debe caber completamente en la disponibilidad semanal y no cruzar
  bloqueos ni citas `scheduled` o `in_progress` del profesional.
- Solo puede existir una cita `in_progress` por profesional.
- Cancelar libera el intervalo inmediatamente.
- Marcar inasistencia o interrumpir libera el resto del intervalo desde la
  siguiente posición futura de la cuadrícula.
- El horario mostrado es informativo: reservar y reprogramar siempre lo
  validan nuevamente.
- Dos operaciones concurrentes sobre intervalos superpuestos del mismo
  profesional producirán una sola reserva. La perdedora no dejará contactos,
  eventos ni correos parciales.
- La aplicación validará los cruces y PostgreSQL los impedirá mediante una
  restricción transaccional para citas `scheduled` o `in_progress`.
- Iniciar comprobará atómicamente que no exista otra cita `in_progress` del mismo
  tenant y profesional, incluso cuando sus intervalos previstos sean adyacentes.
- Crear o ampliar un bloqueo no podrá cruzar una cita vigente. Si compite con una
  reserva sobre el mismo profesional e intervalo, ambas operaciones se
  serializarán y solo una tendrá éxito.
- Reservar y reprogramar verificarán una versión vigente del horario, los
  bloqueos y el catálogo. Si un cambio termina primero, se aplicará la nueva
  configuración; si la reserva termina primero, la cita se conservará y cualquier
  bloqueo que la cruce será rechazado.

## Reserva pública

1. El servidor resuelve la organización desde su identificador público; no
   acepta un `tenant_id` elegido por el cliente.
2. El cliente solicita un enlace temporal y de un solo uso para verificar su
   correo. Verificar el correo no crea una cuenta ni una identidad global.
3. La verificación queda ligada al mismo correo y organización. Después, el
   cliente elige servicio, profesional y horario.
4. El servidor vuelve a validar y reserva inmediatamente si el horario sigue
   libre; de lo contrario solicita elegir otro.
5. El cliente recibe un acceso independiente para consultar, confirmar,
   reprogramar o cancelar esa cita.

La verificación previa evita que correos ajenos bloqueen la agenda. Se aplicarán
límites de frecuencia y controles contra automatización tanto a la solicitud del
correo como a la reserva.

## Enlaces seguros del invitado

- Ambos enlaces contendrán un token opaco, aleatorio y suficientemente largo;
  solo se almacenará su resumen seguro.
- El token de verificación estará ligado al tenant, correo, propósito,
  vencimiento y un único uso.
- El token de gestión estará ligado en el servidor a una cita, un tenant,
  acciones permitidas, vencimiento y revocación.
- Cada `GET` mostrará una página inerte. Una acción `POST` intercambiará el token
  por una sesión restringida en una cookie segura y lo retirará de la URL.
- Ningún `GET` ejecutará cambios y el token no aparecerá en registros, métricas ni
  servicios de analítica.
- Un acceso inválido, vencido o ajeno responderá sin revelar si el correo o la
  cita existen.
- Cambiar el correo, reenviar el acceso o detectar compromiso revocará el enlace
  de gestión anterior.
- El cliente podrá solicitar otro acceso con una respuesta no enumerable y
  limitada por frecuencia; solo se enviará al correo registrado en la cita.
- El cliente solo verá los datos necesarios de su propia cita; nunca notas
  internas, auditoría ni información de otras personas.

## Comunicaciones por correo

- La reserva pública enviará primero el enlace de verificación y, después de
  reservar, el acceso de gestión al mismo correo.
- La creación interna, reprogramación y cancelación enviarán el cambio al
  contacto solo cuando tenga correo.
- Una reserva interna sin correo seguirá siendo válida, no generará acceso de
  autoservicio y deberá gestionarla el administrador.
- Si no puede entregarse la verificación, no existirá todavía una reserva pública
  y el cliente podrá solicitar otro envío.
- Después de crear una cita, los fallos de entrega no cambiarán el resultado de
  la operación; quedarán visibles para reintento administrativo.

## Cambios y excepciones

- El cliente puede reprogramar o cancelar solo antes del inicio previsto.
- Al reprogramar, el cliente conserva el servicio y puede cambiar horario o
  profesional. Usa la duración y demás datos guardados del servicio; solo el
  nuevo horario actualiza la instantánea de sede y zona horaria. Cambiar el
  servicio requiere cancelar y crear otra cita.
- Reprogramar vuelve a validar la elegibilidad del canal. El invitado no podrá
  hacerlo si la página, el servicio o el profesional dejaron de estar
  publicados; conservará las demás acciones y podrá solicitar ayuda a la
  empresa. El administrador podrá usar elementos `private`, no `archived`.
- Una cita `overdue` se resolverá mediante inicio tardío, registro de la atención
  realizada, inasistencia o cancelación, según los permisos, datos y motivos
  exigidos por la transición correspondiente.
- Una cita iniciada solo puede finalizarse o interrumpirse según la tabla.
- La inasistencia se registra manualmente; el sistema nunca la presume.
- Cambiar la disponibilidad semanal o desactivar un servicio o profesional
  afectará solo nuevos horarios libres; las citas existentes se conservarán. El
  administrador deberá reprogramarlas o cancelarlas cuando ya no puedan
  atenderse.
- Si una atención se extiende sobre la siguiente cita, el sistema alerta, pero no
  cambia reservas automáticamente.
- Las citas y eventos no se eliminan mediante la operación ordinaria. Una
  corrección conserva los valores anterior y nuevo, el actor, la fecha y el
  motivo. Una política aprobada antes del piloto definirá retención y
  anonimización de datos personales sin destruir los hechos operativos mínimos.

## Consistencia y efectos secundarios

- Toda operación mutable será idempotente y verificará la versión vigente de la
  cita.
- Reserva, historial y evento de salida se escribirán en una sola transacción.
- El correo se enviará después del commit. Un fallo no revertirá ni duplicará la
  cita y podrá reintentarse o reenviarse.
- Cada evento registrará tenant, cita, actor, instante, origen, cambio y motivo
  cuando corresponda.
- Servicios, profesionales, contactos y citas de tenants distintos nunca podrán
  relacionarse.

## Escenarios mínimos de prueba

- Reserva pública e interna, confirmación repetida y acceso seguro del invitado.
- Reserva interna con correo y solo con teléfono, con las comunicaciones
  correspondientes en cada caso.
- Horario pasado, fuera de disponibilidad, bloqueado, incompatible o de otro
  tenant.
- Reservas simultáneas, reintentos y carreras entre reprogramar, cancelar e
  iniciar.
- Reprogramación exitosa y fallida sin perder el intervalo anterior.
- Cancelación, inicio, finalización e inasistencia en estados válidos e inválidos.
- Interrupción, atención registrada tarde y corrección entre resultado completo
  e inasistencia.
- Fallo y reintento del correo sin duplicar la cita ni el historial.
- Revocación del enlace y denegación de permisos entre actores y organizaciones.
- Carreras de una reserva contra un bloqueo y contra la desactivación del
  servicio o profesional.

