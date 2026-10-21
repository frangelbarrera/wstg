# Defensas contra el uso indebido de aplicaciones

|ID          |
|------------|
|WSTG-BUSL-07|

## Resumen

El uso indebido y el uso inválido de funcionalidades válidas pueden revelar ataques que intentan enumerar la aplicación web, identificar debilidades y explotar vulnerabilidades. Se deben realizar pruebas para determinar si existen mecanismos defensivos en la capa de aplicación que protejan la aplicación.

La falta de defensas activas permite a un atacante buscar vulnerabilidades sin que se tomen medidas para impedirlo. Por lo tanto, el propietario de la aplicación no sabrá que esta está bajo ataque.

### Ejemplo

Un usuario autenticado realiza la siguiente secuencia (poco probable) de acciones:

1. Intenta acceder a un ID de archivo que su rol no le permite descargar
2. Sustituye el número de ID del archivo por una comilla simple `'`
3. Cambia una solicitud GET por una POST
4. Añade un parámetro adicional
5. Duplica un par de nombre/valor de parámetro

La aplicación monitorea el uso indebido y, después del quinto evento, responde con un nivel de confianza extremadamente alto de que el usuario es un atacante. Por ejemplo, la aplicación:

- Deshabilita funcionalidades críticas
- Habilita pasos de autenticación adicionales para acceder a las funcionalidades restantes
- Añade demoras a cada ciclo de solicitud-respuesta
- Comienza a registrar datos adicionales sobre las interacciones del usuario (por ejemplo, encabezados de solicitudes HTTP saneados, cuerpos de solicitudes y cuerpos de respuestas)

Si la aplicación no responde de ninguna manera y el atacante puede seguir abusando de las funcionalidades y enviando contenido claramente malicioso a la aplicación, esta ha fallado este caso de prueba. En la práctica, es poco probable que las acciones concretas del ejemplo anterior ocurran de esa manera. Es mucho más probable que se utilice una herramienta de fuzzing para identificar debilidades en cada parámetro, uno por uno. Esto es lo que también habría hecho un evaluador de seguridad.

## Objetivos de prueba

- Generar notas de todas las pruebas realizadas contra el sistema.
- Revisar qué pruebas dieron lugar a un funcionamiento diferente debido a entradas agresivas.
- Entender las defensas implementadas y verificar si son suficientes para proteger el sistema frente a técnicas de evasión.

## Cómo probar

Esta prueba es inusual, ya que el resultado puede deducirse de todas las demás pruebas realizadas contra la aplicación web. Mientras se realizan las demás pruebas, se deben anotar las medidas que podrían indicar que la aplicación cuenta con mecanismos de autoprotección:

- Respuestas modificadas
- Solicitudes bloqueadas
- Acciones que cierran la sesión de un usuario o bloquean su cuenta

Es posible que estas medidas solo sean localizadas. Algunas defensas localizadas comunes (por función) son:

- Rechazar entradas que contengan ciertos caracteres
- Bloquear temporalmente una cuenta después de varios errores de autenticación

Los controles de seguridad localizados no son suficientes. A menudo, no hay defensas contra el uso indebido general, como:

- Navegación forzada
- Evasión de la validación de entradas en la capa de presentación
- Múltiples errores de control de acceso
- Nombres de parámetros adicionales, duplicados o faltantes
- Múltiples fallos de validación de entradas o de verificación de la lógica de negocio con valores que no pueden ser resultado de errores o erratas del usuario
- Recepción de datos estructurados (por ejemplo, JSON o XML) con formato inválido
- Recepción de cargas útiles evidentes de cross-site scripting o inyección SQL
- Uso de la aplicación a una velocidad que no sería posible sin herramientas de automatización
- Cambio de la ubicación geográfica continental de un usuario
- Cambio del agente de usuario
- Acceso a un proceso de negocio de varias etapas en el orden equivocado
- Un gran número de usos, o una alta frecuencia de uso, de funcionalidades específicas de la aplicación (por ejemplo, envío de códigos de vales, pagos con tarjeta de crédito fallidos, cargas de archivos, descargas de archivos, cierres de sesión, etc.).

### Representación engañosa de la interfaz de usuario / Falsificación de contenido

La representación engañosa de la interfaz de usuario ocurre cuando las entradas controladas por el usuario se muestran en elementos de interfaz de usuario confiables de una manera que puede engañar a usuarios o administradores, sin necesidad de ejecutar scripts. A diferencia del cross-site scripting, estos problemas se basan en el engaño visual o contextual y pueden permitir el abuso de flujos de trabajo o la ingeniería social dentro de la aplicación.

Estos problemas constituyen un uso indebido de la aplicación porque abusan de flujos de trabajo legítimos de la interfaz de usuario y de límites de confianza, en lugar de explotar una vulnerabilidad técnica.

Algunos ejemplos comunes son:

- Nombres de archivo, títulos o etiquetas controlados por el usuario que se muestran como mensajes generados por el sistema
- Archivos cargados con nombres modificados que parecen documentos confiables o artefactos del sistema
- Texto proporcionado por el usuario que se muestra como estados de aprobación, nombres de remitentes o indicadores de flujo de trabajo

Por ejemplo: una aplicación permite a los usuarios asignar nombres a los archivos que cargan. Un atacante carga un archivo llamado
"Pago aprobado - Sistema financiero". Si este nombre de archivo se muestra en un flujo de trabajo de
revisión administrativa sin indicar claramente que lo proporcionó el usuario, los revisores podrían ser engañados
y aprobar un proceso fraudulento.

Durante las pruebas, se debe evaluar si:

- Los datos controlados por el usuario se reflejan en contextos de interfaz de usuario privilegiados o autoritativos
- El texto inyectado puede imitar mensajes del sistema, estados de flujo de trabajo o etiquetas confiables
- La presentación de la interfaz de usuario podría influir en las decisiones de los usuarios o en los procesos de negocio, aunque no se produzca ningún exploit técnico

Estas defensas funcionan mejor en las partes autenticadas de la aplicación, aunque la tasa de creación de cuentas nuevas o de acceso a contenido (por ejemplo, para extraer información) puede ser útil en áreas públicas.

No es necesario que la aplicación monitoree todos los casos anteriores, pero sí es problemático que no monitoree ninguno. Al probar la aplicación web y realizar acciones como las anteriores, ¿se tomó alguna medida contra el evaluador? Si no, el evaluador debería informar que la aplicación parece no tener defensas activas contra el uso indebido en toda la aplicación. Cabe señalar que, en ocasiones, todas las respuestas a la detección de ataques pueden ser silenciosas para el usuario (por ejemplo, cambios en el registro, aumento del monitoreo, alertas a administradores y redireccionamiento de solicitudes mediante un proxy), por lo que no se puede garantizar la certeza de este hallazgo. En la práctica, muy pocas aplicaciones (o infraestructuras relacionadas, como un firewall de aplicaciones web) detectan estos tipos de uso indebido.

## Casos de prueba relacionados

Todos los demás casos de prueba son pertinentes.

## Remediación

Las aplicaciones deben implementar defensas activas para detener a atacantes y abusadores.

## Referencias

- [Software Assurance](https://www.cisa.gov/uscert/sites/default/files/publications/infosheet_SoftwareAssurance.pdf), US Department Homeland Security
- [IR 7684](https://csrc.nist.gov/publications/detail/nistir/7864/final) Common Misuse Scoring System (CMSS), NIST
- [Common Attack Pattern Enumeration and Classification](https://capec.mitre.org/) (CAPEC), The Mitre Corporation
- [OWASP AppSensor Project](https://owasp.org/www-project-appsensor/)
- Watson C, Coates M, Melton J and Groves G, [Creating Attack-Aware Software Applications with Real-Time Defenses](https://www.semanticscholar.org/paper/02365631792fa6c953e82cadb0e7268be35df905), CrossTalk The Journal of Defense Software Engineering, Vol. 24, No. 5, Sep/Oct 2011