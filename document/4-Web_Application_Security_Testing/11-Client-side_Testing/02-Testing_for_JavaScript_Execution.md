# Pruebas de Ejecución de JavaScript

|ID          |
|------------|
|WSTG-CLNT-02|

## Resumen

Una vulnerabilidad de inyección de JavaScript es un subtipo de cross site scripting (XSS) que involucra la capacidad de inyectar código JavaScript arbitrario que es ejecutado por la aplicación dentro del navegador de la víctima. Esta vulnerabilidad puede tener muchas consecuencias, como la divulgación de las cookies de sesión del usuario que podrían ser usadas para suplantar a la víctima, o, más generalmente, puede permitir al atacante modificar el contenido de la página visto por las víctimas o el comportamiento de la aplicación.

Las vulnerabilidades de inyección de JavaScript pueden ocurrir cuando la aplicación carece de validación apropiada de entrada y salida proporcionada por el usuario. Como JavaScript se usa para poblar dinámicamente páginas web, esta inyección ocurre durante esta fase de procesamiento de contenido y consecuentemente afecta a la víctima.

Al probar esta vulnerabilidad, considerar que algunos caracteres son tratados de manera diferente por diferentes navegadores. Como referencia, ver [XSS basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).

Aquí hay un ejemplo de un script que no realiza ninguna validación de la variable `rr`. La variable contiene entrada proporcionada por el usuario vía la cadena de consulta, y adicionalmente no aplica ninguna forma de codificación:

```js
var rr = location.search.substring(1);
if(rr) {
    window.location=decodeURIComponent(rr);
}
```

Esto implica que un atacante podría inyectar código JavaScript simplemente enviando la siguiente cadena de consulta: `www.victim.com/?javascript:alert(1)`.

## Objetivos de Prueba

- Identificar sinks y posibles puntos de inyección de JavaScript.

## Cómo Probar

Considerar lo siguiente: [ejercicio de DOM XSS](http://www.domxss.com/domxss/01_Basics/04_eval.html)

La página contiene el siguiente script:

```html
<script>
function loadObj(){
    var cc=eval('('+aMess+')');
    document.getElementById('mess').textContent=cc.message;
}

if(window.location.hash.indexOf('message')==-1) {
    var aMess='({"message":"Hello User!"})';
} else {
    var aMess=location.hash.substr(window.location.hash.indexOf('message=')+8)
}
</script>
```

El código anterior contiene una fuente `location.hash` que es controlada por el atacante que puede inyectar directamente en el valor `message` un código JavaScript para tomar el control del navegador del usuario.
