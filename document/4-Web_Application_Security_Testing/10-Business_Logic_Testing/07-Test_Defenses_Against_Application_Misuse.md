# Probar Defensas contra el Uso Indebido de la Aplicación

|ID          |
|------------|
|WSTG-BUSL-07|

## Resumen

El uso indebido y uso inválido de funcionalidad válida puede identificar ataques que intentan enumerar la aplicación web, identificar debilidades, y explotar vulnerabilidades. Se deberían realizar pruebas para determinar si hay mecanismos defensivos a nivel de aplicación en su lugar para proteger la aplicación.

La falta de defensas activas permite a un atacante cazar vulnerabilidades sin ningún recurso. El propietario de la aplicación así no sabrá que su aplicación está bajo ataque.

### Ejemplo

Un usuario autenticado emprende la siguiente secuencia (improbable) de acciones:

1. Intenta acceder a un ID de archivo que su rol no tiene permitido descargar
2. Sustituye un apóstrofe simple `'` en lugar del número de ID de archivo
3. Altera una solicitud GET a POST
4. Añade un parámetro extra
5. Duplica un par nombre/valor de parámetro

La aplicación está monitoreando en busca de uso indebido y responde después del 5to evento con extrema alta confianza que el usuario es un atacante. Por ejemplo la aplicación:

- Deshabilita funcionalidad crítica
- Habilita pasos de autenticación adicionales a la funcionalidad restante
- Añade demoras de tiempo en cada ciclo solicitud-respuesta
- Comienza a registrar datos adicionales sobre las interacciones del usuario (por ejemplo encabezados de solicitud HTTP saneados, cuerpos y cuerpos de respuesta)

Si la aplicación no responde de ninguna manera y el atacante puede continuar abusando funcionalidad y enviando contenido claramente malicioso a la aplicación, la aplicación ha fallado este caso de prueba. En la práctica las acciones de ejemplo discretas en el ejemplo anterior son improbables que ocurran así. Es mucho más probable que una herramienta de fuzzing sea usada para identificar debilidades en cada parámetro por turno. Esto es lo que un tester de seguridad también habrá emprendido.

## Objetivos de Prueba

- Generar notas de todas las pruebas conducidas contra el sistema.
- Revisar qué pruebas tuvieron una funcionalidad diferente basada en entrada agresiva.
- Entender las defensas en su lugar y verificar si son suficientes para proteger el sistema contra técnicas de evasión.

## Cómo Probar

Esta prueba es inusual en que el resultado puede extraerse de todas las otras pruebas realizadas contra la aplicación web. Mientras se realizan todas las otras pruebas, tomar nota de medidas que podrían indicar que la aplicación tiene auto-defensa incorporada:

- Respuestas cambiadas
- Solicitudes bloqueadas
- Acciones que cierran la sesión de un usuario o bloquean su cuenta

Estas podrían solo ser localizadas. Las defensas localizadas comunes (por función) son:

- Rechazar entrada que contenga ciertos caracteres
- Bloquear una cuenta temporalmente después de un número de fallos de autenticación

Los controles de seguridad localizados no son suficientes. A menudo no hay defensas contra el uso indebido general tal como:

- Navegación forzada
- Evasión de validación de entrada de capa de presentación
- Múltiples errores de control de acceso
- Nombres de parámetro adicionales, duplicados o faltantes
- Múltiples fallos de validación de entrada o verificación de lógica de negocio con valores que no pueden ser el resultado de errores del usuario o typos
- Datos estructurados (por ejemplo JSON, XML) de un formato inválido son recibidos
- Payloads blatantes de cross-site scripting o inyección SQL son recibidos
- Utilizar la aplicación más rápido de lo que sería posible sin herramientas de automatización
- Cambio en la geo-localización continental de un usuario
- Cambio de user agent
- Acceder a un proceso de negocio multi-etapa en el orden equivocado
- Gran número de, o alta tasa de uso de, funcionalidad específica de la aplicación (por ejemplo envío de código de voucher, pagos de tarjeta de crédito fallidos, subidas de archivos, descargas de archivos, log outs, etc).

### Tergiversación de UI / Falsificación de Contenido

La tergiversación de UI ocurre cuando entrada controlada por el usuario se renderiza en elementos de UI confiables de una manera que puede engañar a usuarios o administradores, sin requerir ejecución de script. A diferencia del cross-site scripting, estos problemas dependen de engaño visual o contextual y pueden habilitar abuso de flujo de trabajo o ingeniería social dentro de la aplicación.

Estos problemas representan uso indebido de la aplicación porque abusan de flujos de trabajo de UI legítimos y límites de confianza en lugar de explotar una vulnerabilidad técnica.

Ejemplos comunes incluyen:

- Nombres de archivo, títulos o etiquetas controlados por el usuario mostrados como mensajes generados por el sistema
- Subidas de archivo renombradas que aparecen como documentos confiables o artefactos del sistema
- Texto proporcionado por el usuario renderizado como estados de aprobación, nombres de remitente, o indicadores de flujo de trabajo

Por ejemplo: una aplicación permite a los usuarios nombrar archivos subidos. Un atacante sube un archivo llamado
"Pago Aprobado – Sistema de Finanzas". Cuando este nombre de archivo se muestra en un flujo de trabajo de
revisión administrativa sin indicación clara de que es proporcionado por el usuario, los revisores podrían ser engañados
a aprobar un proceso fraudulento.

Durante las pruebas, evaluar si:

- Datos controlados por el usuario se reflejan en contextos de UI privilegiados o autoritativos
- Texto inyectado puede imitar mensajes del sistema, estados de flujo de trabajo, o etiquetas confiables
- La presentación de UI podría influir en decisiones del usuario o procesos de negocio a pesar de que no ocurra ningún exploit técnico

Estas defensas funcionan mejor en partes autenticadas de la aplicación, aunque la tasa de creación de nuevas cuentas o acceso a contenido (por ejemplo para raspar información) puede ser de uso en áreas públicas.

No todas las anteriores necesitan ser monitoreadas por la aplicación, pero hay un problema si ninguna de ellas lo está. Al probar la aplicación web, haciendo el tipo de acciones anteriores, ¿se tomó alguna respuesta contra el tester? Si no, el tester debería reportar que la aplicación parece no tener defensas activas a nivel de aplicación contra uso indebido. Notar que a veces es posible que todas las respuestas a detección de ataque sean silenciosas para el usuario (por ejemplo cambios de logging, monitoreo aumentado, alertas a administradores y proxying de solicitudes), así que la confianza en este hallazgo no puede ser garantizada. En la práctica, muy pocas aplicaciones (o infraestructura relacionada tal como un firewall de aplicaciones web) están detectando estos tipos de uso indebido.

## Casos de Prueba Relacionados

Todos los otros casos de prueba son relevantes.

## Remediación

Las aplicaciones deberían implementar defensas activas para rechazar atacantes y abusadores.

## Referencias

- [Software Assurance](https://www.cisa.gov/uscert/sites/default/files/publications/infosheet_SoftwareAssurance.pdf), US Department Homeland Security
- [IR 7684](https://csrc.nist.gov/publications/detail/nistir/7864/final) Common Misuse Scoring System (CMSS), NIST
- [Common Attack Pattern Enumeration and Classification](https://capec.mitre.org/) (CAPEC), The Mitre Corporation
- [OWASP AppSensor Project](https://owasp.org/www-project-appsensor/)
- Watson C, Coates M, Melton J and Groves G, [Creating Attack-Aware Software Applications with Real-Time Defenses](https://pdfs.semanticscholar.org/0236/5631792fa6c953e82cadb0e7268be35df905.pdf), CrossTalk The Journal of Defense Software Engineering, Vol. 24, No. 5, Sep/Oct 2011
