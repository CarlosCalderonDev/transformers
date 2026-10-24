# ADR 0010: alcance de la cuenta personal sin organización

**Estado:** aceptada

**Fecha:** 2026-10-23

**Aceptación:** 2026-10-24 por el director (opción A). El MVP mantiene la
cuenta sin beneficios de reserva; se fija desde ahora la regla de vinculación
futura descrita en «Evolución y ciclo de vida».

## Contexto

TRANSFORMERS permite registrar una identidad global antes de crear una
organización. Esa separación es necesaria para que una persona conserve una
sola identidad aunque después participe en varias empresas o no cree ninguna.

El MVP también permite reservar como cliente invitado y excluye una cuenta de
cliente con historial unificado. Sin una regla explícita, una implementación
podría unir por correo la cuenta global con contactos privados de distintas
organizaciones, conceder acceso por una coincidencia o convertir el perfil en
un depósito prematuro de datos de cliente.

Esta decisión no define la propiedad ni los roles de una organización; esos
asuntos pertenecen a ADR 0009.

## Decisión

### Cuenta global mínima

- `Person`, `PersonProfile` y `Account` podrán existir válidamente con cero
  membresías y cero organizaciones. «Cuenta sin organización» será un estado
  derivado, no otro tipo de cuenta ni un rol `customer`.
- En esta decisión, «sin organización» significa que no existe ninguna relación
  organizacional para la cuenta. Una membresía suspendida o terminada y una
  organización suspendida son estados diferentes, quedan fuera de este alcance
  y no podrán presentarse como si nunca hubiera existido relación.
- La cuenta permitirá registrarse y verificar sus métodos de acceso, iniciar y
  cerrar sesión, recuperar su credencial local, consultar y corregir el perfil
  mínimo, obtener la lista de organizaciones autorizadas —que podrá estar
  vacía— e iniciar el alta de una organización conforme a ADR 0005. La
  recuperación de una identidad externa pertenece a su proveedor; su
  vinculación y desvinculación siguen ADR 0004.
- El perfil global contendrá únicamente nombre y apellido durante el MVP. El
  correo y su verificación pertenecerán al método de autenticación. No se
  agregarán teléfono, dirección, preferencias, pagos ni atributos de marketing
  sin un caso de uso aprobado.
- La cuenta no tendrá tenant predeterminado. Conocer un identificador público o
  mantener una sesión global no concederá acceso a una organización; cada
  operación interna seguirá exigiendo membresía, estado y permiso vigentes.
- Crear una organización agregará las relaciones aprobadas para ella sin
  duplicar ni convertir `Person`, `PersonProfile` o `Account` en datos del
  tenant.

### Separación del cliente invitado

- Estar autenticado no convertirá a la persona en un cliente identificado. Una
  reserva pública seguirá el mismo flujo invitado aunque el navegador mantenga
  una sesión global.
- La reserva seguirá exigiendo la verificación temporal ligada al correo, la
  organización y el propósito. Creará un contacto nuevo y tenant-scoped, como
  exige el flujo público aprobado, y emitirá una capacidad independiente para
  gestionar esa cita. La sesión global no sustituirá esa capacidad.
- Una coincidencia de correo o teléfono nunca reclamará, combinará ni revelará
  contactos o citas. Tampoco permitirá omitir la verificación de la reserva.
- `Customer` continuará siendo un rol contextual y `CustomerContact` un dato
  privado de la organización. Contactos, citas e instantáneas no requerirán
  `person_id` ni `account_id` durante el MVP.
- Modificar el perfil global no cambiará contactos ni instantáneas. Modificar un
  contacto tampoco cambiará el perfil global ni las citas anteriores.

### Evolución y ciclo de vida

- No se creará anticipadamente una relación entre `CustomerContact` y `Person`.
  Una futura vinculación requerirá otra decisión que defina prueba de control,
  consentimiento cuando corresponda, revocación, duplicados, visibilidad e
  historial por tenant.
- La única prueba de control admitida para esa vinculación será una reserva
  realizada por la propia persona con su sesión iniciada. Solo esa cita, y el
  contacto creado por ella, podrán quedar asociados a su `Person`. Nunca se
  vincularán citas o contactos existentes por coincidencia de correo o
  teléfono, ni reservas creadas por el personal en su nombre. Con esa base, un
  corte posterior podrá autocompletar datos, omitir el enlace de verificación
  y ofrecer «mis citas» sin reinterpretar datos históricos.
- Esa futura relación se apoyará en `Person`, que representa la identidad
  estable, no en `Account`, que representa el acceso. Vincular un contacto no
  concederá membresía ni acceso interno, y desvincularlo no borrará contactos,
  citas ni sus instantáneas.
- Deshabilitar una cuenta y revocar todas sus sesiones y desafíos pendientes
  capaces de modificar o recuperar el acceso será una sola operación atómica.
  Mientras siga deshabilitada se rechazarán la autenticación y las mutaciones
  de métodos de acceso; reactivarla no devolverá validez a sesiones ni desafíos
  anteriores. `Person` y `PersonProfile` se conservarán y no habrá cascadas
  desde identidad o acceso hacia contactos, citas o hechos operativos.
- Las capacidades independientes de una cita invitada no se revocarán por la
  mera coincidencia de su correo con el de una cuenta deshabilitada. Su propio
  ciclo de vida seguirá siendo tenant-scoped.
- Cierre, exportación, retención y anonimización necesitan una política
  específica antes de usar datos reales. Hasta aprobarla no se inferirán plazos
  ni se prometerá eliminación total; se mantendrán separados el ciclo de vida
  de la identidad global y el de los registros operativos de cada tenant.

## Visibilidad mínima

| Superficie | Puede exponer | No puede exponer ni inferir |
| --- | --- | --- |
| Cuenta propia | Perfil permitido, estado de sus métodos de acceso y selectores de organizaciones autorizadas | Contactos, citas o historial obtenidos por coincidencia |
| Personal de una organización | Contactos e instantáneas de su tenant según permisos; una proyección mínima de integrante solo mediante una membresía autorizada independiente | Inferir una cuenta o perfil desde un contacto o coincidencia; métodos de acceso y relaciones con otros tenants |
| Canal invitado | Datos públicos y la cita autorizada por su capacidad segura | Perfil global, membresías y canal interno |
| Cuenta u organización sin relación autorizada | Nada de la cuenta personal | Existencia, correo, estado o relaciones de la cuenta |

Las respuestas usarán proyecciones permitidas explícitamente. Nunca expondrán
hashes, sujetos OIDC, identificadores de sesión, desafíos ni datos internos de
auditoría como parte del perfil.

## Alternativas descartadas

- **Exigir una organización durante el registro:** acopla identidad y tenant,
  contradice el alta opcional y obliga a rediseñar la cuenta para otros usos.
- **Vincular contactos automáticamente por correo o teléfono:** esos datos no
  prueban identidad; la opción permitiría correlación y exposición entre
  organizaciones.
- **Incluir ahora una cuenta de cliente con historial:** es una capacidad de
  producto completa, expresamente fuera del MVP, que necesita nuevos flujos de
  vinculación, privacidad, autorización y revocación.
- **Agregar `account_id` opcional a contactos o citas:** acopla hechos del tenant
  al método de acceso, crea cascadas peligrosas y no resuelve cómo se demuestra
  la relación con una persona.

## Consecuencias

- Una persona puede adoptar TRANSFORMERS gradualmente: primero conserva su
  identidad y perfil; después puede crear o recibir acceso a organizaciones sin
  migrar la cuenta.
- La cuenta por sí sola no mejora todavía la experiencia de reserva ni presenta
  un historial personal. Esa limitación es visible y deliberada.
- La plataforma evita construir una identidad comercial global a partir de
  datos privados de cada empresa y mantiene abierta una vinculación futura
  explícita.
- `person_profiles`, actualmente introducida por la migración provisional de
  onboarding, pertenece conceptualmente a identidad. La adopción del historial
  de migraciones deberá corregir esa responsabilidad sin editar una migración
  que ya sea inmutable.

## Verificación obligatoria

- Registrar persona, perfil, cuenta y método de acceso no crea organización,
  membresía, contacto ni cita; una cuenta con cero organizaciones puede iniciar
  sesión y gestionar exclusivamente su superficie global.
- La lista de organizaciones de esa cuenta es vacía y cualquier operación
  tenant-scoped se rechaza aunque la persona conozca un selector válido. La
  superficie global muestra el perfil y la acción para crear una organización,
  no un error ni una redirección automática hacia un tenant. Los estados
  suspendidos o terminados no se disfrazan como este estado vacío.
- En la superficie HTTP, un selector interno existente pero no autorizado y uno
  inexistente son indistinguibles en estado, código estable de problema, cuerpo
  y efectos observables. La aplicación puede conservar causas internas
  distintas sin exponerlas al solicitante.
- Crear posteriormente una organización reutiliza la misma persona y cuenta, y
  no altera contactos ni reservas preexistentes.
- Correos o teléfonos coincidentes entre una cuenta y contactos de uno o varios
  tenants no crean relaciones, no cambian filas y no revelan su existencia.
- Reservar con una sesión global activa continúa exigiendo el flujo invitado y
  no agrega `person_id` ni `account_id` a contacto, cita o instantánea.
- La capacidad de una cita invitada no permite consultar el perfil, las
  membresías o el canal interno; la sesión global tampoco permite gestionar esa
  cita sin su capacidad independiente.
- Actualizar perfil, contacto o instantánea demuestra independencia en ambas
  direcciones.
- Deshabilitar la cuenta, sus sesiones y desafíos se confirma como una operación
  atómica; reactivarla no revive credenciales temporales anteriores y la cuenta
  deshabilitada no puede mutar métodos de acceso. Persona y perfil se conservan,
  mientras contactos, citas, hechos históricos y capacidades de invitado por
  coincidencia de correo no se borran ni modifican.
- Las respuestas de registro, recuperación y consulta no permiten enumerar
  cuentas ni serializan campos internos de autenticación.
