# Identificación del servidor web

|ID          |
|------------|
|WSTG-INFO-02|

## Resumen

La identificación del servidor web consiste en determinar el tipo y la versión del servidor web que ejecuta un objetivo. Aunque suele estar integrada en herramientas de pruebas automatizadas, es importante que los investigadores comprendan los fundamentos de cómo estas herramientas intentan identificar el software y por qué resulta útil.

Identificar con precisión el tipo de servidor web que ejecuta una aplicación puede permitir a los evaluadores de seguridad determinar si la aplicación es vulnerable a ataques. En particular, los servidores que ejecutan versiones antiguas de software sin los parches de seguridad más recientes pueden ser susceptibles a exploits conocidos específicos de esa versión.

## Objetivos de las pruebas

- Determinar la versión y el tipo de un servidor web en ejecución para facilitar la búsqueda de vulnerabilidades conocidas.

## Cómo realizar las pruebas

Entre las técnicas para identificar un servidor web se incluyen la [obtención de banners](https://en.wikipedia.org/wiki/Banner_grabbing), provocar respuestas mediante solicitudes malformadas y usar herramientas automatizadas para realizar análisis más exhaustivos que combinan varias tácticas. Todas estas técnicas se basan en la misma premisa fundamental: buscan obtener alguna respuesta del servidor web que luego pueda compararse con una base de datos de respuestas y comportamientos conocidos y, de ese modo, asociarse con un tipo de servidor conocido.

### Obtención de banners

La obtención de un banner se realiza enviando una solicitud HTTP al servidor web y examinando su [encabezado de respuesta](https://developer.mozilla.org/en-US/docs/Glossary/Response_header). Esto puede hacerse con `curl -I` (que envía una solicitud `HEAD` y muestra solo los encabezados de respuesta), con [httpx](https://github.com/projectdiscovery/httpx) para sondear listas extensas de hosts a gran escala, o con `openssl s_client` para realizar solicitudes mediante TLS/SSL.

Por ejemplo, el siguiente comando recupera únicamente los encabezados de respuesta de un servidor:

```bash
curl -I https://example.org
```

A continuación, se muestra la respuesta a una solicitud enviada a un servidor Apache.

```http
HTTP/1.1 200 OK
Date: Thu, 05 Sep 2019 17:42:39 GMT
Server: Apache/2.4.43 (Unix)
Last-Modified: Thu, 05 Sep 2026 17:40:42 GMT
ETag: "75-591d1d21b6167"
Accept-Ranges: bytes
Content-Length: 117
Connection: close
Content-Type: text/html
...
```

A continuación se muestra otra respuesta, esta vez enviada por Nginx.

```http
HTTP/1.1 200 OK
Server: nginx/1.31.5
Date: Thu, 05 Sep 2026 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2026 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
...
```

Así es una respuesta enviada por lighttpd.

```http
HTTP/1.1 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4192788355"
Last-Modified: Thu, 05 Sep 2026 17:40:42 GMT
Content-Length: 117
Connection: close
Date: Thu, 05 Sep 2026 17:57:57 GMT
Server: lighttpd/1.4.85
...
```

En estos ejemplos, se muestran claramente el tipo y la versión del servidor. Sin embargo, las aplicaciones que prestan atención a la seguridad pueden ocultar la información del servidor modificando el encabezado. Por ejemplo, este es un fragmento de la respuesta a una solicitud para un sitio con un encabezado modificado:

```sh
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

Cuando la información del servidor está oculta, los evaluadores pueden intentar deducir el tipo de servidor a partir del orden de los campos del encabezado. Observe que, en el ejemplo anterior de Apache, los campos aparecen en este orden:

- Date
- Server
- Last-Modified
- ETag
- Accept-Ranges
- Content-Length
- Connection
- Content-Type

Sin embargo, tanto en el ejemplo de Nginx como en el del servidor con la información oculta, los campos comunes aparecen en este orden:

- Server
- Date
- Content-Type

Los evaluadores pueden usar esta información para deducir que el servidor cuya información está oculta es Nginx. Sin embargo, dado que varios servidores web distintos pueden compartir el mismo orden de campos y que estos pueden modificarse o eliminarse, este método no es concluyente.

### Envío de solicitudes malformadas

Los servidores web pueden identificarse examinando sus respuestas de error y, si no se han personalizado, sus páginas de error predeterminadas. Una forma de obligar a un servidor a presentarlas consiste en enviar solicitudes intencionalmente incorrectas o malformadas.

Por ejemplo, esta es la respuesta de un servidor Apache a una solicitud que usa la versión HTTP inexistente `SANTA CLAUS`.

```sh
GET / SANTA CLAUS/1.1


HTTP/1.1 400 Bad Request
Date: Fri, 06 Sep 2019 19:21:01 GMT
Server: Apache/2.4.41 (Unix)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
```

Esta es la respuesta de Nginx a la misma solicitud.

```sh
GET / SANTA CLAUS/1.1


<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.3</center>
</body>
</html>
```

Esta es la respuesta de lighttpd a la misma solicitud.

```sh
GET / SANTA CLAUS/1.1


HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Sun, 08 Sep 2019 21:56:17 GMT
Server: lighttpd/1.4.54

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "https://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="https://www.w3.org/1999/xhtml/" xml:lang="en" lang="en">
 <head>
  <title>400 Bad Request</title>
 </head>
 <body>
  <h1>400 Bad Request</h1>
 </body>
</html>
```

Las páginas de error predeterminadas ofrecen muchos elementos que permiten diferenciar los tipos de servidores web; por eso, examinarlas puede ser un método eficaz para identificarlos incluso cuando los campos del encabezado del servidor están ocultos.  
Además, este [recurso](https://0xdf.gitlab.io/cheatsheets/404) puede resultar útil, especialmente al encontrar páginas de error predeterminadas que no revelan el tipo de servidor web.

### Uso de herramientas de análisis automatizado

Como se indicó anteriormente, la identificación del servidor web suele estar incluida como una funcionalidad de las herramientas de análisis automatizado. Estas herramientas pueden realizar solicitudes similares a las mostradas anteriormente, así como enviar otras sondas más específicas para cada servidor. Las herramientas automatizadas pueden comparar las respuestas de los servidores web mucho más rápido que las pruebas manuales y utilizan grandes bases de datos de respuestas conocidas para intentar identificar el servidor. Por estos motivos, es más probable que las herramientas automatizadas produzcan resultados precisos.

A continuación se enumeran algunas herramientas de análisis de uso común que incluyen funciones para identificar servidores web.

- [Netcraft Site Report](https://sitereport.netcraft.com/), una herramienta en línea que analiza sitios para recopilar información, incluidos datos sobre el servidor web.
- [Nikto](https://github.com/sullo/nikto), una herramienta de análisis de código abierto para la línea de comandos.
- [Nmap](https://nmap.org/), una herramienta de código abierto para la línea de comandos que también dispone de una interfaz gráfica, [Zenmap](https://nmap.org/zenmap/).
- [WhatWeb](https://github.com/urbanadventurer/WhatWeb) y [httpx](https://github.com/projectdiscovery/httpx) (`-tech-detect`), que identifican el software del servidor web junto con otras tecnologías (consulte [Identificación del framework de la aplicación web](08-Fingerprint_Web_Application_Framework.md) e [Identificación de la aplicación web](09-Fingerprint_Web_Application.md) para obtener más información sobre su uso para detectar el conjunto de tecnologías).

## Mitigación

La información expuesta sobre el servidor no constituye necesariamente una vulnerabilidad por sí sola, pero puede ayudar a los atacantes a aprovechar otras vulnerabilidades existentes. También puede ayudarles a encontrar vulnerabilidades del servidor específicas de una versión que permitan atacar servidores sin parches. Por este motivo, se recomienda tomar ciertas precauciones, entre ellas:

- Ocultar la información del servidor web en los encabezados, por ejemplo, mediante el módulo [mod_headers](https://httpd.apache.org/docs/current/mod/mod_headers.html) de Apache.
- Usar un [servidor proxy inverso](https://en.wikipedia.org/wiki/Proxy_server#Reverse_proxies) reforzado para crear una capa de seguridad adicional entre el servidor web e Internet.
- Asegurarse de que los servidores web estén actualizados con el software y los parches de seguridad más recientes.