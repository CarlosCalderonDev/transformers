# ADR 0008: evolución de la base de datos y fronteras de acceso

**Estado:** aceptada

**Fecha:** 2026-10-23

**Aceptación:** 2026-10-24 por el director. Se confirmaron las dos decisiones
macro: una sola base PostgreSQL compartida con aislamiento por filas, y acceso
runtime exclusivamente mediante funciones estrechas que revalidan sesión,
membresía y permiso dentro de la misma transacción.

## Contexto

`arcee` ya demuestra aislamiento básico mediante restricciones y RLS, pero el
entorno actual inicializa una base vacía y depende de scripts de prueba para
aplicar SQL. Los roles runtime aún conservan privilegios más amplios que sus
capacidades y `optimus` usa pools intercambiables del mismo tipo.

Sin un protocolo de migraciones, compatibilidad y privilegio mínimo, una nueva
versión puede romper una instancia existente, usar accidentalmente el canal de
aprovisionamiento o producir una verificación verde sin haber ejecutado la
integración real.

## Decisión

### Migraciones y compatibilidad

- Las migraciones SQLx serán la única secuencia oficial del esquema. Serán
  planas, ordenadas y transaccionales por defecto. Una excepción no
  transaccional deberá declararse, revisarse y ser reanudable, con precondición,
  poscondición y recuperación propias.
- Una migración será inmutable después de fusionarse a la rama principal o
  aplicarse a una base compartida o no descartable, lo que ocurra primero.
- Antes de adoptar SQLx se inventariarán las bases existentes. Si todas son
  locales y descartables, `0001` y `0002` se validarán y formarán el historial
  inicial; `0003` solo entrará después de resolver la propiedad. Si existe una
  base no descartable sin `_sqlx_migrations`, se comprobará una huella completa
  del esquema y se usará un procedimiento de adopción explícito; nunca se
  insertarán versiones a ciegas ni se borrará esa base.
- Un rol `transformers_object_owner` sin inicio de sesión poseerá los objetos
  ordinarios. El despliegue usará `transformers_migrator`, separado y autorizado
  para asumirlo; ningún runtime tendrá DDL ni pertenecerá a esos roles.
- Cada cambio se probará desde una base vacía y desde una revisión anterior
  fijada por tag, commit o artefacto, con datos representativos y comparación de
  filas, relaciones e invariantes. Los cambios incompatibles seguirán
  `expand → backfill → contract`.
- La reversión ordinaria se hará con una migración correctiva hacia delante. La
  recuperación ante desastre usará respaldos y restauraciones probados antes de
  almacenar datos reales; esa prueba operativa no bloquea este fundamento, pero
  sí bloquea el piloto.
- `arcee` expondrá mediante una capacidad restringida una versión de contrato
  actual y un piso de compatibilidad, separados del historial interno de SQLx.
  `optimus` declarará la versión que consume y solo estará listo cuando
  `piso ≤ versión de optimus ≤ versión actual`. Un `contract` no elevará el piso
  mientras siga desplegada una versión anterior. El runtime nunca leerá
  `_sqlx_migrations` ni ejecutará migraciones.

### Esquemas, roles y capacidades

- Una única base PostgreSQL separará objetos en `identity`, `tenancy`,
  `scheduling` y `platform` cuando exista una capacidad real que los necesite.
  Los objetos existentes se moverán mediante migraciones compatibles, no
  reescribiendo el historial aplicado.
- Los canales iniciales serán `transformers_identity_runtime`,
  `transformers_internal_runtime` y `transformers_provision_runtime`;
  `transformers_public_runtime` aparecerá con la primera capacidad pública.
  Cada uno tendrá `NOINHERIT`, ninguna membresía privilegiada y una matriz
  explícita de `CONNECT`, esquemas, tablas, secuencias, funciones y operaciones
  permitidas y denegadas. Se revocará `CREATE` en esquemas compartidos y se
  configurarán privilegios predeterminados seguros.
- `optimus` representará cada canal con un tipo de pool distinto. Un caso de uso
  solo recibirá la capacidad mínima necesaria y nunca una conexión general que
  pueda saltarse permisos o contexto. Al arrancar comprobará el usuario efectivo,
  la versión de contrato y una sonda de capacidad; CI probará además que las URL
  intercambiadas o los privilegios excesivos fallan.
- RLS será de denegación por defecto para datos tenant-scoped. Las claves y
  restricciones también impedirán relaciones cruzadas.
- Las funciones `SECURITY DEFINER` se limitarán a capacidades de persistencia
  estrechas que necesiten cruzar la frontera de autorización; no existirá un
  ejecutor SQL o CRUD genérico. Fijarán un `search_path` seguro y revocarán
  `EXECUTE` a `PUBLIC`. Las poseerán roles `NOLOGIN` de capacidad estrecha —no
  el propietario general— y solo recibirán los objetos exactos que necesitan.

### Sesión, autorización y revocación

Una transacción no abarcará dos pools. Tampoco se confiará en un parámetro
personalizado de PostgreSQL como prueba de autorización: el rol de sesión puede
intentar cambiarlo.

Los roles runtime no tendrán `SELECT`, `INSERT`, `UPDATE` ni `DELETE` directos
sobre tablas tenant-scoped. Cada puerto de persistencia se implementará mediante
una función estrecha que recibe el hash del token, el selector público y los
datos propios de esa operación. El permiso requerido quedará fijado por la
función, no será una cadena elegida por el llamador. Antes de devolver control,
la misma función:

1. revalidará sesión, cuenta, organización, membresía, rol y permiso;
2. bloqueará con `FOR SHARE` las filas cuya revocación o suspensión cambiaría
   el resultado;
3. establecerá tenant, actor y canal con alcance local;
4. ejecutará la consulta o mutación exacta bajo RLS; y
5. devolverá únicamente su resultado y el contexto no secreto necesario.

Si la autorización adquiere primero los bloqueos, la operación termina antes de
la revocación; si la revocación llega primero, la operación se rechaza. Ese será
el punto de linealización y toda operación posterior observará la revocación.
El runtime no podrá ejecutar `SET ROLE`, acceder directamente a la tabla ni
separar autorización de operación. Si intenta modificar `app.tenant_id`,
`app.actor_id`, `app.channel` o un permiso contextual, no obtendrá por ello
ningún acceso porque no existe DML runtime que confíe en esos valores.

Un caso de uso con varias operaciones conservará una transacción Rust, pero su
adaptador solo expondrá métodos tipados que invocan esas funciones; nunca
entregará `PgConnection`. Cada función revalidará o reutilizará únicamente
bloqueos adquiridos dentro de esa misma transacción.

El alta autoservicio se inicia desde la superficie web, pero su adaptador usa
`transformers_provision_runtime`, que solo puede ejecutar la función exacta de
aprovisionamiento. Esa función vuelve a validar y bloquear la sesión dentro de
la misma transacción; no recibe un `account_id` previamente confiado ni concede
DML general. Las operaciones globales de identidad usarán su propio pool y
transacción.

### Clasificación y evidencia

Cada tabla se clasificará como global, raíz de tenant, propiedad de tenant,
relación global–tenant o plataforma interna. Un manifiesto legible por máquina
y su documentación registrarán datos personales, propietario lógico, estado de
retención, RLS, restricciones, índices y roles autorizados. Una retención aún no
aprobada figurará como `pending_no_real_data` y bloqueará datos reales sin
impedir cerrar la clasificación técnica.

Las suites serán independientes y fallarán explícitamente si no existe la base,
una credencial o una migración requerida. PostgreSQL 18 será el mayor soportado;
los entornos y CI fijarán versión de parche e imagen por digest, con una
actualización deliberada y probada.

## Alternativas descartadas

- **Aplicar SQL manualmente al iniciar contenedores:** no demuestra actualización
  ni conserva checksums o historial operativo.
- **Un usuario de aplicación con acceso amplio:** convierte un error de código
  en acceso transversal y contradice la defensa en profundidad.
- **Una base o esquema por tenant desde el MVP:** multiplica migraciones y
  operación sin evidencia de que el aislamiento compartido sea insuficiente.
- **Editar o deshacer migraciones aplicadas:** impide reproducir el estado y
  hace ambiguas las recuperaciones.
- **Validar sesión en un pool y autorizar después en otro:** deja una carrera de
  revocación sin un punto transaccional definido.

## Consecuencias

- El bootstrap de roles queda separado de las migraciones ordinarias y requiere
  credenciales de despliegue protegidas.
- Las pruebas necesitarán varias conexiones reales y una matriz negativa de
  privilegios, no solo consultas exitosas.
- Algunas correcciones actuales deberán añadirse como migraciones hacia delante
  y coordinarse con una ventana de compatibilidad de `optimus`.
- Las funciones privilegiadas estrechas añaden diseño y pruebas, pero evitan
  resolver la carrera de revocación mediante un usuario general o `SET ROLE`.
- Se conserva una operación inicial sencilla con una sola base, sin renunciar a
  límites que permitan evolucionar o separar cargas en el futuro.

## Verificación obligatoria

- El inventario decide de forma explícita cómo se adopta cada base existente y
  preserva cualquier instancia no descartable.
- Una base vacía converge al esquema esperado y una base en la revisión previa,
  poblada con fixtures, se actualiza sin pérdida de filas, relaciones o
  invariantes.
- Ejecutar nuevamente el migrador es un no-op seguro; alterar una migración
  aplicada produce un fallo visible de checksum. Una migración fallida no deja
  versión ni objetos parciales, salvo la excepción no transaccional cuya
  recuperación se prueba por separado.
- Cada rol runtime supera únicamente sus casos positivos y falla en todos los
  accesos fuera de su matriz, incluida la capacidad de asumir roles superiores.
- Las pruebas demuestran ausencia de fugas entre tenants y conexiones
  reutilizadas usando los mismos roles de producción.
- Carreras entre operación y revocación demuestran el punto de linealización y
  ningún caso de uso puede configurar tenant o reutilizar una sesión vieja por
  fuera de la función de capacidad.
- Pruebas hostiles modifican directamente todos los parámetros de contexto y
  confirman que el runtime no puede leer ni mutar otra tabla o tenant, antes o
  después de invocar una capacidad válida.
- `optimus` rechaza esquemas demasiado antiguos y futuros incompatibles, y se
  prueba la convivencia durante `expand`, `backfill` y `contract`.
- La integración de `optimus` no puede omitirse y contabilizarse como exitosa
  cuando faltan PostgreSQL, variables o revisiones de esquema.

## Referencias

- [PostgreSQL: Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL: Schemas](https://www.postgresql.org/docs/current/ddl-schemas.html)
- [PostgreSQL: CREATE FUNCTION](https://www.postgresql.org/docs/current/sql-createfunction.html)
