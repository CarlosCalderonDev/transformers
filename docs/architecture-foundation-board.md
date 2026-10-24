# Tablero de cierre del fundamento arquitectónico

**Estado:** activo; construcción funcional pausada

**Fecha de corte:** 2026-10-23

## Objetivo y alcance

Este tablero coordina `transformers`, `arcee`, `optimus` y `bumblebee` hasta
convertir la arquitectura propuesta en límites de código y evidencia
reproducible. Es la única fuente del avance transversal; los repositorios de
código conservarán sus diseños y pruebas, pero no copias de las prioridades.

Durante la pausa solo se acepta trabajo de caracterización, arquitectura,
seguridad, migraciones, contratos, pruebas e infraestructura mínima. No se
agregan funcionalidades de agenda ni endpoints reales de negocio.

## Significado de los estados

- **verificado:** resultado implementado y evidencia revisada;
- **en curso:** tiene cambios iniciados, pero aún falta evidencia obligatoria;
- **listo:** no tiene una dependencia pendiente y puede comenzar;
- **pendiente:** espera trabajo técnico anterior;
- **decisión:** necesita una elección macro del director;
- **bloqueado:** no debe comenzar hasta aceptar la decisión indicada.

Una tarea no pasa a **verificado** solo porque existan archivos o compile. Debe
cumplir su criterio, enlazar la evidencia exacta y no poder omitir una prueba
obligatoria contabilizándola como exitosa.

## Ruta crítica

```text
                         ┌─ FND-002 propiedad y roles ────────┐
FND-001 auditoría ───────┼─ FND-003 cuenta personal           │
                         ├─ FND-004 arquitectura modular       ├─ Puertas 1 y 2
XCHAR-001 evidencia ─────┼─ FND-005 frontera web              │
                         └─ FND-006 datos y migraciones ───────┘
                                                                  │
                                     contratos + pruebas no omitibles
                                                                  │
                                         revisión transversal de salida
                                                                  │
                              registro → verificación → sesión → organización
```

Las decisiones se presentan al director de una en una. `XCHAR-001` puede
avanzar en paralelo porque solo hace obligatoria y visible la evidencia del
comportamiento existente.

## Puerta 0 — decisiones y dirección

| ID | Estado | Responsable | Resultado y criterio de aceptación | Evidencia |
| --- | --- | --- | --- | --- |
| `FND-001` | verificado | transversal | Auditar estructura, dependencias, seguridad, datos y pruebas de los tres repositorios de código. | [Evaluación arquitectónica](architecture-assessment.md) y hallazgos trazados en este tablero. |
| `FND-002` | decisión | `transformers` | Distinguir propietario, administrador y profesional; definir transferencia excepcional, suspensión, recuperación y protección del último responsable. Debe existir una ruta válida para que la empresa del MVP tenga profesional. | [ADR 0009 propuesto](decisions/0009-organization-ownership-and-initial-roles.md), escenarios y matriz actor–acción–permiso. |
| `FND-003` | decisión | `transformers` | Definir si la cuenta sin empresa solo conserva perfil y habilita crear organizaciones o si se vincula a contactos e historial durante el MVP. | [ADR 0010 propuesto](decisions/0010-personal-account-without-organization.md), escenarios incluidos/excluidos y documentos de MVP y glosario coherentes. |
| `FND-004` | verificado | `transformers` | Aceptar, ajustar o rechazar la estructura modular y su grafo de dependencias. | [ADR 0006 aceptado](decisions/0006-modular-codebase-architecture.md) el 2026-10-24: monolito modular y cuatro repositorios separados. |
| `FND-005` | verificado | `transformers` | Aceptar, ajustar o rechazar origen único, superficies HTTP y distribución del contrato. | [ADR 0007 aceptado](decisions/0007-web-api-boundary.md) el 2026-10-24: origen único y `optimus` como única autoridad HTTP. |
| `FND-006` | verificado | `transformers` | Aceptar, ajustar o rechazar adopción de migraciones, compatibilidad, roles y protocolo transaccional de autorización. | [ADR 0008 aceptado](decisions/0008-database-evolution-and-access-boundaries.md) el 2026-10-24: base única con RLS y acceso runtime solo por funciones estrechas. |

## Puerta 1 — evidencia primero y fundamento ejecutable

| ID | Estado | Repositorio | Depende de | Criterio de aceptación | Evidencia mínima |
| --- | --- | --- | --- | --- | --- |
| `XCHAR-001` | en curso | `arcee` + `optimus` | — | Caracterizar antes de mover código: una ejecución limpia aplica `0001`–`0003`, corre todas las suites SQL y la integración Rust; la ausencia de PostgreSQL o credenciales falla, nunca omite. Se instala CI mínimo para esos comandos. | Unitarias, formato y Clippy pasan. Ambas URL ausentes fallan por su causa exacta; el arnés incluye las tres suites, preflight, exclusión por puerto, limpieza inicial y verificación de limpieza final. Faltan ejecución PostgreSQL completa y workflow porque Docker Desktop/WSL devuelve `Input/output error`. |
| `ARC-ADOPT-001` | bloqueado | `arcee` | `FND-006` | Inventariar bases, clasificar cuáles son descartables y adoptar el historial SQLx sin modificar ni marcar a ciegas una instancia no descartable. Resolver expresamente `0001`, `0002` y el estado provisional de `0003`. | Inventario sin secretos, huella del esquema y ensayo del procedimiento elegido sobre una copia. |
| `ARC-MIG-001` | pendiente | `arcee` | `XCHAR-001`, `ARC-ADOPT-001` | SQLx es el ciclo oficial; separa bootstrap, fija PostgreSQL 18 por parche y digest y aplica migraciones transaccionales o excepciones reanudables. | Base limpia, no-op repetido, fallo parcial y actualización desde un artefacto anterior con fixtures. |
| `ARC-TEST-001` | pendiente | `arcee` | `ARC-MIG-001` | Suites independientes y repetibles en Linux y PowerShell ofrecen arneses para esquema, privilegios, RLS, migraciones, carreras e invariantes. | Dos ejecuciones limpias consecutivas y reporte individual de cada suite. |
| `OPT-ARCH-001` | bloqueado | `optimus` | `FND-004`, `XCHAR-001` | Convertir a Cargo Workspace conservando comportamiento, sin crates vacíos ni cajones genéricos, y hacer comprobable el grafo permitido. | `cargo metadata`; inspección automática de dependencias; `fmt --all --check`; `clippy --workspace --all-targets --all-features -- -D warnings`; `test --workspace --all-targets`. |
| `OPT-MODEL-001` | pendiente | `optimus` | `OPT-ARCH-001` | IDs, selector público, permisos y errores son tipos del dominio; la aplicación no filtra SQLx ni acepta valores que PostgreSQL rechazará después. | Pruebas de límites, Unicode, IDs no intercambiables y mapeo estable de errores. |
| `API-CONVENTIONS-001` | bloqueado | `optimus` | `FND-005` | Crear OpenAPI 3.1 base sin endpoints ficticios y definir versionado, problemas, fechas, caché, idempotencia y concurrencia por operación. | Especificación validada, política de cambios y fixtures de contrato comunes. |
| `OPT-RUNTIME-001` | pendiente | `optimus` | `OPT-ARCH-001`, `API-CONVENTIONS-001` | Shell Axum sin negocio ficticio: configuración validada, secretos redactados, request ID, trazas seguras, límites, health/readiness privados y apagado ordenado. | Pruebas del router/configuración y arranque contra roles y contrato de esquema correctos. |
| `BMB-ARCH-001` | bloqueado | `bumblebee` | `FND-004`, `FND-005` | Diseño local define App Router server-first, `app/features/shared`, importaciones, caché, estado, formularios, estilos, seguridad de respuesta y accesibilidad. | ADR local y reglas de dependencias, serialización y caché convertibles en pruebas. |
| `BMB-BASE-001` | pendiente | `bumblebee` | `BMB-ARCH-001` | Aplicación mínima sin flujos ficticios, con TypeScript estricto, entorno validado, CSS base, `lang="es"`, foco/teclado/semántica, encabezados seguros y arneses unitario/E2E. | Instalación reproducible; formato, lint, typecheck, build, prueba de bundle sin secretos y CI propio verde. |

## Puerta 2 — límites de seguridad P0

| ID | Estado | Repositorio | Depende de | Criterio de aceptación | Evidencia mínima |
| --- | --- | --- | --- | --- | --- |
| `ARC-CAT-001` | bloqueado | `arcee` | `FND-002`, `FND-006`, `ARC-MIG-001` | Un manifiesto clasifica cada tabla como global, raíz tenant, propiedad tenant, relación global–tenant o plataforma, con PII, propietario, retención, RLS, restricciones, índices y roles. Retención pendiente bloquea datos reales, no la clasificación. | Documento generado y prueba que falla ante una tabla nueva no clasificada o tenant sin protección. |
| `ARC-SEC-001` | pendiente | `arcee` | `ARC-TEST-001` | Roles prefijados, propietarios de función estrechos y privilegios predeterminados aplican mínimo acceso; ningún runtime puede DDL, `SET ROLE` privilegiado ni DML directo sobre tablas tenant. | Matriz positiva/negativa de base, esquema, tabla, secuencia, función, membresía y contexto con roles LOGIN reales. |
| `ARC-AUTH-DB-001` | pendiente | `arcee` | `ARC-SEC-001` | Cambiar correo invalida verificación; desafíos validan propósito, cuenta, correo, expiración y consumo atómico; rotación conserva cuenta y sucesor único; eventos de seguridad revocan las sesiones definidas. | Pruebas SQL de repetición, expiración, carreras, cambio de correo, rotación, revocación y fallo parcial. |
| `ARC-TEN-DB-001` | bloqueado | `arcee` | `FND-002`, `ARC-CAT-001`, `ARC-SEC-001`, `ARC-AUTH-DB-001` | Tenant público inmutable y no reutilizable; nombre y propietario obligatorios; exactamente una membresía inicial con roles aprobados; último responsable protegido; cada función mínima fija su permiso, autoriza y ejecuta antes de devolver control. | Pruebas positivas, negativas y concurrentes, incluida linealización operación–revocación e intentos de falsificar todos los parámetros de contexto. |
| `OPT-AUTH-001` | pendiente | `optimus` | `OPT-MODEL-001`, `ARC-AUTH-DB-001` | Adaptadores de identidad usan pool tipado, consumen sesión una sola vez por límite y no entregan conexión ni errores SQLx. | Unitarias e integración real de sesión vencida, revocada, rotada y perteneciente a otra cuenta. |
| `OPT-SEC-001` | pendiente | `optimus` + `arcee` | `OPT-MODEL-001`, `OPT-AUTH-001`, `ARC-TEN-DB-001` | Pools tipados verifican rol/capacidad; token, selector, permiso fijo y operación comparten función y transacción; los adaptadores solo exponen métodos tipados y ninguna API entrega `PgConnection`. | Fallo con URL/rol intercambiado o excesivo; pruebas de tenant cruzado, contexto falsificado, pool reutilizado y carreras de revocación. |
| `OPT-AUTHZ-001` | bloqueado | `optimus` | `FND-002`, `OPT-SEC-001` | Cada caso tenant-scoped exige un permiso concreto y una membresía activa no basta por sí sola; capacidades de UI son proyecciones, no autorización. | Matriz automatizada actor–acción–permiso y pruebas de revalidación servidor. |
| `OPT-WEBSEC-001` | pendiente | `optimus` | `API-CONVENTIONS-001`, `OPT-RUNTIME-001`, `OPT-AUTH-001`, `OPT-SEC-001` | Implementar protocolo y middleware de cookie/CSRF/Origin con rutas de arnés, no endpoints funcionales: contexto anónimo, rotación, logout simulado, múltiples pestañas, OIDC simulado y HTTPS. | Integración HTTP negativa/positiva con PostgreSQL real y verificación de que secretos no llegan a trazas. |
| `XCUT-DESIGN-001` | bloqueado | transversal | `FND-005`, `FND-006` | Definir contratos mínimos y momento de creación para auditoría, idempotencia y entrega duradera. No crear tablas, bus u outbox hasta su primer uso. | Invariantes, clasificación, reintento, retención pendiente y pruebas exigidas al primer corte documentados. |

## Puerta 3 — contratos y evidencia no omitible

| ID | Estado | Repositorio | Depende de | Criterio de aceptación | Evidencia mínima |
| --- | --- | --- | --- | --- | --- |
| `XDB-CONTRACT-001` | pendiente | `arcee` + `optimus` | `ARC-MIG-001`, `OPT-RUNTIME-001`, `OPT-SEC-001` | Capacidad restringida expone versión actual y piso; readiness rechaza esquema antiguo y futuro incompatible; despliegue prueba convivencia `expand/backfill/contract` sin leer `_sqlx_migrations`. | Pipeline conjunto contra artefacto/tag exacto y matriz antiguo–compatible–futuro. |
| `API-ARTIFACT-001` | pendiente | `optimus` + `bumblebee` | `API-CONVENTIONS-001`, `BMB-BASE-001` | CI publica OpenAPI web inmutable con versión, commit y checksum; Bumblebee fija procedencia, genera esquemas/tipos y detecta deriva sin checkout hermano. | Regeneración sin diferencias, generador fijado y prueba deliberada de cambio incompatible. |
| `HTTP-CONFORMANCE-001` | pendiente | `optimus` + `bumblebee` | `OPT-WEBSEC-001`, `API-ARTIFACT-001` | El arnés comprueba proxy confiable, superficies privadas, cookies, CSRF, Problem Details, caché, encabezados y ausencia de secretos con el mismo origen. | Pruebas HTTP y Playwright sobre HTTPS en Chromium, Firefox y WebKit. |
| `QUALITY-GATE-001` | pendiente | los tres repositorios | `XDB-CONTRACT-001`, `API-ARTIFACT-001`, `HTTP-CONFORMANCE-001` | Auditar que cada pipeline requerido sea fail-fast, use versiones fijadas, publique sus suites y no permita `continue-on-error`, skips implícitos ni artefactos desactualizados. | Ejecución verde completa y mutaciones deliberadas que producen rojo en cada frontera crítica. |

## Puerta 4 — revisión y descongelación

| ID | Estado | Responsable | Depende de | Criterio de aceptación | Evidencia mínima |
| --- | --- | --- | --- | --- | --- |
| `XREV-001` | pendiente | transversal | puertas 0–3 | Revisar grafo, seguridad, migraciones, contratos y coherencia documental; no quedan hallazgos críticos abiertos. | Lista de revisión enlazada y diferencias de los tres repositorios inspeccionadas. |
| `FND-EXIT-001` | pendiente | director + agente | `XREV-001` | El director acepta el fundamento verificado y se levanta la pausa funcional. | ADR aceptados, CI verde y registro explícito de aceptación. |

## Trabajo expresamente diferido

Hasta superar `FND-EXIT-001` no se implementan:

- endpoints funcionales de registro, sesión u organizaciones;
- invitaciones, empleados, nuevos roles o transferencia de propiedad;
- tablas y casos de uso de agenda;
- pantallas funcionales o un sistema de diseño amplio;
- tablas de auditoría, idempotencia u outbox antes del efecto real que las use.

El primer corte posterior será
`registro → verificación → sesión → creación de organización`. Antes de exponerlo
deberá materializar los contratos de `XCUT-DESIGN-001`, lista de contraseñas
comprometidas, límites de intentos y cuerpo/tiempo, resistencia a enumeración y
fijación de sesión, y ausencia de contraseñas, cookies y tokens en trazas.

La política de privacidad y la prueba de respaldo/restauración siguen siendo
obligatorias antes de usar datos reales o iniciar el piloto, aunque no bloquean
esta puerta arquitectónica.

## Política de actualización

Al cerrar un paquete se enlazará aquí su evidencia exacta. Si una revisión
descubre una dependencia o riesgo nuevo, se agrega antes de continuar. Los
detalles viven en el repositorio responsable; este archivo conserva estado,
dependencias, criterio de aceptación y evidencia transversal.
