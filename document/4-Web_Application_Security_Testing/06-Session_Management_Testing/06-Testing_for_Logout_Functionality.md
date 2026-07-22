# Pruebas de funcionalidad de cierre de sesión

|ID          |
|------------|
|WSTG-SESS-06|

## Resumen

La terminación de la sesión es una parte importante del ciclo de vida de la sesión. Reducir al mínimo la duración de los tokens de sesión disminuye la probabilidad de un ataque exitoso de secuestro de sesión. Esto puede verse como un control contra la prevención de otros ataques como Scripting entre sitios (XSS) y Falsificación de solicitud entre sitios (CSRF). Se sabe que estos ataques dependen de que un usuario tenga una sesión autenticada presente. No tener una terminación segura de la sesión solo aumenta la superficie de ataque para cualquiera de estos ataques.

Una terminación segura de la sesión requiere al menos los siguientes componentes:

- Disponibilidad de controles de interfaz de usuario que permitan al usuario cerrar sesión manualmente.
- Terminación de la sesión después de un período determinado de inactividad (tiempo de espera de sesión).
- Invalidación adecuada del estado de la sesión del lado del servidor.

Existen múltiples problemas que pueden impedir la terminación efectiva de una sesión. Para la aplicación web idealmente segura, un usuario debería poder terminar en cualquier momento a través de la interfaz de usuario. Cada página debería contener un botón de cierre de sesión en un lugar donde sea directamente visible. Funciones de cierre de sesión poco claras o ambiguas podrían hacer que el usuario no confíe en tal funcionalidad.

Otro error común en la terminación de la sesión es que el token de sesión del lado del cliente se establece en un nuevo valor mientras que el estado del lado del servidor permanece activo y puede reutilizarse configurando la cookie de sesión de vuelta al valor anterior. A veces solo se muestra un mensaje de confirmación al usuario sin realizar ninguna acción adicional. Esto debería evitarse.

Algunos marcos de aplicaciones web dependen únicamente de la cookie de sesión para identificar al usuario conectado. El ID del usuario está incrustado en el valor de la cookie (cifrada). El servidor de aplicaciones no realiza ningún seguimiento del lado del servidor de la sesión. Al cerrar sesión, la cookie de sesión se elimina del navegador. Sin embargo, dado que la aplicación no realiza ningún seguimiento, no sabe si una sesión está cerrada o no. Por lo tanto, reutilizando una cookie de sesión es posible obtener acceso a la sesión autenticada. Un ejemplo bien conocido de esto es la funcionalidad de autenticación de formularios en ASP.NET.

Los usuarios de navegadores web a menudo no se preocupan de que una aplicación siga abierta y simplemente cierran el navegador o una pestaña. Una aplicación web debería ser consciente de este comportamiento y terminar la sesión automáticamente en el lado del servidor después de una cantidad definida de tiempo.

El uso de un sistema de inicio de sesión único (SSO) en lugar de un esquema de autenticación específico de la aplicación a menudo causa la coexistencia de múltiples sesiones que deben terminarse por separado. Por ejemplo, la terminación de la sesión específica de la aplicación no termina la sesión en el sistema SSO. Navegando de vuelta al portal SSO ofrece al usuario la posibilidad de volver a iniciar sesión en la aplicación donde se realizó el cierre de sesión justo antes. Por otro lado, una función de cierre de sesión en un sistema SSO no necesariamente causa la terminación de la sesión en aplicaciones conectadas.

## Objetivos de la prueba

- Evaluar la interfaz de usuario de cierre de sesión.
- Analizar el tiempo de espera de sesión y si la sesión se elimina correctamente después del cierre de sesión.

## Cómo probar

### Pruebas de interfaz de usuario de cierre de sesión

Verifique la apariencia y visibilidad de la funcionalidad de cierre de sesión en la interfaz de usuario. Para este propósito, vea cada página desde la perspectiva de un usuario que tiene la intención de cerrar sesión de la aplicación web.

> Hay algunas propiedades que indican una buena interfaz de usuario de cierre de sesión:
>
> - Un botón de cierre de sesión está presente en todas las páginas de la aplicación web.
> - El botón de cierre de sesión debería identificarse rápidamente por un usuario que quiere cerrar sesión de la aplicación web.
> - Después de cargar una página, el botón de cierre de sesión debería ser visible sin desplazarse.
> - Idealmente, el botón de cierre de sesión se coloca en un área de la página que está fija en la ventana gráfica del navegador y no afectada por el desplazamiento del contenido.

### Pruebas de terminación de sesión del lado del servidor

Primero, almacene los valores de las cookies que se usan para identificar una sesión. Invoque la función de cierre de sesión y observe el comportamiento de la aplicación, especialmente con respecto a las cookies de sesión. Intente navegar a una página que solo sea visible en una sesión autenticada, por ejemplo, mediante el uso del botón de retroceso del navegador. Si se muestra una versión en caché de la página, use el botón de recarga para refrescar la página desde el servidor. Si la función de cierre de sesión causa que las cookies de sesión se establezcan en un nuevo valor, restaure el valor anterior de las cookies de sesión y recargue una página del área autenticada de la aplicación. Si estas pruebas no muestran vulnerabilidades en una página particular, intente al menos algunas páginas adicionales de la aplicación que se consideren críticas para la seguridad, para asegurar que la terminación de la sesión sea reconocida correctamente por estas áreas de la aplicación.

> Ningún dato que debería ser visible solo por usuarios autenticados debería ser visible en las páginas examinadas mientras se realizan las pruebas. Idealmente, la aplicación redirige a un área pública o un formulario de inicio de sesión al acceder a áreas autenticadas después de la terminación de la sesión. No debería ser necesario para la seguridad de la aplicación, pero establecer cookies de sesión en nuevos valores después del cierre de sesión se considera generalmente como una buena práctica.

### Pruebas de tiempo de espera de sesión

Intente determinar un tiempo de espera de sesión realizando solicitudes a una página en el área autenticada de la aplicación web con retrasos crecientes. Si aparece el comportamiento de cierre de sesión, el retraso usado coincide aproximadamente con el valor de tiempo de espera de sesión.

> Se esperan los mismos resultados que para las pruebas de terminación de sesión del lado del servidor descritas anteriormente por un cierre de sesión causado por un tiempo de espera de inactividad.
>
> El valor adecuado para el tiempo de espera de sesión depende del propósito de la aplicación y debería ser un equilibrio entre seguridad y usabilidad. En aplicaciones bancarias no tiene sentido mantener una sesión inactiva más de 15 minutos. Por otro lado, un tiempo de espera corto en un wiki o foro podría molestar a los usuarios que están escribiendo artículos largos con solicitudes de inicio de sesión innecesarias. Allí, tiempos de espera de una hora y más pueden ser aceptables.

### Pruebas de terminación de sesión en entornos de inicio de sesión único (cierre de sesión único)

Realice un cierre de sesión en la aplicación probada. Verifique si hay un portal central o directorio de aplicaciones que permita al usuario volver a iniciar sesión en la aplicación sin autenticación. Pruebe si la aplicación solicita al usuario autenticarse, si se solicita la URL de un punto de entrada a la aplicación. Mientras está conectado en la aplicación probada, realice un cierre de sesión en el sistema SSO. Luego intente acceder a un área autenticada de la aplicación probada.

> Se espera que la invocación de una función de cierre de sesión en una aplicación web conectada a un sistema SSO o en el sistema SSO mismo cause la terminación global de todas las sesiones. Se debería requerir una autenticación del usuario para obtener acceso a la aplicación después del cierre de sesión en el sistema SSO y la aplicación conectada.

## Herramientas

- [Burp Suite - Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)

## Referencias

### Documentos técnicos

- [Ataques de reproducción de cookies en ASP.NET cuando se usa autenticación de formularios](https://www.vanstechelman.eu/content/cookie-replay-attacks-in-aspnet-when-using-forms-authentication)