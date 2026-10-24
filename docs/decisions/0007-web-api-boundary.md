# ADR 0007: frontera web y contrato HTTP

**Estado:** aceptada

**Fecha:** 2026-10-23

**Aceptación:** 2026-10-24 por el director. Se confirmaron las dos decisiones
macro: un único origen público con `/api/v1` enrutado a `optimus`, y `optimus`
como única autoridad HTTP con `bumblebee` consumiendo el contrato generado sin
BFF.

## Contexto

`bumblebee` necesita consumir capacidades autenticadas y públicas sin crear una
segunda autoridad de negocio. Mantener frontend y API en orígenes distintos
añadiría CORS y complejidad de cookies; convertir Next.js en un BFF general
duplicaría contratos, errores y autorización que pertenecen a `optimus`.

También se necesita una fuente verificable para generar tipos, representar
errores de forma estable y evolucionar el contrato sin acoplar la interfaz a
estructuras internas de Rust o PostgreSQL.

## Decisión

### Origen y superficies

- Un único origen público enrutará `/api/v1` y sus subrutas, conservando el
  prefijo, al listener web de `optimus`; las demás rutas irán a `bumblebee`.
- Ese listener y su OpenAPI expondrán únicamente la superficie web aprobada.
  Migraciones, soporte, administración excepcional, health detallado y
  aprovisionamiento técnico usarán CLI, listener o red no públicos. El alta
  autoservicio será una capacidad web limitada, no acceso a esas superficies.
- El proxy eliminará encabezados `Forwarded` y `X-Forwarded-*` recibidos de
  clientes no confiables y los reconstruirá. `optimus` confiará solo en proxies
  configurados. Límites de cuerpo, tiempo y streaming se declararán por ruta.
- Los Server Components usarán una URL interna fija y validada por configuración,
  nunca derivada de `Host`, rutas o parámetros del usuario.
- CORS permanecerá deshabilitado en producción. Una integración entre orígenes
  necesitará canal, credencial y ADR propios.

### Propiedad y distribución del contrato

- `optimus` será la única autoridad HTTP y poseerá OpenAPI 3.1 para la superficie
  web versionada en `/api/v1`.
- Su CI publicará un artefacto inmutable con versión de contrato, commit de
  origen y SHA-256. `bumblebee` fijará esa procedencia en un manifiesto y
  conservará el snapshot generado; no dependerá de un checkout hermano
  implícito.
- Se generarán esquemas y tipos, no un cliente ejecutable obligatorio. Dos
  adaptadores pequeños y revisables manejarán las diferencias: servidor con URL
  interna y lista permitida de cookies; navegador con ruta relativa y CSRF.
  Código, URL o secretos exclusivos de servidor nunca llegarán al bundle.
- Antes de publicar una versión externa, un cambio incompatible exige actualizar
  coordinadamente el artefacto fijado. Después, los cambios incompatibles
  requerirán nueva versión mayor o una deprecación compatible comprobada.

### Navegador, sesión y CSRF

- Las mutaciones del navegador llamarán directamente a la ruta relativa de la
  API. No se creará un BFF ni un conjunto paralelo de Server Actions por
  defecto.
- La sesión opaca de ADR 0004 viajará únicamente en
  `__Host-transformers_session`, con `Secure`, `HttpOnly`, `SameSite=Lax`,
  `Path=/` y sin `Domain`. Se eliminará usando exactamente los mismos atributos.
  No contendrá roles ni tenant.
- El servidor entregará un token CSRF no secreto en una respuesta explícita y
  mantendrá su vínculo con la sesión o, antes de autenticar, con el identificador
  opaco de `__Host-transformers_csrf_context`. Esa cookie anónima tendrá
  `Secure`, `HttpOnly`, `SameSite=Lax`, `Path=/` y ningún `Domain`; no será una
  cuenta ni concederá acceso. El navegador conservará el token en memoria y lo
  devolverá en `X-CSRF-Token`; nunca se guardará en `localStorage`.
- Registro, login y recuperación usarán contexto CSRF anónimo. Autenticar o
  rotar sesión invalidará el contexto anónimo y rotará también el vínculo CSRF;
  cerrar sesión lo invalidará. Ambas cookies se borrarán con los mismos
  atributos con que se emitieron. Una pestaña con token anterior deberá renovar
  su contexto y no será aceptada por tolerancia.
- Todo `POST`, `PUT`, `PATCH` y `DELETE` verificará el token con comparación
  segura y exigirá un `Origin` permitido. Un origen ausente, `null` o no
  confiable se rechazará en el canal navegador. El callback OIDC será la
  excepción explícita: se protegerá mediante `state`, `nonce`, PKCE y redirección
  exacta, no mediante el encabezado CSRF.
- Desarrollo integrado y E2E ejecutarán HTTPS para probar los mismos atributos
  de cookie que producción.

### Renderizado, caché y respuestas

- Perfil, sesión, organizaciones y todo dato privado o tenant-scoped usarán
  `Cache-Control: private, no-store` y lecturas `no-store` en Next.js. Nunca
  entrarán en cachés compartidas de ruta o función.
- Una caché pública deberá aprobarse por operación e incluir el identificador
  público y demás variantes en su clave. Cachear disponibilidad nunca reservará
  un cupo ni sustituirá su comprobación transaccional al confirmar.
- Cookies, tokens CSRF y datos privados no aparecerán serializados en RSC, HTML,
  logs o telemetría. Una operación que rote o emita cookie responderá
  directamente al navegador o implementará retransmisión explícita; una lectura
  SSR no descartará silenciosamente `Set-Cookie`.
- Las respuestas usarán CSP —primero report-only y después enforcement—,
  `frame-ancestors`, `X-Content-Type-Options: nosniff`, `Referrer-Policy` y una
  `Permissions-Policy` mínima. Las rutas que contienen tokens no cargarán
  terceros ni enviarán referentes.

### Errores, repetición y concurrencia

- Los errores usarán `application/problem+json` conforme a RFC 9457. Además de
  los miembros estándar, expondrán `code`, `request_id` y, cuando corresponda,
  `errors: [{ field, code }]`. Solo esos códigos estables controlarán la UI;
  `title` y `detail` serán seguros para mostrar, pero no se interpretarán como
  lógica.
- `X-Request-ID` y `request_id` coincidirán. Un valor entrante solo se aceptará
  si cumple formato y tamaño; de lo contrario se generará uno nuevo. Nunca se
  reflejará texto arbitrario.
- Las convenciones mapearán explícitamente autenticación, ocultación de recursos,
  conflictos, precondiciones, validación, abuso e indisponibilidad a `401`,
  `403`/`404`, `409`, `412`, `422`, `429` y `503`, sin permitir enumeración de
  cuentas o tenants.
- Cuando una operación sea reintentable, `Idempotency-Key` se acotará por actor
  o contexto anónimo, tenant si existe, método y ruta. Se conservará el hash del
  cuerpo y una expiración; duplicados concurrentes se serializarán, el mismo
  cuerpo reproducirá el resultado y un cuerpo diferente devolverá conflicto.
- Los recursos con riesgo real de edición concurrente usarán versión o
  `ETag`/`If-Match`, con semántica `412` documentada por operación.
- Las fechas de transporte serán RFC 3339 en UTC y las zonas de negocio serán
  identificadores IANA. Los identificadores internos de tenant no serán
  selectores públicos.

## Alternativas descartadas

- **CORS entre frontend y API como topología inicial:** amplía la configuración
  y superficie de seguridad sin aportar una capacidad al MVP.
- **Next.js como BFF obligatorio:** crea dos capas HTTP y puede trasladar
  autorización al frontend.
- **Esquemas y DTO duplicados a mano:** permite divergencias silenciosas entre la
  API y sus consumidores.
- **JWT con roles como sesión de aplicación:** contradice la revocación y
  revalidación inmediata aceptadas en ADR 0004.

## Consecuencias

- El despliegue deberá proporcionar enrutamiento por ruta bajo un mismo origen.
- El despliegue también deberá separar la superficie web de las capacidades
  operativas y conservar una lista explícita de proxies confiables.
- `optimus` deberá estabilizar primero cada contrato antes de que `bumblebee`
  construya el flujo correspondiente.
- OpenAPI, ejemplos y respuestas HTTP formarán parte de las pruebas y de la
  revisión de cambios incompatibles.
- Una necesidad futura de API pública o cliente nativo se resolverá como un
  canal y contrato explícitos, no relajando por defecto la frontera web.

## Verificación obligatoria

- El contrato OpenAPI se valida; su artefacto, procedencia y generación de tipos
  son reproducibles y libres de diferencias en CI.
- Pruebas HTTP comprueban emisión, rotación y borrado de cookies; contexto CSRF
  anónimo y autenticado; orígenes ausentes o falsos; múltiples pestañas; tipo de
  problema, códigos estables y correlación mediante `request_id`.
- Un análisis de dependencias demuestra que `bumblebee` no implementa reglas de
  autorización, no importa detalles internos de `optimus` y no incluye módulos
  exclusivos de servidor en su bundle cliente.
- Las pruebas de navegador usan el mismo origen y no requieren CORS en el flujo
  del MVP; inspeccionan caché, encabezados y ausencia de secretos serializados.
- Pruebas de proxy demuestran precedencia de rutas, encabezados confiables,
  límites y que ninguna superficie operativa es alcanzable desde el origen web.

## Referencias

- [OpenAPI Specification](https://spec.openapis.org/oas/)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
