# Pruebas de HTTP Response Splitting

|ID          |
|------------|
|WSTG-INPV-15|

## Resumen

HTTP Response Splitting es una vulnerabilidad que ocurre cuando una aplicación incorpora entrada de usuario no saneada en encabezados de respuesta HTTP, permitiendo a un atacante inyectar caracteres Carriage Return (CR) y Line Feed (LF). Como resultado, una sola respuesta HTTP puede ser interpretada como múltiples respuestas distintas por clientes o sistemas intermediarios.

La explotación exitosa de HTTP Response Splitting puede llevar a varios impactos, incluyendo web cache poisoning, cross-site scripting (XSS), content spoofing, session fixation, u otros ataques del lado del cliente, dependiendo de cómo se procese la respuesta inyectada.

Esta sección se enfoca exclusivamente en identificar y probar vulnerabilidades de HTTP Response Splitting en la capa de aplicación. HTTP Request Smuggling, que depende de inconsistencias de análisis entre múltiples agentes HTTP, se cubre en un capítulo separado.

## Objetivos de Prueba

- Identificar entrada controlada por el usuario que se refleja en encabezados de respuesta HTTP.
- Evaluar si los caracteres CR (`\r`) y LF (`\n`) pueden inyectarse en encabezados de respuesta.
- Determinar el impacto potencial de ataques exitosos de HTTP Response Splitting, tales como cache poisoning o explotación del lado del cliente.

## Cómo Probar

### Pruebas de Caja Negra

Algunas aplicaciones web usan entrada proporcionada por el usuario para generar los valores de ciertos encabezados de respuesta HTTP. Un ejemplo común es la lógica de redirección, donde la URL de destino se deriva de un parámetro de solicitud.

Por ejemplo, asumir que se pide a un usuario elegir entre una interfaz estándar o avanzada. La opción seleccionada se pasa como parámetro y se refleja en un encabezado de respuesta de redirección.

Si el parámetro `interface` tiene el valor `advanced`, la aplicación podría responder con:

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
```

Cuando el navegador recibe esta respuesta, sigue la URL especificada en el encabezado `Location`. Sin embargo, si la aplicación no valida o sanea apropiadamente la entrada del usuario, un atacante podría inyectar la secuencia `%0d%0a`, representando caracteres CRLF usados para separar líneas de encabezado HTTP.

Al inyectar secuencias CRLF, un tester podría causar que la respuesta se interprete como dos respuestas HTTP separadas por clientes downstream o sistemas intermediarios, tales como cachés web. Este comportamiento puede explotarse para envenenar cachés o entregar contenido malicioso a usuarios.

Por ejemplo, el tester proporciona el siguiente valor para el parámetro `interface`:

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

Una caché web procesando esta respuesta podría interpretarla como dos respuestas distintas. Si el atacante emite inmediatamente una solicitud subsiguiente por `/index.html`, la caché podría asociar esa solicitud con la segunda respuesta y almacenarla. Como resultado, todos los usuarios subsiguientes que accedan a `victim.com/index.html` a través de esa caché podrían recibir el contenido controlado por el atacante.

Alternativamente, el atacante podría inyectar un payload de JavaScript para realizar un ataque de cross-site scripting contra usuarios servidos por la caché envenenada. Aunque la vulnerabilidad reside en la aplicación, los objetivos primarios son sus usuarios.

Para identificar este problema, los testers deberían localizar toda entrada controlada por el usuario que influya en encabezados de respuesta HTTP y verificar si se pueden inyectar secuencias CRLF.

Los encabezados de respuesta más comúnmente asociados con HTTP Response Splitting incluyen:

- `Location`
- `Set-Cookie`

La explotación exitosa en escenarios del mundo real podría requerir consideración cuidadosa de factores adicionales:

- El tester podría necesitar diseñar encabezados de respuesta adecuados para caching (por ejemplo, `Last-Modified` establecido a una fecha futura) y potencialmente invalidar entradas de caché existentes usando encabezados tales como `Pragma: no-cache`.
- Las aplicaciones podrían filtrar caracteres CRLF pero permitir codificaciones alternativas o representaciones de caracteres, lo cual a veces puede aprovecharse para evadir validación de entrada.
- Algunas plataformas URL-codifican porciones de encabezados de respuesta (tales como la ruta en el encabezado `Location`) mientras dejan la cadena de consulta sin codificar, permitiendo inyección a través de componentes específicos de la URL.

Para una discusión más profunda de esta clase de ataque y escenarios de explotación adicionales, referirse a los whitepapers listados en la sección de Referencias.

### Pruebas de Caja Gris

En un escenario de pruebas de caja gris, el conocimiento de la arquitectura de aplicación y el comportamiento del servidor mejora la fiabilidad de explotación.

Diferentes servidores o intermediarios podrían determinar límites de mensaje de manera diferente (por ejemplo, usando buffers de tamaño fijo), requiriendo offsets precisos o padding. Cuando los parámetros vulnerables se transmiten vía GET, los límites de longitud de URL podrían truncar payloads. Los testers deberían identificar puntos de inyección alternativos o métodos de solicitud, tales como POST, para ganar mejor control sobre la longitud y posicionamiento del payload.

## Remediación

Asegurar que la entrada proporcionada por el usuario nunca se coloque en encabezados HTTP sin validación y saneamiento estrictos.

- **Validación de Entrada:** Rechazar o stripear entrada que contenga caracteres Carriage Return (`\r`, `%0d`) o Line Feed (`\n`, `%0a`) antes de que se use en encabezados HTTP.
- **Codificación URL:** Si la entrada es parte de una URL (por ejemplo, en un encabezado `Location`), asegurar que esté propiamente URL-codificada para prevenir que caracteres de control se interpreten como delimitadores.
- **Usar Frameworks Seguros:** Utilizar funciones de framework integradas para establecer encabezados (por ejemplo, `setHeader()`, `addHeader()`) en lugar de construir manualmente cadenas crudas de respuesta HTTP. Los entornos modernos típicamente bloquean la inyección de encabezados por defecto.

## Herramientas

- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- [CRLFuzz](https://github.com/dwisiswant0/crlfuzz) - Una herramienta diseñada específicamente para escanear vulnerabilidades CRLF.
- [Nuclei](https://github.com/projectdiscovery/nuclei) - Puede usarse con plantillas específicas para detectar patrones de inyección CRLF.

## Referencias

- [Amit Klein, "Divide and Conquer: HTTP Response Splitting, Web Cache Poisoning Attacks, and Related Topics"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper.html)
- [Amit Klein: "HTTP Message Splitting, Smuggling and Other Animals"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit Klein: "HTTP Request Smuggling - ERRATA (the IIS 48K buffer phenomenon)"](https://web.archive.org/web/20210614052317/https://www.securityfocus.com/archive/1/411418)
- [Amit Klein: "HTTP Response Smuggling"](https://web.archive.org/web/20210126213458/https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin: "HTTP Request Smuggling"](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
