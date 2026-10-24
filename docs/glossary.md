# Glosario del dominio

Estos términos son canónicos para la documentación. Los identificadores del
código utilizarán sus equivalentes en inglés.

| Término | Identificador | Definición |
| --- | --- | --- |
| Persona | `Person` | Identidad global de una persona natural reconocida por la plataforma. No pertenece a una organización. |
| Perfil personal | `PersonProfile` | Nombre y apellido globales de una persona. El correo se conserva y verifica por su método de autenticación. |
| Cuenta de acceso | `Account` | Medio global mediante el que una persona se autentica. Deshabilitarla no elimina a la persona ni su historial. |
| Organización | `Organization` | Empresa que configura y opera sus servicios, personal, horarios y citas. Es propietaria de sus datos operativos. |
| Tenant | `tenant_id` | Frontera técnica de aislamiento. En el MVP corresponde a una organización, pero no se autentica ni actúa. |
| Membresía | `Membership` | Relación entre una persona y una organización que contiene estado y roles. Solo una membresía activa concede acceso. |
| Administrador | `Administrator` | Rol de membresía que configura la organización y gestiona todas sus citas. |
| Profesional | `Professional` | Rol de membresía que presta servicios, tiene disponibilidad y atiende citas asignadas. |
| Contacto de cliente | `CustomerContact` | Registro privado de una organización con los datos mínimos de quien recibe un servicio. Puede existir sin persona ni cuenta global. |
| Contacto invitado | — | Condición de un contacto de cliente que todavía no está vinculado con una persona global; no es otra entidad. |
| Cliente | `Customer` | Rol contextual de quien recibe un servicio. En el MVP se representa mediante un contacto de cliente. |
| Servicio | `Service` | Prestación ofrecida por una organización, con nombre, duración y precio informativo opcional en la moneda de la organización. |
| Disponibilidad | `AvailabilityWindow` | Intervalo recurrente en el que un profesional podría atender; no garantiza que el horario siga libre. |
| Bloqueo | `TimeBlock` | Excepción fechada que vuelve indisponible a un profesional durante un intervalo. |
| Horario libre | `AvailableSlot` | Intervalo calculado que satisface disponibilidad, bloqueos, citas vigentes y duración del servicio. No se almacena ni se garantiza hasta reservar. |
| Cita | `Appointment` | Acuerdo para que un profesional preste un servicio a un cliente en un intervalo determinado. |
| Reserva | `Booking` | Operación que valida la disponibilidad y crea una cita, ocupando el intervalo de forma atómica. |
| Confirmación de asistencia | `AttendanceConfirmation` | Manifestación posterior del cliente de que pretende asistir. No crea la cita ni libera el horario si falta. |
| Correo verificado | `EmailVerification` | Comprobación temporal de acceso a una dirección para una organización y flujo concretos. No demuestra la identidad de la persona. |

## Reglas conceptuales

- “Organización” es el término del modelo; “empresa” puede usarse en lenguaje
  común.
- Administrador y profesional son roles, no tipos diferentes de persona. Una
  membresía puede poseer ambos.
- Cliente es un rol dentro de una cita, no una identidad global.
- Correo y teléfono son datos de contacto, no pruebas suficientes de identidad.
- Reservar como invitado no crea una persona, una cuenta ni una membresía.
- Los contactos de organizaciones distintas nunca se fusionarán automáticamente
  por compartir correo o teléfono.
- Una futura vinculación entre contacto y persona será explícita, verificable y
  no compartirá automáticamente información entre organizaciones.

