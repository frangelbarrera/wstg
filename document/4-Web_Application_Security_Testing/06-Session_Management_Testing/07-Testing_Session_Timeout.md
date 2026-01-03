# Prueba de tiempo de espera de sesión

|ID          |
|------------|
|WSTG-SESS-07|

## Resumen

En esta fase, los evaluadores verifican que la aplicación cierre automáticamente la sesión de un usuario cuando este ha estado inactivo durante un cierto período de tiempo, asegurando que no sea posible "reutilizar" la misma sesión y que no queden datos sensibles almacenados en la caché del navegador.

Todas las aplicaciones deben implementar un tiempo de espera por inactividad para las sesiones. Este tiempo de espera define la cantidad de tiempo que una sesión permanecerá activa en caso de que no haya actividad por parte del usuario, cerrando e invalidando la sesión tras el período de inactividad definido desde la última solicitud HTTP recibida por la aplicación web para un ID de sesión determinado. El tiempo de espera más apropiado debe ser un equilibrio entre seguridad (tiempo de espera más corto) y usabilidad (tiempo de espera más largo) y depende en gran medida del nivel de sensibilidad de los datos manejados por la aplicación. Por ejemplo, un tiempo de cierre de sesión de 60 minutos para un foro público puede ser aceptable, pero tal tiempo sería demasiado largo en una aplicación de banca en línea (donde se recomienda un tiempo de espera máximo de 15 minutos). En cualquier caso, cualquier aplicación que no aplique un cierre de sesión basado en tiempo de espera debe considerarse no segura, a menos que tal comportamiento sea requerido por un requisito funcional específico.

El tiempo de espera por inactividad limita las oportunidades que tiene un atacante para adivinar y usar un ID de sesión válido de otro usuario, y bajo ciertas circunstancias podría proteger las computadoras públicas de la reutilización de sesiones. Sin embargo, si el atacante es capaz de secuestrar una sesión determinada, el tiempo de espera por inactividad no limita las acciones del atacante, ya que puede generar actividad en la sesión periódicamente para mantener la sesión activa durante períodos más largos.

La gestión del tiempo de espera de sesión y la expiración deben aplicarse del lado del servidor. Si se utilizan algunos datos bajo el control del cliente para aplicar el tiempo de espera de sesión, por ejemplo utilizando valores de cookies u otros parámetros del cliente para rastrear referencias de tiempo (por ejemplo, número de minutos desde el tiempo de inicio de sesión), un atacante podría manipular estos para extender la duración de la sesión. Por lo tanto, la aplicación debe rastrear el tiempo de inactividad del lado del servidor y, después de que expire el tiempo de espera, invalidar automáticamente la sesión del usuario actual y eliminar todos los datos almacenados en el cliente.

Ambas acciones deben implementarse con cuidado, para evitar introducir debilidades que podrían ser explotadas por un atacante para obtener acceso no autorizado si el usuario olvidó cerrar sesión en la aplicación. Más específicamente, como en la función de cierre de sesión, es importante asegurar que todos los tokens de sesión (por ejemplo, cookies) sean destruidos o inutilizados correctamente, y que se apliquen controles apropiados del lado del servidor para prevenir la reutilización de tokens de sesión. Si tales acciones no se llevan a cabo correctamente, un atacante podría reproducir estos tokens de sesión para "resucitar" la sesión de un usuario legítimo e impersonarlo (este ataque se conoce generalmente como 'reproducción de cookies'). Por supuesto, un factor mitigante es que el atacante necesita poder acceder a esos tokens (que están almacenados en la PC de la víctima), pero en una variedad de casos, esto puede no ser imposible o particularmente difícil.

El escenario más común para este tipo de ataque es una computadora pública que se utiliza para acceder a información privada (por ejemplo, correo web, cuenta bancaria en línea). Si el usuario se aleja de la computadora sin cerrar sesión explícitamente y el tiempo de espera de sesión no está implementado en la aplicación, entonces un atacante podría acceder a la misma cuenta simplemente presionando el botón "atrás" del navegador.

## Objetivos de la prueba

- Validar que existe un tiempo de espera de sesión duro.

## Cómo probar

### Pruebas de caja negra

Se puede aplicar el mismo enfoque visto en la sección [Prueba de funcionalidad de cierre de sesión](06-Testing_for_Logout_Functionality-es.md) al medir el cierre de sesión por tiempo de espera.
La metodología de prueba es muy similar. Primero, los evaluadores deben verificar si existe un tiempo de espera, por ejemplo, iniciando sesión y esperando a que se active el cierre de sesión por tiempo de espera. Como en la función de cierre de sesión, después de que pase el tiempo de espera, todos los tokens de sesión deben ser destruidos o inutilizados.

Luego, si el tiempo de espera está configurado, los evaluadores necesitan entender si el tiempo de espera se aplica por el cliente o por el servidor (o ambos). Si la cookie de sesión no es persistente (o, más en general, la cookie de sesión no almacena ningún dato sobre el tiempo), los evaluadores pueden asumir que el tiempo de espera se aplica por el servidor. Si la cookie de sesión contiene algunos datos relacionados con el tiempo (por ejemplo, tiempo de inicio de sesión, o último tiempo de acceso, o fecha de expiración para una cookie persistente), entonces es posible que el cliente esté involucrado en la aplicación del tiempo de espera. En este caso, los evaluadores podrían intentar modificar la cookie (si no está protegida criptográficamente) y ver qué sucede con la sesión. Por ejemplo, los evaluadores pueden establecer la fecha de expiración de la cookie en el futuro lejano y ver si la sesión puede prolongarse.

Como regla general, todo debe verificarse del lado del servidor y no debe ser posible, restableciendo las cookies de sesión a valores anteriores, acceder nuevamente a la aplicación.

### Pruebas de caja gris

El evaluador necesita verificar que:

- La función de cierre de sesión destruye efectivamente todos los tokens de sesión, o al menos los hace inutilizables,
- El servidor realiza verificaciones apropiadas sobre el estado de la sesión, impidiendo que un atacante reproduzca identificadores de sesión previamente destruidos
- Se aplica un tiempo de espera y se aplica correctamente por el servidor. Si el servidor utiliza un tiempo de expiración que se lee de un token de sesión enviado por el cliente (pero esto no es aconsejable), entonces el token debe estar protegido criptográficamente contra manipulaciones.

Tenga en cuenta que lo más importante es que la aplicación invalide la sesión del lado del servidor. Generalmente esto significa que el código debe invocar los métodos apropiados, por ejemplo `HttpSession.invalidate()` en Java y `Session.abandon()` en .NET. Limpiar las cookies del navegador es aconsejable, pero no estrictamente necesario, ya que si la sesión se invalida correctamente en el servidor, tener la cookie en el navegador no ayudará a un atacante.

## Referencias

### Recursos de OWASP

- [Hoja de referencia de gestión de sesiones](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)