# Pruebas de contraseñas recordadas vulnerables

|ID          |
|------------|
|WSTG-ATHN-05|

## Summary

Las credenciales son la tecnología de autenticación más ampliamente utilizada. Debido a tal amplio uso de pares usuario-contraseña, los usuarios ya no pueden manejar correctamente sus credenciales en la multitud de aplicaciones utilizadas.

Para ayudar a los usuarios con sus credenciales, surgieron múltiples tecnologías:

- Las aplicaciones proporcionan una funcionalidad de "recordarme" que permite al usuario permanecer autenticado durante largos períodos de tiempo, sin pedirle nuevamente sus credenciales.
- Administradores de contraseñas - incluyendo administradores de contraseñas del navegador - que permiten al usuario almacenar sus credenciales de manera segura y posteriormente inyectarlas en formularios de usuario sin intervención del usuario.

## Test Objectives

- Validar que la sesión generada se gestione de manera segura y no ponga en peligro las credenciales del usuario.

## How to Test

Como estos métodos proporcionan una mejor experiencia de usuario y permiten al usuario olvidar todo sobre sus credenciales, aumentan el área de superficie de ataque. Algunas aplicaciones:

- Almacenan las credenciales de manera codificada en los mecanismos de almacenamiento del navegador, lo que puede verificarse siguiendo el [escenario de pruebas de almacenamiento web](../11-Client-side_Testing/12-Testing_Browser_Storage.md) y pasando por los escenarios de [análisis de sesión](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md#session-analysis). Las credenciales no deberían almacenarse de ninguna manera en la aplicación del lado cliente, y deberían sustituirse por tokens generados del lado servidor.
- Inyectan automáticamente las credenciales del usuario que pueden ser abusadas por:
    - Ataques de [ClickJacking](../11-Client-side_Testing/09-Testing_for_Clickjacking.md).
    - Ataques de [CSRF](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md).
- Los tokens deberían analizarse en términos de vida útil del token, donde algunos tokens nunca expiran y ponen a los usuarios en peligro si esos tokens alguna vez son robados. Asegúrese de seguir el escenario de pruebas de [tiempo de espera de sesión](../06-Session_Management_Testing/07-Testing_Session_Timeout.md).

## Remediation

- Seguir las buenas prácticas de [gestión de sesión](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
- Asegurarse de que ninguna credencial se almacene en texto claro o sea fácilmente recuperable en formas codificadas o encriptadas en mecanismos de almacenamiento del navegador; deberían almacenarse del lado servidor y seguir buenas prácticas de [almacenamiento de contraseñas](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).