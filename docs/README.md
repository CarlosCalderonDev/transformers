# Documentación de TRANSFORMERS

Este directorio reúne las definiciones funcionales y técnicas del proyecto.
Cada documento trata un tema específico y debe mantenerse coherente con los
demás.

## Contenido

- [Estado del proyecto](project-status.md): etapa actual, objetivo, decisiones
  pendientes y siguiente paso.
- [Tablero del fundamento arquitectónico](architecture-foundation-board.md):
  paquetes, dependencias, criterios y evidencia para cerrar la arquitectura de
  los tres repositorios de código.
- [Evaluación arquitectónica](architecture-assessment.md): dictamen transversal,
  brechas encontradas y diseño objetivo propuesto.
- [MVP inicial](mvp.md): segmento, problema, actores, alcance, exclusiones y
  criterios de aceptación aprobados.
- [Proceso de citas](appointment-process.md): reglas, estados, transiciones,
  permisos y excepciones del primer proceso.
- [Configuración operativa](operational-configuration.md): organización,
  integrantes, catálogo, horarios, publicación y contactos.
- [Glosario del dominio](glossary.md): términos canónicos y sus equivalentes en
  el código.
- [Decisión sobre el modelo de reserva](decisions/0001-booking-model.md): canales
  autorizados, aceptación y consecuencias.
- [Aprobación integral del MVP](decisions/0002-mvp-scope.md): alcance aprobado y
  control de cambios.
- [Fundamento multi-tenant](decisions/0003-multi-tenancy-foundation.md): propiedad,
  aislamiento, contexto confiable, responsabilidades y verificación obligatoria.
- [Autenticación propia, federada y resolución del tenant](decisions/0004-authentication-membership-tenant-resolution.md):
  cuenta global, credencial local, Google, Microsoft, sesiones, membresías y
  alta inicial.
- [Creación autoservicio de organizaciones](decisions/0005-self-service-organization-onboarding.md):
  alta de empresa por una cuenta autenticada y membresía administradora inicial.
- [Arquitectura modular de código](decisions/0006-modular-codebase-architecture.md):
  límites y dependencias de `arcee`, `optimus` y `bumblebee`.
- [Frontera web y contrato HTTP](decisions/0007-web-api-boundary.md):
  origen único, autoridad de la API, sesiones y contrato generado.
- [Evolución y acceso a datos](decisions/0008-database-evolution-and-access-boundaries.md):
  migraciones, compatibilidad, esquemas, roles y evidencia obligatoria.
- [Propiedad y roles iniciales](decisions/0009-organization-ownership-and-initial-roles.md):
  propietario único y membresía inicial administradora y profesional.
- [Cuenta personal sin organización propuesta](decisions/0010-personal-account-without-organization.md):
  identidad y perfil globales separados de contactos e historial de clientes.
- [Visión del producto](product-vision.md): propósito, participantes, alcance y
  estrategia de evolución.
- [Arquitectura y diseño técnico](architecture.md): organización del sistema,
  módulos, persistencia, integraciones e infraestructura.
- [Multi-tenancy](multi-tenancy.md): modelo de pertenencia, aislamiento entre
  organizaciones y recomendaciones de implementación.
- [Stack tecnológico](technology-stack.md): tecnologías adoptadas y reglas para
  su utilización.

## Criterio de mantenimiento

Las decisiones confirmadas deben expresarse como requisitos. Las alternativas,
recomendaciones o ideas pendientes de validación deben identificarse como tales
para evitar que una posibilidad se interprete como una decisión arquitectónica.

Cuando una decisión importante cambie, se actualizarán todos los documentos
afectados en el mismo cambio.

