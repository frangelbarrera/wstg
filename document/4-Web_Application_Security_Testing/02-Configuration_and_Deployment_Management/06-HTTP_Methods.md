# Métodos HTTP

|ID          |
|------------|
|WSTG-CONF-06|

## Resumen

HTTP ofrece varios métodos (o verbos) que pueden utilizarse para realizar acciones en el servidor web. Aunque GET y POST son, con diferencia, los métodos más comunes para acceder a la información proporcionada por un servidor web, también pueden admitirse otros métodos que, en ocasiones, pueden ser aprovechados por atacantes.

[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) define los principales métodos de solicitud HTTP válidos (o verbos), aunque se han añadido métodos adicionales en otros RFC, como [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789). Varios de estos verbos se han reutilizado con distintos propósitos en aplicaciones [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer), como se indica en la tabla siguiente.

| Método | Propósito original | Propósito RESTful |
|--------|--------------------|-------------------|
| [`GET`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1) | Solicitar un archivo. | Solicitar un objeto.|
| [`HEAD`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2) | Solicitar un archivo, pero devolver solo los encabezados HTTP. | |
| [`POST`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3) | Enviar datos. | |
| [`PUT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.4) | Subir un archivo. | Crear un objeto. |
| [`DELETE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5) | Eliminar un archivo. | Eliminar un objeto. |
| [`CONNECT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) | Establecer una conexión con otro sistema. | |
| [`OPTIONS`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.7) | Enumerar los métodos HTTP admitidos. | Realizar una solicitud [CORS Preflight](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request). |
| [`TRACE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.8) | Devolver la solicitud HTTP para fines de depuración. | |
| [`PATCH`](https://datatracker.ietf.org/doc/html/rfc5789#section-2) |  | Modificar un objeto. |
| [`QUERY`](https://datatracker.ietf.org/doc/html/rfc9110#section-9.3.7) | Recuperar datos de forma segura mediante un cuerpo de solicitud. | Consultar objetos o recursos con parámetros complejos. |

## Objetivos de la prueba

- Enumerar los métodos HTTP admitidos.
- Probar si es posible eludir los controles de acceso.
- Probar técnicas para sobrescribir el método HTTP.

## Cómo probar

### Descubrir los métodos admitidos

Para realizar esta prueba, el evaluador necesita una forma de identificar qué métodos HTTP admite el servidor web examinado. La forma más sencilla de hacerlo es enviar una solicitud `OPTIONS` al servidor:

```http
OPTIONS / HTTP/1.1
Host: example.org
```

A continuación, el servidor debería responder con una lista de los métodos admitidos:

```http
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
```

Sin embargo, es posible que algunos servidores no respondan a las solicitudes OPTIONS e incluso que otros devuelvan información inexacta. También cabe tener en cuenta que los servidores pueden admitir distintos métodos en diferentes rutas. Esto significa que, aunque no se admita un método en el directorio raíz `/`, no necesariamente se deduce que tampoco se admita en otras ubicaciones.

Una forma más fiable de comprobar qué métodos se admiten consiste en enviar una solicitud con ese método y examinar la respuesta del servidor. Si el método no está permitido, el servidor debería devolver el estado `405 Method Not Allowed`.

Cabe señalar que algunos servidores tratan los métodos desconocidos como si fueran equivalentes a `GET`, por lo que pueden responder a métodos arbitrarios, como el de la solicitud que se muestra a continuación. En ocasiones, esto puede ser útil para evadir un firewall de aplicaciones web o cualquier otro filtro que bloquee métodos específicos.

```http
FOO / HTTP/1.1
Host: example.org
```

También se pueden enviar solicitudes con métodos arbitrarios mediante curl y la opción `-X`:

```bash
curl -X FOO https://example.org
```

También hay varias herramientas automatizadas que pueden intentar determinar qué métodos se admiten, como el script [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html) de Nmap. Sin embargo, es posible que estas herramientas no prueben métodos peligrosos (es decir, métodos que puedan provocar cambios, como `PUT` o `DELETE`) o que provoquen cambios involuntarios en el servidor web si se admiten dichos métodos. Por lo tanto, deben utilizarse con cuidado.

#### Pruebas a través de proxies y CDN

Cuando la aplicación se encuentra detrás de un proxy inverso, un balanceador de carga, una CDN o un WAF, el tratamiento de los métodos en el perímetro puede diferir del que se aplica en el servidor de origen, por lo que conviene comparar los resultados:

- Muchas CDN almacenan en caché y reenvían de forma predeterminada solo un conjunto limitado de métodos (normalmente `GET`/`HEAD` y, en ocasiones, `OPTIONS`), y pueden rechazar o eliminar otros (`TRACE`, `CONNECT` y, a veces, `PUT`/`DELETE`) antes de que lleguen al servidor de origen. Por lo tanto, un encabezado `Allow` o una respuesta `OPTIONS` observados en el perímetro no reflejan necesariamente los métodos que admite el propio servidor de origen.
- Cuando sea posible (por ejemplo, en una prueba de caja gris con acceso al servidor de origen o mediante una conexión directa basada en la dirección IP o el encabezado `Host`), pruebe también el servidor de origen directamente, además de hacerlo a través del perímetro. Si un método está bloqueado en el perímetro, pero el servidor de origen lo admite, podría aprovecharse si alguna vez se pudiera acceder directamente al servidor de origen (por reglas de firewall mal configuradas, SSRF, otros hosts virtuales que compartan el servidor de origen, etc.).
- Algunos proxies y WAF normalizan o convierten a minúsculas el token del método antes de compararlo con una lista de bloqueo. Probar variantes con mayúsculas y minúsculas (`DeLeTe`) o espacios en blanco no estándar puede revelar incoherencias en la aplicación de los controles entre el proxy y el servidor de origen.

### PUT y DELETE

Los métodos `PUT` y `DELETE` pueden tener efectos distintos según si los interpreta el servidor web o la aplicación que se ejecuta en él.

#### Servidores web heredados

Algunos servidores web heredados permitían utilizar el método `PUT` para crear archivos en el servidor. Por ejemplo, si el servidor está configurado para permitirlo, la solicitud siguiente crearía en el servidor un archivo llamado `test.html` con el contenido `<script>alert(1)</script>`.

```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

<script>alert(1)</script>
```

También se pueden enviar solicitudes similares con curl:

```bash
curl https://example.org --upload-file test.html
```

Esto permite a un atacante subir archivos arbitrarios al servidor web, lo que podría dar lugar a un compromiso completo del sistema si se le permite subir código ejecutable, como archivos PHP. Sin embargo, esta configuración es extremadamente poco frecuente y es improbable encontrarla en sistemas modernos.

De forma similar, el método `DELETE` puede utilizarse para eliminar archivos del servidor web. Tenga en cuenta que esta es una **acción destructiva**; por lo tanto, se debe extremar la precaución al probar este método.

```http
DELETE /test.html HTTP/1.1
Host: example.org
```

O con curl:

```bash
curl https://example.org/test.html -X DELETE
```

#### API RESTful

En cambio, las aplicaciones RESTful modernas suelen utilizar los métodos `PUT` y `DELETE` para crear y eliminar objetos. Por ejemplo, la siguiente solicitud a la API podría utilizarse para crear un usuario llamado "foo" con el rol "user":

```http
PUT /api/users/foo HTTP/1.1
Host: example.org
Content-Length: 34

{
    "role": "user"
}
```

Una solicitud similar con el método DELETE podría utilizarse para eliminar un objeto.

```http
DELETE /api/users/foo HTTP/1.1
Host: example.org
```

Aunque las herramientas de análisis automatizado puedan notificarlo, la presencia de estos métodos en una API RESTful **no es un problema de seguridad**. Sin embargo, esta funcionalidad puede presentar otras vulnerabilidades (como controles de acceso débiles), por lo que debe probarse exhaustivamente.

### TRACE

El método `TRACE` (o el método equivalente de Microsoft, `TRACK`) hace que el servidor devuelva el contenido de la solicitud. Esto dio lugar a la publicación, en [2003](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf) (PDF), de una vulnerabilidad denominada Cross-Site Tracing (XST), que podía utilizarse para acceder a cookies que tenían activado el indicador `HttpOnly`. El método `TRACE` está bloqueado en todos los navegadores y complementos desde hace muchos años; por ello, este problema ya no es explotable. Sin embargo, es posible que las herramientas de análisis automatizado sigan señalándolo, y que el método `TRACE` esté habilitado en un servidor web sugiere que no se ha reforzado correctamente su seguridad.

### CONNECT

El método `CONNECT` hace que el servidor web abra una conexión TCP con otro sistema y, a continuación, transmita a ese sistema el tráfico procedente del cliente. Esto podría permitir a un atacante usar el servidor como proxy para ocultar su dirección de origen, acceder a sistemas internos o acceder a servicios vinculados a localhost. A continuación se muestra un ejemplo de una solicitud `CONNECT`:

```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### PATCH

El método `PATCH` está definido en [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) y se utiliza para indicar cómo se debe modificar un objeto. El propio RFC no define el formato que deben tener estas instrucciones, pero hay varios métodos definidos en otros estándares, como el [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://datatracker.ietf.org/doc/html/rfc6902).

Por ejemplo, si tenemos un usuario llamado "foo" con las siguientes propiedades:

```json
{
    "role": "user",
    "email": "foo@example.org"
}
```

La siguiente solicitud JSON PATCH podría utilizarse para cambiar el rol de este usuario a "admin" sin modificar la dirección de correo electrónico:

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{ "op": "replace", "path": "/role", "value": "admin" }
```

Aunque el RFC establece que debe incluir instrucciones sobre cómo modificar el objeto, el método `PATCH` suele utilizarse (indebidamente) para incluir el contenido modificado en su lugar, como se muestra a continuación. Al igual que en la solicitud anterior, esto cambiaría el valor de "role" a "admin" sin modificar el resto del objeto. Esto contrasta con el método `PUT`, que sobrescribiría el objeto completo y, por tanto, daría como resultado un objeto sin el atributo "email".

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{
    "role": "admin"
}
```

Al igual que con el método `PUT`, esta funcionalidad puede presentar debilidades en los controles de acceso u otras vulnerabilidades. Además, es posible que las aplicaciones no realicen el mismo nivel de validación de entradas al modificar un objeto que al crearlo. Esto podría permitir inyectar valores maliciosos (como en un ataque de cross-site scripting almacenado) o crear objetos dañados o no válidos que podrían ocasionar problemas relacionados con la lógica de negocio.

### Métodos WebDAV

Cuando WebDAV está habilitado en el servidor web (por ejemplo, IIS con el rol WebDAV Publishing o `mod_dav` de Apache), pueden exponerse métodos adicionales, como `PROPFIND`, `PROPPATCH`, `MKCOL`, `COPY`, `MOVE`, `LOCK` y `UNLOCK`. La compatibilidad con WebDAV es heredada y rara vez es necesaria para las aplicaciones web modernas, por lo que su presencia merece señalarse por sí sola, independientemente de que exista alguna vulnerabilidad específica.

`PROPFIND` puede utilizarse para enumerar los contenidos de directorios y los metadatos de recursos, incluso si la exploración de directorios está deshabilitada:

```http
PROPFIND /uploads/ HTTP/1.1
Host: example.org
Depth: 1
```

`MKCOL`, `COPY` y `MOVE` pueden utilizarse para crear directorios o reubicar y duplicar archivos. Combinados con un método `PUT` permisivo, históricamente han permitido eludir restricciones de carga de archivos (por ejemplo, subir un archivo con una extensión bloqueada y luego utilizar `MOVE`/`COPY` para cambiarle el nombre y asignarle una extensión ejecutable). Compruebe si los filtros de carga basados en la extensión se aplican de forma coherente a través de `PUT`, `MOVE` y `COPY`.

Si WebDAV está presente, pero no es necesario, recomiende deshabilitarlo. Si es necesario, asegúrese de que esté limitado únicamente a las rutas que lo requieran y de que no sea accesible en las rutas que sirven la aplicación principal.

### QUERY

El método `QUERY` es un método HTTP relativamente reciente, definido en [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110), diseñado para operaciones de recuperación segura que requieren un cuerpo de solicitud. Al igual que `POST`, acepta datos estructurados en el cuerpo de la solicitud; a diferencia de `POST`, es seguro e idempotente, es decir, no modifica el estado del servidor y se puede almacenar en caché. Esto lo hace semánticamente similar a `GET`, pero permite expresar parámetros de filtro complejos y otros similares que resultarían engorrosos en una URL.

Por ejemplo, el método `QUERY` podría utilizarse para enviar una consulta de búsqueda compleja con varios criterios de filtrado:

```http
QUERY /api/users/search HTTP/1.1
Host: example.org
Content-Type: application/json

{
    "filters": {
        "role": "admin",
        "status": "active",
        "lastLogin": {"after": "2024-01-01"}
    }
}
```

El método `QUERY` está diseñado para ser almacenable en caché y seguro (de solo lectura), lo que lo diferencia de `POST`. No debe utilizarse para operaciones que modifiquen el estado del servidor. Los evaluadores deben verificar que:

- El servidor trate correctamente `QUERY` como una operación segura y no modifique el estado del servidor.
- Los controles de acceso se apliquen correctamente a las solicitudes `QUERY`.
- El método no se utilice como una vía alternativa para eludir los controles de seguridad que restringen otros métodos.

### Elusión de controles de acceso

Si una página de la aplicación redirige a los usuarios a una página de inicio de sesión con un código 302 cuando intentan acceder a ella directamente, es posible que se pueda eludir este comportamiento enviando una solicitud con un método HTTP diferente, como `HEAD`, `POST` o incluso un método inventado, como `FOO`. Si la aplicación web responde con `HTTP/1.1 200 OK` en lugar del `HTTP/1.1 302 Found` esperado, podría ser posible eludir la autenticación o la autorización. El ejemplo siguiente muestra cómo una solicitud `HEAD` puede hacer que una página establezca cookies administrativas en lugar de redirigir al usuario a una página de inicio de sesión:

```http
HEAD /admin/ HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: adminSessionCookie=[...];
```

También es posible que se puedan enviar solicitudes directas a páginas que ejecutan acciones, como:

```http
HEAD /admin/createUser.php?username=foo&password=bar&role=admin HTTP/1.1
Host: example.org
```

O bien:

```http
FOO /admin/createUser.php
Host: example.org
Content-Length: 36

username=foo&password=bar&role=admin
```

### Sobrescritura del método HTTP

Algunos frameworks web ofrecen una forma de sobrescribir el método HTTP real de una solicitud. Para ello, emulan los verbos HTTP que faltan y envían algunos encabezados personalizados en las solicitudes. Su propósito principal es eludir una aplicación de middleware (como un proxy o un firewall de aplicaciones web) que bloquea métodos específicos. Podrían utilizarse los siguientes encabezados HTTP alternativos:

- `X-HTTP-Method`
- `X-HTTP-Method-Override`
- `X-Method-Override`

Para probarlo, considere los casos en que los verbos restringidos, como `PUT` o `DELETE`, devuelven `405 Method not allowed`. En esos casos, vuelva a enviar la misma solicitud, pero añada los encabezados alternativos para sobrescribir el método HTTP. A continuación, observe la respuesta del sistema. Si se admite la sobrescritura de métodos, la aplicación debería responder con un código de estado diferente (por ejemplo, `200 OK`).

Varios frameworks también admiten la sobrescritura de métodos mediante un parámetro de solicitud, en lugar de un encabezado o además de este. Se suele utilizar `_method` en la cadena de consulta o en el cuerpo del formulario (por ejemplo, `POST /resource.html?_method=DELETE`). Pruebe esta opción junto con las variantes basadas en encabezados, ya que un filtro que solo inspeccione los encabezados podría pasarla por alto.

Dado que el propósito de la sobrescritura de métodos suele ser hacer que una solicitud atraviese un dispositivo intermediario, compare siempre el comportamiento en el perímetro (proxy/CDN/WAF) con el del servidor de origen. Si un método se bloquea en el perímetro, pero se sobrescribe y se acepta en el servidor de origen, esto indica que el servidor de origen confía en las señales de sobrescritura suministradas por el cliente, aunque el perímetro no tuviera previsto permitirlas.

El servidor web del ejemplo siguiente no permite el método `DELETE` y lo bloquea:

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

- Asegúrese de permitir únicamente los métodos necesarios y de que estén configurados correctamente.
- Asegúrese de no implementar mecanismos alternativos para eludir las medidas de seguridad implementadas por los agentes de usuario, los frameworks o los servidores web.
- Deshabilite WebDAV (`mod_dav`, IIS WebDAV Publishing) a menos que sea necesario de forma explícita; si lo es, limite su alcance.
- Asegúrese de que las restricciones de métodos se apliquen de forma coherente entre cualquier proxy/CDN/WAF perimetral y el servidor de origen, y de que el servidor de origen no confíe en encabezados o parámetros de sobrescritura de métodos proporcionados por el cliente, salvo que el perímetro dependa explícitamente de ellos.

## Herramientas

- [Ncat](https://nmap.org/ncat/)
- [curl](https://curl.se/)
- [Script http-methods NSE de Nmap](https://nmap.org/nsedoc/scripts/http-methods.html)

## Referencias

- [RFC 7231 - Protocolo de transferencia de hipertexto (HTTP/1.1)](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 5789 - Método PATCH para HTTP](https://datatracker.ietf.org/doc/html/rfc5789)
- [RFC 9110 - Semántica de HTTP](https://datatracker.ietf.org/doc/html/rfc9110)
- [HTACCESS: método BILBAO expuesto](https://web.archive.org/web/20160616172703/https://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Fortify - Uso indebido de la sobrescritura de métodos HTTP](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [Mozilla Developer Network - Métodos HTTP seguros](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)