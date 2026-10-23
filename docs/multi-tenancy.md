# Multi-tenancy

**Estado:** fundamento aprobado en [ADR 0003](decisions/0003-multi-tenancy-foundation.md)

## Modelo adoptado

TRANSFORMERS funcionará como una plataforma multiempresa con identidad global de
personas. Inicialmente utilizará una base de datos PostgreSQL compartida y
tablas compartidas, con aislamiento lógico por organización.

Una persona podrá relacionarse con varias organizaciones y tener funciones y
permisos diferentes en cada una. Por ello, la identidad personal no pertenecerá
necesariamente a un solo tenant.

En este documento, un **tenant** es una organización que utiliza la plataforma.
`tenant_id` identifica la organización propietaria o responsable de un registro
cuando ese registro está sujeto a aislamiento organizacional.

## Clasificación de los datos

Antes de definir una tabla, sus datos deberán clasificarse explícitamente:

- **Globales:** pertenecen a la plataforma y se comparten de manera controlada,
  como determinados catálogos del sistema.
- **Propiedad de una organización:** contienen datos operativos o configuraciones
  exclusivos de un tenant y requieren `tenant_id`.
- **Relacionados con organizaciones:** representan vínculos entre una entidad
  global y uno o más tenants, como la relación entre una persona y una empresa.

`tenant_id` no debe añadirse indiscriminadamente a todas las tablas. Su uso debe
derivarse de esta clasificación y del modelo de propiedad de cada entidad.

## Reglas de aislamiento

Para los datos que pertenezcan a una organización se aplicarán estas reglas:

1. `tenant_id` será obligatorio y no podrá ser nulo.
2. Toda operación usará un contexto explícito de tenant y canal. Las operaciones
   protegidas exigirán una membresía autorizada o una capacidad pública limitada,
   como el acceso seguro de un cliente invitado.
3. El servidor resolverá el tenant desde una fuente confiable, como la sesión,
   el identificador público validado o un token limitado; nunca confiará en un
   `tenant_id` arbitrario enviado por el cliente.
4. Las relaciones entre entidades pertenecientes a un tenant impedirán
   referencias cruzadas entre organizaciones.
5. Las verificaciones de aislamiento se aplicarán también a trabajos
   asíncronos, scripts y procesos internos.

El tenant podrá resolverse a partir de un subdominio, un token validado o una
combinación coherente de ambos. El flujo inicial y sus límites están definidos
en [ADR 0004](decisions/0004-authentication-membership-tenant-resolution.md).

## Defensa en profundidad

El aislamiento no dependerá de un único filtro en la aplicación. Se aplicarán
tres capas complementarias:

1. **Aplicación:** los casos de uso y consultas incluyen explícitamente el
   contexto del tenant.
2. **Base de datos:** PostgreSQL aplica políticas de Row Level Security (RLS)
   para los datos que requieren aislamiento.
3. **Integridad:** restricciones y relaciones impiden referencias cruzadas entre
   organizaciones.

RLS es obligatoria para las tablas sujetas a aislamiento. Cada transacción
establecerá de forma segura el contexto del tenant, por ejemplo mediante
`set_config(..., true)`, y las políticas usarán ese contexto para autorizar o
rechazar operaciones. La ausencia de contexto deberá denegar el acceso.

El rol ordinario de la aplicación no podrá omitir RLS. Las migraciones y las
operaciones administrativas excepcionales utilizarán credenciales separadas,
restringidas y auditadas.

Se deberá comprobar especialmente el comportamiento del pool de conexiones para
evitar que el contexto de una solicitud permanezca activo en otra conexión o
transacción.

## Integridad entre entidades

Cuando dos tablas pertenezcan a un tenant, sus relaciones deberán incluir el
contexto organizacional. Una opción es utilizar claves únicas y foráneas
compuestas:

```text
customers UNIQUE (tenant_id, id)
orders    FOREIGN KEY (tenant_id, customer_id)
          REFERENCES customers (tenant_id, id)
```

Esto impide que una orden del tenant A haga referencia a un cliente del tenant
B. La forma exacta de las claves primarias se decidirá de acuerdo con el modelo
de identificadores; no es obligatorio que todas las claves primarias sean
compuestas si una restricción única equivalente protege la relación.

## Índices y rendimiento

Los índices se diseñarán según los patrones reales de consulta. Para consultas
frecuentes limitadas por organización, suele ser útil que un índice compuesto
comience por `tenant_id`, por ejemplo `(tenant_id, created_at)`.

Esta es una guía, no una regla universal. Cada índice tiene un costo de
almacenamiento y escritura y deberá justificarse mediante consultas y mediciones
concretas.

## Riesgos y mitigaciones

| Riesgo | Mitigación principal |
| --- | --- |
| Una consulta omite el filtro del tenant | Contexto obligatorio, RLS y pruebas de aislamiento |
| Un `JOIN` mezcla organizaciones | Relaciones que incluyan `tenant_id`, RLS y convenciones de consulta |
| Una tabla se clasifica erróneamente como global | Clasificación explícita durante el diseño y revisión del esquema |
| El contexto se filtra entre conexiones | Transacciones acotadas y pruebas con el pool de conexiones |
| Se crean índices innecesarios | Revisión de planes de ejecución y métricas reales |

## Lista mínima para la primera implementación

- [ ] Definir cómo se identifica y valida el tenant en cada solicitud.
- [ ] Clasificar las tablas como globales, propiedad de una organización o de
      relación.
- [ ] Hacer obligatorio `tenant_id` en las tablas que pertenezcan a un tenant.
- [ ] Proteger las relaciones para impedir referencias entre organizaciones.
- [x] Validar RLS y el manejo del contexto con SQLx y el pool de conexiones.
- [x] Verificar que el rol real de la aplicación no pueda omitir RLS.
- [x] Probar que el tenant A no puede leer ni modificar datos del tenant B.
- [x] Probar que el tenant A no puede relacionar sus registros con objetos del
      tenant B.
- [ ] Incluir `tenant_id` en la observabilidad cuando sea útil, sin exponer
      información sensible.
- [ ] Comprobar el aislamiento en trabajos asíncronos, cachés, archivos e
      idempotencia.

La validación inicial se implementó conjuntamente en `arcee` y `optimus`. Cubre
RLS, rol de ejecución, relaciones compuestas, contexto local a la transacción y
reutilización segura del pool. Los puntos restantes se completarán al incorporar
autenticación, trabajos asíncronos y almacenamiento externo.

