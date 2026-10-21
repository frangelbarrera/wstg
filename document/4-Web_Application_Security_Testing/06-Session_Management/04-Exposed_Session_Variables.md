# Variables de sesión expuestas

|ID          |
|------------|
|WSTG-SESS-04|

## Resumen

Los tokens de sesión (cookie, SessionID, campo oculto), si se exponen, suelen permitir que un atacante suplante a una víctima y acceda a la aplicación de manera ilegítima. Es importante protegerlos de la interceptación en todo momento, en particular mientras están en tránsito entre el navegador del cliente y los servidores de la aplicación.

La información que se presenta aquí se refiere a la aplicación de la seguridad de transporte a la transferencia de datos sensibles de Session ID, y no a los datos en general; por ello, puede ser más estricta que las políticas de almacenamiento en caché y transporte aplicadas a los datos que sirve el sitio.

Con un proxy personal, es posible determinar lo siguiente sobre cada solicitud y respuesta:

- Protocolo utilizado (por ejemplo, HTTP frente a HTTPS)
- Cabeceras HTTP
- Cuerpo del mensaje (por ejemplo, POST o contenido de la página)

Cada vez que se transmitan datos de Session ID entre el cliente y el servidor, se deben examinar el protocolo, las directivas de caché y privacidad, y el cuerpo. La seguridad de transporte en este contexto se refiere a los Session IDs transmitidos en solicitudes GET o POST, cuerpos de mensajes u otros medios mediante solicitudes HTTP válidas.

## Objetivos de la prueba

- Asegurarse de que se implemente el cifrado adecuado.
- Revisar la configuración del almacenamiento en caché.
- Evaluar la seguridad del canal y de los métodos.

## Cómo probar

### Vulnerabilidades de cifrado y reutilización de tokens de sesión

La protección contra la interceptación suele proporcionarse mediante cifrado TLS, aunque también puede incorporar otros túneles o mecanismos de cifrado. Cabe señalar que el cifrado o el hash criptográfico del Session ID deben considerarse por separado del cifrado de transporte, ya que lo que se protege es el propio Session ID, no los datos que este pueda representar.

Si un atacante pudiera presentar el Session ID a la aplicación para obtener acceso, este debe protegerse en tránsito para mitigar ese riesgo. Por lo tanto, se debe garantizar que el cifrado sea tanto el valor predeterminado como obligatorio en cualquier solicitud o respuesta en la que se transmita el Session ID, independientemente del mecanismo utilizado (por ejemplo, un campo de formulario oculto). Se deben realizar comprobaciones sencillas, como reemplazar `https://` por `http://` durante la interacción con la aplicación, así como modificar los envíos de formularios para determinar si existe una segregación adecuada entre los sitios seguros y los no seguros.

Tenga en cuenta que, si el sitio también incluye una funcionalidad que rastrea al usuario mediante Session IDs sin aplicar medidas de seguridad (por ejemplo, registrar qué documentos públicos descarga un usuario registrado), es esencial utilizar un Session ID diferente. Por lo tanto, se debe supervisar el Session ID cuando el cliente pase de las funcionalidades seguras a las no seguras, a fin de comprobar que se utilice uno diferente.

> Cada vez que la autenticación se realice correctamente, el usuario debe esperar recibir:
>
> - Un token de sesión diferente
> - Un token enviado a través de un canal cifrado cada vez que realice una solicitud HTTP

### Vulnerabilidades de proxies y almacenamiento en caché

También se deben tener en cuenta los proxies al revisar la seguridad de la aplicación. En muchos casos, los clientes accederán a la aplicación mediante proxies corporativos, de ISP u otros, o mediante pasarelas que reconocen el protocolo (por ejemplo, firewalls). El protocolo HTTP proporciona directivas para controlar el comportamiento de los proxies intermedios, y también se debe evaluar la implementación correcta de estas directivas.

En general, el Session ID nunca debe transmitirse a través de un transporte sin cifrar ni almacenarse en caché. Se debe examinar la aplicación para garantizar que las comunicaciones cifradas sean tanto el valor predeterminado como obligatorias en cualquier transferencia de Session IDs. Además, cada vez que se transmita el Session ID, deben aplicarse directivas para impedir que se almacene en caché, tanto en cachés intermedias como locales.

La aplicación también debe configurarse para proteger los datos almacenados en cachés con HTTP/1.0 y HTTP/1.1. RFC 2616 describe los controles adecuados para HTTP. HTTP/1.1 proporciona varios mecanismos de control de caché. `Cache-Control: no-cache` indica que un proxy no debe reutilizar ningún dato. Aunque `Cache-Control: Private` parece una directiva adecuada, aun así permite que un proxy no compartido almacene datos en caché. En el caso de cibercafés u otros sistemas compartidos, esto supone un riesgo evidente. Incluso en estaciones de trabajo de un solo usuario, el Session ID almacenado en caché puede quedar expuesto si se compromete el sistema de archivos o si se utilizan almacenes de red. Las cachés HTTP/1.0 no reconocen la directiva `Cache-Control: no-cache`.

> Se deben usar las directivas `Expires: 0` y `Cache-Control: max-age=0` para garantizar aún más que las cachés no expongan los datos. Se debe examinar cada solicitud o respuesta que transmita datos de Session ID para comprobar que utilice las directivas de caché adecuadas.

### Vulnerabilidades de GET y POST

En general, no se deben usar solicitudes GET, ya que el Session ID puede quedar expuesto en los registros de proxies o firewalls. Además, son mucho más fáciles de manipular que otros tipos de transporte, aunque cabe señalar que el cliente puede manipular casi cualquier mecanismo si dispone de las herramientas adecuadas. Asimismo, los ataques de [Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) se explotan con mayor facilidad enviando a la víctima un enlace especialmente diseñado. Esto es mucho menos probable si el cliente envía los datos mediante solicitudes POST.

Se debe probar todo el código del lado del servidor que recibe datos de solicitudes POST para comprobar que no acepte esos datos si se envían mediante GET. Por ejemplo, considere la siguiente solicitud POST (`https://owaspapp.com/login.asp`) generada por una página de inicio de sesión.

```http
POST /login.asp HTTP/1.1
Host: owaspapp.com
[...]
Cookie: ASPSESSIONIDABCDEFG=ASKLJDLKJRELKHJG
Content-Length: 51

Login=Username&password=Password&SessionID=12345678
```

Si login.asp está mal implementado, es posible que se pueda iniciar sesión mediante la siguiente URL: `https://owaspapp.com/login.asp?Login=Username&password=Password&SessionID=12345678`

Se pueden identificar scripts del lado del servidor potencialmente inseguros comprobando de esta manera cada solicitud POST.

### Vulnerabilidades de transporte

Se debe probar toda interacción entre el cliente y la aplicación, como mínimo, con los siguientes criterios:

- ¿Cómo se transfieren los Session IDs? Por ejemplo, mediante GET, POST o campos de formulario (incluidos los campos ocultos).
- ¿Se envían siempre los Session IDs mediante transporte cifrado de forma predeterminada?
- ¿Es posible manipular la aplicación para que envíe Session IDs sin cifrar? Por ejemplo, ¿cambiando HTTPS por HTTP?
- ¿Qué directivas de control de caché se aplican a las solicitudes o respuestas que transmiten Session IDs?
- ¿Están siempre presentes estas directivas? Si no es así, ¿dónde están las excepciones?
- ¿Se utilizan solicitudes GET que incorporan el Session ID?
- Si se utiliza POST, ¿se puede sustituir por GET?

## Referencias

### Documentos técnicos

- [RFCs 2109 y 2965 - Mecanismo de gestión del estado HTTP - D. Kristol, L. Montulli](https://www.ietf.org/rfc/rfc2965.txt)
- [RFC 2616 - Protocolo de transferencia de hipertexto - HTTP/1.1](https://www.ietf.org/rfc/rfc2616.txt)