# ADR 0009: propiedad de la organización y roles iniciales

**Estado:** propuesta

**Fecha:** 2026-10-23

## Contexto

ADR 0005 crea una organización con un propietario y una única membresía que
solo recibe `administrator`. Sin embargo, el MVP excluye empleados y exige al
menos un `professional` para ofrecer servicios, definir disponibilidad y
recibir citas. Una empresa recién creada no puede completar el flujo reservable
con ambas reglas.

También falta definir si **propietario** es solo el creador histórico, un rol
operativo o una responsabilidad permanente. Dejarlo ambiguo afectaría salida de
miembros, suspensión de cuentas, recuperación y una futura transferencia.

## Decisión propuesta

- Cada organización tendrá exactamente un propietario, representado por
  `owner_person_id`, que referencia a una persona global. Una persona podrá ser
  propietaria de varias organizaciones independientes.
- La propiedad será una responsabilidad de continuidad distinta de los roles
  operativos. No concederá por sí sola acceso general a datos ni sustituirá la
  membresía, el permiso concreto o la revalidación de sesión.
- La persona que crea la organización será su propietaria inicial. La misma
  transacción creará exactamente una membresía `active` y le asignará
  simultáneamente `administrator` y `professional`.
- Este corte asume un negocio operado por su propietario: la persona propietaria
  también prestará los servicios. Un dueño que solo administra y necesita otra
  persona para atender queda fuera del MVP hasta incorporar empleados.
- Durante el MVP no existirán invitaciones, empleados, administración de
  miembros ni transferencia de propiedad. El propietario no podrá salir,
  eliminar su membresía, perder `administrator` o perder `professional` por un
  flujo ordinario.
- Si la única cuenta administradora se deshabilita por seguridad, la misma
  transacción revocará sus sesiones y suspenderá todas las organizaciones donde
  sea la única administradora efectiva. La suspensión bloqueará accesos,
  disponibilidad y reservas nuevas, pero conservará citas y propiedad. No se
  eliminará ni reasignará silenciosamente `owner_person_id`.
- La recuperación o transferencia por indisponibilidad permanente será una
  operación futura excepcional, atómica, con autenticación reforzada y
  auditoría. Una futura transferencia deberá elegir una persona con cuenta
  habilitada, correo requerido verificado y membresía `active`, conservar
  siempre exactamente un propietario y no revivir automáticamente el acceso
  del propietario anterior. Sus permisos posteriores se decidirán
  explícitamente en ese futuro alcance.

### Autoridad operativa mínima

| Concepto | Efecto durante el MVP |
| --- | --- |
| Propiedad | Mantiene un responsable único y protege continuidad. No concede acceso operativo por sí sola. |
| `administrator` | Configura y activa la organización; administra servicio, perfil profesional, asignación, horario, bloqueos, contactos y todas las citas incluidas en el MVP. |
| `professional` | Permite representar a la membresía como profesional y consultar/operar sus citas asignadas: iniciar, finalizar y registrar inasistencia conforme al proceso aprobado. |

El rol `professional` es permiso y requisito de elegibilidad, pero no vuelve a
la persona reservable por sí solo. También se requieren cuenta, organización y
membresía activas, perfil, asignación, horario y estados de publicación que
correspondan. Solo `administrator` configura o publica esos elementos. En el
MVP la misma persona puede hacerlo porque posee ambos roles.

La propiedad no se modelará como un tercer rol. Así se evita que quitar un rol
operativo deje a la empresa sin responsable o que marcar a alguien como
propietario le conceda acceso implícito fuera de la autorización ordinaria.

## Cambios sobre ADR 0005

Si se acepta esta propuesta, sustituirá únicamente estas partes de ADR 0005:

- donde dice que el alta crea solo el rol `administrator`, creará ambos roles;
- donde protege al propietario frente a perder el rol administrador, protegerá
  también el rol profesional durante el alcance sin empleados;
- `owner_person_id` dejará de ser ambiguo y representará responsabilidad vigente,
  no solo autoría histórica.

La creación autoservicio, atomicidad, cuenta global, múltiples organizaciones y
aislamiento definidos en ADR 0005 se mantendrán.

## Consecuencias

- El propietario puede operar por sí solo una agenda mínima sin introducir un
  modelo provisional de empleados.
- Agregar empleados después no cambia identidad, propiedad ni aislamiento: se
  crean nuevas membresías y roles sin convertirlos en copropietarios.
- La transferencia futura requerirá un caso de uso específico; no será una
  actualización directa de columna ni una consecuencia de cambiar roles.
- Suspender al único propietario puede dejar temporalmente la empresa sin acceso
  operativo y sin reservas nuevas. Es preferible a ignorar una medida de
  seguridad; la recuperación excepcional deberá resolverlo sin debilitar la
  suspensión y las citas conservadas deberán atenderse, reprogramarse o
  cancelarse durante esa recuperación.

## Verificación obligatoria

- El alta crea toda la organización o nada: propietario no nulo, una membresía
  inicial activa y exactamente los roles `administrator` y `professional`.
- La persona propietaria coincide con la persona de esa membresía y otra cuenta
  no puede reclamarla ni actuar mediante ella.
- Ningún flujo ordinario puede borrar la membresía inicial ni retirar cualquiera
  de sus dos roles durante el MVP.
- La persona, actuando con `administrator`, puede configurar su propio perfil,
  asignación y horario; actuando con `professional`, puede operar sus citas. La
  organización alcanza el estado reservable sin crear otra persona.
- Deshabilitar la única cuenta administradora revoca sus sesiones y suspende
  atómicamente todas las organizaciones afectadas; no admite acceso ni reservas
  nuevas aunque la relación de propiedad permanezca.
- Las pruebas concurrentes impiden cero propietarios, más de uno o una
  transferencia parcial cuando ese caso de uso sea incorporado.
