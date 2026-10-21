# División de respuestas HTTP

|ID          |
|------------|
|WSTG-INJT-15|

## Resumen

La división de respuestas HTTP es una vulnerabilidad que se produce cuando una aplicación incorpora entradas de usuario no saneadas en los encabezados de respuesta HTTP, lo que permite a un atacante inyectar caracteres de retorno de carro (CR) y salto de línea (LF). Como resultado, los clientes o los sistemas intermediarios pueden interpretar una sola respuesta HTTP como varias respuestas distintas.

La explotación exitosa de la división de respuestas HTTP puede tener diversos impactos, como envenenamiento de caché web, cross-site scripting (XSS), suplantación de contenido, fijación de sesión u otros ataques del lado del cliente, según cómo se procese la respuesta inyectada.

Esta sección se centra exclusivamente en identificar y probar vulnerabilidades de división de respuestas HTTP en la capa de aplicación. HTTP Request Smuggling, que depende de inconsistencias de análisis entre varios agentes HTTP, se trata en un capítulo aparte.

## Objetivos de las pruebas

- Identificar entradas controladas por el usuario que se reflejen en los encabezados de respuesta HTTP.
- Evaluar si es posible inyectar caracteres CR (`\r`) y LF (`\n`) en los encabezados de respuesta.
- Determinar el impacto potencial de los ataques exitosos de división de respuestas HTTP, como el envenenamiento de caché o la explotación del lado del cliente.

## Cómo probar

### Pruebas de caja negra

Algunas aplicaciones web utilizan entradas proporcionadas por el usuario para generar los valores de ciertos encabezados de respuesta HTTP. Un ejemplo habitual es la lógica de redirección, en la que la URL de destino se deriva de un parámetro de la solicitud.

Por ejemplo, supongamos que se pide a un usuario que elija entre una interfaz estándar y una avanzada. La opción seleccionada se pasa como parámetro y se refleja en un encabezado de respuesta de redirección.

Si el parámetro `interface` tiene el valor `advanced`, la aplicación podría responder con:

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
```

Cuando el navegador recibe esta respuesta, sigue la URL especificada en el encabezado `Location`. Sin embargo, si la aplicación no valida o sanea correctamente la entrada del usuario, un atacante podría inyectar la secuencia `%0d%0a`, que representa caracteres CRLF utilizados para separar líneas de encabezado HTTP.

Al inyectar secuencias CRLF, un evaluador podría hacer que clientes o sistemas intermediarios posteriores, como las cachés web, interpreten la respuesta como dos respuestas HTTP distintas. Este comportamiento puede explotarse para envenenar cachés o entregar contenido malicioso a los usuarios.

Por ejemplo, el evaluador proporciona el siguiente valor para el parámetro `interface`:

`advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>`

La respuesta resultante de la aplicación vulnerable podría ser:

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 35

<html>Sorry,%20System%20Down</html>
```

Una caché web que procese esta respuesta podría interpretarla como dos respuestas distintas. Si el atacante envía inmediatamente una solicitud posterior para `/index.html`, la caché podría asociar esa solicitud con la segunda respuesta y almacenarla. Como resultado, todos los usuarios posteriores que accedan a `victim.com/index.html` a través de esa caché podrían recibir contenido controlado por el atacante.

Como alternativa, el atacante podría inyectar una carga útil de JavaScript para llevar a cabo un ataque de cross-site scripting contra los usuarios a los que sirve la caché envenenada. Aunque la vulnerabilidad reside en la aplicación, los objetivos principales son sus usuarios.

Para identificar este problema, los evaluadores deben localizar todas las entradas controladas por el usuario que influyan en los encabezados de respuesta HTTP y verificar si es posible inyectar secuencias CRLF.

Los encabezados de respuesta más comúnmente asociados con la división de respuestas HTTP incluyen:

- `Location`
- `Set-Cookie`

La explotación exitosa en escenarios reales podría requerir tener en cuenta otros factores:

- El evaluador podría necesitar crear encabezados de respuesta adecuados para el almacenamiento en caché (por ejemplo, con `Last-Modified` establecido en una fecha futura) y, posiblemente, invalidar las entradas de caché existentes mediante encabezados como `Pragma: no-cache`.
- Es posible que las aplicaciones filtren los caracteres CRLF, pero permitan codificaciones o representaciones alternativas de caracteres que, en ocasiones, pueden aprovecharse para eludir la validación de entradas.
- Algunas plataformas codifican con URL partes de los encabezados de respuesta (como la ruta del encabezado `Location`), pero dejan sin codificar la cadena de consulta, lo que permite inyectar datos a través de componentes específicos de la URL.

Para obtener un análisis más detallado de esta clase de ataque y conocer otros escenarios de explotación, consulte los documentos técnicos que se indican en la sección Referencias.

### Pruebas de caja gris

En un escenario de pruebas de caja gris, conocer la arquitectura de la aplicación y el comportamiento del servidor mejora la fiabilidad de la explotación.

Diferentes servidores o intermediarios pueden determinar los límites de los mensajes de distintas maneras (por ejemplo, mediante búferes de tamaño fijo), lo que requiere desplazamientos o relleno precisos. Cuando los parámetros vulnerables se transmiten mediante GET, los límites de longitud de la URL pueden truncar las cargas útiles. Los evaluadores deben identificar puntos de inyección o métodos de solicitud alternativos, como POST, para tener un mayor control sobre la longitud y la posición de la carga útil.

## Mitigación

Asegúrese de que las entradas proporcionadas por el usuario nunca se incluyan en encabezados HTTP sin una validación y un saneamiento estrictos.

- **Validación de entradas:** Rechace o elimine las entradas que contengan caracteres de retorno de carro (`\r`, `%0d`) o salto de línea (`\n`, `%0a`) antes de utilizarlas en encabezados HTTP.
- **Codificación de URL:** Si la entrada forma parte de una URL (por ejemplo, en un encabezado `Location`), asegúrese de que esté correctamente codificada con URL para evitar que los caracteres de control se interpreten como delimitadores.
- **Utilice frameworks seguros:** Utilice las funciones integradas del framework para establecer encabezados (por ejemplo, `setHeader()`, `addHeader()`) en lugar de construir manualmente cadenas de respuesta HTTP sin procesar. Por lo general, los entornos modernos bloquean de forma predeterminada la inyección de encabezados.

## Herramientas

- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- [CRLFuzz](https://github.com/dwisiswant0/crlfuzz) - Una herramienta diseñada específicamente para buscar vulnerabilidades CRLF.
- [Nuclei](https://github.com/projectdiscovery/nuclei) - Puede utilizarse con plantillas específicas para detectar patrones de inyección CRLF.

## Referencias

- [Amit Klein, "Divide and Conquer: HTTP Response Splitting, Web Cache Poisoning Attacks, and Related Topics"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper.html)
- [Amit Klein: "HTTP Message Splitting, Smuggling and Other Animals"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit Klein: "HTTP Request Smuggling - ERRATA (the IIS 48K buffer phenomenon)"](https://web.archive.org/web/20210614052317/https://www.securityfocus.com/archive/1/411418)
- [Amit Klein: "HTTP Response Smuggling"](https://web.archive.org/web/20210126213458/https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin: "HTTP Request Smuggling"](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)