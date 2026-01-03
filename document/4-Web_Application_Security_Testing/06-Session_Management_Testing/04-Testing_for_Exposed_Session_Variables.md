# Pruebas de Variables de Sesión Expuestas

|ID          |
|------------|
|WSTG-SESS-04|

## Resumen

Los Tokens de Sesión (Cookie, SessionID, Campo Oculto), si se exponen, generalmente permitirán a un atacante suplantar a una víctima y acceder a la aplicación de manera ilegítima. Es importante que estén protegidos contra la interceptación en todo momento, particularmente mientras están en tránsito entre el navegador del cliente y los servidores de la aplicación.

La información aquí se relaciona con cómo se aplica la seguridad de transporte a la transferencia de datos sensibles de Session ID en lugar de datos en general, y puede ser más estricta que las políticas de almacenamiento en caché y transporte aplicadas a los datos servidos por el sitio.

Usando un proxy personal, es posible determinar lo siguiente sobre cada solicitud y respuesta:

- Protocolo utilizado (por ejemplo, HTTP vs. HTTPS)
- Cabeceras HTTP
- Cuerpo del mensaje (por ejemplo, POST o contenido de página)

Cada vez que se pasan datos de Session ID entre el cliente y el servidor, se deben examinar el protocolo, las directivas de caché y privacidad, y el cuerpo. La seguridad de transporte aquí se refiere a Session IDs pasados en solicitudes GET o POST, cuerpos de mensaje u otros medios sobre solicitudes HTTP válidas.

## Objetivos de la Prueba

- Asegurar que se implemente el cifrado adecuado.
- Revisar la configuración de almacenamiento en caché.
- Evaluar la seguridad del canal y los métodos.

## Cómo Probar

### Pruebas de Vulnerabilidades de Cifrado y Reutilización de Tokens de Sesión

La protección contra la interceptación a menudo se proporciona mediante cifrado TLS, pero puede incorporar otros túneles o cifrado. Debe notarse que el cifrado o hash criptográfico del Session ID debe considerarse por separado del cifrado de transporte, ya que es el Session ID en sí el que se protege, no los datos que puede representar.

Si el Session ID pudiera ser presentado por un atacante a la aplicación para obtener acceso, entonces debe protegerse en tránsito para mitigar ese riesgo. Por lo tanto, se debe asegurar que el cifrado sea tanto el predeterminado como obligatorio para cualquier solicitud o respuesta donde se pase el Session ID, independientemente del mecanismo utilizado (por ejemplo, un campo de formulario oculto). Se deben realizar comprobaciones simples como reemplazar `https://` con `http://` durante la interacción con la aplicación, junto con la modificación de publicaciones de formularios para determinar si se implementa una segregación adecuada entre los sitios seguros y no seguros.

Tenga en cuenta que si también hay un elemento en el sitio donde el usuario se rastrea con Session IDs pero no hay seguridad presente (por ejemplo, notando qué documentos públicos descarga un usuario registrado), es esencial que se use un Session ID diferente. El Session ID debe monitorearse a medida que el cliente cambia de elementos seguros a no seguros para asegurar que se use uno diferente.

> Cada vez que la autenticación sea exitosa, el usuario debe esperar recibir:
>
> - Un token de sesión diferente
> - Un token enviado a través de un canal cifrado cada vez que hacen una Solicitud HTTP

### Pruebas de Vulnerabilidades de Proxies y Almacenamiento en Caché

Los proxies también deben considerarse al revisar la seguridad de la aplicación. En muchos casos, los clientes accederán a la aplicación a través de proxies corporativos, ISP u otros proxies o pasarelas conscientes del protocolo (por ejemplo, Firewalls). El protocolo HTTP proporciona directivas para controlar el comportamiento de proxies descendentes, y la implementación correcta de estas directivas también debe evaluarse.

En general, el Session ID nunca debe enviarse sobre transporte no cifrado y nunca debe almacenarse en caché. La aplicación debe examinarse para asegurar que las comunicaciones cifradas sean tanto el predeterminado como obligatorio para cualquier transferencia de Session IDs. Además, cada vez que se pase el Session ID, deben estar en vigor directivas para prevenir su almacenamiento en caché por cachés intermedias e incluso locales.

La aplicación también debe configurarse para asegurar datos en cachés sobre HTTP/1.0 y HTTP/1.1 – RFC 2616 discute los controles apropiados con referencia a HTTP. HTTP/1.1 proporciona una serie de mecanismos de control de caché. `Cache-Control: no-cache` indica que un proxy no debe reutilizar ningún dato. Mientras que `Cache-Control: Private` parece ser una directiva adecuada, esto aún permite que un proxy no compartido almacene datos en caché. En el caso de cibercafés u otros sistemas compartidos, esto presenta un riesgo claro. Incluso con estaciones de trabajo de un solo usuario, el Session ID almacenado en caché puede exponerse a través de un compromiso del sistema de archivos o donde se usan almacenes de red. Las cachés HTTP/1.0 no reconocen la directiva `Cache-Control: no-cache`.

> Las directivas `Expires: 0` y `Cache-Control: max-age=0` deben usarse para asegurar aún más que las cachés no expongan los datos. Cada solicitud/respuesta que pase datos de Session ID debe examinarse para asegurar que se usen directivas de caché apropiadas.

### Pruebas de Vulnerabilidades GET y POST

En general, las solicitudes GET no deben usarse, ya que el Session ID puede exponerse en registros de Proxy o Firewall. También son mucho más fácilmente manipulables que otros tipos de transporte, aunque debe notarse que casi cualquier mecanismo puede manipularse por el cliente con las herramientas adecuadas. Además, los ataques [Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) se explotan más fácilmente enviando un enlace especialmente construido a la víctima. Esto es mucho menos probable si los datos se envían desde el cliente como POSTs.

Todo el código del lado del servidor que recibe datos de solicitudes POST debe probarse para asegurar que no acepta los datos si se envían como GET. Por ejemplo, considere la siguiente solicitud POST (`https://owaspapp.com/login.asp`) generada por una página de inicio de sesión.

```http
POST /login.asp HTTP/1.1
Host: owaspapp.com
[...]
Cookie: ASPSESSIONIDABCDEFG=ASKLJDLKJRELKHJG
Content-Length: 51

Login=Username&password=Password&SessionID=12345678
```

Si login.asp está mal implementado, puede ser posible iniciar sesión usando la siguiente URL: `https://owaspapp.com/login.asp?Login=Username&password=Password&SessionID=12345678`

Los scripts del lado del servidor potencialmente inseguros pueden identificarse verificando cada POST de esta manera.

### Pruebas de Vulnerabilidades de Transporte

Toda la interacción entre el Cliente y la Aplicación debe probarse al menos contra los siguientes criterios.

- ¿Cómo se transfieren los Session IDs? por ejemplo, GET, POST, Campo de Formulario (incluyendo campos ocultos)
- ¿Se envían siempre los Session IDs sobre transporte cifrado por defecto?
- ¿Es posible manipular la aplicación para enviar Session IDs sin cifrar? por ejemplo, cambiando HTTPS a HTTP?
- ¿Qué directivas de control de caché se aplican a solicitudes/respuestas que pasan Session IDs?
- ¿Están siempre presentes estas directivas? Si no, ¿dónde están las excepciones?
- ¿Se usan solicitudes GET que incorporan el Session ID?
- Si se usa POST, ¿puede intercambiarse con GET?

## Referencias

### Documentos Técnicos

- [RFCs 2109 y 2965 – Mecanismo de Gestión de Estado HTTP - D. Kristol, L. Montulli](https://www.ietf.org/rfc/rfc2965.txt)
- [RFC 2616 – Protocolo de Transferencia de Hipertexto - HTTP/1.1](https://www.ietf.org/rfc/rfc2616.txt)