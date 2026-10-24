# Estado del proyecto

**Etapa:** diseño técnico — cierre del fundamento arquitectónico

**Objetivo actual:** resolver las decisiones de producto que condicionan la
arquitectura y materializar límites seguros, migraciones, contratos y pruebas
reales en `arcee`, `optimus` y `bumblebee` antes de continuar funcionalidades.

## Criterio de terminado de la etapa

- [x] Desarrollo funcional pausado por prioridad del director.
- [x] Auditoría transversal de los tres repositorios de código completada.
- [x] Propietario, administrador y profesional definidos sin contradicciones.
- [x] Alcance de la cuenta personal sin empresa definido para el MVP.
- [ ] Modelo de configuración controlada y catálogos definido.
- [x] ADR de modularidad, frontera web y evolución de datos aceptados.
- [ ] Estructuras base materializadas sin módulos especulativos.
- [ ] Fronteras de seguridad P0 corregidas.
- [ ] Migraciones, integración y CI incapaces de producir falsos verdes.
- [ ] Revisión transversal de salida completada y aceptada.

## Estado actual

- La definición del [MVP inicial](mvp.md) fue aprobada integralmente y está
  registrada en [ADR 0002](decisions/0002-mvp-scope.md).
- Por decisión del director, el trabajo funcional está temporalmente pausado y
  el fundamento arquitectónico es la prioridad principal, por encima de la
  velocidad de entrega.
- La [evaluación arquitectónica](architecture-assessment.md) concluyó que la
  dirección conceptual es válida y el código existente es recuperable, pero
  sus límites prácticos, migraciones y evidencia todavía no permiten continuar
  con seguridad.
- El [tablero transversal](architecture-foundation-board.md) registra la ruta
  crítica, dependencias, criterios de aceptación y evidencia exigida por
  repositorio.
- El [ADR 0006](decisions/0006-modular-codebase-architecture.md) fue aceptado
  el 2026-10-24: monolito modular con fronteras por capacidad y cuatro
  repositorios separados.
- El [ADR 0007](decisions/0007-web-api-boundary.md) fue aceptado el
  2026-10-24: un único origen público con `/api/v1` hacia `optimus`, que es la
  única autoridad HTTP; `bumblebee` consume el contrato generado sin BFF.
- El [ADR 0008](decisions/0008-database-evolution-and-access-boundaries.md)
  fue aceptado el 2026-10-24: una sola base PostgreSQL con RLS y acceso runtime
  exclusivamente mediante funciones estrechas que revalidan sesión, membresía y
  permiso en la misma transacción.
- El [ADR 0009](decisions/0009-organization-ownership-and-initial-roles.md)
  fue aceptado el 2026-10-24: propietario único y permanente distinto de los
  roles; el MVP asume negocios operados por su propietario, cuya membresía
  inicial recibe `administrator` y `professional`. El ADR 0005 quedó
  confirmado con ese ajuste.
- El [ADR 0010](decisions/0010-personal-account-without-organization.md) fue
  aceptado el 2026-10-24: en el MVP la cuenta del cliente no cambia el flujo de
  reserva; una vinculación futura solo se admitirá por reservas que la propia
  persona realice con su sesión iniciada.
- El director identificó la configuración controlada y los catálogos por
  sector como una base del producto. Se registró como `FND-007` y se propone en
  [ADR 0011](decisions/0011-controlled-configuration-and-catalogs.md).
- El fundamento de aislamiento entre organizaciones fue aprobado en
  [ADR 0003](decisions/0003-multi-tenancy-foundation.md). Su implementación
  deberá validarse antes de construir los módulos funcionales.
- La prueba conjunta existente entre `arcee` y `optimus` aporta evidencia de
  RLS, relaciones entre tenants, sesión, membresía y contexto transaccional. La
  omisión silenciosa ya fue eliminada: la ausencia de cada URL falla por su
  causa exacta. Unitarias, formato y Clippy pasan; la nueva caracterización
  incluye las tres suites SQL, preflight, exclusión por puerto y una comprobación
  explícita de limpieza final, pero su ejecución completa y el CI siguen
  pendientes porque la integración Docker Desktop/WSL devuelve `Input/output
  error` al ejecutar el cliente Docker.
- La autenticación global, las sesiones, la autorización por membresía y la
  resolución segura del tenant fueron aprobadas en
  [ADR 0004](decisions/0004-authentication-membership-tenant-resolution.md).
- La creación autoservicio de una organización fue aprobada en
  [ADR 0005](decisions/0005-self-service-organization-onboarding.md); su
  membresía inicial recibe ambos roles conforme a ADR 0009, lo que resuelve el
  conflicto con la necesidad de un profesional en el MVP.
- Una cuenta puede crear varias organizaciones independientes; el selector de
  organización nunca sustituye la validación de su membresía y permiso.
- El alta solicita nombre visible e identificador público único de la
  organización; el identificador se propone desde el nombre y puede ajustarse
  antes de crearla.
- El modelo aprobado combina autenticación local propia con Google y Microsoft,
  vinculables explícitamente a una misma cuenta sin fusionar por correo.
- El primer fundamento técnico ya implementa y prueba el esquema de cuentas,
  credenciales, identidades, sesiones e invitaciones en `arcee`, y Argon2id,
  política de contraseña, tokens opacos, validación de sesión, claves externas y
  transacción tenant autorizada en `optimus`.
- La creación autoservicio reutilizará esas cuentas globales, membresías y
  fronteras tenant sin convertir a la persona en un dato de la organización.
- `arcee` ya persiste el perfil global mínimo y exige correo verificado para
  crear atómicamente la organización, su propietario, membresía y rol inicial;
  `optimus` expone el caso de uso técnico sin endpoint HTTP todavía.
- La lógica canónica de reserva está en
  [Proceso de citas](appointment-process.md).
- La preparación de la empresa, personal, servicios, horarios y contactos está
  en [Configuración operativa](operational-configuration.md).
- Los términos canónicos están en [Glosario del dominio](glossary.md).

## Siguiente paso

Decidir el modelo de configuración controlada y catálogos propuesto en
[ADR 0011](decisions/0011-controlled-configuration-and-catalogs.md). Es la
última decisión de la puerta 0.

## Ruta de cierre de la etapa

1. **Completado:** auditar los tres repositorios y detener el crecimiento sobre
   límites provisionales.
2. **En curso:** cerrar las decisiones de producto de la puerta 0 (0009 y 0010
   aceptados; falta 0011).
3. **Completado:** ADR 0006, 0007 y 0008 aceptados el 2026-10-24.
4. **Pendiente:** materializar estructuras, migraciones y límites de seguridad
   según las puertas 1 y 2 del tablero.
5. **Pendiente:** establecer contrato y pruebas no omitibles según la puerta 3.
6. **Pendiente:** superar la revisión transversal y obtener aceptación de salida.
7. **Posterior:** reanudar el corte vertical
   `registro → verificación → sesión → organización`.

## Decisiones macro pendientes

- Modelo de configuración controlada y catálogos por sector (ADR 0011).
- Antes del piloto: política de privacidad, retención y anonimización para usar
  datos reales.

## Supuestos activos

- El problema y la utilidad comercial siguen siendo hipótesis hasta validarse
  con negocios reales durante el piloto.
- No se usarán datos personales reales en el piloto hasta aprobar su política de
  privacidad, retención y anonimización.

## Bloqueos

- La semántica de propiedad bloquea el modelo definitivo de organizaciones,
  autorización y onboarding; no bloquea preparar migraciones, estructura y CI
  después de aceptar sus ADR.
- Ningún endpoint funcional, tabla de agenda o pantalla de negocio se construirá
  hasta superar la puerta de salida arquitectónica.
- La evidencia PostgreSQL de `XCHAR-001` requiere recuperar la integración de
  Docker Desktop con WSL. No requiere credenciales del director; probablemente
  exigirá reiniciar WSL y Docker Desktop desde Windows. Se realizará en un corte
  controlado porque ese reinicio detiene todo el entorno WSL actual.
- Google y Microsoft requerirán aplicaciones y credenciales externas cuando
  llegue su corte; esas credenciales nunca se guardarán en Git y no bloquean el
  fundamento actual.

