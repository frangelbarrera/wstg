# Atributos de las cookies

|ID          |
|------------|
|WSTG-SESS-02|

## Resumen

Las cookies web (en adelante, cookies) suelen ser un vector de ataque clave para usuarios maliciosos (que normalmente tienen como objetivo a otros usuarios), por lo que la aplicación siempre debe tomar las precauciones necesarias para protegerlas.

HTTP es un protocolo sin estado, lo que significa que no mantiene referencias a las solicitudes enviadas por un mismo usuario. Para solucionar este problema, se crearon las sesiones y se añadieron a las solicitudes HTTP. Como se explica en [pruebas del almacenamiento del navegador](../11-Client-side/12-Browser_Storage.md), los navegadores cuentan con multitud de mecanismos de almacenamiento. En esa sección de la guía se analiza cada uno en detalle.

El mecanismo de almacenamiento de sesiones más utilizado en los navegadores es el almacenamiento de cookies. El servidor puede establecer cookies al incluir una cabecera [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie) en la respuesta HTTP, o bien estas pueden establecerse mediante JavaScript. Las cookies pueden utilizarse por diversos motivos, como:

- gestión de sesiones
- personalización
- seguimiento

Para proteger los datos de las cookies, la industria ha desarrollado mecanismos que ayudan a restringirlas y limitar su superficie de ataque. Con el tiempo, las cookies se han convertido en un mecanismo de almacenamiento preferido para las aplicaciones web, ya que permiten gran flexibilidad de uso y protección.

Los mecanismos para proteger las cookies son:

- [Atributos de las cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Creating_cookies)
- [Prefijos de las cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Cookie_prefixes)

## Objetivos de la prueba

- Comprobar que las cookies tengan la configuración de seguridad adecuada.

## Cómo probar

A continuación, se describen todos los atributos y prefijos. El evaluador debe comprobar que la aplicación los utilice correctamente. Las cookies pueden revisarse mediante un [proxy de interceptación](#intercepting-proxy) o examinando el almacén de cookies del navegador.

### Atributos de las cookies

#### Atributo Secure

El atributo [`Secure`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie#Secure) indica al navegador que solo envíe la cookie si la solicitud se realiza a través de un canal seguro, como `HTTPS`. Esto ayuda a proteger la cookie para que no se envíe en solicitudes sin cifrar. Si se puede acceder a la aplicación tanto mediante `HTTP` como mediante `HTTPS`, un atacante podría redirigir al usuario para que envíe su cookie como parte de solicitudes no protegidas.

#### Atributo HttpOnly

El atributo [`HttpOnly`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie#HttpOnly) ayuda a prevenir ataques como la filtración de sesiones, ya que impide que se acceda a la cookie mediante un script del lado del cliente, como JavaScript.

> Esto no limita toda la superficie de ataque de los ataques XSS, ya que un atacante aún podría enviar solicitudes en nombre del usuario, pero limita considerablemente el alcance de los vectores de ataque XSS.

#### Atributo Domain

El atributo [`Domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Scope_of_cookies) se utiliza para comparar el dominio de la cookie con el dominio del servidor al que se envía la solicitud HTTP. Si el dominio coincide o es un subdominio, se comprobará a continuación el atributo [`path`](#path-attribute).

Solo los hosts que pertenecen al dominio especificado pueden establecer cookies para ese dominio. Además, el atributo `domain` no puede ser un dominio de nivel superior (como `.gov` o `.com`), lo que impide que los servidores establezcan cookies arbitrarias para otro dominio (por ejemplo, establecer una cookie para `owasp.org`). Si no se establece el atributo `domain`, se utiliza como valor predeterminado el nombre de host del servidor que generó la cookie.

Por ejemplo, si una aplicación en `app.mydomain.com` establece una cookie sin definir el atributo domain, esta se volverá a enviar en todas las solicitudes posteriores a `app.mydomain.com`, pero no a sus subdominios (como `hacker.app.mydomain.com`) ni a `otherapp.mydomain.com`. (Sin embargo, las versiones antiguas de Edge/IE se comportan de manera distinta y _sí_ envían estas cookies a los subdominios). Si un desarrollador quisiera flexibilizar esta restricción, podría establecer el atributo `domain` en `mydomain.com`. En ese caso, la cookie se enviaría en todas las solicitudes a `app.mydomain.com` y a los subdominios de `mydomain.com`, como `hacker.app.mydomain.com` e incluso `bank.mydomain.com`. Si hubiera un servidor vulnerable en un subdominio (por ejemplo, `otherapp.mydomain.com`) y el atributo `domain` se hubiera establecido con demasiada amplitud (por ejemplo, en `mydomain.com`), el servidor vulnerable podría utilizarse para recolectar cookies (como tokens de sesión) de todo el ámbito de `mydomain.com`.

#### Atributo Path

El atributo [`Path`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Scope_of_cookies) desempeña un papel importante a la hora de establecer el alcance de las cookies, junto con [`domain`](#domain-attribute). Además del dominio, se puede especificar la ruta URL para la que la cookie es válida. Si coinciden el dominio y la ruta, la cookie se enviará en la solicitud. Al igual que con el atributo domain, si el atributo path se establece con demasiada amplitud, la aplicación podría quedar expuesta a ataques de otras aplicaciones del mismo servidor. Por ejemplo, si el atributo path se establece en la raíz del servidor web `/`, las cookies de la aplicación se enviarán a todas las aplicaciones del mismo dominio (si hay varias aplicaciones alojadas en el mismo servidor). Algunos ejemplos de rutas para varias aplicaciones en el mismo servidor:

- `path=/bank`
- `path=/private`
- `path=/docs`
- `path=/docs/admin`

#### Atributo Expires

El atributo [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Permanent_cookies) se utiliza para:

- establecer cookies persistentes
- limitar la duración de una sesión si se prolonga demasiado
- eliminar una cookie de manera forzada estableciendo una fecha pasada

A diferencia de las [cookies de sesión](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Session_cookies), el navegador utilizará las cookies persistentes hasta que caduquen. Una vez que se supera la fecha de expiración establecida, el navegador eliminará la cookie.

#### Atributo SameSite

El atributo [`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#SameSite_cookies) permite especificar si una cookie debe enviarse junto con solicitudes entre sitios. Esta característica permite al servidor mitigar el riesgo de filtración de información entre orígenes. En algunos casos, también se utiliza como estrategia de reducción de riesgos (o mecanismo de defensa en profundidad) para prevenir ataques de [falsificación de solicitudes entre sitios](05-Cross_Site_Request_Forgery.md). Este atributo puede configurarse en tres modos distintos:

- `Strict`
- `Lax`
- `None`

##### Valor Strict

El valor `Strict` es el uso más restrictivo de `SameSite`: permite que el navegador envíe la cookie únicamente en un contexto de primera parte sin navegación de nivel superior. En otras palabras, los datos asociados a la cookie solo se enviarán en solicitudes que coincidan con el sitio actual que se muestra en la barra de direcciones del navegador. La cookie no se enviará en solicitudes generadas por sitios de terceros. Este valor se recomienda especialmente para acciones realizadas en el mismo dominio. Sin embargo, puede presentar limitaciones con algunos sistemas de gestión de sesiones y afectar negativamente a la experiencia de navegación del usuario. Como el navegador no enviaría la cookie en ninguna solicitud generada desde un dominio de terceros o un correo electrónico, el usuario tendría que volver a iniciar sesión aunque ya tuviera una sesión autenticada.

##### Valor Lax

El valor `Lax` es menos restrictivo que `Strict`. Permite enviar la cookie en navegaciones de nivel superior entre sitios que utilizan un método HTTP «seguro» (en la práctica, `GET`), por ejemplo, cuando un usuario sigue un enlace desde otro sitio. __No__ envía la cookie en solicitudes de subrecursos entre sitios, como las que generan las etiquetas `<img>`, `<script>` o `<iframe>`, `fetch()` o `XMLHttpRequest`, ni en navegaciones entre sitios que utilizan `POST` u otros métodos no seguros.

Desde principios de 2020, los navegadores basados en Chromium (Chrome, Edge, Opera) tratan de forma predeterminada cualquier cookie establecida __sin__ un atributo `SameSite` explícito como `SameSite=Lax`. Firefox y Safari no aplican ese valor predeterminado. En esos navegadores, una cookie que omita `SameSite` sigue pudiendo enviarse en solicitudes entre sitios, sujeta en cambio a sus propios controles de prevención del seguimiento y de cookies de terceros (la Protección mejorada contra el rastreo / Protección total de cookies de Firefox y la Prevención inteligente del rastreo de Safari). Ten en cuenta que omitir el atributo no equivale a establecer [`SameSite=None`](#none-value), que es un valor explícito que además requiere el atributo [`Secure`](#secure-attribute) para que se acepte. Estos controles de prevención del seguimiento son mecanismos de privacidad, no sustituyen a establecer `SameSite`, y se comportan de manera distinta a una restricción explícita `SameSite=Lax`. Los evaluadores no deben asumir que una cookie está protegida contra CSRF solo porque parezca que una solicitud entre sitios se bloquea en Firefox o Safari: deben verificar la cabecera `Set-Cookie` real en lugar de deducir la protección a partir del comportamiento observado del navegador.

##### Valor None

El valor `None` especifica que el navegador enviará la cookie en todos los contextos, incluidas las solicitudes entre sitios (el comportamiento normal antes de la implementación de `SameSite`). Si se establece `Samesite=None`, también debe establecerse el atributo Secure; de lo contrario, los navegadores modernos ignorarán el atributo SameSite, _por ejemplo,_ `SameSite=None; Secure`.

### Prefijos de las cookies

Por diseño, las cookies no permiten garantizar la integridad ni la confidencialidad de la información almacenada en ellas. Estas limitaciones impiden que el servidor pueda confiar en cómo se establecieron los atributos de una cookie determinada al crearla. Para proporcionar estas características a los servidores de manera compatible con versiones anteriores, la industria introdujo el concepto de [`Prefijos de nombre de cookie`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-prefixes-00), que permite incluir esos detalles en el nombre de la cookie.

#### Prefijo Host

El prefijo [`__Host-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Cookie_prefixes) exige que las cookies cumplan las siguientes condiciones:

  1. La cookie debe establecerse con el [atributo `Secure`](#secure-attribute).
  2. La cookie debe establecerse desde un URI que el agente de usuario considere seguro.
  3. Solo debe enviarse al host que estableció la cookie y NO DEBE incluir ningún [atributo `Domain`](#domain-attribute).
  4. La cookie debe establecerse con el [atributo `Path`](#path-attribute) con el valor `/`, para que se envíe con cada solicitud al host.

Por este motivo, se aceptaría la cookie `Set-Cookie: __Host-SID=12345; Secure; Path=/`, mientras que se rechazaría siempre cualquiera de las siguientes:
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.example`
`Set-Cookie: __Host-SID=12345; Domain=site.example; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.example; Path=/`

#### Prefijo Secure

El prefijo [`__Secure-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies#Cookie_prefixes) es menos restrictivo y puede añadirse al nombre de la cookie mediante la cadena `__Secure-`, que distingue entre mayúsculas y minúsculas. Se espera que toda cookie cuyo nombre coincida con el prefijo `__Secure-` cumpla las siguientes condiciones:

  1. La cookie debe establecerse con el atributo `Secure`.
  2. La cookie debe establecerse desde un URI que el agente de usuario considere seguro.

### Prácticas recomendadas

Los atributos y prefijos deben aplicarse según las necesidades de la aplicación y el funcionamiento previsto de la cookie. Cuanto más restringida esté la cookie, mejor.

Con todo esto en cuenta, la configuración más segura de los atributos de una cookie sería: `Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict`.

## Herramientas

### Proxy de interceptación

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Web Proxy Burp Suite](https://portswigger.net)

### Complementos del navegador

- Herramientas para desarrolladores del navegador (panel Application o Storage)
- ["FireSheep" para FireFox](https://github.com/codebutler/firesheep)
- ["Cookie-Editor" para Chrome](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
- ["Cookiebro - Cookie Manager" para FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## Referencias

- [RFC 2965 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 - Hypertext Transfer Protocol - HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Same-Site Cookies - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [The important "expires" attribute of Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [HttpOnly Session ID in URL and Page Body](https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)
- [Internet Explorer Cookie Internals (FAQ)](https://learn.microsoft.com/en-gb/archive/blogs/ieinternals/internet-explorer-cookie-internals-faq)