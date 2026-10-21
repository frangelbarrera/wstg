# Contraseña recordada vulnerable

|ID          |
|------------|
|WSTG-ATHN-05|

## Resumen

Las credenciales son la tecnología de autenticación más utilizada. Debido al uso generalizado de pares de nombre de usuario y contraseña, los usuarios ya no pueden gestionar correctamente sus credenciales en la multitud de aplicaciones que utilizan.

Para ayudar a los usuarios a gestionar sus credenciales, surgieron varias tecnologías:

- Las aplicaciones ofrecen una funcionalidad de *recordarme* que permite al usuario permanecer autenticado durante largos períodos de tiempo, sin volver a pedirle sus credenciales.
- Los administradores de contraseñas —incluidos los administradores de contraseñas de los navegadores— permiten al usuario almacenar sus credenciales de forma segura e introducirlas posteriormente en formularios de usuario sin ninguna intervención.

## Objetivos de las pruebas

- Validar que la sesión generada se gestione de forma segura y no ponga en peligro las credenciales del usuario.

## Cómo probar

Como estos métodos mejoran la experiencia del usuario y le permiten olvidarse por completo de sus credenciales, aumentan la superficie de ataque. Algunas aplicaciones:

- Almacenan las credenciales codificadas en los mecanismos de almacenamiento del navegador, lo que puede verificarse siguiendo el [escenario de pruebas de almacenamiento web](../11-Client-side/12-Browser_Storage.md) y realizando los escenarios de [análisis de sesión](../06-Session_Management/01-Session_Management_Schema.md#session-analysis). Las credenciales no deberían almacenarse de ninguna manera en la aplicación del lado del cliente; deberían sustituirse por tokens generados en el servidor.
- Introducen automáticamente las credenciales del usuario, lo que puede aprovecharse mediante:
    - Ataques de [ClickJacking](../11-Client-side/09-Clickjacking.md).
    - Ataques de [CSRF](../06-Session_Management/05-Cross_Site_Request_Forgery.md).
- Los tokens deberían analizarse en cuanto a su duración, ya que algunos nunca expiran y ponen en peligro a los usuarios si llegan a ser robados. Asegúrese de seguir el escenario de pruebas de [tiempo de espera de sesión](../06-Session_Management/07-Session_Timeout.md).

## Remediación

- Siga las buenas prácticas de [gestión de sesiones](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
- Asegúrese de que las credenciales no se almacenen en texto sin cifrar ni en formas codificadas o cifradas que puedan recuperarse fácilmente desde los mecanismos de almacenamiento del navegador; deberían almacenarse en el servidor y seguir buenas prácticas de [almacenamiento de contraseñas](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).