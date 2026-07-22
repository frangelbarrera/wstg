# Pruebas de Sesiones Concurrentes

|ID          |
|------------|
|WSTG-SESS-11|

## Resumen

Las sesiones concurrentes son un aspecto común de las aplicaciones web que habilitan múltiples interacciones simultáneas de usuario. Este caso de prueba tiene como objetivo evaluar la capacidad de la aplicación para manejar múltiples sesiones activas para un solo usuario. Esta funcionalidad es esencial para gestionar efectivamente sesiones de usuario concurrentes, particularmente en áreas sensibles tales como paneles de administración que contienen Información de Identificación Personal (PII), cuentas de usuario personales, o APIs que dependen de servicios de terceros para enriquecer datos proporcionados por el usuario. El objetivo primario es asegurar que las sesiones concurrentes se alineen con los requisitos de seguridad de la aplicación.

Entender las necesidades de seguridad en una aplicación es clave para evaluar si habilitar sesiones concurrentes corresponde con las características previstas. Permitir sesiones concurrentes no es inherentemente perjudicial y está intencionalmente permitido en muchas aplicaciones. Sin embargo, es crucial asegurar que la funcionalidad de la aplicación esté efectivamente alineada con sus medidas de seguridad respecto a sesiones concurrentes. Si las sesiones concurrentes están previstas, es vital asegurar controles de seguridad adicionales, tales como gestionar sesiones activas, terminar sesiones, y notificaciones potenciales de nuevas sesiones. A la inversa, si las sesiones concurrentes no están previstas o planificadas dentro de la aplicación, es crucial validar verificaciones existentes para vulnerabilidades de gestión de sesión.

Para reconocer que las sesiones concurrentes son esenciales, deberías considerar los siguientes factores:

- Entender la naturaleza de la aplicación, particularmente situaciones donde los usuarios podrían requerir acceso simultáneo desde diferentes ubicaciones o dispositivos.
- Identificar operaciones críticas, tales como transacciones financieras que requieren acceso seguro.
- Manejar datos sensibles como Información de Identificación Personal (PII), indicando la necesidad de interacciones seguras.
- Distinguir entre un panel de gestión y un dashboard de usuario estándar para acceso normal de usuario.

## Objetivos de Prueba

- Evaluar la gestión de sesión de la aplicación evaluando el manejo de múltiples sesiones activas para una sola cuenta de usuario.

## Cómo Probar

1. **Generar Sesión Válida:**
   - Enviar credenciales válidas (nombre de usuario y contraseña) para crear una sesión.
   - Ejemplo de Solicitud HTTP:

     ```http
     POST /login HTTP/1.1
     Host: www.example.com
     Content-Length: 32

     username=admin&password=admin123
     ```

   - Ejemplo de Respuesta:

     ```http
     HTTP/1.1 200 OK
     Set-Cookie: SESSIONID=0add0d8eyYq3HIUy09hhus; Path=/; Secure
     ```

   - Almacenar la cookie de autenticación generada. En algunos casos, la cookie de autenticación generada se reemplaza por tokens tales como JSON Web Tokens (JWT).

2. **Probar Generación de Sesiones Activas:**
   - Intentar crear múltiples cookies de autenticación enviando solicitudes de login (por ejemplo, cien veces).

   Nota: Utilizar modo de navegación privada o contenedores multi-cuenta podría ser beneficioso para conducir estas pruebas, ya que pueden proporcionar entornos separados para probar la gestión de sesión sin interferencia de sesiones existentes o cookies almacenadas en el navegador.

3. **Probar Validación de Sesiones Activas:**
   - Intentar acceder a la aplicación usando el token de sesión inicial (por ejemplo, `SESSIONID=0add0d8eyYq3HIUy09hhus`).
   - Si ocurre autenticación exitosa con el primer token generado, considerarlo un problema potencial indicando gestión de sesión inadecuada.

Además, hay casos de prueba adicionales que extienden el alcance de la metodología de pruebas para incluir escenarios que involucran múltiples sesiones originadas de varias IPs y ubicaciones. Estos casos de prueba ayudan a identificar vulnerabilidades potenciales o irregularidades en el manejo de sesión relacionadas con factores geográficos o basados en red:

- Probar múltiples sesiones desde la misma IP.
- Probar múltiples sesiones desde diferentes IPs.
- Probar múltiples sesiones desde ubicaciones que es improbable o imposible que sean visitadas por el mismo usuario en un período corto de tiempo (por ejemplo, una sesión creada en un país específico, seguida por otra sesión generada cinco minutos después desde un país diferente).

## Remediación

La aplicación debería monitorear y limitar el número de sesiones activas por cuenta de usuario. Si se superan las sesiones máximas permitidas, el sistema debe invalidar sesiones previas para mantener seguridad. Implementar soluciones adicionales puede mitigar aún más esta vulnerabilidad:

   1. **Notificación al Usuario:** Notificar a los usuarios después de cada login exitoso para aumentar la conciencia de sesiones activas.
   2. **Página de Gestión de Sesión:** Crear una página dedicada para mostrar y permitir la terminación de sesiones activas para mayor control del usuario.
   3. **Seguimiento de Dirección IP:** Rastrear las direcciones IP de usuarios que inician sesión en una cuenta y marcar cualquier actividad sospechosa, tal como múltiples logins desde diferentes ubicaciones.
   4. **Restricciones de Dirección IP:** Permitir a los usuarios especificar direcciones IP o rangos confiables desde los cuales pueden acceder a sus cuentas, mejorando la seguridad restringiendo sesiones a ubicaciones conocidas y aprobadas.

## Herramientas Recomendadas

### Herramientas de Proxy de Intercepción

- [Zed Attack Proxy](https://www.zaproxy.org)
- [Burp Suite Web Proxy](https://portswigger.net)
