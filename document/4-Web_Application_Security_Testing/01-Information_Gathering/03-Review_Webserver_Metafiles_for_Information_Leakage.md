# Revisar los archivos de metadatos del servidor web para detectar fugas de información

|ID          |
|------------|
|WSTG-INFO-03|

## Resumen

Esta sección describe cómo probar diversos archivos de metadatos para detectar fugas de información sobre las rutas o funcionalidades de la aplicación web. Además, la lista de directorios que deben evitar los spiders, robots o crawlers también puede crearse como dependencia para [Mapear las rutas de ejecución a través de la aplicación](07-Map_Execution_Paths_Through_Application.md). También se puede recopilar otra información para identificar la superficie de ataque, los detalles tecnológicos o utilizarla en actividades de ingeniería social.

## Objetivos de la prueba

- Identificar rutas y funcionalidades ocultas u ofuscadas mediante el análisis de archivos de metadatos.
- Extraer y mapear otra información que pueda permitir comprender mejor los sistemas en cuestión.

## Cómo probar

> Cualquiera de las acciones que se indican a continuación y que se realicen con `wget` también se puede realizar con `curl`. Muchas herramientas de pruebas de seguridad de aplicaciones dinámicas (DAST), como ZAP y Burp Suite, incluyen comprobaciones o análisis de estos recursos como parte de sus funciones de spider/crawler. También se pueden identificar mediante diversos [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) o aprovechando funciones de búsqueda avanzada como `inurl:`.

### Robots

Los spiders, robots o crawlers web recuperan una página web y luego recorren recursivamente sus hipervínculos para recuperar más contenido web. Su comportamiento permitido se especifica mediante el [Protocolo de exclusión de robots](https://datatracker.ietf.org/doc/html/rfc9309) ([RFC 9309](https://datatracker.ietf.org/doc/html/rfc9309)), a través de un archivo `robots.txt` en el directorio raíz del sitio web.

Como ejemplo, a continuación se cita una muestra del archivo `robots.txt` de [Google](https://www.google.com/robots.txt), obtenida el 13 de septiembre de 2026:

```text
User-agent: *
User-agent: Yandex
Disallow: /search
Allow: /search/about
Allow: /search/howsearchworks
Disallow: /sdch
Disallow: /groups
...
# Crawlers of certain social media sites are allowed to access page markup when
# google.com/imgres* links are shared. To learn more, please contact
# images-robots-allowlist@google.com.
User-agent: facebookexternalhit
User-agent: Twitterbot
Allow: /imgres
Allow: /search
Disallow: /groups
Disallow: /hosted/images/
Disallow: /m/

Sitemap: https://www.google.com/sitemap.xml
```

La directiva [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent) hace referencia al spider, robot o crawler web específico. Por ejemplo, `User-Agent: Googlebot` hace referencia al spider de Google, mientras que `User-Agent: bingbot` hace referencia a un crawler de Microsoft. `User-Agent: *` en el ejemplo anterior se aplica a todos los [spiders, robots y crawlers web](https://developers.google.com/search/docs/crawling-indexing/robots/intro).

La directiva `Disallow` especifica qué recursos están prohibidos para los spiders, robots o crawlers. En el ejemplo anterior, se prohíbe el acceso a los siguientes:

```text
...
Disallow: /search
...
Disallow: /sdch
...
```

Los spiders, robots o crawlers web pueden [ignorar intencionalmente](https://blog.cloudflare.com/perplexity-is-using-stealth-undeclared-crawlers-to-evade-website-no-crawl-directives/) las directivas `Disallow` especificadas en un archivo `robots.txt`. Por lo tanto, `robots.txt` no debe considerarse un mecanismo para imponer restricciones sobre cómo terceros acceden al contenido web, lo almacenan o lo vuelven a publicar.

El archivo `robots.txt` se recupera del directorio raíz del sitio web del servidor. Por ejemplo, para recuperar el `robots.txt` de `www.google.com` con `wget` o `curl`:

```bash
$ curl -O -Ss https://www.google.com/robots.txt && head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
...
```

#### Crawlers de IA

Se ha observado que muchos crawlers de IA, utilizados por empresas de IA para recopilar datos de entrenamiento, ignoran las directivas de `robots.txt`. Los desarrolladores y propietarios de sitios han explorado contramedidas para proteger su contenido; como se describe en [un artículo de TechCrunch de 2025](https://techcrunch.com/2025/03/27/open-source-devs-are-fighting-ai-crawlers-with-cleverness-and-vengeance/), la comunidad de código abierto ha respondido con tácticas como generar contenido falso sin fin para consumir los recursos de los crawlers o proporcionar datos engañosos para degradar la calidad de los conjuntos de datos de entrenamiento de IA. La [Free Software Foundation](https://www.fsf.org/blogs/sysadmin/our-small-team-vs-millions-of-bots) también ha documentado la carga operativa que imponen a los equipos pequeños los grandes volúmenes de crawlers de IA, y ha destacado que incluso las reglas de `robots.txt` bienintencionadas se ignoran habitualmente.

Desde la perspectiva de las pruebas, las entradas de `robots.txt` destinadas a bloquear crawlers de IA (por ejemplo, `User-Agent: GPTBot` y `User-Agent: ClaudeBot`) pueden ser ineficaces, y cualquier crawler que ignore el protocolo puede descubrir las rutas confidenciales incluidas en `robots.txt`.

#### Analizar robots.txt mediante Google Webmaster Tools

Los propietarios de sitios pueden usar la función de Google «Analizar robots.txt» para analizar el sitio como parte de [Google Webmaster Tools](https://search.google.com/search-console/about). Esta herramienta puede ayudar con las pruebas. El procedimiento es el siguiente:

1. Inicia sesión en Google Webmaster Tools con una cuenta de Google.
2. En el panel, introduce la URL del sitio que se va a analizar.
3. Elige uno de los métodos disponibles y sigue las instrucciones en pantalla.

### Etiquetas META

Las etiquetas `<META>` se encuentran en la sección `HEAD` de cada documento HTML y deben ser coherentes en todo el sitio, por si el punto de inicio del robot, spider o crawler no parte de un enlace a un documento distinto del webroot, es decir, de un [enlace profundo](https://en.wikipedia.org/wiki/Deep_linking). La directiva Robots también se puede especificar mediante una [etiqueta META](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag) específica.

#### Etiqueta META Robots

Si no hay una entrada `<META NAME="ROBOTS" ... >`, el «Protocolo de exclusión de robots» toma de forma predeterminada los valores `INDEX,FOLLOW`, respectivamente. Por lo tanto, las otras dos entradas válidas definidas por el «Protocolo de exclusión de robots» llevan el prefijo `NO...`; es decir, `NOINDEX` y `NOFOLLOW`.

A partir de las directivas `Disallow` incluidas en el archivo `robots.txt` del webroot, se realiza una búsqueda mediante una expresión regular de `<META NAME="ROBOTS"` en cada página web. Luego, el resultado se compara con el archivo `robots.txt` del webroot.

#### Etiquetas META de información diversa

Las organizaciones suelen incluir etiquetas META informativas en el contenido web para admitir diversas tecnologías, como lectores de pantalla, vistas previas en redes sociales e indexación en motores de búsqueda, entre otras. Esta metainformación puede ser útil para que los evaluadores identifiquen las tecnologías utilizadas y otras rutas o funcionalidades que explorar y probar. La siguiente metainformación se obtuvo de `www.whitehouse.gov` mediante la opción Ver código fuente de la página el 13 de septiembre de 2026:

```html
...
<meta name="description" content="President Donald J. Trump and Vice President JD Vance are committed to lowering costs for all Americans, securing our borders, unleashing American energy dominance, restoring peace through strength, and making all Americans safe and secure once again." />
<link rel="canonical" href="https://www.whitehouse.gov/" />
<meta property="og:locale" content="en_US" />
<meta property="og:type" content="website" />
<meta property="og:title" content="The White House" />
<meta property="og:description" content="President Donald J. Trump and Vice President JD Vance are committed to lowering costs for all Americans, securing our borders, unleashing American energy dominance, restoring peace through strength, and making all Americans safe and secure once again." />
<meta property="og:url" content="https://www.whitehouse.gov/" />
<meta property="og:site_name" content="The White House" />
<meta property="article:publisher" content="https://www.facebook.com/WhiteHouse/" />
<meta property="article:modified_time" content="2026-07-22T13:21:31+00:00" />
<meta property="og:image" content="https://www.whitehouse.gov/wp-content/uploads/2025/03/WH47-Social-Share-Card.jpg" />
...
<meta property="og:image:type" content="image/jpeg" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="The White House" />
...
<meta name="twitter:image" content="https://www.whitehouse.gov/wp-content/uploads/2025/03/WH47-Social-Share-Card.jpg" />
<meta name="twitter:site" content="@whitehouse" />
...
```

### Mapas del sitio

Un mapa del sitio es un archivo en el que un desarrollador u organización puede proporcionar información sobre las páginas, los videos y otros archivos que ofrece el sitio o la aplicación, así como sobre las relaciones entre ellos. Los motores de búsqueda pueden usar este archivo para navegar por el sitio de forma más eficiente. De igual modo, los evaluadores pueden utilizar archivos `sitemap.xml` para obtener información más detallada sobre el sitio o la aplicación objeto de investigación.

El siguiente fragmento pertenece al mapa del sitio principal de Google, obtenido el 5 de mayo de 2020.

```bash
$ wget --no-verbose https://www.google.com/sitemap.xml && head -n8 sitemap.xml
2020-05-05 12:23:30 URL:https://www.google.com/sitemap.xml [2049] -> "sitemap.xml" [1]

<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="https://www.google.com/schemas/sitemap/0.84">
  <sitemap>
    <loc>https://www.google.com/gmail/sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.google.com/forms/sitemaps.xml</loc>
  </sitemap>
...
```

A partir de ahí, un evaluador podría querer recuperar el mapa del sitio de Gmail `https://www.google.com/gmail/sitemap.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="https://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
...
```

### Security TXT

[security.txt](https://securitytxt.org) fue ratificado por la IETF como [RFC 9116: formato de archivo para facilitar la divulgación de vulnerabilidades de seguridad](https://www.rfc-editor.org/rfc/rfc9116.html), que permite a los sitios definir políticas de seguridad y datos de contacto. Esto puede resultar de interés en escenarios de pruebas por diversos motivos, entre ellos:

- Identificar otras rutas o recursos para incluir en la fase de descubrimiento o análisis.
- Recopilar inteligencia de fuentes abiertas.
- Encontrar información sobre programas de recompensas por errores, etc.
- Ingeniería social.

El archivo puede estar en la raíz del servidor web o en el directorio `.well-known/`, por ejemplo:

- `https://example.com/security.txt`
- `https://example.com/.well-known/security.txt`

A continuación se muestra un ejemplo real obtenido de LinkedIn el 5 de mayo de 2020:

```bash
$ wget --no-verbose https://www.linkedin.com/.well-known/security.txt && cat security.txt
2020-05-07 12:56:51 URL:https://www.linkedin.com/.well-known/security.txt [333/333] -> "security.txt" [1]
# Conforms to IETF `draft-foudil-securitytxt-07`
Contact: mailto:security@linkedin.com
Contact: https://www.linkedin.com/help/linkedin/answer/62924
Encryption: https://www.linkedin.com/help/linkedin/answer/79676
Canonical: https://www.linkedin.com/.well-known/security.txt
Policy: https://www.linkedin.com/help/linkedin/answer/62924
```

Las claves públicas OpenPGP contienen metadatos que pueden proporcionar información sobre la propia clave. A continuación se indican algunos elementos de metadatos comunes que se pueden extraer de una clave pública OpenPGP:

- **ID de clave**: el ID de clave es un identificador corto derivado del material de la clave pública. Ayuda a identificar la clave y suele mostrarse como un valor hexadecimal de ocho caracteres.
- **Huella digital de la clave**: la huella digital de la clave es un identificador más largo y único, derivado del material de la clave. Suele mostrarse como un valor hexadecimal de 40 caracteres. Las huellas digitales de claves se utilizan habitualmente para verificar la integridad y autenticidad de una clave pública.
- **Algoritmo de clave**: el algoritmo de clave representa el algoritmo criptográfico que utiliza la clave pública. OpenPGP admite diversos algoritmos, como RSA, DSA y ECC (criptografía de curva elíptica).
- **Tamaño de clave**: el tamaño de clave se refiere a la longitud o el tamaño de la clave criptográfica, expresado en bits. Indica la robustez de la clave y determina el nivel de seguridad que proporciona.
- **Fecha de creación de la clave**: indica cuándo se generó o creó la clave.
- **Fecha de vencimiento de la clave**: las claves públicas OpenPGP pueden tener una fecha de vencimiento, después de la cual se consideran no válidas. La fecha de vencimiento de la clave especifica cuándo deja de ser válida.
- **ID de usuario**: las claves públicas pueden tener uno o más ID de usuario asociados que identifican al propietario o a la entidad asociada con la clave. Los ID de usuario suelen incluir datos como el nombre, la dirección de correo electrónico y comentarios opcionales del propietario de la clave.

### Humans TXT

`humans.txt` es una iniciativa para conocer a las personas que están detrás de un sitio. Consiste en un archivo de texto que contiene información sobre las diferentes personas que han contribuido a crear el sitio. Este archivo suele contener información relacionada con puestos de trabajo o rutas de empleo, aunque no siempre.

El siguiente ejemplo se obtuvo de Google el 5 de mayo de 2020:

```bash
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```

### Otras fuentes de información de .well-known

Hay otros RFC y borradores de Internet que proponen usos estandarizados de archivos dentro del directorio `.well-known/`. Se pueden encontrar listas de estos [aquí en WikiPedia](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) o [aquí, en IANA](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml).

A un evaluador le resultaría bastante sencillo revisar los RFC o borradores y crear una lista para proporcionársela a un crawler o fuzzer, con el fin de verificar la existencia o el contenido de esos archivos.

## Herramientas

- Navegador (función Ver código fuente o Herramientas para desarrolladores)
- curl
- wget
- Burp Suite
- ZAP