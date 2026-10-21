# Funcionalidad de cierre de sesión

|ID          |
|------------|
|WSTG-SESS-06|

## Resumen

La terminación de la sesión es una parte importante del ciclo de vida de la sesión. Reducir al mínimo la duración de los tokens de sesión disminuye la probabilidad de que un ataque de secuestro de sesión tenga éxito. Esto puede considerarse un control para prevenir otros ataques, como el scripting entre sitios (XSS) y la falsificación de solicitudes entre sitios (CSRF). Se sabe que estos ataques dependen de que el usuario tenga una sesión autenticada activa. No contar con una terminación segura de la sesión solo aumenta la superficie de ataque para cualquiera de estos ataques.

Una terminación segura de la sesión requiere al menos los siguientes componentes:

- Disponibilidad de controles de interfaz de usuario que permitan al usuario cerrar sesión manualmente.
- Terminación de la sesión después de un período determinado de inactividad (tiempo de espera de sesión).
- Invalidación adecuada del estado de la sesión en el lado del servidor.

Existen varios problemas que pueden impedir la terminación efectiva de una sesión. En una aplicación web segura ideal, el usuario debería poder terminar la sesión en cualquier momento desde la interfaz de usuario. Las funciones de cierre de sesión poco claras o ambiguas pueden hacer que los usuarios desconfíen de ellas.

Otro error común en la terminación de la sesión es que el token de sesión del lado del cliente se establece en un nuevo valor, mientras que el estado del lado del servidor permanece activo y puede reutilizarse al restablecer la cookie de sesión a su valor anterior. A veces, solo se muestra un mensaje de confirmación al usuario, sin realizar ninguna otra acción. Esto debería evitarse.

Algunos marcos de aplicaciones web dependen únicamente de la cookie de sesión para identificar al usuario que inició sesión. El ID del usuario está incrustado en el valor de la cookie (cifrada). El servidor de aplicaciones no realiza ningún seguimiento de la sesión en el lado del servidor. Al cerrar sesión, la cookie de sesión se elimina del navegador. Sin embargo, como la aplicación no realiza ningún seguimiento, no sabe si una sesión se ha cerrado o no. Por lo tanto, al reutilizar una cookie de sesión es posible acceder a la sesión autenticada. Un ejemplo conocido de esto es la funcionalidad de autenticación de formularios de ASP.NET.

A los usuarios de navegadores web a menudo no les importa que una aplicación siga abierta y simplemente cierran el navegador o una pestaña. Una aplicación web debería tener en cuenta este comportamiento y terminar la sesión automáticamente en el lado del servidor después de un período definido.

El uso de un sistema de inicio de sesión único (SSO) en lugar de un esquema de autenticación específico de la aplicación suele provocar la coexistencia de varias sesiones que deben terminarse por separado. Por ejemplo, terminar la sesión específica de la aplicación no termina la sesión en el sistema SSO. Volver al portal SSO ofrece al usuario la posibilidad de volver a iniciar sesión en la aplicación en la que acaba de cerrar sesión. Por otro lado, una función de cierre de sesión en un sistema SSO no necesariamente termina la sesión en las aplicaciones conectadas.

## Objetivos de la prueba

- Evaluar la interfaz de usuario de cierre de sesión.
- Analizar el tiempo de espera de sesión y comprobar si la sesión se termina correctamente después del cierre de sesión.

## Cómo probar

### Interfaz de usuario de cierre de sesión

Verifique la apariencia y visibilidad de la función de cierre de sesión en la interfaz de usuario. Para ello, revise cada página desde la perspectiva de un usuario que desea cerrar sesión en la aplicación web.

> Algunas propiedades indican que una interfaz de usuario de cierre de sesión es adecuada:
>
> - Hay un botón para cerrar sesión en todas las páginas de la aplicación web.
> - El usuario que desea cerrar sesión en la aplicación web debería poder identificar rápidamente el botón de cierre de sesión.
> - Después de cargar una página, el botón de cierre de sesión debería estar visible sin tener que desplazarse.
> - Idealmente, el botón de cierre de sesión se encuentra en un área de la página fija en la ventana gráfica del navegador y no se ve afectada por el desplazamiento del contenido.

### Terminación de sesión en el lado del servidor

Primero, almacene los valores de las cookies que se utilizan para identificar una sesión. Invoque la función de cierre de sesión y observe el comportamiento de la aplicación, especialmente en lo que respecta a las cookies de sesión. Intente navegar a una página que solo sea visible en una sesión autenticada, por ejemplo, mediante el botón de retroceso del navegador. Si se muestra una versión en caché de la página, utilice el botón de recarga para actualizarla desde el servidor. Si la función de cierre de sesión hace que las cookies de sesión adopten un nuevo valor, restablezca los valores anteriores de las cookies de sesión y recargue una página del área autenticada de la aplicación. Si estas pruebas no revelan vulnerabilidades en una página determinada, pruebe al menos algunas otras páginas de la aplicación consideradas críticas para la seguridad, para asegurarse de que esas áreas de la aplicación reconozcan correctamente la terminación de la sesión.

> Durante las pruebas, no deberían mostrarse en las páginas examinadas datos que solo deberían estar visibles para usuarios autenticados. Lo ideal es que, al acceder a áreas autenticadas después de terminar la sesión, la aplicación redirija a un área pública o a un formulario de inicio de sesión. No debería ser necesario para la seguridad de la aplicación, pero, por lo general, se considera una buena práctica establecer nuevos valores para las cookies de sesión después de cerrar sesión.

### Tiempo de espera de sesión

Intente determinar el tiempo de espera de sesión enviando solicitudes a una página del área autenticada de la aplicación web con retrasos cada vez mayores. Si se produce el cierre de sesión, el retraso utilizado coincide aproximadamente con el valor del tiempo de espera de sesión.

> Se esperan los mismos resultados que en las pruebas de terminación de sesión en el lado del servidor descritas anteriormente cuando el cierre de sesión se produce por un tiempo de espera por inactividad.
>
> El valor adecuado del tiempo de espera de sesión depende del propósito de la aplicación y debería equilibrar la seguridad y la usabilidad. En las aplicaciones bancarias, no tiene sentido mantener una sesión inactiva durante más de 15 minutos. Por otro lado, un tiempo de espera breve en un wiki o foro podría molestar a los usuarios que están escribiendo artículos largos y reciben solicitudes de inicio de sesión innecesarias. En esos casos, pueden ser aceptables tiempos de espera de una hora o más.

### Terminación de sesión en entornos de inicio de sesión único (cierre de sesión único)

Cierre sesión en la aplicación sometida a prueba. Verifique si hay un portal central o un directorio de aplicaciones que permita al usuario volver a iniciar sesión en la aplicación sin autenticarse. Compruebe si la aplicación solicita al usuario que se autentique cuando se solicita la URL de un punto de entrada a la aplicación. Mientras la sesión esté iniciada en la aplicación sometida a prueba, cierre sesión en el sistema SSO. Luego, intente acceder a un área autenticada de la aplicación sometida a prueba.

> Se espera que invocar una función de cierre de sesión en una aplicación web conectada a un sistema SSO, o en el propio sistema SSO, termine globalmente todas las sesiones. Para acceder a la aplicación después de cerrar sesión en el sistema SSO y en la aplicación conectada, debería ser necesario que el usuario se autentique.

### Reutilización de sesiones entre dispositivos y dependencia de un único valor en SSO

> Aunque las pruebas tradicionales del cierre de sesión de SSO se centran en los flujos de reautenticación dentro del mismo contexto del navegador, los evaluadores también deben comprobar si un artefacto de autenticación (por ejemplo, una cookie, un encabezado o un token bearer) puede reproducirse en distintos dispositivos.

En algunas implementaciones de inicio de sesión único (SSO), la autenticación en varias aplicaciones depende de un único artefacto de autenticación emitido tras iniciar sesión correctamente (por ejemplo, después de verificar la contraseña y el MFA).

> Por ejemplo, una cookie puede configurarse con Secure y HttpOnly, pero su seguridad depende por completo de los controles de validación y revocación del lado del servidor. Si el ecosistema SSO utiliza una única cookie como única prueba de autenticación, la vulneración de esa cookie puede otorgar acceso inmediato a varias aplicaciones.

Los evaluadores deberían comprobar si:

- Se requiere un único artefacto de autenticación para mantener el estado autenticado.
- El artefacto de autenticación puede reutilizarse en distintos navegadores o dispositivos.
- El cierre de sesión invalida correctamente el artefacto de autenticación en el lado del servidor.
- Todas las aplicaciones que confían en el artefacto de autenticación de SSO lo rechazan después del cierre de sesión.

Procedimiento de prueba:

- Paso 1 - Autenticarse y capturar todos los artefactos de autenticación emitidos

1. Inicie sesión en el Dispositivo 1 con credenciales válidas y complete el MFA.
2. Intercepte la respuesta de autenticación.
3. Registre todos los artefactos de autenticación (por ejemplo, una cookie, un encabezado o un token bearer) emitidos por la aplicación o el dominio SSO.

Por ejemplo: `auth-cookie=XYZ123`, `session_meta=ABC456`, `tracking_id=DEF789`...

- Paso 2 - Identificar los artefactos críticos para la autenticación

El propósito de este paso es determinar si la implementación de SSO depende de un único artefacto de autenticación (por ejemplo, una cookie, un encabezado o un token bearer).

1. Envíe solicitudes a endpoints protegidos.
2. Elimine cada artefacto de autenticación individualmente y vuelva a enviar la solicitud.
3. Observe cuáles son necesarios para mantener una respuesta 200 OK.

Si solo se necesita un artefacto de autenticación (por ejemplo, `auth-cookie`) para mantener la autenticación, esto indica que el límite de confianza de SSO podría depender únicamente de ese valor.

> Nota de seguridad:
> Cuando un único artefacto de autenticación representa todo el estado autenticado, se convierte en un objetivo de alto valor. Un atacante que obtenga ese valor único podría no necesitar metadatos de sesión adicionales ni información del contexto del dispositivo para obtener acceso.

- Paso 3 - Inyección de sesión entre dispositivos

1. Copie el valor del artefacto crítico para la autenticación (por ejemplo, `auth-cookie=XYZ123`).
2. Abra otro navegador o dispositivo (Dispositivo 2).
3. Inserte manualmente el artefacto único copiado.
4. Intente acceder a endpoints protegidos en:
    - La aplicación original
    - Otras aplicaciones que participan en el mismo ecosistema SSO

Si el Dispositivo 2 obtiene acceso autenticado sin introducir credenciales ni completar el MFA, el artefacto se puede reutilizar en distintos contextos.

- Paso 4 - Validación del cierre de sesión entre dispositivos

1. En el Dispositivo 1, cierre sesión.
2. En el Dispositivo 2, actualice la página o acceda a recursos protegidos utilizando el mismo valor inyectado del artefacto de autenticación.

Resultado esperado:

- El cierre de sesión debe invalidar el artefacto de autenticación (por ejemplo, una cookie o un token bearer) en el lado del servidor.
- Deben terminarse todas las sesiones que utilicen ese artefacto.
- El acceso no debe seguir siendo válido hasta que expire el artefacto de autenticación.
- Todas las aplicaciones que confían en el artefacto de autenticación de SSO deben rechazarlo.

Si una implementación de SSO:

- Depende principalmente de un único artefacto de autenticación
- Permite reutilizar ese artefacto en varios dispositivos
- No lo revoca en el lado del servidor al cerrar sesión

entonces:

- El robo de ese artefacto de autenticación de valor único puede comprometer de inmediato las sesiones activas.
- Las protecciones de MFA se eluden efectivamente después del inicio de sesión inicial.
- Es posible que el cierre de sesión no termine globalmente las sesiones.
- El acceso puede persistir durante toda la vida útil del artefacto (por ejemplo, `auth-cookie=XYZ123; max-age=28800`, 8 horas).

Las implementaciones deberían evitar arquitecturas en las que un artefacto de autenticación de valor único represente todo el estado autenticado sin controles de validación adicionales.

## Herramientas

- [Burp Suite - Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)

## Referencias

- [La cookie dorada: cuando fallan los límites de sesión de SSO](https://allaboutinfosec.blogspot.com/2026/03/the-golden-cookie-when-sso-session.html)

### Documentos técnicos

- [Ataques de reproducción de cookies en ASP.NET al usar la autenticación de formularios](https://www.vanstechelman.eu/content/cookie-replay-attacks-in-aspnet-when-using-forms-authentication)