# Transformers

Repositorio central de documentación del conjunto de proyectos Transformers.

## Responsabilidad

`transformers` es un repositorio exclusivamente documental. Debe conservar y actualizar la documentación relacionada con todos los proyectos que pertenecen al ecosistema Transformers.

## Proyectos relacionados

- `arcee`
- `bumblebee`
- `optimus`

Cada proyecto tiene su propio repositorio remoto e independiente.

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
