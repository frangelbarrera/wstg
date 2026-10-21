# Sesiones concurrentes

|ID          |
|------------|
|WSTG-SESS-11|

## Resumen

Las sesiones concurrentes son un aspecto común de las aplicaciones web que permiten múltiples interacciones simultáneas de los usuarios. Este caso de prueba tiene como objetivo evaluar la capacidad de la aplicación para gestionar varias sesiones activas de un mismo usuario. Esta funcionalidad es esencial para gestionar eficazmente las sesiones concurrentes de los usuarios, en particular en áreas sensibles como los paneles de administración que contienen Información de Identificación Personal (PII), las cuentas personales de usuario o las API que dependen de servicios de terceros para enriquecer los datos proporcionados por los usuarios. El objetivo principal es garantizar que las sesiones concurrentes se ajusten a los requisitos de seguridad de la aplicación.

Comprender las necesidades de seguridad de una aplicación es clave para evaluar si habilitar sesiones concurrentes se corresponde con las funcionalidades previstas. Permitir sesiones concurrentes no es intrínsecamente perjudicial y está permitido intencionalmente en muchas aplicaciones. Sin embargo, es fundamental garantizar que la funcionalidad de la aplicación esté debidamente alineada con sus medidas de seguridad en lo que respecta a las sesiones concurrentes. Si se prevén sesiones concurrentes, es fundamental garantizar controles de seguridad adicionales, como la gestión de sesiones activas, la finalización de sesiones y las posibles notificaciones de nuevas sesiones. Por el contrario, si las sesiones concurrentes no están previstas ni planificadas en la aplicación, es fundamental validar los controles existentes para detectar vulnerabilidades en la gestión de sesiones.

Para determinar si las sesiones concurrentes son esenciales, se deben considerar los siguientes factores:

- Comprender la naturaleza de la aplicación, en particular las situaciones en las que los usuarios podrían necesitar acceso simultáneo desde diferentes ubicaciones o dispositivos.
- Identificar las operaciones críticas, como las transacciones financieras que requieren acceso seguro.
- Gestionar datos sensibles, como la Información de Identificación Personal (PII), que indican la necesidad de interacciones seguras.
- Distinguir entre un panel de administración y un panel estándar para el acceso normal de los usuarios.

## Objetivos de prueba

- Evaluar la gestión de sesiones de la aplicación mediante el análisis de cómo gestiona varias sesiones activas para una misma cuenta de usuario.

## Cómo realizar las pruebas

1. **Generar una sesión válida:**
   - Enviar credenciales válidas (nombre de usuario y contraseña) para crear una sesión.
   - Ejemplo de solicitud HTTP:

     ```http
     POST /login HTTP/1.1
     Host: www.example.com
     Content-Length: 32

     username=admin&password=admin123
     ```

   - Ejemplo de respuesta:

     ```http
     HTTP/1.1 200 OK
     Set-Cookie: SESSIONID=0add0d8eyYq3HIUy09hhus; Path=/; Secure
     ```

   - Guardar la cookie de autenticación generada. En algunos casos, la cookie de autenticación generada se sustituye por tokens como los JSON Web Tokens (JWT).

2. **Probar la generación de sesiones activas:**
   - Intentar crear varias cookies de autenticación enviando solicitudes de inicio de sesión (por ejemplo, cien veces).

   Nota: Puede ser útil utilizar el modo de navegación privada o contenedores multicuenta para realizar estas pruebas, ya que permiten disponer de entornos separados para probar la gestión de sesiones sin interferencias de las sesiones existentes o de las cookies almacenadas en el navegador.

3. **Probar la validación de sesiones activas:**
   - Intentar acceder a la aplicación usando el token de sesión inicial (por ejemplo, `SESSIONID=0add0d8eyYq3HIUy09hhus`).
   - Si la autenticación se realiza correctamente con el primer token generado, considerarlo un posible problema que indica una gestión inadecuada de las sesiones.

Además, hay otros casos de prueba que amplían el alcance de la metodología de pruebas para incluir escenarios con varias sesiones originadas desde distintas direcciones IP y ubicaciones. Estos casos ayudan a identificar posibles vulnerabilidades o irregularidades en la gestión de sesiones relacionadas con factores geográficos o de red:

- Probar varias sesiones desde la misma IP.
- Probar varias sesiones desde diferentes IP.
- Probar varias sesiones desde ubicaciones que probablemente el mismo usuario no podría visitar en un período breve (por ejemplo, una sesión creada en un país específico, seguida cinco minutos después por otra sesión iniciada desde otro país).

## Remediación

La aplicación debe supervisar y limitar el número de sesiones activas por cuenta de usuario. Si se supera el número máximo de sesiones permitidas, el sistema debe invalidar las sesiones anteriores para mantener la seguridad. La implementación de soluciones adicionales puede mitigar aún más esta vulnerabilidad:

   1. **Notificación al usuario:** Notificar a los usuarios después de cada inicio de sesión correcto para que estén al tanto de las sesiones activas.
   2. **Página de gestión de sesiones:** Crear una página específica que muestre las sesiones activas y permita finalizarlas, para que los usuarios tengan más control.
   3. **Seguimiento de direcciones IP:** Registrar las direcciones IP de los usuarios que inician sesión en una cuenta y señalar cualquier actividad sospechosa, como varios inicios de sesión desde diferentes ubicaciones.
   4. **Restricciones de direcciones IP:** Permitir a los usuarios especificar direcciones IP o rangos de confianza desde los que pueden acceder a sus cuentas, para mejorar la seguridad restringiendo las sesiones a ubicaciones conocidas y aprobadas.

## Herramientas recomendadas

### Herramientas de proxy de interceptación

- [Zed Attack Proxy](https://www.zaproxy.org)
- [Burp Suite Web Proxy](https://portswigger.net)