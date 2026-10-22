# Arquitectura y diseño técnico

## Definición arquitectónica

TRANSFORMERS será una plataforma web multiempresa con un frontend desarrollado en
Next.js y un backend desarrollado en Rust con Axum. El backend seguirá una
arquitectura de monolito modular orientada al dominio, organizada mediante un
Cargo Workspace y respaldada por PostgreSQL.

El sistema tendrá módulos funcionales independientes, separación de
responsabilidades, procesamiento asíncrono con Tokio y capacidades internas
para coordinar procesos empresariales estandarizados.

La plataforma conservará el control de su lógica de negocio y sus operaciones
fundamentales. Las integraciones externas complementarán únicamente capacidades
específicas.

## Arquitectura general

La aplicación será un único sistema compuesto por módulos independientes en su
lógica e integrados mediante contratos internos explícitos. Esta organización
permitirá incorporar nuevas funcionalidades y mantener el código ordenado sin
introducir prematuramente la complejidad de los microservicios.

TRANSFORMERS aplicará DDD pragmático y una arquitectura hexagonal ligera con cuatro
responsabilidades principales:

- **Presentación:** interfaces web, componentes visuales y puntos de entrada
  HTTP.
- **Aplicación:** casos de uso, coordinación de operaciones y ejecución de
  acciones.
- **Dominio:** entidades, reglas de negocio, estados, validaciones e invariantes.
- **Infraestructura:** persistencia, consultas, servicios técnicos e
  integraciones externas.

Las reglas de negocio permanecerán independientes de las tecnologías de
interfaz, persistencia e integración.

## Organización de repositorios

El sistema se administrará mediante repositorios hermanos e independientes
dentro de una carpeta local llamada `workspace`:

```text
workspace/
├── transformers/  # lógica de negocio y arquitectura general
├── arcee/         # base de datos
├── bumblebee/     # frontend
└── optimus/       # backend y trabajadores
```

`optimus` organizará el backend como un Cargo Workspace y conservará sus módulos
funcionales, casos de uso, adaptadores y trabajadores. `bumblebee` consumirá sus
capacidades mediante contratos HTTP explícitos. `arcee` será la fuente técnica
de esquemas, migraciones, restricciones y políticas de acceso a datos.

Cada repositorio tendrá sus propias pruebas, automatización y documentación
técnica detallada. Las reglas compartidas y decisiones generales se mantendrán
en `transformers`; los contratos entre componentes estarán versionados y sus
cambios se coordinarán entre los repositorios afectados.

Cada módulo del backend podrá contener componentes de dominio, aplicación,
infraestructura y pruebas. Las carpetas, archivos, tablas, campos, estructuras y
funciones utilizarán nombres técnicos en inglés de manera consistente.

## Modularidad funcional

Los módulos representarán capacidades del negocio, como personas,
autenticación, empresas, permisos, catálogos, agendamiento, inventarios,
operaciones, pedidos, comunicaciones y procesos. Su número crecerá según las
necesidades reales.

Cada módulo administrará sus reglas, estados y operaciones. La colaboración
entre módulos ocurrirá mediante interfaces internas, casos de uso y eventos
cuando corresponda, sin accesos directos no controlados a la información de
otro módulo.

La modularidad no implica crear un servicio, una aplicación o una base de datos
independiente para cada capacidad.

## Datos y persistencia

PostgreSQL será el sistema central de almacenamiento y SQLx gestionará el acceso
desde Rust. El diseño priorizará:

- Estructuras relacionales normalizadas y extensibles.
- Identificadores estables y relaciones explícitas.
- Integridad referencial y restricciones en la base de datos.
- Transacciones para las operaciones que requieran consistencia.
- Control de concurrencia.
- Índices basados en patrones reales de consulta.
- Migraciones versionadas.
- Auditoría de operaciones relevantes.
- Protección de la información personal y empresarial.

JSONB se reservará para atributos genuinamente variables y no sustituirá sin
necesidad el modelado relacional. Se evitarán optimizaciones prematuras, como el
particionamiento generalizado, la duplicación de información o el uso de varias
bases de datos sin necesidades comprobadas.

El modelo de aislamiento entre organizaciones se describe en
[Multi-tenancy](multi-tenancy.md).

## Procesamiento asíncrono e integraciones

Los sistemas externos se utilizarán cuando sea necesario complementar
capacidades, por ejemplo, para enviar correos, mensajes SMS o notificaciones de
WhatsApp, procesar pagos y transferencias, o prestar servicios de facturación
electrónica.

Las integraciones usarán adaptadores desacoplados para permitir el reemplazo de
proveedores sin alterar las reglas de negocio.

Las operaciones diferidas serán ejecutadas por trabajadores asíncronos basados
en Tokio, con persistencia de tareas, reintentos, idempotencia y control de
fallos. Cuando un cambio operativo y su evento deban registrarse atómicamente,
se aplicará el patrón de outbox transaccional.

PostgreSQL se utilizará inicialmente para las tareas duraderas. Solo se
incorporarán tecnologías adicionales de mensajería cuando existan necesidades
demostradas.

## Frontend

La interfaz web se desarrollará con Next.js, React, TypeScript, HTML y CSS y se
comunicará por HTTP con el backend en Axum.

La plataforma proporcionará una experiencia unificada para personas y empresas,
adaptada a sus permisos y contextos. El frontend se organizará mediante
componentes reutilizables, funcionalidades separadas y estructuras de
navegación compartidas.

Se utilizarán renderizado del lado del servidor e hidratación selectiva cuando
resulte conveniente, procurando eficiencia, accesibilidad y compatibilidad con
navegadores de computadores, tabletas y dispositivos móviles. Las reglas
críticas de negocio y seguridad permanecerán en el servidor.

## Escalabilidad, infraestructura y calidad

La infraestructura inicial estará compuesta principalmente por:

- La aplicación web en Next.js.
- El servidor en Axum y Tokio.
- Trabajadores asíncronos para tareas diferidas.
- PostgreSQL como base de datos central.

Docker permitirá contenerizar los componentes y GitHub Actions automatizará la
integración, verificación y entrega continuas.

La arquitectura deberá soportar un crecimiento sostenido en personas, empresas,
procesos e interacciones sin degradar significativamente la experiencia de uso.
Los componentes podrán escalar progresivamente según lo requieran las
mediciones, sin depender inicialmente de una infraestructura distribuida
compleja.

Se incorporarán registros estructurados, métricas, pruebas automatizadas,
copias de seguridad y procedimientos de recuperación. El rendimiento se
equilibrará con la claridad, la estabilidad y la mantenibilidad. Toda
optimización deberá responder a mediciones o necesidades reales.

