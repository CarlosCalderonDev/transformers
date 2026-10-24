# Configuración operativa del MVP

**Estado:** reglas canónicas definidas; alta autoservicio aprobada en
[ADR 0005](decisions/0005-self-service-organization-onboarding.md)

Este documento define cómo una organización queda preparada para operar el
[proceso de citas](appointment-process.md). Solo el administrador modifica la
configuración; el profesional la consulta y opera sus citas asignadas.

## Organización y publicación

| Estado | Comportamiento |
| --- | --- |
| `setup` | Permite configurar la organización, pero no reservar. |
| `active` | Permite operar y reservar cuando se cumplen las demás reglas. |
| `suspended` | Bloquea accesos y reservas; conserva datos y citas. |

- Toda organización comienza en `setup` y su página en `unpublished`.
- Una persona con cuenta autenticada y habilitada puede crear una organización
  en `setup`; la misma transacción le asigna su única membresía inicial,
  `active` con roles `administrator` y `professional`, y la registra como
  propietaria conforme a
  [ADR 0009](decisions/0009-organization-ownership-and-initial-roles.md).
- El alta solicita el nombre visible. El sistema propone su identificador
  público único para la URL y el administrador puede cambiarlo antes de crear
  la organización.
- La misma persona puede crear varias organizaciones. Cada creación conserva su
  propio tenant y no comparte configuración ni datos operativos con las demás.
- La configuración base contiene nombre, identificador público único, una sede
  con nombre y dirección, zona horaria IANA y moneda ISO 4217.
- Activar exige la configuración base completa y al menos un administrador
  efectivo; ese administrador ejecuta la activación.
- Solo un operador autorizado de TRANSFORMERS puede suspender y restaurar una
  organización mediante operaciones excepcionales y auditadas; no estarán
  disponibles para el tenant.
- El MVP no cerrará ni eliminará organizaciones.
- Después de activar, el identificador público, la zona horaria y la moneda son
  inmutables en el flujo ordinario del MVP. Una corrección requerirá una
  operación asistida y auditada. Editar el nombre o la sede afecta nuevas citas
  y citas existentes solo cuando se reprograman; las demás conservan su
  instantánea.
- La página de reservas tiene un estado independiente: `unpublished` o
  `published`. Despublicarla impide nuevas reservas públicas, pero no afecta
  citas existentes, sus accesos ni la gestión interna.

La primera publicación exige una organización activa y al menos una ruta
reservable: servicio publicado, profesional público y operativo, asignación
activa y horario semanal capaz de contener su duración. Si después deja de haber
horarios, la página sigue publicada y muestra que no hay disponibilidad.

## Invitaciones, membresías y roles

Una invitación es independiente de una membresía. Tiene estado `pending`,
`accepted` o `revoked`; la expiración se deriva de su fecha límite.

- Toda invitación comienza `pending`.
- Un administrador activo invita por correo y asigna uno o ambos roles fijos:
  `administrator` y `professional`.
- Solo un administrador activo puede revocar invitaciones pendientes, cambiar
  roles o suspender, restaurar y terminar membresías de su organización. Un
  integrante puede terminar la propia con las restricciones indicadas abajo.
- Aceptar exige una cuenta autenticada, demostrar acceso al correo y consumir un
  token temporal de un solo uso.
- La aceptación crea o reactiva la membresía en una transacción. Repetirla para
  la misma persona es idempotente.
- Una invitación vencida, revocada o destinada a otro correo no concede acceso.
- Las invitaciones y la gestión de integrantes quedan fuera del MVP. La única
  membresía inicial se crea junto con la organización conforme a ADR 0005.

La membresía tiene estado `active`, `suspended` o `ended`. Puede suspenderse,
restaurarse o terminarse con motivo y auditoría. Una membresía terminada solo se
reactiva mediante una nueva invitación aceptada. Existe como máximo una por
persona y organización.

- Los roles forman un conjunto no vacío; retirar el último termina la membresía.
- Un integrante puede retirarse, excepto si es el último administrador.
- Los cambios de estado o roles surten efecto inmediato en el servidor, aunque
  exista una sesión iniciada.
- Nunca se podrá suspender, terminar o despojar del rol al último administrador
  efectivo. Las operaciones concurrentes se serializarán por organización.
- Si la única cuenta administradora debe deshabilitarse por seguridad, la
  organización se suspende en la misma operación hasta su recuperación asistida.

Un administrador efectivo tiene cuenta habilitada, membresía `active` y rol
`administrator`. Una invitación pendiente no cuenta. Cada organización creada
en el MVP tiene desde su origen un administrador efectivo.

## Servicios, profesionales y asignaciones

| Elemento | Estados del MVP |
| --- | --- |
| Servicio | `private`, `published`, `archived` |
| Visibilidad profesional | `private`, `published` |
| Asignación profesional-servicio | `active`, `inactive` |

- Todo servicio y perfil profesional comienza `private`. Una asignación nueva
  comienza `active` cuando cumple sus invariantes.
- Un servicio privado puede reservarse internamente; uno publicado también puede
  aparecer en el canal público; uno archivado no admite nuevas reservas.
- Archivar un servicio es reversible únicamente hacia `private`. No se elimina
  si tiene historial.
- El nombre será único por normalización entre servicios no archivados. La
  duración será positiva y múltiplo de quince minutos; el precio será opcional y
  no negativo en la moneda de la organización.
- Editar nombre, duración o precio afecta solo nuevas citas. Las existentes
  conservan su instantánea, incluso al reprogramarse.
- Un profesional es operativo cuando la organización, su cuenta y su membresía
  están activas y posee el rol `professional`.
- La visibilidad `published` solo controla el canal público. El canal interno
  también puede usar profesionales privados que sean operativos.
- Solo habrá una asignación por organización, profesional y servicio. Activarla
  exige que todos pertenezcan al mismo tenant, el profesional sea operativo y el
  servicio no esté archivado.
- Inactivar una asignación o un elemento relacionado impide nuevas reservas y
  reprogramaciones hacia esa combinación, sin alterar las citas existentes.

## Horario semanal y bloqueos

El horario semanal de cada profesional se reemplaza como un conjunto completo,
atómico y versionado. La ausencia de ventanas significa que no está disponible.

- Cada ventana usa día de semana y horas locales de la organización, con
  intervalos semiabiertos `[inicio, fin)`.
- Una ventana no cruza medianoche; se divide entre ambos días. Los solapamientos
  se rechazan y las ventanas adyacentes se unen de forma determinista.
- Los cambios afectan nuevos horarios libres, no citas existentes. La operación
  devuelve las citas futuras que quedaron fuera del nuevo horario.
- Los horarios locales inexistentes o ambiguos por cambios estacionales no se
  ofrecen automáticamente.

Un bloqueo es una excepción fechada de un profesional. Tiene estado `active` o
`cancelled`; `expired` es una condición derivada.

- Puede abarcar varios días y su motivo es interno y opcional.
- Crear, ampliar o mover un bloqueo se rechaza si cruza una cita vigente u otro
  bloqueo activo del profesional. Los intervalos adyacentes sí se permiten.
- Cancelar o acortar un bloqueo libera disponibilidad inmediatamente. Ningún
  cambio elimina su historial.

Las citas vigentes y los bloqueos activos comparten una única regla de ocupación
por tenant, profesional e intervalo. Las carreras reserva-reserva,
reserva-bloqueo y bloqueo-bloqueo tendrán un solo resultado válido.

## Contactos de clientes

- Un contacto pertenece a un tenant y exige nombre y al menos un correo o
  teléfono. No constituye una persona, cuenta ni prueba de identidad.
- La reserva pública crea un contacto nuevo junto con la cita, después de
  verificar el correo y reclamar el horario. Una reserva fallida no deja un
  contacto huérfano.
- El canal público no reutiliza ni fusiona contactos por coincidencia de datos.
  El administrador puede seleccionar explícitamente uno existente o crear otro
  durante una reserva interna.
- Correo y teléfono no son únicos. La fusión manual queda fuera del MVP.
- Cada cita copia el nombre y las vías de contacto utilizadas. Editar el contacto
  afecta solo citas nuevas; solo el administrador puede corregir la instantánea
  de una cita y debe quedar auditado.
- Cambiar el correo de una cita revoca el acceso anterior. El nuevo correo debe
  verificarse antes de obtener autoservicio.
- Solo el administrador edita o archiva contactos. Un contacto archivado no
  puede seleccionarse para nuevas citas y nunca se elimina si tiene historial.
- El profesional ve únicamente los datos necesarios de sus citas asignadas; el
  invitado solo la instantánea de su propia cita mediante acceso seguro.

La futura anonimización será una excepción controlada: deberá retirar
identificadores directos de contactos, instantáneas, historiales con datos
personales y envíos pendientes, además de revocar accesos, sin borrar los hechos
operativos mínimos. Su política debe aprobarse antes del piloto.

## Efectos sobre citas existentes

| Cambio | Resultado |
| --- | --- |
| Edición del servicio o contacto | La instantánea de la cita no cambia. |
| Edición del nombre o sede | Los datos nuevos se usan al crear o reprogramar; las demás citas conservan su instantánea. |
| Servicio o profesional vuelto privado | Sale del canal público; las citas y la gestión interna continúan. |
| Servicio archivado, profesional inactivo o asignación inactiva | La cita se conserva y sigue ocupando; no puede reprogramarse hacia una configuración inelegible. |
| Cambio del horario semanal | La cita se conserva y se informa al administrador si queda fuera. |
| Bloqueo que cruza una cita | El bloqueo se rechaza. |
| Página despublicada | Se detienen reservas y reprogramaciones públicas; continúan la consulta, confirmación, cancelación y gestión interna. |
| Organización suspendida | Se bloquea toda operación hasta su restauración. |

Cuando una cita ya no pueda atenderse, el administrador deberá conservarla,
reprogramarla o cancelarla según sus reglas; ningún cambio de configuración la
modificará silenciosamente.

## Consistencia y auditoría

- Cada operación valida tenant, actor, permisos y versión vigente.
- Creaciones y transiciones repetibles usan claves de idempotencia con alcance de
  tenant y actor o canal.
- Entidad, historial y evento de salida se escriben en una transacción; una
  carrera perdida no deja estados parciales.
- Si una configuración cambia antes de reservar, rige la nueva versión. Si la
  reserva termina primero, se conserva y el cambio posterior informa su impacto
  o se rechaza cuando sea un bloqueo incompatible.
- Toda modificación registra actor, canal, instante, valores relevantes y motivo
  cuando corresponda. Los datos personales no se copian a registros genéricos.
- La proyección pública nunca revela estados internos, motivos de bloqueo,
  integrantes ni datos de clientes.

## Escenarios mínimos de prueba

- Activación, suspensión, publicación válida y organización sin configuración.
- Invitación aceptada, repetida, vencida y revocada; cambio inmediato de permisos
  y carrera para retirar al último administrador.
- Matriz de servicios y profesionales privados, publicados, archivados e
  inactivos en los canales público e interno.
- Horarios con descansos, límites adyacentes, conjunto vacío, reemplazo atómico y
  cambios estacionales.
- Bloqueos adyacentes, solapados y concurrentes con una reserva.
- Citas preservadas y alertas de impacto ante cada cambio de configuración.
- Contacto público atómico, reutilización interna explícita, duplicados seguros,
  archivo, corrección de instantánea y aislamiento entre tenants.

## Alta inicial aprobada

Una persona autenticada crea su propia organización en `setup`, queda como su
propietaria y recibe en la misma transacción su membresía activa con
`administrator` y `professional`. Después configura y
activa la organización. El MVP no permite invitar integrantes ni administrar
roles, pero conserva la membresía como base para incorporarlos en el futuro.

