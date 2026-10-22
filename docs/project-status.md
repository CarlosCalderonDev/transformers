# Estado del proyecto

**Etapa:** negocio

**Objetivo actual:** completar y verificar la lógica de negocio necesaria para
implementar el MVP aprobado.

## Criterio de terminado de la etapa

- [x] Etapa de definición cerrada con alcance y criterios aprobados.
- [x] Proceso de citas, estados, excepciones e invariantes definidos.
- [x] Configuración de organización, catálogo, horarios y contactos definida.
- [x] Membresías, roles y permisos operativos definidos.
- [ ] Modelo de creación de la organización y su primer administrador aprobado.
- [ ] Revisión integral de negocio completada sin vacíos materiales.

## Estado actual

- La definición del [MVP inicial](mvp.md) fue aprobada integralmente y está
  registrada en [ADR 0002](decisions/0002-mvp-scope.md).
- La lógica canónica de reserva está en
  [Proceso de citas](appointment-process.md).
- La preparación de la empresa, personal, servicios, horarios y contactos está
  en [Configuración operativa](operational-configuration.md).
- Los términos canónicos están en [Glosario del dominio](glossary.md).

## Siguiente paso

Decidir cómo se crearán las organizaciones piloto y su primer administrador. El
agente recomienda un alta asistida mediante una operación interna confiable y
una invitación inicial, dejando el registro empresarial autoservicio fuera del
MVP.

## Ruta de cierre de la etapa

1. **Completado:** definición del MVP aprobada y cerrada.
2. **Completado:** proceso de citas definido de extremo a extremo.
3. **Completado:** configuración operativa y membresías definidas.
4. **Director:** aprobar o cambiar el modelo de alta inicial.
5. **Agente:** integrar la decisión y revisar toda la lógica de negocio.
6. **Agente:** cerrar la etapa y preparar el diseño técnico del primer corte.

## Decisiones macro pendientes

- Actual: creación de las organizaciones piloto y su primer administrador.
- Antes del piloto: política de privacidad, retención y anonimización para usar
  datos reales.

## Supuestos activos

- El problema y la utilidad comercial siguen siendo hipótesis hasta validarse
  con negocios reales durante el piloto.
- No se usarán datos personales reales en el piloto hasta aprobar su política de
  privacidad, retención y anonimización.

## Bloqueos

El cierre de la etapa de negocio depende de decidir el modelo de alta inicial.

