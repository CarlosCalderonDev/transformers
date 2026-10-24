# ADR 0005: creación autoservicio de organizaciones

**Estado:** aceptada

**Fecha:** 2026-10-23

## Contexto

El MVP de agenda debe permitir que una persona se registre, se identifique y,
solo si lo necesita, cree una empresa para administrar sus propias citas. La
persona puede también actuar únicamente como cliente de una o varias empresas.

La decisión inicial de aprovisionamiento en ADR 0004 requería que un operador
de TRANSFORMERS creara cada organización e invitara al primer administrador.
Esa dependencia añade fricción al flujo principal del MVP y no es necesaria
para preservar el aislamiento entre organizaciones.

## Decisión

- Una persona con cuenta autenticada, habilitada y correo verificado podrá crear
  una organización desde el canal interno de autoservicio.
- Una misma persona podrá crear y administrar varias organizaciones
  independientes. Cada una tendrá su propio tenant, propietario, configuración
  y datos operativos aislados.
- El perfil global mínimo de una cuenta contiene nombre y apellido. El correo
  verificado pertenece al método de autenticación, no se duplica en el perfil.
- La operación creará atómicamente la organización en estado `setup`, con su
  nombre visible e identificador público único, y una membresía `active` de esa
  persona con el rol `administrator`.
- El alta solicitará el nombre visible de la organización. El sistema propondrá
  un identificador público derivado de ese nombre para la URL; la persona podrá
  modificarlo antes de crearla y la unicidad la decidirá el servidor.
- Esa membresía inicial es la única del MVP. No se expondrán invitaciones,
  empleados, roles adicionales ni administración de miembros en la interfaz o
  los contratos del MVP.
- La organización y su administrador inicial se crearán en una transacción. Si
  una de las dos creaciones falla, no persistirá ninguna de ellas.
- La cuenta y la persona continúan siendo globales. Crear una organización no
  las convierte en datos del tenant ni concede acceso a otra organización.
- El identificador de organización enviado por el navegador sigue siendo solo
  un selector: cada operación interna revalidará sesión, cuenta, membresía y
  organización antes de abrir el contexto tenant-scoped.
- El propietario inicial no podrá eliminar su única membresía ni perder el rol
  de administrador mediante los flujos ordinarios del MVP. La suspensión por
  seguridad seguirá siendo una operación excepcional y auditada.

## Consecuencias

- El alta empresarial deja de depender de un operador humano y pasa a formar
  parte del flujo de una cuenta autenticada.
- El esquema y los casos de uso conservan la entidad `Membership`, aunque el
  MVP cree una sola por organización. Esto permite incorporar después empleados
  y roles sin rediseñar identidad, tenants ni aislamiento.
- ADR 0004 queda sustituido únicamente en su sección **Alta inicial aprobada**;
  sus demás decisiones sobre autenticación, sesiones, autorización y resolución
  del tenant se mantienen vigentes.

## Verificación obligatoria

- Una cuenta autenticada crea exactamente una organización y una membresía
  administradora activa, o no crea ninguna de las dos.
- La nueva organización comienza en `setup` y no permite reservas hasta cumplir
  las reglas de activación.
- Otra cuenta no puede leer, administrar ni usar la organización recién creada.
- Una misma cuenta puede crear organizaciones independientes sin compartir sus
  datos operativos.
