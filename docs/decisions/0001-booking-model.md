# ADR 0001: modelo de reserva

**Estado:** aceptada

**Fecha:** 2026-10-21

## Contexto

El MVP necesita permitir que el cliente reserve sin cuenta y que la empresa
registre solicitudes recibidas por otros medios, sin agregar una cola de
aprobación que retrase el proceso.

## Decisión

- Un cliente invitado podrá reservar desde la página pública después de
  demostrar acceso al correo informado.
- Un administrador podrá crear la cita desde el canal interno.
- La cita se aceptará inmediatamente si el horario continúa libre.
- La confirmación de asistencia será posterior e independiente; no aprobará la
  reserva ni liberará el horario cuando falte.

## Consecuencias

- No existirá aprobación manual de la empresa antes de reservar.
- El servidor deberá volver a validar y reclamar el intervalo atómicamente.
- El correo será obligatorio para la reserva pública y opcional en la interna.
  No creará una cuenta ni probará la identidad de la persona.
- El administrador podrá ayudar a clientes que reserven por teléfono o en el
  establecimiento.

