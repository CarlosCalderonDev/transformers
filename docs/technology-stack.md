# Stack tecnológico

Este documento registra la base tecnológica adoptada para el desarrollo de
TRANSFORMERS.

## Tecnologías oficiales

- **Lenguajes principales:** TypeScript para el frontend y Rust para el backend.
- **Frontend:** Next.js, React, TypeScript, HTML y CSS.
- **Backend:** Axum y Tokio.
- **Base de datos:** PostgreSQL y SQLx.
- **Autenticación e integración:** PASETO y JWT.
- **Contenedores:** Docker.
- **Integración y despliegue continuos:** GitHub Actions.

## Uso de tokens

- **PASETO:** se utilizará para los procesos internos de autenticación y
  comunicación de la aplicación.
- **JWT:** se utilizará para la autenticación e integración con sistemas y
  servicios externos.

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

