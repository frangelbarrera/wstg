# Pruebas de atributos de cookies

|ID          |
|------------|
|WSTG-SESS-02|

## Resumen

Las cookies web (aquí denominadas cookies) son a menudo un vector de ataque clave para usuarios maliciosos (generalmente dirigidos a otros usuarios) y la aplicación siempre debe tomar las precauciones necesarias para proteger las cookies.

HTTP es un protocolo sin estado, lo que significa que no mantiene ninguna referencia a las solicitudes enviadas por el mismo usuario. Para solucionar este problema, se crearon sesiones y se anexaron a las solicitudes HTTP. Los navegadores, como se discute en [pruebas de almacenamiento del navegador](../11-Client-side_Testing/12-Testing_Browser_Storage.md), contienen una multitud de mecanismos de almacenamiento. En esa sección de la guía, cada uno se discute exhaustivamente.

El mecanismo de almacenamiento de sesiones más utilizado en los navegadores es el almacenamiento de cookies. Las cookies pueden ser configuradas por el servidor, incluyendo una cabecera [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) en la respuesta HTTP o mediante JavaScript. Las cookies pueden utilizarse por una multitud de razones, tales como:

- gestión de sesiones
- personalización
- seguimiento

Para asegurar los datos de las cookies, la industria ha desarrollado medios para bloquear estas cookies y limitar su superficie de ataque. Con el tiempo, las cookies se han convertido en un mecanismo de almacenamiento preferido para las aplicaciones web, ya que permiten una gran flexibilidad en el uso y la protección.

Los medios para proteger las cookies son:

- [Atributos de cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Creating_cookies)
- [Prefijos de cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)

## Objetivos de la prueba

- Asegurar que se establece la configuración de seguridad adecuada para las cookies.

## Cómo probar

A continuación, se describe cada atributo y prefijo. El probador debe validar que están siendo utilizados correctamente por la aplicación. Las cookies pueden revisarse utilizando un [proxy interceptador](#intercepting-proxy), o revisando el jar de cookies del navegador.

### Atributos de cookies

#### Atributo Secure

El atributo [`Secure`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Secure) indica al navegador que solo envíe la cookie si la solicitud se está enviando a través de un canal seguro como `HTTPS`. Esto ayudará a proteger la cookie de ser pasada en solicitudes no encriptadas. Si la aplicación puede accederse tanto a través de `HTTP` como de `HTTPS`, un atacante podría ser capaz de redirigir al usuario para enviar su cookie como parte de solicitudes no protegidas.

#### Atributo HttpOnly

El atributo [`HttpOnly`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#HttpOnly) se utiliza para ayudar a prevenir ataques como la fuga de sesiones, ya que no permite que la cookie sea accedida mediante un script del lado cliente como JavaScript.

> Esto no limita toda la superficie de ataque de los ataques XSS, ya que un atacante podría aún enviar solicitudes en lugar del usuario, pero limita inmensamente el alcance de los vectores de ataque XSS.

#### Atributo Domain

El atributo [`Domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) se utiliza para comparar el dominio de la cookie contra el dominio del servidor para el cual se está haciendo la solicitud HTTP. Si el dominio coincide o si es un subdominio, entonces se comprobará a continuación el atributo [`path`](#path-attribute).

Tenga en cuenta que solo los hosts que pertenecen al dominio especificado pueden configurar una cookie para ese dominio. Además, el atributo `domain` no puede ser un dominio de nivel superior (como `.gov` o `.com`) para prevenir que los servidores configuren cookies arbitrarias para otro dominio (como configurar una cookie para `owasp.org`). Si el atributo domain no se establece, entonces el nombre de host del servidor que generó la cookie se utiliza como el valor predeterminado del `domain`.

Por ejemplo, si una cookie es configurada por una aplicación en `app.mydomain.com` sin atributo domain establecido, entonces la cookie sería reenviada para todas las solicitudes subsiguientes para `app.mydomain.com`, pero no para sus subdominios (como `hacker.app.mydomain.com`), o a `otherapp.mydomain.com`. (Sin embargo, versiones antiguas de Edge/IE se comportan de manera diferente, y _sí_ envían estas cookies a subdominios.) Si un desarrollador quisiera aflojar esta restricción, entonces podrían establecer el atributo `domain` a `mydomain.com`. En este caso la cookie sería enviada a todas las solicitudes para `app.mydomain.com` y subdominios de `mydomain.com`, tales como `hacker.app.mydomain.com`, e incluso `bank.mydomain.com`. Si había un servidor vulnerable en un subdominio (por ejemplo, `otherapp.mydomain.com`) y el atributo `domain` ha sido establecido demasiado laxamente (por ejemplo, `mydomain.com`), entonces el servidor vulnerable podría ser utilizado para cosechar cookies (como tokens de sesión) a través del alcance completo de `mydomain.com`.

#### Atributo Path

El atributo [`Path`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) juega un papel mayor en establecer el alcance de las cookies en conjunción con el [`domain`](#domain-attribute). Además del dominio, se puede especificar la ruta URL para la cual la cookie es válida. Si el dominio y la ruta coinciden, entonces la cookie será enviada en la solicitud. Al igual que con el atributo domain, si el atributo path se establece demasiado laxamente, entonces podría dejar la aplicación vulnerable a ataques por otras aplicaciones en el mismo servidor. Por ejemplo, si el atributo path se estableció en la raíz del servidor web `/`, entonces las cookies de la aplicación serán enviadas a cada aplicación dentro del mismo dominio (si múltiples aplicaciones residen bajo el mismo servidor). Un par de ejemplos para múltiples aplicaciones bajo el mismo servidor:

- `path=/bank`
- `path=/private`
- `path=/docs`
- `path=/docs/admin`

#### Atributo Expires

El atributo [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Permanent_cookies) se utiliza para:

- configurar cookies persistentes
- limitar la vida útil si una sesión vive demasiado tiempo
- remover una cookie forzosamente configurándola a una fecha pasada

A diferencia de las [cookies de sesión](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Session_cookies), las cookies persistentes serán utilizadas por el navegador hasta que la cookie expire. Una vez que la fecha de expiración ha excedido el tiempo establecido, el navegador eliminará la cookie.

#### Atributo SameSite

El atributo [`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies) puede ser utilizado para afirmar si una cookie debería ser enviada junto con solicitudes cross-site. Esta característica permite al servidor mitigar el riesgo de fuga de información cross-origin. En algunos casos, se utiliza también como una estrategia de reducción de riesgo (o mecanismo de defensa en profundidad) para prevenir ataques de [falsificación de solicitud cross-site](05-Testing_for_Cross_Site_Request_Forgery.md). Este atributo puede ser configurado en tres modos diferentes:

- `Strict`
- `Lax`
- `None`

##### Valor Strict

El valor `Strict` es el uso más restrictivo de `SameSite`, permitiendo al navegador enviar la cookie solo a contexto de primera parte sin navegación de nivel superior. En otras palabras, los datos asociados con la cookie solo serán enviados en solicitudes que coincidan con el sitio actual mostrado en la barra de URL del navegador. La cookie no será enviada en solicitudes generadas por sitios de terceros. Este valor es especialmente recomendado para acciones realizadas en el mismo dominio. Sin embargo, puede tener algunas limitaciones con algunos sistemas de gestión de sesiones afectando negativamente la experiencia de navegación del usuario. Dado que el navegador no enviaría la cookie en ninguna solicitud generada desde un dominio de terceros o email, el usuario sería requerido para iniciar sesión nuevamente incluso si ya tienen una sesión autenticada.

##### Valor Lax

El valor `Lax` es menos restrictivo que `Strict`. La cookie será enviada si la URL es igual al dominio de la cookie (primera parte) incluso si el enlace viene de un dominio de terceros. Este valor es considerado por la mayoría de los navegadores el comportamiento predeterminado ya que proporciona una mejor experiencia de usuario que el valor `Strict`. No se activa para activos, tales como imágenes, donde las cookies podrían no ser necesarias para acceder a ellos.

##### Valor None

El valor `None` especifica que el navegador enviará la cookie en todos los contextos, incluyendo solicitudes cross-site (el comportamiento normal antes de la implementación de `SameSite`). Si `Samesite=None` se establece, entonces el atributo Secure debe establecerse, de lo contrario los navegadores modernos ignorarán el atributo SameSite, _ej._ `SameSite=None; Secure`.

### Prefijos de cookies

Por diseño las cookies no tienen las capacidades para garantizar la integridad y confidencialidad de la información almacenada en ellas. Esas limitaciones hacen imposible para un servidor tener confianza sobre cómo los atributos de una cookie dada fueron configurados en la creación. Para dar a los servidores tales características de manera compatible hacia atrás, la industria ha introducido el concepto de [`Prefijos de nombre de cookie`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-prefixes-00) para facilitar pasar tales detalles embebidos como parte del nombre de la cookie.

#### Prefijo Host

El prefijo [`__Host-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) espera que las cookies cumplan las siguientes condiciones:

1. La cookie debe configurarse con el atributo [`Secure`](#secure-attribute).
2. La cookie debe configurarse desde un URI considerado seguro por el agente de usuario.
3. Enviada solo al host que configuró la cookie y NO DEBE incluir ningún atributo [`Domain`](#domain-attribute).
4. La cookie debe configurarse con el atributo [`Path`](#path-attribute) con un valor de `/` para que sea enviada con cada solicitud al host.

Por esta razón, la cookie `Set-Cookie: __Host-SID=12345; Secure; Path=/` sería aceptada mientras que cualquiera de las siguientes siempre serían rechazadas:
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.example`
`Set-Cookie: __Host-SID=12345; Domain=site.example; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.example; Path=/`

#### Prefijo Secure

El prefijo [`__Secure-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) es menos restrictivo y puede introducirse agregando la cadena sensible a mayúsculas y minúsculas `__Secure-` al nombre de la cookie. Cualquier cookie que coincida con el prefijo `__Secure-` se esperaría que cumpla las siguientes condiciones:

1. La cookie debe configurarse con el atributo `Secure`.
2. La cookie debe configurarse desde un URI considerado seguro por el agente de usuario.

### Prácticas sólidas

Basado en las necesidades de la aplicación, y cómo la cookie debería funcionar, los atributos y prefijos deben aplicarse. Cuanto más bloqueada esté la cookie, mejor.

Poniendo todo esto junto, podemos definir la configuración de atributos de cookie más segura como: `Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict`.

## Herramientas

### Proxy interceptador

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Web Proxy Burp Suite](https://portswigger.net)

### Plug-in del navegador

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
- ["FireSheep" for FireFox](https://github.com/codebutler/firesheep)
- ["EditThisCookie" for Chrome](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)
- ["Cookiebro - Cookie Manager" for FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## Referencias

- [RFC 2965 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Same-Site Cookies - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [The important "expires" attribute of Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [HttpOnly Session ID in URL and Page Body](https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)
- [Internet Explorer Cookie Internals (FAQ)](https://learn.microsoft.com/en-gb/archive/blogs/ieinternals/internet-explorer-cookie-internals-faq)