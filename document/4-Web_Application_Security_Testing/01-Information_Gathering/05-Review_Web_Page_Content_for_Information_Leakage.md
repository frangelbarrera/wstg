# Revisar el contenido de las páginas web para detectar fugas de información

|ID          |
|------------|
|WSTG-INFO-05|

## Resumen

Es muy común, e incluso recomendable, que los programadores incluyan comentarios detallados y metadatos en su código fuente. Sin embargo, los comentarios y metadatos incluidos en el código HTML podrían revelar información interna que no debería estar disponible para posibles atacantes. Deben revisarse los comentarios y metadatos para determinar si se está filtrando información. Además, algunas aplicaciones pueden filtrar información en el cuerpo de las respuestas de redirección.

En las aplicaciones web modernas, el uso de JavaScript del lado del cliente para el frontend es cada vez más habitual. Las tecnologías populares para construir el frontend utilizan JavaScript del lado del cliente, como React, Angular o Vue. Al igual que ocurre con los comentarios y metadatos del código HTML, muchos programadores también codifican información sensible en variables JavaScript del frontend. La información sensible puede incluir, entre otros datos, claves API privadas (*p. ej.*, una clave API de Google Maps sin restricciones), direcciones IP internas, rutas sensibles (*p. ej.*, rutas a páginas de administración ocultas o a funcionalidades ocultas) o incluso credenciales. Esta información sensible puede filtrarse a través de ese código JavaScript del frontend. Debe realizarse una revisión para determinar si se ha filtrado información sensible que los atacantes podrían aprovechar.

En las aplicaciones web grandes, los problemas de rendimiento son una preocupación importante para los programadores. Estos han utilizado distintos métodos para optimizar el rendimiento del frontend, incluidos Syntactically Awesome Style Sheets (Sass), Sassy CSS (SCSS), webpack, etc. Con estas tecnologías, el código del frontend a veces se vuelve más difícil de entender y depurar, por lo que los programadores suelen implementar archivos de mapas de origen con fines de depuración. Un «mapa de origen» es un archivo especial que relaciona una versión minificada u ofuscada de un recurso (CSS o JavaScript) con la versión original escrita por el autor. Los programadores siguen debatiendo si se deben incluir o no archivos de mapas de origen en el entorno de producción. Sin embargo, es innegable que, si se publican en el entorno de producción, los archivos de mapas de origen o los archivos de depuración harán que el código fuente sea más legible para las personas. Esto puede facilitar que los atacantes encuentren vulnerabilidades en el frontend o recopilen información sensible de este. Debe realizarse una revisión del código JavaScript para determinar si hay archivos de depuración expuestos en el frontend. Según el contexto y la sensibilidad del proyecto, un experto en seguridad debe decidir si esos archivos deberían estar o no en el entorno de producción.

## Objetivos de la prueba

- Revisar los comentarios, metadatos y cuerpos de redirección de las páginas web para encontrar posibles fugas de información.
- Recopilar archivos JavaScript y revisar el código JS para comprender mejor la aplicación y encontrar posibles fugas de información.
- Identificar si existen archivos de mapas de origen u otros archivos de depuración del frontend.

## Cómo probar

### Revisar los comentarios y metadatos de las páginas web

Los desarrolladores suelen utilizar comentarios HTML para incluir información de depuración sobre la aplicación. A veces se olvidan de los comentarios y los dejan en los entornos de producción. Los evaluadores deben buscar comentarios HTML que comiencen con `<!--`.

Revisar el código fuente HTML en busca de comentarios que contengan información sensible y puedan ayudar al atacante a conocer mejor la aplicación. Puede tratarse de código SQL, nombres de usuario y contraseñas, direcciones IP internas o información de depuración.

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

El evaluador podría incluso encontrar algo como esto:

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

Revisar la información de la versión de HTML para verificar que los números de versión y las URL de Data Type Definition (DTD) sean válidos.

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "https://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` -- DTD estricto predeterminado
- `loose.dtd` -- DTD flexible
- `frameset.dtd` -- DTD para documentos de conjuntos de marcos

Algunas etiquetas `META` no proporcionan vectores de ataque activos, sino que permiten al atacante perfilar una aplicación:

```html
<META name="Author" content="Andrew Muller">
```

Una etiqueta `META` habitual (aunque no cumple con [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/)) es [Refresh](https://en.wikipedia.org/wiki/Meta_refresh).

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Un uso habitual de la etiqueta `META` es especificar palabras clave que un motor de búsqueda podría utilizar para mejorar la calidad de los resultados de búsqueda.

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Aunque la mayoría de los servidores web gestionan la indexación de los motores de búsqueda mediante el archivo `robots.txt`, también se puede gestionar con etiquetas `META`. La etiqueta siguiente indica a los robots que no indexen ni sigan los enlaces de la página HTML que la contiene.

```html
<META name="robots" content="none">
```

La [Platform for Internet Content Selection (PICS)](https://www.w3.org/PICS/) y el [Protocol for Web Description Resources (POWDER)](https://www.w3.org/2007/powder/) proporcionan infraestructura para asociar metadatos con contenido de Internet.

### Identificar código JavaScript y recopilar archivos JavaScript

Los programadores suelen codificar información sensible en variables JavaScript del frontend. Los evaluadores deben revisar el código fuente HTML y buscar código JavaScript entre las etiquetas `<script>` y `</script>`. También deben identificar archivos JavaScript externos para revisar su código (los archivos JavaScript tienen la extensión `.js` y el nombre del archivo JavaScript suele aparecer en el atributo `src` (source) de una etiqueta `<script>`).

Revisar el código JavaScript en busca de fugas de información sensible que los atacantes podrían utilizar para abusar del sistema o manipularlo. Buscar valores como claves API, direcciones IP internas, rutas sensibles o credenciales. Por ejemplo:

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAccessKey: config('AWSS3SecretAccessKey'),
};
```

El evaluador podría incluso encontrar algo como esto:

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

Cuando se encuentra una clave API, los evaluadores pueden comprobar si sus restricciones están configuradas por servicio, dirección IP, referente HTTP, aplicación, SDK, etc.

Por ejemplo, si los evaluadores encuentran una clave API de Google Maps, pueden comprobar si está restringida por dirección IP o únicamente a las API de Google Maps. Si la clave API de Google está restringida únicamente a las API de Google Maps, los atacantes aún pueden utilizarla para consultar API de Google Maps sin restricciones, y el propietario de la aplicación tendrá que pagar por ello.

```html
<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

En algunos casos, los evaluadores pueden encontrar rutas sensibles en el código JavaScript, como enlaces a páginas de administración internas u ocultas, o a funcionalidades de administración.

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://staging-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

#### Análisis de JavaScript y del código fuente del cliente

Las aplicaciones modernas de página única y el código abundante del lado del cliente suelen incluir endpoints de API, indicadores de funcionalidades y, ocasionalmente, secretos. Después de recopilar archivos JavaScript (manualmente, mediante un proxy o con herramientas como Uproot), los evaluadores pueden extraer datos de interés con:

```bash
# Example with jsluice
jsluice urls bundle.js
jsluice secrets bundle.js

# Example with LinkFinder
python linkfinder.py -i bundle.js -o cli
```

Contrastar los endpoints y parámetros extraídos con los resultados de la recopilación de URL de archivos históricos (gau / waybackurls / waymore) y las técnicas descritas en [Identificar los puntos de entrada de la aplicación](06-Identify_Application_Entry_Points.md) y en el capítulo de [reconocimiento de API](../12-API_Testing/01-API_Reconnaissance.md).

### Identificar archivos de mapas de origen

Los archivos de mapas de origen suelen cargarse al abrir DevTools. Los evaluadores también pueden encontrar archivos de mapas de origen añadiendo la extensión «.map» después de la extensión de cada archivo JavaScript externo. Por ejemplo, si un evaluador ve un archivo `/static/js/main.chunk.js`, puede buscar su archivo de mapa de origen visitando `/static/js/main.chunk.js.map`.

Revisar los archivos de mapas de origen en busca de información sensible que pueda ayudar al atacante a conocer mejor la aplicación. Por ejemplo:

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

Cuando los sitios cargan archivos de mapas de origen, el código fuente del frontend se vuelve legible y más fácil de depurar.

### Identificar respuestas de redirección que filtran información

Aunque, por lo general, no se espera que las respuestas de redirección contengan contenido web significativo, no hay garantías de que no puedan contenerlo. Por tanto, si bien las respuestas de la serie 300 (redirección) suelen contener texto del tipo «redireccionando a `https://example.com/`», también pueden filtrar contenido.

Pensemos en una situación en la que una respuesta de redirección sea el resultado de una comprobación de autenticación o autorización. Si esa comprobación falla, el servidor puede responder redirigiendo al usuario a una página «segura» o «predeterminada»; sin embargo, la propia respuesta de redirección aún puede contener contenido que no se muestra en el navegador, pero que sí se transmite al cliente. Esto puede observarse con las herramientas para desarrolladores del navegador o mediante un proxy personal (como ZAP, Burp, Fiddler o Charles).

### Revisar los archivos generados en busca de fugas de metadatos

Las aplicaciones web pueden generar archivos descargables, como PDF, hojas de cálculo, facturas o informes. Estos archivos pueden contener metadatos incrustados que describen el software o las bibliotecas utilizados para crearlos.

Los campos de metadatos, como *Producer*, *Creator*, *Application* o *Library Version*, pueden revelar tecnologías del backend y números de versión específicos. Esta información puede ayudar a los atacantes a identificar la tecnología de la aplicación y a encontrar vulnerabilidades conocidas públicamente (CVE).

Por ejemplo, un PDF generado podría revelar la biblioteca utilizada para crearlo:

`Producer: iText 2.1.7`

A continuación, un atacante podría buscar vulnerabilidades conocidas que afecten a esa versión.

#### Cómo probar

Descargar los archivos generados por la aplicación e inspeccionar sus metadatos.

Para archivos PDF:

```bash
exiftool file.pdf
pdfinfo file.pdf
strings file.pdf | grep -i producer
```

Ejemplo de salida:

`Producer: iText 2.1.7`

Para documentos de Office (DOCX, XLSX):

```bash
exiftool file.docx
```

Revisar campos de metadatos como:

- Producer
- Creator
- Application
- Creation Tool
- Library Version

## Herramientas

- [wget](https://www.gnu.org/software/wget)
- Función «ver código fuente» del navegador
- Inspección visual
- [Curl](https://curl.se/)
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
- [gau](https://github.com/lc/gau)
- [waymore](https://github.com/xnl-h4ck3r/waymore)
- Extracción de endpoints y secretos de JavaScript:
    - [LinkFinder](https://github.com/GerbenJavado/LinkFinder)
    - [xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder)
    - [jsluice](https://github.com/BishopFox/jsluice)
    - [SecretFinder](https://github.com/m4ll0k/SecretFinder) (o herramientas similares basadas en expresiones regulares)
- [Google Maps API Scanner](https://github.com/ozguralp/gmapsapiscanner/)
- [exiftool](https://exiftool.org/)

## Referencias

- [KeyHacks](https://github.com/streaak/keyhacks)
- [RingZer0 Online CTF](https://ringzer0ctf.com/challenges/104) - Desafío 104 «Admin Panel».

### Documentos técnicos

- [Versión HTML 4.01](https://www.w3.org/TR/1999/REC-html401-19991224)
- [XHTML](https://www.w3.org/TR/2010/REC-xhtml-basic-20101123/)
- [Estándar vivo HTML de WHATWG](https://html.spec.whatwg.org/multipage/)