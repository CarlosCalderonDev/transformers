# ADR 0003: fundamento multi-tenant

**Estado:** aceptada

**Fecha:** 2026-10-22

## Contexto

TRANSFORMERS deberá soportar muchas organizaciones, personas y procesos sin
mezclar información ni exigir un rediseño de sus fundamentos. El aislamiento no
puede depender solamente de que cada consulta recuerde filtrar por `tenant_id`.

La identidad de una persona es global y puede relacionarse con varias
organizaciones. Los datos operativos, configuraciones y procesos sí pertenecen a
una organización concreta.

## Decisión

### Modelo de propiedad

- Un tenant corresponde a una organización y posee un identificador estable,
  inmutable y no reutilizable.
- Cada entidad se clasificará antes de implementarse como global, propiedad de
  un tenant o relación entre ámbitos.
- Las entidades propiedad de una organización tendrán `tenant_id NOT NULL`.
- Personas y cuentas serán globales. Sus permisos existirán mediante membresías
  independientes para cada organización.

### Aislamiento obligatorio

El aislamiento utilizará tres capas complementarias:

1. **Backend:** cada caso de uso recibe un contexto de tenant validado y aplica
   autorización según actor, membresía y canal.
2. **Base de datos:** las tablas sujetas a aislamiento usarán Row Level Security
   de PostgreSQL con denegación por defecto.
3. **Integridad:** claves únicas y foráneas incluirán el tenant cuando sea
   necesario para impedir relaciones cruzadas incluso ante un error de código.

La aplicación no operará con un rol que pueda omitir RLS. Las migraciones y las
operaciones administrativas excepcionales usarán credenciales separadas y de
acceso restringido.

### Contexto confiable

- El backend resolverá el tenant desde una sesión, membresía, ruta pública o
  token previamente validados; nunca aceptará como autoridad un `tenant_id`
  arbitrario enviado por el frontend.
- Cada operación de base de datos protegida se ejecutará dentro de una
  transacción que establezca el contexto mediante configuración local a la
  transacción. Sin contexto válido, la política deberá negar el acceso.
- El pool de conexiones no conservará contexto entre transacciones.
- Los permisos se verificarán en el servidor. Los datos de sesión no serán la
  única fuente cuando una suspensión o cambio de rol deba tener efecto inmediato.
- Las operaciones globales o de soporte serán explícitas, mínimas y auditadas;
  no reutilizarán silenciosamente el flujo ordinario de un tenant.

### Componentes y datos derivados

- Trabajos asíncronos, eventos, claves de idempotencia, cachés, archivos y
  búsquedas conservarán el contexto de tenant cuando procesen datos aislados.
- Los registros de auditoría incluirán tenant, actor, acción, canal y resultado,
  evitando copiar datos personales innecesarios.
- `tenant_id` podrá aparecer en trazas y registros protegidos, pero no se usará
  por defecto como etiqueta de métricas de alta cardinalidad.
- Importaciones, exportaciones, respaldos y recuperaciones deberán conservar la
  propiedad y no permitir restauraciones cruzadas.

### Rendimiento y evolución

- Los índices se diseñarán a partir de consultas reales; en accesos acotados por
  organización se evaluará comenzar por `tenant_id`.
- El particionamiento, las réplicas y la distribución entre bases de datos se
  incorporarán solo con evidencia de capacidad, tamaño o aislamiento que lo
  justifique.
- El modelo mantendrá explícita la propiedad por tenant para permitir en el
  futuro mover organizaciones grandes sin cambiar la semántica del dominio.

### Responsabilidad por repositorio

- `arcee` implementará esquemas, restricciones, RLS, roles, migraciones y
  pruebas de aislamiento en PostgreSQL.
- `optimus` implementará resolución del tenant, autorización, transacciones,
  propagación de contexto y contratos seguros.
- `bumblebee` permitirá seleccionar o mostrar el contexto autorizado, pero
  nunca será la autoridad que conceda acceso a un tenant.
- `transformers` conservará las reglas canónicas de negocio y las decisiones
  generales de aislamiento.

## Verificación obligatoria

Antes de considerar lista la base multi-tenant deberá demostrarse que:

- una operación sin contexto no obtiene datos protegidos;
- el tenant A no puede leer, crear, modificar, eliminar ni relacionar datos del
  tenant B;
- RLS continúa activa mediante el rol real de la aplicación;
- el contexto no se filtra entre conexiones reutilizadas;
- trabajos asíncronos e idempotencia mantienen el aislamiento;
- cambios de membresía o suspensión revocan el acceso según las reglas;
- las consultas críticas tienen planes e índices medidos con datos
  representativos.

Estas comprobaciones se automatizarán y serán una condición para avanzar con
los módulos funcionales.

## Consecuencias

- La primera implementación requiere más disciplina en esquemas, consultas y
  pruebas, a cambio de reducir el riesgo de exposición entre organizaciones.
- La base compartida mantiene simple la operación inicial, pero una recuperación
  selectiva por tenant requerirá herramientas específicas.
- Ninguna capa aislada se considerará suficiente: backend, RLS e integridad
  referencial deberán permanecer coherentes.
- El agendamiento y los procesos futuros heredarán este fundamento en vez de
  definir su propia estrategia de aislamiento.
