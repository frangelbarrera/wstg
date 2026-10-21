# Inyección HTML

|ID          |
|------------|
|WSTG-CLNT-03|

## Resumen

La inyección HTML es un tipo de vulnerabilidad de inyección que ocurre cuando un usuario puede controlar un punto de entrada e inyectar código HTML arbitrario en una página web vulnerable. Esta vulnerabilidad puede tener muchas consecuencias, como la divulgación de las cookies de sesión de un usuario, que podrían usarse para suplantar a la víctima o, en términos más generales, permitir al atacante modificar el contenido de la página que ven las víctimas.

Esta vulnerabilidad ocurre cuando la entrada del usuario no se sanea correctamente y la salida no se codifica. Una inyección permite al atacante enviar una página HTML maliciosa a una víctima. El navegador objetivo no podrá distinguir (ni confiar en) las partes legítimas y las maliciosas de la página y, en consecuencia, analizará y ejecutará la página completa en el contexto de la víctima.

Existe una amplia variedad de métodos y atributos que pueden usarse para representar contenido HTML. Si estos métodos reciben una entrada no confiable, existe un alto riesgo de vulnerabilidad de inyección HTML. Por ejemplo, se puede inyectar código HTML malicioso mediante el método JavaScript `innerHTML`, que suele usarse para representar código HTML insertado por el usuario. Si las cadenas no se sanean correctamente, el método puede permitir la inyección HTML. Una función JavaScript que puede usarse con este propósito es `document.write()`.

El siguiente ejemplo muestra un fragmento de código vulnerable que permite usar una entrada no validada para crear HTML dinámico en el contexto de la página:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.getElementById("Welcome").innerHTML=" Hello, "+user;
```

El siguiente ejemplo muestra código vulnerable que usa la función `document.write()`:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.write("<h1>Hello, " + user +"</h1>");
```

En ambos ejemplos, esta vulnerabilidad puede explotarse con una entrada como la siguiente:

```text
https://vulnerable.site/page.html?user=<img%20src='aaa'%20onerror=alert(1)>
```

Esta entrada añadirá una etiqueta de imagen a la página que ejecutará código JavaScript arbitrario insertado por el usuario malicioso en el contexto HTML.

## Objetivos de la prueba

- Identificar los puntos de inyección HTML y evaluar la gravedad del contenido inyectado.

## Cómo probar

Considere el siguiente código HTML y script:

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