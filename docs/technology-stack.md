# Stack tecnológico

Este documento registra la base tecnológica adoptada para el desarrollo de
TRANSFORMERS.

## Tecnologías oficiales

- **Lenguajes principales:** TypeScript para el frontend y Rust para el backend.
- **Frontend:** Next.js, React, TypeScript, HTML y CSS.
- **Backend:** Axum y Tokio.
- **Base de datos:** PostgreSQL 18 y SQLx. Cada entorno fijará la versión de
  parche y la imagen por digest; su actualización será deliberada y probada.
- **Autenticación e integración:** sesiones opacas, OAuth 2.0 y OpenID Connect;
  JWT únicamente cuando un protocolo o integración externa lo requiera.
- **Contenedores:** Docker.
- **Integración y despliegue continuos:** GitHub Actions.

## Uso de sesiones y tokens

- **Sesión de la aplicación:** identificador opaco, aleatorio y de alta
  entropía; la base conserva únicamente su hash y el servidor revalida cuenta,
  expiración y revocación. Esta es la decisión vigente de
  [ADR 0004](decisions/0004-authentication-membership-tenant-resolution.md).
- **OAuth 2.0 y OpenID Connect:** protocolos para Google, Microsoft y futuras
  identidades federadas aprobadas.
- **JWT:** se aceptará solo en los puntos donde el estándar o un proveedor lo
  requiera y se validará por emisor, audiencia, firma, tiempo y propósito. No
  será la sesión interna ni transportará roles persistentes de la aplicación.
- **PASETO:** no forma parte del fundamento actual. Solo se incorporará mediante
  una decisión posterior si aparece un caso concreto que una sesión opaca u
  otro contrato estándar no resuelvan.

La autenticación, la gestión de sesiones y la autorización se tratarán como
responsabilidades diferentes. Las verificaciones de autorización se realizarán
siempre en el servidor, independientemente de las restricciones visibles en la
interfaz.

## Alcance tecnológico inicial

La primera etapa estará enfocada en aplicaciones web con diseño responsivo,
compatibles con computadores, tabletas y dispositivos móviles mediante
navegadores web.

No se incorporarán tecnologías adicionales de infraestructura, almacenamiento
o mensajería hasta que una necesidad comprobada justifique su adopción.

