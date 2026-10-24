# Evaluación y plan del fundamento arquitectónico

**Estado:** auditoría completada; decisiones estructurales propuestas

**Fecha:** 2026-10-23

## Objetivo

Cerrar la arquitectura práctica de `arcee`, `optimus` y `bumblebee` antes de
continuar con endpoints o interfaz. La meta no es anticipar todas las funciones
futuras, sino establecer límites que permitan crecer sin trasladar reglas entre
capas, debilitar el aislamiento o acoplar el producto a una tecnología.

## Dictamen

La dirección canónica es válida: monolito modular, DDD pragmático, arquitectura
hexagonal ligera, PostgreSQL y una interfaz Next.js. Sin embargo, todavía no se
encuentra materializada como arquitectura de código:

| Repositorio | Fundamento válido | Brecha antes de continuar |
| --- | --- | --- |
| `arcee` | Integridad relacional, RLS, roles separados y pruebas de aislamiento. | Migraciones reales, fronteras por capacidad, privilegios mínimos, evolución y pruebas independientes. |
| `optimus` | Criptografía, sesión opaca y contexto tenant transaccional. | Cargo Workspace, módulos y adaptadores, autorización por permiso, sesión fresca, HTTP y evidencia de integración no omisible. |
| `bumblebee` | Reglas de seguridad documentadas. | Aún no existe aplicación, contrato HTTP, estructura, pruebas ni límite servidor/cliente. |

El código actual es pequeño y recuperable. No se requiere reiniciar el proyecto;
este es el momento de reorganizarlo con bajo costo.

## Hallazgos materiales observados

### `arcee`

- `compose.yaml` crea PostgreSQL, pero no aplica una secuencia oficial de
  migraciones ni comprueba su revisión. Los scripts de prueba actúan de manera
  accidental como instalador del esquema.
- El rol ordinario conserva `SELECT`, `INSERT`, `UPDATE` y `DELETE` sobre
  organizaciones, membresías y roles; el aprovisionamiento reutiliza ese canal
  en vez de una capacidad tipada y mínima.
- `owner_person_id` y el nombre visible fueron agregados como columnas
  opcionales para conservar compatibilidad, pero no existe todavía una
  migración que cierre su obligatoriedad. El identificador público puede
  modificarse o reutilizarse pese a que ADR 0003 lo define estable.
- Las restricciones actuales no aseguran por sí solas que cambiar un correo
  invalide su verificación, que un desafío solo se consuma antes de vencer ni
  que la rotación de sesión mantenga cuenta y sucesor únicos.
- Las suites comparten estado y orden de ejecución. El arnés de caracterización
  ya monta `0001`–`0003` y ejecuta las tres suites actuales, pero todavía no
  existe CI que pruebe instalación y actualización con los roles reales.

### `optimus`

- El backend es todavía un crate y `src/database.rs` mezcla validación,
  onboarding, autenticación, autorización y SQL.
- `AuthorizedTenantTransaction::connection()` entrega un `PgConnection` a sus
  consumidores. Esto permite ejecutar cualquier consulta autorizada por el rol
  sin expresar el permiso ni la capacidad que justificó la transacción.
- Abrir contexto tenant comprueba membresía, pero no un permiso concreto, y
  recibe una `AuthenticatedSession` reutilizable sin revalidar en ese mismo
  límite que siga vigente.
- Identidad, acceso interno y aprovisionamiento reciben el mismo tipo `PgPool`;
  el compilador no puede impedir intercambiar credenciales.
- Los errores públicos conservan variantes de SQLx y la validación del
  identificador de organización no coincide en todos los caminos de Rust y SQL.
- Al iniciar la auditoría, la prueba de integración devolvía éxito cuando
  faltaban las URL de PostgreSQL. Ya falla explícitamente y el arnés comprueba
  por separado la ausencia de cada URL; aún falta ejecutarlo con PostgreSQL real
  y llevar la misma evidencia a CI.

### `bumblebee`

- El repositorio contiene instrucciones y README, pero aún no tiene aplicación,
  lockfile, contrato generado, reglas de importación ni arneses de prueba.
- Esta ausencia no es deuda que deba conservarse: permite adoptar la frontera
  correcta antes de escribir componentes o estado de interfaz.

Estos hallazgos justifican la pausa. No significan que lo implementado deba
desecharse; cada corrección aparece como paquete verificable en el
[tablero del fundamento](architecture-foundation-board.md).

## Bloqueadores de producto descubiertos

### Propietario, administrador y profesional

El MVP necesita un profesional operativo para publicar disponibilidad y atender
citas. A la vez, el alta actual crea una única membresía solo con rol
`administrator` y excluye empleados. Una organización nueva no puede completar
una ruta reservable bajo ambas reglas.

También falta distinguir si **propietario** significa solamente creador
histórico o una responsabilidad permanente diferente de los roles operativos.
Esto afecta transferencias futuras, recuperación, suspensión de cuentas y la
regla del último administrador.

**Recomendación:** conservar un único propietario explícito y transferible solo
mediante una operación excepcional futura. En el MVP, su membresía inicial
recibe simultáneamente `administrator` y `professional`. No se habilitan
empleados ni administración de miembros. La decisión completa está propuesta
en [ADR 0009](decisions/0009-organization-ownership-and-initial-roles.md).

### Cuenta global del cliente

La visión permite que una persona cree una cuenta aunque no tenga una empresa,
pero el alcance original reserva como invitado y excluye historial global de
cliente. Falta decidir si el MVP vincula una cuenta autenticada con contactos y
reservas o si, por ahora, la cuenta solo conserva el perfil global y habilita la
creación de organizaciones.

**Recomendación:** mantener la reserva invitada en el primer MVP y no fusionar
contactos por correo. Una cuenta sin empresa conserva su perfil, pero la
vinculación verificable con contactos e historial global queda como un corte
posterior explícito. Esto protege el alcance sin cerrar la evolución.

## Arquitectura objetivo transversal

```text
persona / navegador
        │
        ▼
un único origen público
├── /*          → bumblebee / Next.js
└── /api/v1/*   → optimus / Axum
                       │
                       ▼
                casos de uso y permisos
                       │
                       ▼
                adaptadores tipados
                       │
                       ▼
                arcee / PostgreSQL
```

- `transformers` define reglas, vocabulario y decisiones compartidas.
- `optimus` es la única autoridad HTTP y ejecuta casos de uso.
- `arcee` protege persistencia, aislamiento, integridad y concurrencia.
- `bumblebee` presenta capacidades; no decide permisos ni replica reglas.
- Los contratos cambian de forma versionada y se prueban en sus fronteras.

## Fundamento propuesto por repositorio

### `optimus`

Cargo Workspace con módulos de capacidad y adaptadores externos:

```text
optimus/
├── Cargo.toml
├── contracts/http/v1/openapi.yaml
├── apps/api/
├── crates/
│   ├── identity/
│   ├── tenancy/
│   ├── organizations/
│   └── adapters/
├── docs/
├── scripts/
└── tests/
```

Reglas de dependencia:

- `identity`, `tenancy` y `organizations` no dependen de Axum, SQLx,
  configuración ni proveedores.
- Cada capacidad expone una fachada pequeña; sus detalles permanecen privados.
- `adapters` implementa puertos concretos de PostgreSQL, criptografía y futuras
  integraciones. No contiene política de negocio.
- `apps/api` compone dependencias y traduce HTTP; no ejecuta SQL.
- Los puertos representan capacidades de casos de uso, no CRUD genérico.
- Ningún módulo entrega una conexión SQL a presentación o dominio.
- Toda operación tenant-scoped exige sesión vigente, selector público y un
  permiso concreto dentro del límite transaccional acordado.
- Los pools de identidad, tenant y aprovisionamiento usan tipos distintos para
  impedir intercambios accidentales.
- No se crean crates `common`, `utils`, buses o módulos futuros vacíos.

### `arcee`

Una sola base PostgreSQL, con esquemas alineados con responsabilidades:

```text
identity    # persona, perfil, cuentas, credenciales, sesiones y desafíos
tenancy     # organizaciones, propiedad, membresías, roles y contexto
scheduling  # catálogo, disponibilidad, contactos y citas
platform    # auditoría, idempotencia, outbox y tareas duraderas
```

Los esquemas se incorporan solo cuando existe su primera tabla. Los roles base
serán `transformers_object_owner`, `transformers_migrator`,
`transformers_identity_runtime`, `transformers_internal_runtime` y
`transformers_provision_runtime`; `transformers_public_runtime` aparecerá al
construir el canal público. Las funciones privilegiadas tendrán propietarios
`NOLOGIN` más estrechos por capacidad.

```text
arcee/
├── migrations/       # secuencia SQLx plana, ordenada e inmutable
├── roles/            # bootstrap por entorno
├── docs/
├── tests/
│   ├── fixtures/
│   ├── schema/
│   ├── security/
│   ├── migrations/
│   └── integration/
├── scripts/
└── compose.yaml
```

- Las migraciones se aplican con SQLx usando un rol migrador separado.
- Cada migración es transaccional por defecto; una excepción deberá ser
  explícita, reanudable y probada. Una vez compartida no se edita.
- Los cambios incompatibles siguen `expand → backfill → contract`.
- La reversión ordinaria se realiza con una migración correctiva hacia delante;
  recuperación de desastre usa respaldo y restauración probados.
- Cada tabla documenta clasificación, PII, propietario lógico, RLS, roles,
  retención, restricciones e índices.
- Las funciones `SECURITY DEFINER` se reservan para capacidades atómicas y
  mínimas, con `search_path` seguro y `EXECUTE` revocado a `PUBLIC`.
- Los roles runtime no reciben DML general cuando basta una capacidad limitada.
- Los roles runtime no tienen DML directo sobre tablas tenant. Cada función de
  persistencia fija su permiso, revalida y bloquea sesión y acceso, configura el
  contexto, ejecuta la operación exacta bajo RLS y solo entonces devuelve. Un
  GUC modificado por el llamador nunca concede acceso.

### `bumblebee`

Next.js App Router, server-first y organizado por funcionalidades:

```text
bumblebee/
├── contracts/
│   ├── optimus.openapi.json
│   └── optimus.openapi.lock.json
├── docs/
├── public/
├── src/
│   ├── app/                 # rutas, layouts y composición
│   ├── features/            # flujos de autenticación, empresas y agenda
│   └── shared/
│       ├── api/
│       ├── config/
│       ├── lib/
│       ├── styles/
│       ├── testing/
│       └── ui/
└── tests/e2e/
```

- `app` puede depender de `features` y `shared`.
- `features` depende de `shared`, nunca de detalles internos de otra feature.
- `shared` no depende de `features` ni de `app`.
- Páginas y layouts son Server Components por defecto; las islas cliente se
  limitan a interacción real.
- La sesión se resuelve en el servidor y nunca usa almacenamiento web.
- Datos autenticados o tenant-scoped nunca entran en caché compartida ni se
  serializan con cookies o tokens hacia componentes cliente.
- `optimus` posee OpenAPI y publica un artefacto web inmutable con versión,
  commit y checksum. `bumblebee` fija su procedencia, genera esquemas y tipos y
  adapta DTO a modelos de presentación mediante clientes pequeños revisables.
- No se incorpora estado global, librería de formularios o caché cliente hasta
  que un flujo real lo justifique.
- CSS Modules y variables CSS forman la base visual; WCAG 2.2 AA es el criterio
  de accesibilidad.

## Contrato web propuesto

- Un mismo origen enruta `/api/v1/*` al listener web de Axum y el resto a
  Next.js. Soporte, migraciones, health detallado y administración excepcional
  no forman parte de esa superficie pública.
- OpenAPI 3.1 es la fuente del contrato de `optimus`.
- Los errores usan `application/problem+json` conforme a RFC 9457, con código
  estable, campos inválidos y `request_id`.
- La cookie de sesión es opaca, `__Host-`, `Secure`, `HttpOnly`,
  `SameSite=Lax` y no contiene roles ni tenant.
- Las mutaciones verifican un `Origin` permitido y un token CSRF ligado a la
  sesión o al contexto anónimo; ausencia, `null` o un origen no confiable se
  rechazan en el canal navegador.
- CORS permanece deshabilitado en producción por defecto.
- Creaciones repetibles usan `Idempotency-Key`; actualizaciones concurrentes
  usan versión o `ETag`/`If-Match` cuando corresponda.
- Timestamps son RFC 3339 en UTC; las zonas horarias de negocio son IANA.
- El selector público nunca sustituye la autorización y no se expone
  `tenant_id` como selector de interfaz.

## Calidad y evidencia obligatorias

### `arcee`

- base limpia y actualización desde la revisión anterior;
- matriz positiva y negativa de roles y privilegios;
- RLS, relaciones cruzadas, carreras e invariantes;
- migraciones atómicas y compatibilidad con la revisión de `optimus`.

### `optimus`

- `cargo fmt`, `cargo clippy -D warnings` y pruebas unitarias;
- integración PostgreSQL obligatoria sin auto-skip contabilizado como éxito;
- pruebas negativas de sesión, permiso, tenant y reutilización del pool;
- contrato OpenAPI validado y respuestas HTTP comprobadas.

### `bumblebee`

- formato, lint, TypeScript estricto y build;
- unitarias con Vitest y Testing Library;
- contrato generado sin diferencias;
- Playwright en Chromium, Firefox y WebKit para flujos completos;
- accesibilidad automática y revisión manual de teclado/foco.

## Puerta para reanudar funcionalidades

- [x] Auditoría transversal de los tres repositorios completada.
- [ ] Semántica de propietario y rol profesional resuelta.
- [ ] Alcance de la cuenta global del cliente resuelto.
- [ ] ADR arquitectónicos aceptados.
- [ ] Caracterización y CI mínimo impiden omitir las pruebas actuales.
- [ ] Estructuras base materializadas sin módulos ficticios.
- [ ] Fronteras de seguridad P0 corregidas.
- [ ] Migraciones y pruebas integradas incapaces de producir falsos verdes.
- [ ] CI mínimo activo en los tres repositorios de código.
- [ ] Primera revisión cruzada de arquitectura superada.

Solo después se reanuda el corte registro → verificación → sesión → creación de
organización.

## Referencias técnicas

- [Cargo Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)
- [Next.js App Router](https://nextjs.org/docs/app)
- [Next.js: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Axum middleware](https://docs.rs/axum/latest/axum/middleware/)
- [OpenAPI Specification](https://spec.openapis.org/oas/)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [PostgreSQL: Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL: Schemas](https://www.postgresql.org/docs/current/ddl-schemas.html)
