# ADR 0006: arquitectura modular de los repositorios de código

**Estado:** aceptada

**Fecha:** 2026-10-23

**Aceptación:** 2026-10-24 por el director. Se confirmaron las dos decisiones
macro: backend como monolito modular con fronteras por capacidad y conservación
de los cuatro repositorios separados.

## Contexto

La dirección general de TRANSFORMERS ya establece un monolito modular, dominio
separado de infraestructura y responsabilidades distintas para persistencia,
backend e interfaz. La auditoría transversal encontró que estas ideas aún no se
reflejan en límites de código verificables: `optimus` sigue siendo un único
crate, `arcee` no tiene un ciclo oficial de migraciones y `bumblebee` todavía no
es una aplicación.

Continuar agregando casos de uso sobre esa forma provisional aumentaría el
costo de separar responsabilidades y podría convertir detalles de SQL, HTTP o
interfaz en reglas accidentales del producto.

## Decisión

TRANSFORMERS se implementará como un **sistema multirepositorio cuyo backend es
un monolito modular**, sin servicios distribuidos prematuros:

- `optimus` será un Cargo Workspace. Sus capacidades `identity`, `tenancy` y
  `organizations` contendrán dominio, aplicación y puertos; no dependerán de
  Axum, SQLx, configuración ni proveedores. Un crate de adaptadores implementará
  los puertos y `apps/api` será la raíz de composición y traducción HTTP.
- `arcee` conservará una sola base PostgreSQL y separará responsabilidades en
  esquemas de identidad, tenancy, agenda y plataforma únicamente cuando exista
  la primera tabla real de cada capacidad. Migraciones, roles y pruebas serán
  artefactos de primer nivel del repositorio.
- `bumblebee` utilizará Next.js App Router con componentes de servidor por
  defecto. `app` compondrá rutas, componentes privados y funcionalidades; una
  funcionalidad dependerá de `shared` y solo del punto de entrada público de
  otra funcionalidad. `shared` no contendrá reglas ni tipos exclusivos de una
  capacidad del dominio.
- `transformers` seguirá siendo la fuente canónica de negocio y decisiones
  transversales. Los detalles propios de cada implementación permanecerán en
  el repositorio responsable.

Las dependencias avanzarán hacia el dominio. No se crearán módulos `common`,
`utils`, buses, workers, abstracciones CRUD o capacidades futuras sin una
necesidad concreta y una prueba que justifique su frontera.

La dirección permitida en `optimus` será:

```text
identity ← tenancy ← organizations
    ↑          ↑             ↑
    └──────── adapters ──────┘
                 ↑
               api
```

- `identity` posee persona, cuenta, autenticación y sesión.
- `tenancy` posee identificadores tenant, membresía, roles, permisos y contexto
  autorizado.
- `organizations` posee nombre, estado, propiedad y ciclo de vida de la empresa.
- Los puertos pertenecen al caso de uso consumidor. Una capacidad solo usa la
  fachada pública de la capacidad situada a su izquierda y no accede a sus
  tablas o adaptadores.
- `adapters` depende de los puertos que implementa; las capacidades nunca
  dependen de `adapters`.
- `api` compone fachadas y adaptadores sin convertirse en una capa de negocio.

El grafo se comprobará inspeccionando manifiestos y fuentes, no solo compilando.
Axum y SQLx estarán prohibidos en los crates de capacidad.

La estructura detallada y sus puertas de calidad están en la
[evaluación arquitectónica](../architecture-assessment.md).

## Alternativas descartadas

- **Conservar la estructura provisional y refactorizar después:** oculta ahora
  los cruces de límites y hace más costosa su corrección cuando existan más
  endpoints y migraciones.
- **Microservicios desde el inicio:** agrega fallos distribuidos, operación y
  consistencia eventual sin una escala o equipo que lo justifique.
- **Unificar ahora los repositorios de código:** simplificaría cambios atómicos,
  pero perdería las responsabilidades e historiales independientes ya adoptados.
  Se conserva el modelo multirepositorio con contratos, artefactos y ventanas
  de compatibilidad explícitos; si esa coordinación supera su beneficio, otro
  ADR podrá reevaluarlo con evidencia.
- **Una capa técnica horizontal para todo el dominio:** facilita dependencias
  cruzadas y no expresa las capacidades del producto.

## Consecuencias

- La construcción funcional permanece pausada hasta materializar los límites
  mínimos y corregir las fronteras de seguridad P0.
- Habrá una reorganización temprana de archivos y crates, pero no una
  reescritura del comportamiento válido ya probado.
- Las capacidades podrán evolucionar dentro del monolito y extraerse en el
  futuro solo si mediciones y necesidades operativas lo exigen.
- La arquitectura no se considerará implementada por la mera existencia de
  carpetas: deberá demostrarse mediante reglas de dependencias, pruebas y CI.

## Verificación obligatoria

- Las capacidades de `optimus` compilan sin dependencias de transporte o
  persistencia y una regla automática inspecciona el grafo permitido.
- Ninguna ruta o caso de uso obtiene una conexión SQL cruda que permita omitir
  autorización o contexto tenant.
- Las dependencias de `bumblebee` respetan `app → features → shared` y no existen
  importaciones internas entre funcionalidades.
- Los objetos y pruebas de `arcee` están clasificados por responsabilidad y
  canal, sin crear esquemas vacíos.
- Una revisión transversal confirma que las reglas existen en una sola fuente
  canónica y que cada repositorio materializa únicamente su responsabilidad.
