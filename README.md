# Transformers

Repositorio central de documentación del conjunto de proyectos Transformers.

## Descripción del proyecto

TRANSFORMERS es una plataforma web multiempresa para estandarizar y automatizar procesos operativos de pequeñas empresas. Facilita la interacción entre las empresas y las personas que utilizan sus productos y servicios mediante procesos compartidos, configurables y verificables.

El MVP está definido y aprobado; el proyecto se encuentra en la etapa de lógica de negocio.

## Responsabilidad

`transformers` es un repositorio exclusivamente documental. Debe conservar y actualizar la documentación relacionada con todos los proyectos que pertenecen al ecosistema Transformers.

## Documentación

- [Estado y siguiente paso](docs/project-status.md)
- [MVP inicial](docs/mvp.md)
- [Proceso de citas](docs/appointment-process.md)
- [Configuración operativa](docs/operational-configuration.md)
- [Glosario del dominio](docs/glossary.md)
- [Visión del producto](docs/product-vision.md)
- [Arquitectura y diseño técnico](docs/architecture.md)
- [Multi-tenancy](docs/multi-tenancy.md)
- [Stack tecnológico](docs/technology-stack.md)
- [Índice completo](docs/README.md)

## Principios esenciales

- Una experiencia unificada para personas y empresas.
- Procesos comunes adaptables mediante configuración, no mediante código exclusivo para cada organización.
- Arquitectura modular, segura y mantenible.
- Escalabilidad progresiva basada en necesidades y mediciones reales.

## Alcance aprobado

El MVP será una aplicación web responsiva compatible con computadores, tabletas y dispositivos móviles mediante navegadores web.

## Proyectos relacionados

- `arcee`: base de datos y su documentación técnica detallada.
- `bumblebee`: frontend y su documentación técnica detallada.
- `optimus`: backend y su documentación técnica detallada.

Cada proyecto tiene su propio repositorio remoto e independiente.

`transformers` conserva la documentación canónica de producto, procesos y
lógica de negocio. También puede registrar decisiones técnicas generales que
afecten a varios proyectos, mientras los detalles de implementación pertenecen
al repositorio responsable.

## Configuración Git personal

Todo proyecto nuevo creado directamente dentro de `workspace` debe usar la cuenta personal de Carlos Calderon:

- Nombre: `Carlos Calderon`
- Correo: `CarlosCalderonDev@users.noreply.github.com`
- Remoto SSH: `git@github-personal:CarlosCalderonDev/<repositorio>.git`

El alias SSH `github-personal` selecciona la clave personal. No se debe usar `git@github.com` para estos repositorios.

## Área de trabajo

Los proyectos se desarrollan dentro de una carpeta raíz llamada `workspace`. La ruta absoluta puede cambiar según el dispositivo, pero la estructura lógica se mantiene:

```text
workspace/
├── transformers/   # Documentación central
├── arcee/          # Repositorio independiente
├── bumblebee/      # Repositorio independiente
└── optimus/        # Repositorio independiente
```
