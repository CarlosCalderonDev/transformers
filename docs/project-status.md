# Estado del proyecto

**Etapa:** negocio

**Objetivo actual:** completar y verificar la lógica de negocio necesaria para
implementar el MVP aprobado.

## Criterio de terminado de la etapa

- [x] Etapa de definición cerrada con alcance y criterios aprobados.
- [x] Proceso de citas, estados, excepciones e invariantes definidos.
- [x] Configuración de organización, catálogo, horarios y contactos definida.
- [x] Membresías, roles y permisos operativos definidos.
- [x] Modelo de creación de la organización y su primer administrador aprobado.
- [ ] Revisión integral de negocio completada sin vacíos materiales.

## Estado actual

- La definición del [MVP inicial](mvp.md) fue aprobada integralmente y está
  registrada en [ADR 0002](decisions/0002-mvp-scope.md).
- El fundamento de aislamiento entre organizaciones fue aprobado en
  [ADR 0003](decisions/0003-multi-tenancy-foundation.md). Su implementación
  deberá validarse antes de construir los módulos funcionales.
- La prueba conjunta entre `arcee` y `optimus` valida RLS, relaciones entre
  tenants, sesión, membresía, resolución autorizada, contexto transaccional y
  ausencia de fugas en el pool. Aún faltan trabajos asíncronos, cachés y archivos.
- La autenticación global, las sesiones, la autorización por membresía, la
  resolución segura del tenant y el alta inicial fueron aprobadas en
  [ADR 0004](decisions/0004-authentication-membership-tenant-resolution.md).
- El modelo aprobado combina autenticación local propia con Google y Microsoft,
  vinculables explícitamente a una misma cuenta sin fusionar por correo.
- El primer fundamento técnico ya implementa y prueba el esquema de cuentas,
  credenciales, identidades, sesiones e invitaciones en `arcee`, y Argon2id,
  política de contraseña, tokens opacos, validación de sesión, claves externas y
  transacción tenant autorizada en `optimus`.
- La lógica canónica de reserva está en
  [Proceso de citas](appointment-process.md).
- La preparación de la empresa, personal, servicios, horarios y contactos está
  en [Configuración operativa](operational-configuration.md).
- Los términos canónicos están en [Glosario del dominio](glossary.md).

## Siguiente paso

Implementar los casos de uso y contratos HTTP de registro por invitación, inicio
local, cierre, recuperación y vinculación explícita. Google y Microsoft se
conectarán cuando existan credenciales configuradas fuera de Git.

## Ruta de cierre de la etapa

1. **Completado:** definición del MVP aprobada y cerrada.
2. **Completado:** proceso de citas definido de extremo a extremo.
3. **Completado:** configuración operativa y membresías definidas.
4. **Completado:** alta asistida e invitación inicial aprobadas.
5. **Completado:** integrar la decisión de autenticación y derivar el primer
   diseño técnico.
6. **Completado:** validar sesión y membresía antes de entregar una transacción
   tenant-scoped.
7. **En progreso:** completar la validación multi-tenant iniciada en `arcee` y
   `optimus` antes de implementar agendamiento.

## Decisiones macro pendientes

- Antes del piloto: política de privacidad, retención y anonimización para usar
  datos reales.

## Supuestos activos

- El problema y la utilidad comercial siguen siendo hipótesis hasta validarse
  con negocios reales durante el piloto.
- No se usarán datos personales reales en el piloto hasta aprobar su política de
  privacidad, retención y anonimización.

## Bloqueos

No hay bloqueos para construir la base. La integración real con Google y
Microsoft requerirá registrar las aplicaciones y aportar sus credenciales por
configuración segura; estas nunca se guardarán en Git.

