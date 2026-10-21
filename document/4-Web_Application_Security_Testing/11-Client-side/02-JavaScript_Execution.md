# Ejecución de JavaScript

|ID          |
|------------|
|WSTG-CLNT-02|

## Resumen

Una vulnerabilidad de inyección de JavaScript es un subtipo de secuencias de comandos en sitios cruzados (XSS) que consiste en la capacidad de inyectar código JavaScript arbitrario que la aplicación ejecuta en el navegador de la víctima. Esta vulnerabilidad puede tener muchas consecuencias, como la divulgación de las cookies de sesión de un usuario, que podrían usarse para suplantar a la víctima o, en términos más generales, permitir que el atacante modifique el contenido de la página que ven las víctimas o el comportamiento de la aplicación.

Las vulnerabilidades de inyección de JavaScript pueden ocurrir cuando la aplicación no valida correctamente las entradas proporcionadas por el usuario ni las salidas. Como JavaScript se usa para rellenar páginas web de forma dinámica, esta inyección ocurre durante la fase de procesamiento del contenido y, por consiguiente, afecta a la víctima.

Al probar esta vulnerabilidad, ten en cuenta que algunos caracteres se interpretan de manera diferente en distintos navegadores. Como referencia, consulta [XSS basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).

A continuación, se muestra un ejemplo de un script que no valida la variable `rr`. La variable contiene una entrada proporcionada por el usuario mediante la cadena de consulta y, además, no aplica ningún tipo de codificación:

```js
var rr = location.search.substring(1);
if(rr) {
    window.location=decodeURIComponent(rr);
}
```

Esto implica que un atacante podría inyectar código JavaScript simplemente enviando la siguiente cadena de consulta: `www.victim.com/?javascript:alert(1)`.

## Objetivos de la prueba

- Identificar sumideros y posibles puntos de inyección de JavaScript.

## Cómo probar

Considera una página que contenga el siguiente script:

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

El código anterior contiene una fuente, `location.hash`, que controla el atacante y que puede inyectar directamente código JavaScript en el valor `message` para tomar el control del navegador del usuario.