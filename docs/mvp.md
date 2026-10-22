# MVP inicial: gestión de citas

**Estado:** aprobado integralmente por el director el 22 de octubre de 2026.
Véase [ADR 0002](decisions/0002-mvp-scope.md).

**Segmento aprobado:** pequeñas empresas de servicios por cita, usando
peluquerías y barberías como referencia.

**Modelo de reserva aprobado:** el cliente invitado podrá reservar desde la
página pública sin crear una cuenta y el administrador podrá hacerlo desde el
canal interno. La cita se aceptará inmediatamente si el horario continúa
disponible. La decisión está registrada en
[ADR 0001](decisions/0001-booking-model.md).

Las reglas de la cita están en el [proceso de citas](appointment-process.md), la
preparación de la empresa en
[configuración operativa](operational-configuration.md) y los términos canónicos
en el [glosario del dominio](glossary.md).

## Problema priorizado para validación

Las pequeñas empresas pueden coordinar sus citas mediante conversaciones y
agendas separadas. Esto dificulta conocer la disponibilidad real, evitar cruces
y seguir cada cita hasta su cierre.

El primer MVP deberá ofrecer una agenda compartida y verificable para que una
persona reserve un servicio y la empresa gestione la atención de principio a
fin.

## Actores

- **Administrador:** configura el negocio, los servicios, los profesionales y
  sus horarios; puede gestionar todas las citas.
- **Profesional:** consulta su agenda y registra el inicio y resultado de las
  atenciones que tiene asignadas.
- **Cliente:** reserva, confirma, consulta, reprograma o cancela una cita sin
  necesidad de crear una cuenta.
- **Sistema:** calcula la disponibilidad, impide cruces, comunica los cambios y
  conserva el historial.

Una misma persona podrá desempeñar más de un rol. El administrador es el usuario
operativo principal; el cliente es el beneficiario del canal de reserva.

## Proceso principal

1. La empresa configura sus servicios, profesionales y horarios.
2. El cliente invitado verifica su correo o el administrador inicia la reserva
   desde el canal interno.
3. El cliente o administrador elige servicio, profesional y horario.
4. El sistema vuelve a validar la disponibilidad y reserva el horario.
5. El cliente recibe la información de la cita y puede confirmar su asistencia.
6. El profesional registra el inicio y la finalización de la atención.

La reserva será aceptada inmediatamente cuando el horario continúe disponible.
La confirmación representa la intención del cliente de asistir, no una segunda
aprobación de la empresa. Una cita reservada bloqueará el horario aunque el
cliente todavía no haya confirmado su asistencia; la falta de confirmación no la
cancelará automáticamente.

Como alternativas, una cita podrá reprogramarse, cancelarse o marcarse como
inasistencia. Cada cambio conservará actor, fecha, historial y motivo cuando
corresponda.

## Alcance aprobado

- Una sede, una zona horaria y una moneda por empresa.
- Roles fijos de administrador y profesional.
- Servicios con nombre, duración y precio informativo opcional en la moneda de
  la empresa, sin cobros ni cálculos financieros.
- Profesionales asociados con los servicios que pueden prestar.
- Horario semanal y bloqueos excepcionales por profesional.
- Una cita con un único servicio y un único profesional.
- Creación de citas por el cliente invitado con correo verificado o por el
  administrador.
- Página pública de reserva para cada empresa.
- Verificación del correo mediante un enlace temporal antes de la reserva
  pública, sin crear una cuenta.
- Acceso seguro independiente para consultar, confirmar, reprogramar o cancelar
  la cita.
- Reserva interna con correo o teléfono; sin correo, la gestión corresponderá al
  administrador y no habrá autoservicio del cliente.
- Consulta y gestión de la agenda de la empresa.
- Registro de inicio, finalización, cancelación e inasistencia.
- Prevención transaccional de citas superpuestas.
- Correo básico para creación, cambios y cancelación cuando el contacto tenga
  una dirección registrada.
- Historial de estados y acciones relevantes.
- Autenticación del personal, permisos en el servidor y aislamiento entre
  empresas.
- Interfaz web responsiva.

Los clientes invitados serán inicialmente contactos visibles solo para la
empresa correspondiente. Su futura vinculación con una identidad global no será
un requisito para reservar en el MVP.

## Fuera del alcance inicial

- Pagos, caja, facturación, inventario, nómina y comisiones.
- Tarifas por cancelación o inasistencia.
- WhatsApp, SMS, campañas y chat.
- Varias sedes o recursos reservables como sillas y cabinas.
- Citas recurrentes, grupales o con varios servicios o profesionales.
- Paquetes, promociones, listas de espera y sobrecupos.
- Sincronización con calendarios externos.
- Selección automática de cualquier profesional disponible.
- Roles personalizados y constructor genérico de procesos.
- Aplicaciones móviles nativas.
- Cuenta global e historial unificado para clientes.
- Métricas avanzadas y personalizaciones exclusivas por empresa.

## Criterios de aceptación funcional

- Un administrador puede configurar desde la interfaz la organización, un
  servicio, un profesional, el horario semanal y sus bloqueos.
- Un cliente invitado con correo verificado o administrador puede reservar un
  horario libre.
- Un administrador puede reservar para un contacto con teléfono y sin correo;
  en ese caso no se envía un enlace de gestión.
- Dos intentos concurrentes sobre intervalos superpuestos del mismo profesional
  producen una sola reserva; con profesionales distintos, ambos pueden tener
  éxito.
- Cancelar o reprogramar actualiza correctamente la disponibilidad.
- El profesional puede iniciar y finalizar la atención o registrar inasistencia.
- Tras cada transición, cliente y personal consultan el mismo estado operativo y
  horario; las acciones no autorizadas se ocultan y también se rechazan en el
  servidor.
- Los permisos de administrador y profesional se respetan.
- Una empresa no puede acceder ni relacionar datos de otra.
- Los flujos principales y los fallos relevantes tienen pruebas automatizadas.
- Los flujos completos pasan pruebas en Chromium, Firefox y WebKit con tamaños
  representativos de móvil, tableta y escritorio.

## Validación posterior con negocios reales

Antes de considerar validado el producto, se deberán probar el problema, el
flujo y el vocabulario con negocios del segmento. Los siguientes criterios
fueron aprobados como base del piloto:

- Tres negocios piloto durante cuatro semanas.
- Al menos cincuenta citas con inicio previsto dentro del piloto, gestionadas en
  conjunto.
- Al menos el 80 % de las citas cuyo fin previsto ocurrió hace veinticuatro horas
  o más está en `completed`, `cancelled` o `no_show`; esto mide cierre operativo,
  no éxito de la atención.
- Al menos dos negocios gestionan cinco citas o más durante la última semana.
- Ningún solapamiento entre citas vigentes del mismo profesional ni incidente de
  aislamiento entre empresas.

Estos valores son objetivos de validación, no resultados comprobados ni
garantías de mercado.

## Fundamento del alcance

El alcance aprobado fue contrastado con patrones operativos de herramientas de
agenda: configuración previa de servicios y personal, disponibilidad recurrente
y excepcional, prevención de cruces, confirmaciones, recordatorios,
cancelaciones e inasistencias. Referencias informativas:

- [Creación y programación de citas — Square](https://squareup.com/help/us/en/article/5349-schedule-and-accept-appointments)
- [Disponibilidad del personal — Square](https://squareup.com/help/us/en/article/8443-manage-staff-schedules-and-availability-with-square-appointments)
- [Configuración y estados de citas — Square](https://squareup.com/help/us/en/article/5351-manage-your-square-appointments-account-settings)

Estas referencias ayudan a reconocer el flujo habitual, pero no sustituyen la
validación directa con los futuros usuarios de TRANSFORMERS.

