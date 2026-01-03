# Huella Digital del Servidor Web

|ID          |
|------------|
|WSTG-INFO-02|

## Resumen

La huella digital del servidor web es la tarea de identificar el tipo y versión del servidor web que un objetivo está ejecutando. Mientras que la huella digital del servidor web a menudo está encapsulada en herramientas de pruebas automatizadas, es importante para investigadores entender los fundamentos de cómo estas herramientas intentan identificar software, y por qué esto es útil.

Descubrir con precisión el tipo de servidor web que una aplicación ejecuta puede habilitar a testers de seguridad a determinar si la aplicación es vulnerable a ataque. En particular, servidores ejecutando versiones más viejas de software sin parches de seguridad actualizados pueden ser susceptibles a exploits específicos de versión conocidos.

## Objetivos de Prueba

- Determinar la versión y tipo de un servidor web ejecutándose para habilitar más descubrimiento de cualquier vulnerabilidad conocida.

## Cómo Probar

Técnicas usadas para huella digital del servidor web incluyen [banner grabbing](https://en.wikipedia.org/wiki/Banner_grabbing), eliciting respuestas a solicitudes malformadas, y usando herramientas automatizadas para realizar scans más robustos que usan una combinación de tácticas. La premisa fundamental en la que todas estas técnicas operan es la misma. Todas se esfuerzan por elicitar alguna respuesta del servidor web que puede entonces ser comparada a una base de datos de respuestas y comportamientos conocidos, y así matched a un tipo de servidor conocido.

### Banner Grabbing

Un banner grab se realiza enviando una solicitud HTTP al servidor web y examinando su [response header](https://developer.mozilla.org/en-US/docs/Glossary/Response_header). Esto puede ser logrado usando una variedad de herramientas, incluyendo `telnet` para solicitudes HTTP, o `openssl` para solicitudes sobre TLS/SSL.

Por ejemplo, aquí está la respuesta a una solicitud enviada a un servidor Apache.

```http
HTTP/1.1 200 OK
Date: Thu, 05 Sep 2019 17:42:39 GMT
Server: Apache/2.4.41 (Unix)
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
ETag: "75-591d1d21b6167"
Accept-Ranges: bytes
Content-Length: 117
Connection: close
Content-Type: text/html
...
```

Aquí está otra respuesta, esta vez enviada por nginx.

```http
HTTP/1.1 200 OK
Server: nginx/1.17.3
Date: Thu, 05 Sep 2019 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
...
```

Aquí está cómo se ve una respuesta enviada por lighttpd.

```sh
HTTP/1.0 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4192788355"
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Content-Length: 117
Connection: close
Date: Thu, 05 Sep 2019 17:57:57 GMT
Server: lighttpd/1.4.54
```

En estos ejemplos, el tipo de servidor y versión está claramente expuesto. Sin embargo, aplicaciones conscientes de seguridad pueden ofuscar su información de servidor modificando el header. Por ejemplo, aquí está un excerpt de la respuesta a una solicitud para un sitio con un header modificado:

```sh
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

En casos donde la información del servidor está obscured, testers pueden adivinar el tipo de servidor basado en el ordering de los campos del header. Nota que en el ejemplo de Apache arriba, los campos siguen este orden:

- Date
- Server
- Last-Modified
- ETag
- Accept-Ranges
- Content-Length
- Connection
- Content-Type

Sin embargo, en ambos ejemplos de nginx y servidor obscured, los campos en común siguen este orden:

- Server
- Date
- Content-Type

Testers pueden usar esta información para adivinar que el servidor obscured es nginx. Sin embargo, considerando que un número de diferentes servidores web pueden compartir el mismo field ordering y campos pueden ser modificados o removidos, este método no es definitivo.

### Sending Malformed Requests

Servidores web pueden ser identificados examinando sus respuestas de error, y en los casos donde no han sido customized, sus páginas de error default. Una forma de compelir a un servidor a presentar estas es enviando solicitudes intencionalmente incorrectas o malformadas.

Por ejemplo, aquí está la respuesta a una solicitud para la versión HTTP no-existente `SANTA CLAUS` desde un servidor Apache.

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

Aquí está la respuesta a la misma solicitud desde nginx.

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

Aquí está la respuesta a la misma solicitud desde lighttpd.

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

Como páginas de error default ofrecen muchos factores diferenciadores entre tipos de servidores web, su examen puede ser un método efectivo para fingerprinting incluso cuando campos de header del servidor están obscured.  
Además, este [resource](https://0xdf.gitlab.io/cheatsheets/404) puede ser handy, especialmente cuando te encuentras con páginas de error default que no divulgan el tipo de servidor web.

### Using Automated Scanning Tools

Como se declaró anteriormente, la huella digital del servidor web a menudo está incluida como una funcionalidad de herramientas de scanning automatizadas. Estas herramientas son capaces de hacer solicitudes similares a las demostradas arriba, así como enviar otras probes más server-specific. Herramientas automatizadas pueden comparar respuestas de servidores web mucho más rápido que pruebas manuales, y utilizan grandes bases de datos de respuestas conocidas para intentar identificación de servidor. Por estas razones, herramientas automatizadas son más likely de producir resultados precisos.

Aquí hay algunas herramientas de scan comúnmente usadas que incluyen funcionalidad de huella digital de servidor web.

- [Netcraft](https://toolbar.netcraft.com/site_report), una herramienta online que scans sitios para información, incluyendo detalles de servidor web.
- [Nikto](https://github.com/sullo/nikto), una herramienta de scanning Open Source command-line.
- [Nmap](https://nmap.org/), una herramienta Open Source command-line que también tiene una GUI, [Zenmap](https://nmap.org/zenmap/).

## Remediación

Mientras que información de servidor expuesta no es necesariamente en sí misma una vulnerabilidad, es información que puede asistir a atacantes en exploiting otras vulnerabilidades que pueden existir. Información de servidor expuesta también puede llevar a atacantes a encontrar vulnerabilidades específicas de versión de servidor que pueden ser usadas para exploit servidores sin parches. Por esta razón se recomienda que algunas precauciones sean tomadas. Estas acciones incluyen:

- Ofuscando información de servidor web en headers, tales como con el [mod_headers module](https://httpd.apache.org/docs/current/mod/mod_headers.html) de Apache.
- Usando un hardened [reverse proxy server](https://en.wikipedia.org/wiki/Proxy_server#Reverse_proxies) para crear una capa adicional de seguridad entre el servidor web y el internet.
- Asegurando que servidores web sean mantenidos up-to-date con el último software y parches de seguridad.