# ADR 0004: autenticación, membresías y resolución del tenant

**Estado:** aceptada

**Fecha:** 2026-10-22

## Contexto

TRANSFORMERS necesita una identidad global que pueda participar en varias
organizaciones sin confundir autenticación, autorización y aislamiento. Un
tenant seleccionado por el navegador no demuestra que la persona tenga acceso.
Los cambios de membresía y permisos deben surtir efecto sin depender de tokens
antiguos.

El director decidió conservar la autenticación local bajo administración propia
y permitir también el acceso federado con Google y Microsoft.

## Decisión

### Separación de responsabilidades

- **Autenticación:** demuestra qué cuenta inició sesión.
- **Membresía:** relaciona una persona global con una organización y contiene
  estado y roles.
- **Autorización:** decide si esa membresía puede ejecutar una acción concreta.
- **Resolución del tenant:** transforma un identificador público en el tenant
  candidato; nunca concede acceso por sí sola.

### Cuenta e identidades

- Cada persona tendrá como máximo una cuenta global interna.
- Una cuenta podrá usar una credencial local y vincular varias identidades
  externas sin duplicar personas ni membresías.
- La autenticación local será administrada por TRANSFORMERS. Las contraseñas
  nunca se almacenarán; solo se conservarán verificadores Argon2id con sal única.
- La política inicial admitirá contraseñas Unicode normalizadas de 15 a 128
  caracteres, sin reglas arbitrarias de composición ni rotación periódica. Se
  aplicarán lista de contraseñas bloqueadas y limitación de intentos antes de
  habilitar el flujo para usuarios reales.
- Google y Microsoft serán los proveedores externos iniciales mediante OpenID
  Connect Authorization Code con PKCE, `state`, `nonce` y redirecciones exactas.
- Una identidad externa se reconocerá únicamente por `(issuer, subject)`. El
  correo será un atributo verificable y modificable, nunca su identificador.
- Nunca se fusionarán cuentas automáticamente porque compartan correo. Vincular
  o desvincular una identidad exigirá una sesión autenticada, confirmación
  reciente y conservar al menos un método de acceso válido.
- Apple y proveedores empresariales podrán añadirse después mediante el mismo
  contrato, sin cambiar personas, membresías ni tenants.

### Recuperación y verificación

- Verificación de correo, recuperación de contraseña y vinculación usarán tokens
  aleatorios, de un solo uso, con vencimiento y almacenados únicamente como hash.
- Una respuesta de inicio o recuperación no revelará si un correo existe.
- Cambiar contraseña, recuperar una cuenta o detectar compromiso revocará las
  sesiones afectadas y quedará auditado sin registrar secretos.
- MFA y códigos de recuperación son obligatorios antes de elevar el sistema a
  un nivel de seguridad que los requiera; su diseño queda fuera de este primer
  corte técnico y no debe improvisarse dentro del flujo local.

### Sesión de aplicación

- Después del inicio de sesión, `optimus` emitirá un identificador opaco,
  aleatorio y de alta entropía. `arcee` conservará únicamente su hash.
- La sesión viajará en una cookie `__Host-` con `Secure`, `HttpOnly`,
  `SameSite=Lax`, `Path=/` y sin atributo `Domain`.
- Las contraseñas, tokens de sesión y tokens del proveedor no se almacenarán en
  `localStorage` ni se expondrán al código del navegador cuando no sea necesario.
- Las sesiones tendrán expiración por inactividad y expiración absoluta,
  aplicadas por el servidor. Se rotarán después de autenticar, elevar
  privilegios o ejecutar un evento de riesgo.
- Cerrar sesión, deshabilitar una cuenta o detectar compromiso revocará sus
  sesiones. Las operaciones mutables también usarán protección CSRF.

### Resolución y autorización del tenant

Las rutas internas utilizarán un identificador público, inicialmente con una
forma equivalente a `/o/{organization_public_id}`. El flujo será:

1. validar la sesión y obtener la persona global;
2. resolver el identificador público mediante una operación restringida;
3. iniciar una transacción y establecer tenant, actor y canal;
4. comprobar organización, cuenta y membresía vigentes;
5. construir un `AuthorizedTenantContext` inmutable;
6. autorizar la acción y ejecutarla dentro de la misma transacción.

El identificador recibido del cliente es solamente un selector. No se aceptará
un `tenant_id` de una cabecera, cuerpo, consulta o token como autorización. Cada
nueva operación interna revalidará la membresía; los roles no dependerán de un
JWT de larga duración. Una revocación afectará las operaciones posteriores sin
obligar a esperar la expiración de la sesión.

### Canales y privilegios

- **Autenticación:** acceso mínimo y separado a cuentas, credenciales, identidades
  externas, desafíos y sesiones.
- **Interno:** exige sesión, membresía activa y permiso para la acción.
- **Público:** permite únicamente capacidades expresamente publicadas, como
  consultar disponibilidad o reservar mediante credenciales limitadas.
- **Aprovisionamiento:** crea organizaciones e invitaciones iniciales mediante
  una operación interna confiable.
- **Soporte:** usa capacidades excepcionales, mínimas, temporales y auditadas.

Los canales tendrán tipos, servicios y privilegios de base de datos separados.
El canal público no reutilizará acceso interno general. `bumblebee` podrá mostrar
o solicitar un contexto, pero `optimus` será la autoridad que lo valida y
`arcee` aplicará el aislamiento final.

### Membresías e invitaciones

- Una membresía seguirá siendo única por persona y organización.
- Solo una membresía activa, una cuenta habilitada y una organización permitida
  concederán acceso interno.
- Las invitaciones serán independientes de las membresías. Su token será
  aleatorio, almacenado como hash, de un solo uso y con vencimiento.
- Aceptar una invitación exigirá autenticación y verificación del correo
  destinatario. La aceptación será transaccional e idempotente.
- Una invitación nunca cambiará silenciosamente el tenant activo ni concederá
  permisos en otra organización.

### Alta inicial aprobada

Para el MVP, un operador confiable de TRANSFORMERS creará la organización en
estado `setup` y emitirá la invitación del primer administrador. El destinatario
se autenticará, verificará su correo, aceptará la invitación y completará la
configuración antes de activar la organización. El registro empresarial
autoservicio queda fuera del MVP.

## Verificación obligatoria

- Una cuenta local podrá vincular Google o Microsoft y seguirá siendo la misma
  persona con las mismas membresías.
- Coincidir en correo nunca fusionará dos cuentas automáticamente.
- Contraseñas, tokens de sesión y desafíos no aparecerán en texto claro en la
  base de datos, registros ni respuestas.
- Cambiar el identificador público, una cabecera o el cuerpo nunca permitirá
  acceder a otro tenant.
- Una membresía suspendida o terminada perderá acceso en la siguiente operación.
- Una sesión revocada, vencida o rotada no será reutilizable.
- Invitaciones vencidas, revocadas, repetidas o destinadas a otro correo serán
  rechazadas sin crear estados parciales.
- El canal público no podrá invocar operaciones internas ni leer proyecciones
  privadas.
- Las pruebas cubrirán enumeración de cuentas, limitación de intentos, CSRF,
  fijación de sesión, repetición de tokens y carreras entre autorización y
  revocación.

## Consecuencias

- TRANSFORMERS conserva el control de las cuentas y puede operar sin depender de
  un proveedor externo, pero asume la protección de contraseñas, recuperación,
  prevención de abuso y respuesta a incidentes.
- Google y Microsoft reducen fricción sin convertirse en la identidad central.
- Agregar o cambiar proveedores no altera personas ni membresías porque las
  identidades externas se vinculan mediante `(issuer, subject)`.
- La autorización requerirá consultas adicionales, pero permitirá revocación
  inmediata y reglas coherentes por tenant.
- Se necesitarán privilegios diferenciados para autenticación, acceso interno,
  acceso público, aprovisionamiento y soporte.

## Referencias

- [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/rfc9700)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0-18.html)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Multi-Tenant Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html)
