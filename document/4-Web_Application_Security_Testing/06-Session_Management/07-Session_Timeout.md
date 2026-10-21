# Tiempo de espera de sesión

|ID          |
|------------|
|WSTG-SESS-07|

## Resumen

En esta fase, los evaluadores verifican que la aplicación cierre automáticamente la sesión de un usuario cuando este ha estado inactivo durante cierto tiempo, asegurándose de que no sea posible «reutilizar» la misma sesión y de que no queden datos sensibles almacenados en la caché del navegador.

Todas las aplicaciones deben implementar un tiempo de espera por inactividad para las sesiones. Este tiempo de espera define cuánto tiempo permanecerá activa una sesión si el usuario no realiza ninguna actividad. Al cumplirse el período de inactividad definido desde la última solicitud HTTP recibida por la aplicación web para un ID de sesión determinado, la sesión se cierra y se invalida. El tiempo de espera más apropiado debe equilibrar la seguridad (un tiempo de espera más corto) y la usabilidad (uno más largo), y depende en gran medida del nivel de sensibilidad de los datos que maneja la aplicación. Por ejemplo, un tiempo de cierre de sesión de 60 minutos puede ser aceptable para un foro público, pero sería demasiado largo en una aplicación de banca en línea (donde se recomienda un tiempo de espera máximo de 15 minutos). En cualquier caso, cualquier aplicación que no aplique un cierre de sesión basado en el tiempo de espera debe considerarse no segura, a menos que tal comportamiento sea necesario por un requisito funcional específico.

El tiempo de espera por inactividad limita las oportunidades que tiene un atacante para adivinar y usar un ID de sesión válido de otro usuario y, en determinadas circunstancias, podría proteger las computadoras públicas frente a la reutilización de sesiones. Sin embargo, si el atacante puede secuestrar una sesión determinada, el tiempo de espera por inactividad no limita sus acciones, ya que puede generar actividad en la sesión periódicamente para mantenerla activa durante más tiempo.

La gestión del tiempo de espera de sesión y su expiración deben aplicarse del lado del servidor. Si se utilizan datos bajo el control del cliente para aplicar el tiempo de espera de sesión —por ejemplo, valores de cookies u otros parámetros del cliente para hacer un seguimiento del tiempo (como el número de minutos transcurridos desde el inicio de sesión)—, un atacante podría manipularlos para prolongar la duración de la sesión. Por lo tanto, la aplicación debe hacer un seguimiento del tiempo de inactividad del lado del servidor y, una vez vencido el tiempo de espera, invalidar automáticamente la sesión del usuario actual y eliminar todos los datos almacenados en el cliente.

Ambas acciones deben implementarse con cuidado para evitar introducir debilidades que un atacante pudiera explotar para obtener acceso no autorizado si el usuario olvidó cerrar sesión en la aplicación. Más específicamente, como en la función de cierre de sesión, es importante asegurarse de que todos los tokens de sesión (por ejemplo, las cookies) se destruyan correctamente o queden inutilizables, y de que se apliquen controles adecuados del lado del servidor para impedir la reutilización de tokens de sesión. Si estas acciones no se llevan a cabo correctamente, un atacante podría reproducir esos tokens de sesión para «resucitar» la sesión de un usuario legítimo e suplantar su identidad (este ataque suele conocerse como «reproducción de cookies»). Por supuesto, un factor atenuante es que el atacante debe poder acceder a esos tokens (que están almacenados en la PC de la víctima), pero en diversos casos esto puede no ser imposible ni particularmente difícil.

El escenario más común para este tipo de ataque es una computadora pública que se utiliza para acceder a información privada (por ejemplo, correo web o una cuenta bancaria en línea). Si el usuario se aleja de la computadora sin cerrar sesión explícitamente y la aplicación no implementa el tiempo de espera de sesión, un atacante podría acceder a la misma cuenta simplemente presionando el botón «atrás» del navegador.

## Objetivos de la prueba

- Validar que existe un tiempo de espera de sesión estricto.

## Cómo probar

### Pruebas de caja negra

Al medir el cierre de sesión por tiempo de espera, puede aplicarse el mismo enfoque descrito en la sección [Funcionalidad de cierre de sesión](06-Logout_Functionality.md).
La metodología de prueba es muy similar. Primero, los evaluadores deben comprobar si existe un tiempo de espera, por ejemplo, iniciando sesión y esperando a que se active el cierre de sesión por tiempo de espera. Al igual que con la función de cierre de sesión, una vez transcurrido el tiempo de espera, todos los tokens de sesión deben destruirse o quedar inutilizables.

Luego, si el tiempo de espera está configurado, los evaluadores deben determinar si lo aplica el cliente, el servidor o ambos. Si la cookie de sesión no es persistente (o, en general, no almacena ningún dato relacionado con el tiempo), los evaluadores pueden suponer que el tiempo de espera lo aplica el servidor. Si la cookie de sesión contiene datos relacionados con el tiempo (por ejemplo, la hora de inicio de sesión, la hora del último acceso o la fecha de expiración de una cookie persistente), es posible que el cliente intervenga en la aplicación del tiempo de espera. En ese caso, los evaluadores pueden intentar modificar la cookie (si no está protegida criptográficamente) y observar qué sucede con la sesión. Por ejemplo, pueden establecer la fecha de expiración de la cookie en un momento muy lejano y comprobar si se puede prolongar la sesión.

Como regla general, todo debe comprobarse del lado del servidor y no debe ser posible volver a acceder a la aplicación restableciendo las cookies de sesión a valores anteriores.

### Pruebas de caja gris

El evaluador debe comprobar que:

- La función de cierre de sesión destruya efectivamente todos los tokens de sesión o, al menos, los haga inutilizables.
- El servidor compruebe correctamente el estado de la sesión e impida que un atacante reproduzca identificadores de sesión destruidos previamente.
- Se aplique un tiempo de espera y que el servidor lo aplique correctamente. Si el servidor utiliza un tiempo de expiración leído de un token de sesión enviado por el cliente (aunque no es aconsejable), el token debe estar protegido criptográficamente contra la manipulación.

Lo más importante es que la aplicación invalide la sesión del lado del servidor. En general, esto significa que el código debe invocar los métodos apropiados, por ejemplo, `HttpSession.invalidate()` en Java y `Session.abandon()` en .NET. Es aconsejable borrar las cookies del navegador, pero no es estrictamente necesario, ya que, si la sesión se invalida correctamente en el servidor, tener la cookie en el navegador no ayudará a un atacante.

## Referencias

### Recursos de OWASP

- [Hoja de referencia de gestión de sesiones](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)