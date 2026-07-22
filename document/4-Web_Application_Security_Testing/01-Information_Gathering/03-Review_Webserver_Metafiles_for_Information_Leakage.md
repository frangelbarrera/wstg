# Revisar Metafiles del Servidor Web para Fugas de Información

|ID          |
|------------|
|WSTG-INFO-03|

## Resumen

Esta sección describe cómo probar varios archivos de metadatos para fugas de información de las rutas o funcionalidades de la aplicación web. Además, la lista de directorios que deben evitarse por Spiders, Robots o Crawlers también puede crearse como dependencia para [Mapear rutas de ejecución a través de la aplicación](07-Map_Execution_Paths_Through_Application.md). También se puede recopilar otra información para identificar la superficie de ataque, detalles tecnológicos o para su uso en compromisos de ingeniería social.

## Objetivos de la Prueba

- Identificar rutas o funcionalidades ocultas u ofuscadas mediante el análisis de archivos de metadatos.
- Extraer y mapear otra información que pueda llevar a una mejor comprensión de los sistemas en cuestión.

## Cómo Probar

> Cualquiera de las acciones realizadas a continuación con `wget` también podría hacerse con `curl`. Muchas herramientas de Pruebas de Seguridad de Aplicaciones Dinámicas (DAST) como ZAP y Burp Suite incluyen comprobaciones o análisis para estos recursos como parte de su funcionalidad de spider/crawler. También pueden identificarse utilizando varios [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) o aprovechando funciones de búsqueda avanzadas como `inurl:`.

### Robots

Los Web Spiders, Robots o Crawlers recuperan una página web y luego atraviesan recursivamente los hiperenlaces para recuperar más contenido web. Su comportamiento aceptado se especifica por el [Protocolo de Exclusión de Robots](https://www.robotstxt.org) del archivo [robots.txt](https://www.robotstxt.org/) en el directorio raíz web.

Como ejemplo, el comienzo del archivo `robots.txt` de [Google](https://www.google.com/robots.txt) muestreado el 5 de mayo de 2020 se cita a continuación:

```text
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
...
```

La directiva [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) se refiere al spider/robot/crawler web específico. Por ejemplo, `User-Agent: Googlebot` se refiere al spider de Google mientras que `User-Agent: bingbot` se refiere a un crawler de Microsoft. `User-Agent: *` en el ejemplo anterior se aplica a todos los [web spiders/robots/crawlers](https://support.google.com/webmasters/answer/6062608?visit_id=637173940975499736-3548411022&rd=1).

La directiva `Disallow` especifica qué recursos están prohibidos por spiders/robots/crawlers. En el ejemplo anterior, se prohíben los siguientes:

```text
...
Disallow: /search
...
Disallow: /sdch
...
```

Los web spiders/robots/crawlers pueden [ignorar intencionalmente](https://blog.isc2.org/isc2_blog/2008/07/the-attack-of-t.html) las directivas `Disallow` especificadas en un archivo `robots.txt`. Por lo tanto, `robots.txt` no debería considerarse como un mecanismo para hacer cumplir restricciones sobre cómo se accede, almacena o republica el contenido web por terceros.

El archivo `robots.txt` se recupera del directorio raíz web del servidor web. Por ejemplo, para recuperar el `robots.txt` de `www.google.com` usando `wget` o `curl`:

```bash
$ curl -O -Ss https://www.google.com/robots.txt && head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
...
```

#### Analizar robots.txt Usando Google Webmaster Tools

Los propietarios de sitios pueden usar la función "Analizar robots.txt" de Google para analizar el sitio como parte de sus [Google Webmaster Tools](https://www.google.com/webmasters/tools). Esta herramienta puede ayudar con las pruebas y el procedimiento es el siguiente:

1. Inicia sesión en Google Webmaster Tools con una cuenta de Google.
2. En el panel de control, introduce la URL del sitio a analizar.
3. Elige entre los métodos disponibles y sigue las instrucciones en pantalla.

### Etiquetas META

Las etiquetas `<META>` se ubican dentro de la sección `HEAD` de cada documento HTML y deberían ser consistentes en un sitio en caso de que el punto de inicio del robot/spider/crawler no comience desde un enlace de documento que no sea webroot, es decir, un [enlace profundo](https://en.wikipedia.org/wiki/Deep_linking). La directiva Robots también puede especificarse usando una [etiqueta META](https://www.robotstxt.org/meta.html) específica.

#### Etiqueta META Robots

Si no hay una entrada `<META NAME="ROBOTS" ... >`, entonces el "Protocolo de Exclusión de Robots" por defecto es `INDEX,FOLLOW` respectivamente. Por lo tanto, las otras dos entradas válidas definidas por el "Protocolo de Exclusión de Robots" están prefijadas con `NO...` es decir, `NOINDEX` y `NOFOLLOW`.

Basado en las directivas Disallow listadas dentro del archivo `robots.txt` en webroot, se realiza una búsqueda de expresión regular para `<META NAME="ROBOTS"` dentro de cada página web. El resultado se compara entonces con el archivo robots.txt en webroot.

#### Etiquetas de Información META Misceláneas

Las organizaciones a menudo incrustan etiquetas META informativas en el contenido web para apoyar varias tecnologías como lectores de pantalla, vistas previas de redes sociales, indexación de motores de búsqueda, etc. Tal meta-información puede ser de valor para los evaluadores en la identificación de tecnologías utilizadas, y rutas/funcionalidades adicionales para explorar y probar. La siguiente meta información se recuperó de `www.whitehouse.gov` vía Ver Fuente de Página el 05 de mayo de 2020:

```html
...
<meta property="og:locale" content="en_US" />
<meta property="og:type" content="website" />
<meta property="og:title" content="The White House" />
<meta property="og:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta property="og:url" content="https://www.whitehouse.gov/" />
<meta property="og:site_name" content="The White House" />
<meta property="fb:app_id" content="1790466490985150" />
<meta property="og:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta property="og:image:secure_url" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta name="twitter:title" content="The White House" />
<meta name="twitter:site" content="@whitehouse" />
<meta name="twitter:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:creator" content="@whitehouse" />
...
<meta name="apple-mobile-web-app-title" content="The White House">
<meta name="application-name" content="The White House">
<meta name="msapplication-TileColor" content="#0c2644">
<meta name="theme-color" content="#f5f5f5">
...
```

### Mapas de Sitio

Un mapa de sitio es un archivo donde un desarrollador u organización puede proporcionar información sobre las páginas, videos y otros archivos ofrecidos por el sitio o aplicación, y la relación entre ellos. Los motores de búsqueda pueden usar este archivo para navegar por su sitio de manera más eficiente. Del mismo modo, los evaluadores pueden utilizar archivos 'sitemap.xml' para obtener conocimientos más profundos sobre el sitio o aplicación bajo investigación.

El siguiente extracto es del mapa de sitio principal de Google recuperado el 05 de mayo de 2020.

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

Explorando desde allí, un evaluador puede desear recuperar el mapa de sitio de gmail `https://www.google.com/gmail/sitemap.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="https://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="https://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
...
```

### Security TXT

[security.txt](https://securitytxt.org) fue ratificado por la IETF como [RFC 9116 - Un Formato de Archivo para Ayudar en la Divulgación de Vulnerabilidades de Seguridad](https://www.rfc-editor.org/rfc/rfc9116.html) que permite a los sitios definir políticas de seguridad y detalles de contacto. Hay múltiples razones por las que esto podría ser de interés en escenarios de pruebas, que incluyen, pero no se limitan a:

- Identificar rutas o recursos adicionales para incluir en el descubrimiento/análisis.
- Recopilación de inteligencia de fuentes abiertas.
- Encontrar información sobre Bug Bounties, etc.
- Ingeniería Social.

El archivo puede estar presente ya sea en la raíz del servidor web o en el directorio `.well-known/`, por ejemplo:

- `https://example.com/security.txt`
- `https://example.com/.well-known/security.txt`

Aquí hay un ejemplo del mundo real recuperado de LinkedIn el 05 de mayo de 2020:

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

Las Claves Públicas OpenPGP contienen algunos metadatos que pueden proporcionar información sobre la clave en sí. Aquí hay algunos elementos de metadatos comunes que pueden extraerse de una Clave Pública OpenPGP:

- **ID de Clave**: El ID de Clave es un identificador corto derivado del material de clave pública. Ayuda a identificar la clave y a menudo se muestra como un valor hexadecimal de ocho caracteres.
- **Huella Digital de Clave**: La Huella Digital de Clave es un identificador más largo y único derivado del material de clave. A menudo se muestra como un valor hexadecimal de 40 caracteres. Las huellas digitales de clave se usan comúnmente para verificar la integridad y autenticidad de una clave pública.
- **Algoritmo de Clave**: El Algoritmo de Clave representa el algoritmo criptográfico usado por la clave pública. OpenPGP soporta varios algoritmos como RSA, DSA y ECC (Criptografía de Curva Elíptica).
- **Tamaño de Clave**: El Tamaño de Clave se refiere a la longitud o tamaño de la clave criptográfica en bits. Indica la fuerza de la clave y determina el nivel de seguridad proporcionado por la clave.
- **Fecha de Creación de Clave**: La Fecha de Creación de Clave indica cuándo se generó o creó la clave.
- **Fecha de Expiración de Clave**: Las Claves Públicas OpenPGP pueden tener una fecha de expiración establecida, después de la cual se consideran inválidas. La Fecha de Expiración de Clave especifica cuándo la clave ya no es válida.
- **IDs de Usuario**: Las claves públicas pueden tener uno o más IDs de Usuario asociados que identifican al propietario o entidad asociada con la clave. Los IDs de Usuario típicamente incluyen información como el nombre, dirección de correo electrónico y comentarios opcionales del propietario de la clave.

### Humans TXT

`humans.txt` es una iniciativa para conocer a las personas detrás de un sitio. Toma la forma de un archivo de texto que contiene información sobre las diferentes personas que han contribuido a construir el sitio. Este archivo a menudo (pero no siempre) contiene información relacionada con carreras o rutas de trabajo/sitios.

El siguiente ejemplo se recuperó de Google el 05 de mayo de 2020:

```bash
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```

### Otras Fuentes de Información .well-known

Hay otros RFCs y borradores de internet que sugieren usos estandarizados de archivos dentro del directorio `.well-known/`. Las listas de estos pueden encontrarse [aquí](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) o [aquí](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml).

Sería bastante simple para un evaluador revisar los RFC/borradores y crear una lista para suministrar a un crawler o fuzzer, con el fin de verificar la existencia o contenido de tales archivos.

## Herramientas

- Navegador (funcionalidad Ver Fuente o Herramientas de Desarrollo)
- curl
- wget
- Burp Suite
- ZAP