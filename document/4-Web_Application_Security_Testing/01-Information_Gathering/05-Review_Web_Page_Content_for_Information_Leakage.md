# Revisar Contenido de Página Web para Fugas de Información

|ID          |
|------------|
|WSTG-INFO-05|

## Resumen

Es muy común, e incluso recomendado, que los programadores incluyan comentarios detallados y metadatos en su código fuente. Sin embargo, comentarios y metadatos incluidos en el código HTML podrían revelar información interna que no debería estar disponible para potenciales atacantes. Los comentarios y metadatos deben revisarse para determinar si alguna información está siendo filtrada. Además, algunas aplicaciones pueden filtrar información en el cuerpo de respuestas de redirección.

Para aplicaciones web modernas, el uso de JavaScript del lado del cliente para el frontend se está volviendo más popular. Tecnologías de construcción de frontend populares usan JavaScript del lado del cliente como ReactJS, AngularJS, o Vue. Similar a los comentarios y metadatos en código HTML, muchos programadores también codifican información sensible en variables JavaScript en el frontend. La información sensible puede incluir (pero no se limita a): Claves API privadas (*ej.* una clave API de Google Map sin restricciones), direcciones IP internas, rutas sensibles (*ej.* ruta a páginas o funcionalidad de admin ocultas), o incluso credenciales. Esta información sensible puede filtrarse desde tal código JavaScript frontend. Una revisión debe hacerse para determinar si alguna información sensible filtrada podría ser usada por atacantes para abuso.

Para aplicaciones web grandes, los problemas de rendimiento son una gran preocupación para los programadores. Los programadores han usado diferentes métodos para optimizar el rendimiento del frontend, incluyendo Syntactically Awesome Style Sheets (Sass), Sassy CSS (SCSS), webpack, etc. Usando estas tecnologías, el código frontend a veces se vuelve más difícil de entender y difícil de depurar, y debido a eso, los programadores a menudo despliegan archivos de mapa de fuente para propósitos de depuración. Un "mapa de fuente" es un archivo especial que conecta una versión minificada/uglificada de un activo (CSS o JavaScript) a la versión original autorizada. Los programadores aún debaten si llevar o no archivos de mapa de fuente al entorno de producción. Sin embargo, es innegable que archivos de mapa de fuente o archivos para depuración si se liberan al entorno de producción harán su fuente más legible para humanos. Puede hacer más fácil para atacantes encontrar vulnerabilidades desde el frontend o recopilar información sensible desde él. La revisión de código JavaScript debe hacerse para determinar si algún archivo de depuración está expuesto desde el frontend. Dependiendo del contexto y sensibilidad del proyecto, un experto en seguridad debería decidir si los archivos deberían existir en el entorno de producción o no.

## Objetivos de Prueba

- Revisar comentarios de página web, metadatos, y cuerpos de redirección para encontrar cualquier fuga de información.
- Recopilar archivos JavaScript y revisar el código JS para mejor entender la aplicación y encontrar cualquier fuga de información.
- Identificar si archivos de mapa de fuente u otros archivos de depuración frontend existen.

## Cómo Probar

### Revisar Comentarios de Página Web y Metadatos

Los comentarios HTML a menudo son usados por los desarrolladores para incluir información de depuración sobre la aplicación. A veces, olvidan sobre los comentarios y los dejan en entornos de producción. Los probadores deberían buscar comentarios HTML que empiecen con `<!--`.

Verificar código fuente HTML para comentarios conteniendo información sensible que puede ayudar al atacante a ganar más insight sobre la aplicación. Podría ser código SQL, usernames y passwords, direcciones IP internas, o información de depuración.

```html
...
<div class="table2">
  <div class="col1">1</div><div class="col2">Mary</div>
  <div class="col1">2</div><div class="col2">Peter</div>
  <div class="col1">3</div><div class="col2">Joe</div>

<!-- Query: SELECT id, name FROM app.users WHERE active='1' -->

</div>
...
```

El probador puede incluso encontrar algo como esto:

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

Verificar información de versión HTML para números de versión válidos y URLs de Data Type Definition (DTD)

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "https://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` -- DTD estricto por defecto
- `loose.dtd` -- DTD suelto
- `frameset.dtd` -- DTD para documentos frameset

Algunos tags `META` no proporcionan vectores de ataque activos pero en su lugar permiten a un atacante perfilar una aplicación:

```html
<META name="Author" content="Andrew Muller">
```

Un tag `META` común (pero no compliant con [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/)) es [Refresh](https://en.wikipedia.org/wiki/Meta_refresh).

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Un uso común para tag `META` es especificar keywords que un motor de búsqueda puede usar para mejorar la calidad de resultados de búsqueda.

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Aunque la mayoría de servidores web manejan indexación de motores de búsqueda vía el archivo `robots.txt`, también puede ser manejado por tags `META`. El tag abajo aconsejará a robots no indexar y no seguir links en la página HTML conteniendo el tag.

```html
<META name="robots" content="none">
```

La [Platform for Internet Content Selection (PICS)](https://www.w3.org/PICS/) y [Protocol for Web Description Resources (POWDER)](https://www.w3.org/2007/powder/) proporcionan infraestructura para asociar metadatos con contenido de Internet.

### Identificar Código JavaScript y Recopilar Archivos JavaScript

Los programadores a menudo codifican información sensible con variables JavaScript en el frontend. Los probadores deberían verificar código fuente HTML y buscar código JavaScript entre tags `<script>` y `</script>`. Los probadores también deberían identificar archivos JavaScript externos para revisar el código (archivos JavaScript tienen la extensión de archivo `.js` y nombre del archivo JavaScript usualmente puesto en el atributo `src` (source) de un tag `<script>`).

Verificar código JavaScript para cualquier fuga de información sensible que podría ser usada por atacantes para abusar o manipular el sistema más. Buscar valores como: claves API, direcciones IP internas, rutas sensibles, o credenciales. Por ejemplo:

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAccessKey: config('AWSS3SecretAccessKey'),
};
```

El probador puede incluso encontrar algo como esto:

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

Cuando una Clave API es encontrada, los probadores pueden verificar si las restricciones de Clave API están establecidas por servicio o por IP, HTTP referrer, aplicación, SDK, etc.

Por ejemplo, si los probadores encuentran una Clave API de Google Map, pueden verificar si esta Clave API está restringida por IP o restringida solo por las APIs de Google Map. Si la Clave API de Google está restringida solo por las APIs de Google Map, los atacantes aún pueden usar esa Clave API para consultar APIs de Google Map sin restricciones y el propietario de la aplicación debe pagar por eso.

```html

<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

En algunos casos, los probadores pueden encontrar rutas sensibles desde código JavaScript, como links a páginas o funcionalidad de admin interna u oculta.

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://staging-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

### Identificar Archivos de Mapa de Fuente

Los archivos de mapa de fuente usualmente serán cargados cuando DevTools abran. Los probadores también pueden encontrar archivos de mapa de fuente agregando la extensión ".map" después de la extensión de cada archivo JavaScript externo. Por ejemplo, si un probador ve un archivo `/static/js/main.chunk.js`, pueden entonces verificar su archivo de mapa de fuente visitando `/static/js/main.chunk.js.map`.

Verificar archivos de mapa de fuente para cualquier información sensible que puede ayudar al atacante a ganar más insight sobre la aplicación. Por ejemplo:

```json
{
  "version": 3,
  "file": "static/js/main.chunk.js",
  "sources": [
    "/home/sysadmin/cashsystem/src/actions/index.js",
    "/home/sysadmin/cashsystem/src/actions/reportAction.js",
    "/home/sysadmin/cashsystem/src/actions/cashoutAction.js",
    "/home/sysadmin/cashsystem/src/actions/userAction.js",
    "..."
  ],
  "..."
}
```

Cuando los sitios cargan archivos de mapa de fuente, el código fuente frontend se volverá legible y más fácil de depurar.

### Identificar Respuestas de Redirección que Filtran Información

Aunque las respuestas de redirección generalmente no se esperan que contengan cualquier contenido web significativo no hay garantía de que no puedan contener contenido. Así, mientras las respuestas de serie 300 (redirección) a menudo contienen contenido tipo "redireccionando a `https://example.com/`" también pueden filtrar contenido.

Considera una situación en la que una respuesta de redirección es el resultado de una verificación de autenticación o autorización, si esa verificación falla el servidor puede responder redireccionando al usuario de vuelta a una página "segura" o "por defecto", aún la respuesta de redirección en sí puede aún contener contenido que no se muestra en el navegador pero de hecho se transmite al cliente. Esto puede verse ya sea aprovechando herramientas de desarrollador del navegador o vía un proxy personal (como ZAP, Burp, Fiddler, o Charles).

## Herramientas

- [Wget](https://www.gnu.org/software/wget/wget.html)
- Función "view source" del navegador
- Ojos
- [Curl](https://curl.haxx.se/)
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
- [Google Maps API Scanner](https://github.com/ozguralp/gmapsapiscanner/)

## Referencias

- [KeyHacks](https://github.com/streaak/keyhacks)
- [RingZer0 Online CTF](https://ringzer0ctf.com/challenges/104) - Challenge 104 "Admin Panel".

### Whitepapers

- [HTML version 4.01](https://www.w3.org/TR/1999/REC-html401-19991224)
- [XHTML](https://www.w3.org/TR/2010/REC-xhtml-basic-20101123/)
- [HTML version 5](https://www.w3.org/TR/html5/)