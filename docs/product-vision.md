# Visión del producto

## Propósito

TRANSFORMERS es una plataforma digital para estandarizar y automatizar procesos
operativos de pequeñas empresas y facilitar su interacción con las personas que
utilizan sus productos y servicios.

Su propósito es mejorar la calidad, rapidez y consistencia de la atención
mediante procesos estructurados que puedan utilizar empresas de diferentes
sectores económicos sin desarrollar una solución particular para cada negocio.

## Segmento inicial

El primer MVP se enfocará en pequeñas empresas que prestan servicios mediante
citas, usando peluquerías y barberías como referencia. El primer proceso cubrirá
la programación, confirmación, prestación y finalización de una cita, incluidas
sus principales excepciones.

La reserva podrá realizarla un cliente invitado desde la página pública, sin
crear una cuenta, o un administrador desde el canal interno. En ambos casos se
aceptará inmediatamente cuando el horario continúe disponible. El canal público
comprobará primero el acceso al correo para evitar reservas atribuidas a
terceros.

Este enfoque permitirá validar la propuesta en un contexto concreto. No limita
la aspiración de incorporar posteriormente otros procesos y sectores mediante
capacidades compartidas y configuración controlada.

## Participantes

TRANSFORMERS se concibe como un único sistema con dos tipos principales de
participantes: personas y empresas. Ambos interactúan mediante procesos
compartidos, con funcionalidades, permisos y configuraciones acordes con su
contexto.

Las interacciones podrán comenzar por iniciativa de cualquiera de las partes y
realizarse tanto en entornos digitales como presenciales.

Cada persona que cree o vincule una cuenta tendrá una identidad global en la
plataforma y podrá relacionarse con varias empresas, desempeñando funciones
diferentes en cada una. La identidad de una persona, su relación con una
organización y sus permisos dentro de ese contexto serán conceptos
independientes.

Una empresa podrá mantener contactos invitados que todavía no tengan una cuenta
global. Estos contactos serán privados para esa empresa y solo podrán vincularse
posteriormente mediante un proceso explícito y verificable; no se fusionarán
automáticamente entre empresas por compartir correo o teléfono.

## Capacidades generales

La plataforma busca centralizar:

- La identificación de personas y el reconocimiento entre participantes.
- La información de las empresas.
- Los catálogos de productos y servicios.
- La comunicación y las preferencias de los participantes.
- El registro verificable de las operaciones realizadas.
- La coordinación de procesos empresariales estandarizados.

## Estandarización y configuración

Los procesos mantendrán una estructura común. Las diferencias entre empresas se
administrarán mediante configuraciones predefinidas y validadas, evitando crear
flujos o reglas exclusivas para cada organización.

Cuando existan varios procesos reales que justifiquen una capacidad común, sus
definiciones podrán incluir estados, transiciones, condiciones, participantes y
actividades versionadas. Esta posibilidad no forma parte del primer MVP.

Un futuro coordinador de procesos podrá organizar la interacción entre módulos,
pero no sustituirá las reglas ni las invariantes de los dominios especializados.

## Principios del producto y su desarrollo

1. **Estandarización:** los procesos y capacidades serán compartidos y
   configurables.
2. **Modularidad:** cada componente tendrá responsabilidades claras y
   dependencias controladas.
3. **Independencia del dominio:** las reglas de negocio no dependerán de
   frameworks, bases de datos ni proveedores externos.
4. **Escalabilidad progresiva:** el sistema crecerá sin incorporar
   anticipadamente infraestructura innecesaria.
5. **Consistencia:** las operaciones conservarán la integridad de los datos ante
   concurrencia y fallos.
6. **Seguridad:** la identificación, autenticación, autorización y protección de
   la información formarán parte del diseño.
7. **Reutilización:** las funcionalidades comunes se implementarán una sola vez
   y podrán usarse en distintos procesos y contextos.
8. **Configurabilidad controlada:** las variaciones se expresarán mediante
   configuraciones predefinidas y validadas.
9. **Mantenibilidad:** se favorecerá código comprensible, verificable y fácil de
   modificar.
10. **Complejidad justificada:** los patrones, herramientas y abstracciones
    deberán aportar beneficios concretos.

## Estrategia de evolución

El desarrollo comenzará con un corte vertical del proceso de citas. Incluirá
únicamente las capacidades de identidad, empresas, autenticación, autorización y
relaciones necesarias para completar y validar ese proceso.

Las capacidades transversales, como la coordinación genérica de procesos, las
comunicaciones adicionales, las integraciones y el procesamiento asíncrono,
evolucionarán según necesidades observadas en procesos reales.

Los nuevos módulos respetarán las estructuras y los contratos existentes. La
arquitectura deberá permitir la evolución del producto sin intentar anticipar
desde el comienzo todas sus funcionalidades posibles.

