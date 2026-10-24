# ADR 0011: configuración controlada y catálogos por sector

**Estado:** propuesta

**Fecha:** 2026-10-24

## Contexto

La [visión del producto](../product-vision.md) exige que empresas de distintos
sectores usen los mismos procesos y que sus diferencias se expresen mediante
configuraciones predefinidas y validadas, sin flujos ni código exclusivos por
organización. Ninguna decisión define todavía cómo se materializa ese
principio.

Sin esa regla, cada módulo resolverá la variación a su manera: columnas
sueltas, banderas JSON sin validar, condiciones por cliente en el código u
opciones fijadas en la interfaz. Corregirlo cuando existan muchos negocios y
funcionalidades sería costoso y riesgoso.

## Decisión propuesta

### Tres niveles separados

| Nivel | Qué contiene | Quién lo mantiene | Alcance |
| --- | --- | --- | --- |
| Catálogo de plataforma | Sectores, categorías de servicio y valores de referencia comunes. | TRANSFORMERS | Global, de solo lectura para las organizaciones. |
| Plantilla de sector | Punto de partida sugerido para un sector: servicios típicos con duración, opciones recomendadas y textos base. | TRANSFORMERS | Global y versionada. |
| Configuración de la organización | Valores que cada empresa elige dentro de las opciones permitidas: servicios, horarios, reglas operativas y publicación. | Administrador de la organización | Privada del tenant. |

- Los catálogos y plantillas son datos globales gestionados por una operación
  interna de TRANSFORMERS. No contienen datos de ninguna organización.
- Un elemento de catálogo tendrá un código estable que no se reutiliza. No se
  elimina si está en uso: se marca como obsoleto y deja de ofrecerse.
- Aplicar una plantilla **copia** sus valores a la configuración de la
  organización y registra de qué plantilla y versión provienen. Cambiar después
  la plantilla nunca altera silenciosamente a las organizaciones que la usaron.
- La organización puede modificar todo lo copiado dentro de las reglas del
  módulo; la plantilla solo acelera el inicio.

### Opciones declaradas, no comportamiento libre

- Cada módulo es dueño de sus opciones configurables. Cada opción se declara
  una vez con: clave estable, tipo, valores o rango permitidos, valor
  predeterminado, nivel en que se configura y permiso requerido para cambiarla.
- Una opción solo **elige entre comportamientos implementados y probados una
  vez para todos**. Por ejemplo, la antelación mínima para reservar o si la
  página pública muestra precios. No se admiten scripts, fórmulas, campos de
  código ni reglas escritas por la organización.
- Las opciones se almacenan tipadas en el esquema del módulo dueño, con
  restricciones de base de datos. No se usará una tabla genérica clave-valor ni
  documentos JSON sin esquema para reglas de negocio.
- `optimus` valida cada cambio en el dominio y `arcee` lo impide también con
  restricciones. `bumblebee` obtiene opciones, valores permitidos y textos del
  contrato HTTP; no codifica listas de opciones en la interfaz.
- Ninguna condición del código podrá depender de la identidad de una
  organización concreta. Toda diferencia entre empresas se expresa mediante una
  opción declarada.

### Evolución

- Toda funcionalidad nueva cuyo comportamiento pueda variar entre negocios
  deberá declararlo como opción antes de implementarse.
- Una opción nueva se agrega con un valor predeterminado que conserva el
  comportamiento vigente; las organizaciones existentes no cambian por
  desplegarla.
- Cambiar una configuración se audita con actor, instante, valor anterior y
  nuevo. Los cambios que afectan citas siguen la regla vigente: rigen para
  operaciones nuevas y las existentes conservan su instantánea.
- Un sector nuevo se incorpora agregando catálogo y plantilla, no código.
  Solo si un sector necesita un comportamiento inexistente se implementa una
  opción nueva, disponible para todos.

### Alcance en el MVP

- Se implementa el mecanismo con un único sector, peluquería y barbería.
- Al crear una organización, el administrador puede partir de la plantilla del
  sector, que precarga servicios sugeridos editables, o empezar vacío.
- Las opciones del MVP son únicamente las ya aprobadas en la
  [configuración operativa](../operational-configuration.md). No se agregan
  opciones nuevas para demostrar el mecanismo.
- Quedan fuera: campos personalizados, roles personalizados, constructor de
  procesos, módulos activables por plan y personalizaciones exclusivas por
  empresa.

## Alternativas descartadas

- **Tabla genérica clave-valor o JSON libre por organización:** flexible al
  inicio, pero sin tipos, restricciones ni dueño; las reglas terminan validadas
  en varios lugares o en ninguno.
- **Plantillas vinculadas en vivo:** un cambio central alteraría la operación de
  miles de negocios sin su decisión.
- **Código o reglas por cliente:** contradice la visión y multiplica pruebas y
  mantenimiento por cada organización.
- **Constructor genérico de procesos desde el MVP:** complejidad sin procesos
  reales que la justifiquen; la visión lo reserva para cuando existan varios.

## Consecuencias

- Agregar un sector o un negocio no requiere programar; agregar un
  comportamiento nuevo sí, pero una sola vez para todos.
- Cada módulo tendrá algo más de diseño inicial: declarar sus opciones, valores
  permitidos y valores predeterminados.
- La interfaz de configuración se genera desde el contrato y no se desincroniza
  del servidor.
- Las organizaciones conservan control total de su configuración aunque
  TRANSFORMERS mejore sus plantillas.

## Verificación obligatoria

- Aplicar una plantilla crea configuración del tenant con su origen registrado;
  modificar después la plantilla no cambia ninguna organización existente.
- Un valor fuera de lo permitido se rechaza en `optimus` y, si se intenta
  directamente, en `arcee`.
- Un catálogo obsoleto deja de ofrecerse sin romper a quienes ya lo usan.
- Una opción nueva desplegada conserva el comportamiento de las organizaciones
  existentes.
- Una revisión automática impide condiciones de código basadas en
  identificadores de organizaciones concretas.
- Ninguna organización puede leer ni modificar la configuración de otra; los
  catálogos y plantillas no contienen datos de tenants.
