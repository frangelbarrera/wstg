# Pruebas de Inyección HTML

|ID          |
|------------|
|WSTG-CLNT-03|

## Resumen

La inyección HTML es un tipo de vulnerabilidad de inyección que ocurre cuando un usuario es capaz de controlar un punto de entrada y es capaz de inyectar código HTML arbitrario en una página web vulnerable. Esta vulnerabilidad puede tener muchas consecuencias, como la divulgación de las cookies de sesión del usuario que podrían ser usadas para suplantar a la víctima, o, más generalmente, puede permitir al atacante modificar el contenido de la página visto por las víctimas.

Esta vulnerabilidad ocurre cuando la entrada del usuario no es saneada correctamente y la salida no es codificada. Una inyección permite al atacante enviar una página HTML maliciosa a una víctima. El navegador objetivo no será capaz de distinguir (confiar) entre las partes legítimas y las partes maliciosas de la página, y consecuentemente analizará y ejecutará la página completa en el contexto de la víctima.

Hay una amplia gama de métodos y atributos que podrían ser usados para renderizar contenido HTML. Si estos métodos se proporcionan con una entrada no confiable, entonces hay un alto riesgo de vulnerabilidad de inyección HTML. Por ejemplo, código HTML malicioso puede ser inyectado vía el método JavaScript `innerHTML`, usualmente usado para renderizar código HTML insertado por el usuario. Si las cadenas no son saneadas correctamente, el método puede habilitar la inyección HTML. Una función JavaScript que puede ser usada para este propósito es `document.write()`.

El siguiente ejemplo muestra un fragmento de código vulnerable que permite que una entrada no validada sea usada para crear HTML dinámico en el contexto de la página:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.getElementById("Welcome").innerHTML=" Hello, "+user;
```

El siguiente ejemplo muestra código vulnerable usando la función `document.write()`:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.write("<h1>Hello, " + user +"</h1>");
```

En ambos ejemplos, esta vulnerabilidad puede ser explotada con una entrada como:

```text
https://vulnerable.site/page.html?user=<img%20src='aaa'%20onerror=alert(1)>
```

Esta entrada añadirá una etiqueta de imagen a la página que ejecutará código JavaScript arbitrario insertado por el usuario malicioso en el contexto HTML.

## Objetivos de Prueba

- Identificar puntos de inyección HTML y evaluar la severidad del contenido inyectado.

## Cómo Probar

Considerar el siguiente ejercicio de DOM XSS <https://www.domxss.com/domxss/01_Basics/06_jquery_old_html.html>

El código HTML contiene el siguiente script:

```html
<script src="../js/jquery-1.7.1.js"></script>
<script>
function setMessage(){
    var t=location.hash.slice(1);
    $("div[id="+t+"]").text("The DOM is now loaded and can be manipulated.");
}
$(document).ready(setMessage  );
$(window).bind("hashchange",setMessage)
</script>
<body>
    <script src="../js/embed.js"></script>
    <span><a href="#message" > Show Here</a><div id="message">Showing Message1</div></span>
    <span><a href="#message1" > Show Here</a><div id="message1">Showing Message2</div>
    <span><a href="#message2" > Show Here</a><div id="message2">Showing Message3</div>
</body>
```

Es posible inyectar código HTML.
