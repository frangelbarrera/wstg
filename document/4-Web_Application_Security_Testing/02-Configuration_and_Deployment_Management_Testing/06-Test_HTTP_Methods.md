# Probar Métodos HTTP

|ID          |
|------------|
|WSTG-CONF-06|

## Resumen

HTTP ofrece una serie de métodos (o verbos) que pueden usarse para realizar acciones en el servidor web. Aunque GET y POST son con mucho los métodos más comunes utilizados para acceder a la información proporcionada por un servidor web, hay una variedad de otros métodos que también pueden ser soportados y que a veces pueden ser explotados por atacantes.

[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) define los principales métodos de solicitud HTTP válidos (o verbos), aunque métodos adicionales han sido añadidos en otros RFCs, como [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789). Varios de estos verbos han sido reutilizados para diferentes propósitos en aplicaciones [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer), listados en la tabla a continuación.

| Método | Propósito Original | Propósito RESTful |
|--------|-------------------|-------------------|
| [`GET`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1) | Solicitar un archivo. | Solicitar un objeto.|
| [`HEAD`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2) | Solicitar un archivo, pero solo devolver los encabezados HTTP. | |
| [`POST`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3) | Enviar datos. | |
| [`PUT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.4) | Subir un archivo. | Crear un objeto. |
| [`DELETE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5) | Eliminar un archivo | Eliminar un objeto. |
| [`CONNECT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) | Establecer una conexión a otro sistema. | |
| [`OPTIONS`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.7) | Listar métodos HTTP soportados. | Realizar una solicitud [CORS Preflight](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request). |
| [`TRACE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.8) | Hacer eco de la solicitud HTTP para fines de depuración. | |
| [`PATCH`](https://datatracker.ietf.org/doc/html/rfc5789#section-2) |  | Modificar un objeto. |

## Objetivos de la Prueba

- Enumerar métodos HTTP soportados.
- Probar bypass de control de acceso.
- Probar técnicas de overriding de métodos HTTP.

## Cómo Probar

### Descubrir los Métodos Soportados

Para realizar esta prueba, el evaluador necesita una forma de identificar qué métodos HTTP son soportados por el servidor web que se está examinando. La forma más simple de hacerlo es hacer una solicitud `OPTIONS` al servidor:

```http
OPTIONS / HTTP/1.1
Host: example.org
```

El servidor debería entonces responder con una lista de métodos soportados:

```http
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
```

Sin embargo, no todos los servidores responden a solicitudes OPTIONS, y algunos pueden incluso devolver información inexacta. También vale la pena notar que los servidores pueden soportar diferentes métodos para diferentes rutas. Esto significa que incluso si un método no es soportado para el directorio raíz /, no necesariamente indica que el mismo método no será soportado en otro lugar.

Una forma más confiable de probar métodos soportados es simplemente hacer una solicitud con ese tipo de método y examinar la respuesta del servidor. Si el método no está permitido, el servidor debería devolver un estado `405 Method Not Allowed`.

Nota que algunos servidores tratan métodos desconocidos como equivalentes a `GET`, por lo que pueden responder a métodos arbitrarios, como la solicitud mostrada a continuación. Esto puede ocasionalmente ser útil para evadir un firewall de aplicaciones web, o cualquier otro filtrado que bloquee métodos específicos.

```http
FOO / HTTP/1.1
Host: example.org
```

Las solicitudes con métodos arbitrarios también pueden hacerse usando curl con la opción `-X`:

```bash
curl -X FOO https://example.org
```

También hay una variedad de herramientas automatizadas que pueden intentar determinar métodos soportados, como el script [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html) de Nmap. Sin embargo, estas herramientas pueden no probar métodos peligrosos (es decir, métodos que pueden causar cambios como `PUT` o `DELETE`), o pueden causar cambios no intencionados en el servidor web si estos métodos son soportados. Como tal, deben usarse con cuidado.

### PUT y DELETE

Los métodos `PUT` y `DELETE` pueden tener diferentes efectos, dependiendo de si están siendo interpretados por el servidor web o por la aplicación que se ejecuta en él.

#### Servidores Web Legacy

Algunos servidores web legacy permitían el uso del método `PUT` para crear archivos en el servidor. Por ejemplo, si el servidor está configurado para permitir esto, la solicitud a continuación crearía un archivo en el servidor llamado `test.html` con el contenido `<script>alert(1)</script>`.

```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

<script>alert(1)</script>
```

Solicitudes similares también pueden hacerse con cURL:

```bash
curl https://example.org --upload-file test.html
```

Esto permite a un atacante subir archivos arbitrarios al servidor web, lo que podría potencialmente resultar en un compromiso completo del sistema si se les permite subir código ejecutable como archivos PHP. Sin embargo, esta configuración es extremadamente rara y es poco probable que se vea en sistemas modernos.

De manera similar, el método `DELETE` puede usarse para eliminar archivos del servidor web. Por favor nota que esta es una **acción destructiva**; por lo tanto, se debe ejercer extrema precaución al probar este método.

```http
DELETE /test.html HTTP/1.1
Host: example.org
```

O con cURL:

```bash
curl https://example.org/test.html -X DELETE
```

#### APIs RESTful

Por el contrario, los métodos `PUT` y `DELETE` son comúnmente usados por aplicaciones RESTful modernas para crear y eliminar objetos. Por ejemplo, la solicitud de API a continuación podría usarse para crear un usuario llamado "foo" con un rol de "user":

```http
PUT /api/users/foo HTTP/1.1
Host: example.org
Content-Length: 34

{
    "role": "user"
}
```

Una solicitud similar con el método DELETE podría usarse para eliminar un objeto.

```http
DELETE /api/users/foo HTTP/1.1
Host: example.org
```

Aunque puede ser reportado por herramientas de escaneo automatizadas, la presencia de estos métodos en una API RESTful **no es un problema de seguridad**. Sin embargo, esta funcionalidad puede tener otras vulnerabilidades (como control de acceso débil), y debe ser probada exhaustivamente.

### TRACE

El método `TRACE` (o el equivalente de Microsoft `TRACK`) hace que el servidor haga eco del contenido de la solicitud. Esto llevó a una vulnerabilidad llamada Cross-Site Tracing (XST) a ser publicada en [2003](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf) (PDF), que podía usarse para acceder a cookies que tenían el flag `HttpOnly` establecido. El método `TRACE` ha sido bloqueado en todos los navegadores y plugins durante muchos años; como tal, este problema ya no es explotable. Sin embargo, puede aún ser marcado por herramientas de escaneo automatizadas, y el método `TRACE` habilitado en un servidor web sugiere que no ha sido endurecido correctamente.

### CONNECT

El método `CONNECT` hace que el servidor web abra una conexión TCP a otro sistema, y luego pase el tráfico del cliente a ese sistema. Esto podría permitir a un atacante proxy el tráfico a través del servidor, para ocultar su dirección de origen, acceder a sistemas internos o acceder a servicios que están vinculados a localhost. Un ejemplo de una solicitud `CONNECT` se muestra a continuación:

```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### PATCH

El método `PATCH` está definido en [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789), y se usa para proporcionar instrucciones sobre cómo un objeto debería ser modificado. El RFC en sí no define en qué formato deberían estar estas instrucciones, pero varios métodos están definidos en otros estándares, como el [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://datatracker.ietf.org/doc/html/rfc6902).

Por ejemplo, si tenemos un usuario llamado "foo" con las siguientes propiedades:

```json
{
    "role": "user",
    "email": "foo@example.org"
}
```

La siguiente solicitud JSON PATCH podría usarse para cambiar el rol de este usuario a "admin", sin modificar la dirección de email:

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{ "op": "replace", "path": "/role", "value": "admin" }
```

Aunque el RFC establece que debería incluir instrucciones sobre cómo el objeto debería ser modificado, el método `PATCH` es comúnmente (mal)usado para incluir el contenido cambiado en su lugar, como se muestra a continuación. Al igual que la solicitud anterior, esto cambiaría el valor "role" a "admin" sin modificar el resto del objeto. Esto contrasta con el método `PUT`, que sobrescribiría el objeto entero, y así resultaría en un objeto sin atributo "email".

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{
    "role": "admin"
}
```

Al igual que con el método `PUT`, esta funcionalidad puede tener debilidades de control de acceso u otras vulnerabilidades. Adicionalmente, las aplicaciones pueden no realizar el mismo nivel de validación de entrada al modificar un objeto como lo hacen al crear uno. Esto podría potencialmente permitir valores maliciosos a ser inyectados (como en un ataque de cross-site scripting almacenado), o podría permitir objetos rotos o inválidos que pueden resultar en problemas relacionados con la lógica de negocio.

### Pruebas para Bypass de Control de Acceso

Si una página en la aplicación redirige a los usuarios a una página de login con un código 302 cuando intentan acceder directamente a ella, puede ser posible bypass esto haciendo una solicitud con un método HTTP diferente, como `HEAD`, `POST` o incluso un método inventado como `FOO`. Si la aplicación web responde con un `HTTP/1.1 200 OK` en lugar del esperado `HTTP/1.1 302 Found`, puede entonces ser posible bypass la autenticación o autorización. El ejemplo a continuación muestra cómo una solicitud `HEAD` puede resultar en una página configurando cookies administrativas, en lugar de redirigir al usuario a una página de login:

```http
HEAD /admin/ HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: adminSessionCookie=[...];
```

Alternativamente, puede ser posible hacer solicitudes directas a páginas que causan acciones, como:

```http
HEAD /admin/createUser.php?username=foo&password=bar&role=admin HTTP/1.1
Host: example.org
```

O:

```http
FOO /admin/createUser.php
Host: example.org
Content-Length: 36

username=foo&password=bar&role=admin
```

### Pruebas para Overriding de Métodos HTTP

Algunos frameworks web proporcionan una forma de override el método HTTP real en la solicitud. Logran esto emulando los verbos HTTP faltantes y pasando algunos encabezados personalizados en las solicitudes. El propósito principal de esto es circumventir una aplicación middleware (como un proxy o firewall de aplicaciones web) que bloquea métodos específicos. Los siguientes encabezados HTTP alternativos podrían potencialmente usarse:

- `X-HTTP-Method`
- `X-HTTP-Method-Override`
- `X-Method-Override`

Para probar esto, considera escenarios donde verbos restringidos como `PUT` o `DELETE` devuelven un `405 Method not allowed`. En tales casos, repite la misma solicitud, pero añade los encabezados alternativos para overriding de métodos HTTP. Luego, observa la respuesta del sistema. La aplicación debería responder con un código de estado diferente (*ej.* `200 OK`) en casos donde el overriding de métodos es soportado.

El servidor web en el siguiente ejemplo no permite el método `DELETE` y lo bloquea:

```http
DELETE /resource.html HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 405 Method Not Allowed
[...]
```

Después de añadir el encabezado `X-HTTP-Method`, el servidor responde a la solicitud con un 200:

```http
GET /resource.html HTTP/1.1
Host: example.org
X-HTTP-Method: DELETE
```

```http
HTTP/1.1 200 OK
[...]
```

## Remediación

- Asegúrate de que solo los métodos requeridos sean permitidos y que estos métodos estén correctamente configurados.
- Asegúrate de que no se implementen workarounds para bypass medidas de seguridad implementadas por user-agents, frameworks o servidores web.

## Herramientas

- [Ncat](https://nmap.org/ncat/)
- [cURL](https://curl.haxx.se/)
- [Nmap http-methods NSE script](https://nmap.org/nsedoc/scripts/http-methods.html)

## Referencias

- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1)](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 5789 - PATCH Method for HTTP](https://datatracker.ietf.org/doc/html/rfc5789)
- [HTACCESS: BILBAO Method Exposed](https://web.archive.org/web/20160616172703/https://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Fortify - Misused HTTP Method Override](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [Mozilla Developer Network - Safe HTTP Methods](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)