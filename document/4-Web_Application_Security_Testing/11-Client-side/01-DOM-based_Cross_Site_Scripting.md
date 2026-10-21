# Cross-site scripting basado en DOM

|ID          |
|------------|
|WSTG-CLNT-01|

## Resumen

[Cross-site scripting basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS) es el nombre de facto para los errores [XSS](https://owasp.org/www-community/attacks/xss/) que son resultado del contenido activo de una página en el navegador, normalmente JavaScript, que obtiene datos de entrada del usuario a través de una [fuente](https://github.com/wisec/domxsswiki/wiki/sources) y los utiliza en un [sumidero](https://github.com/wisec/domxsswiki/wiki/Sinks), lo que lleva a la ejecución de código inyectado. Este documento solo analiza errores de JavaScript que provocan XSS.

El DOM, o [Modelo de objetos del documento](https://en.wikipedia.org/wiki/Document_Object_Model), es el formato estructural utilizado para representar documentos en un navegador. El DOM permite que los scripts dinámicos, como JavaScript, hagan referencia a componentes del documento, como un campo de formulario o una cookie de sesión. El navegador también utiliza el DOM con fines de seguridad; por ejemplo, para impedir que los scripts de distintos dominios obtengan las cookies de sesión de otros dominios. Puede producirse una vulnerabilidad de XSS basado en DOM cuando una solicitud especialmente diseñada modifica contenido activo, como una función JavaScript, de manera que un elemento DOM pueda ser controlado por un atacante.

No todos los errores XSS requieren que el atacante controle el contenido que devuelve el servidor; también es posible abusar de malas prácticas de programación en JavaScript para lograr los mismos resultados. Las consecuencias son las mismas que las de un fallo XSS típico; solo cambia el método de entrega.

A diferencia de otros tipos de vulnerabilidades de cross-site scripting ([reflejado y almacenado](https://owasp.org/www-community/attacks/xss/), en las que el servidor recibe un parámetro sin sanear y luego lo devuelve al usuario, donde se ejecuta en el contexto de su navegador), una vulnerabilidad de XSS basado en DOM controla el flujo del código mediante elementos del Modelo de objetos del documento (DOM) y código elaborado por el atacante para modificar dicho flujo.

Debido a su naturaleza, las vulnerabilidades de XSS basado en DOM pueden ejecutarse en muchos casos sin que el servidor pueda determinar qué se está ejecutando realmente. Esto puede hacer que muchas técnicas generales de filtrado y detección de XSS resulten ineficaces contra estos ataques.

Este ejemplo hipotético utiliza el siguiente código del lado del cliente:

```html
<script>
document.write("Site is at: " + document.location.href + ".");
</script>
```

Un atacante podría añadir `#<script>alert('xss')</script>` a la URL de la página afectada; al ejecutarse, se mostraría un cuadro de alerta. En este caso, el código añadido no se enviaría al servidor, ya que el navegador no trata como parte de la consulta nada de lo que aparece después del carácter `#`, sino como un fragmento. En este ejemplo, el código se ejecuta inmediatamente y la página muestra una alerta con el texto "xss". A diferencia de los tipos más comunes de cross-site scripting ([reflejado y almacenado](https://owasp.org/www-community/attacks/xss/), en los que el código se envía al servidor y luego se devuelve al navegador), este código se ejecuta directamente en el navegador del usuario, sin contacto con el servidor.

Las [consecuencias](https://owasp.org/www-community/attacks/xss/) de los fallos de XSS basado en DOM son tan variadas como las de otras formas más conocidas de XSS, e incluyen la obtención de cookies, la inyección de más scripts maliciosos, etc.; por lo tanto, deben tratarse con la misma gravedad.

## Objetivos de prueba

- Identificar los sumideros DOM.
- Crear payloads para cada tipo de sumidero.

## Cómo probar

Las aplicaciones JavaScript difieren significativamente de otros tipos de aplicaciones porque a menudo el servidor las genera dinámicamente. Para entender qué código se ejecuta, es necesario rastrear el sitio web que se está probando a fin de determinar todas las instancias de JavaScript que se ejecutan y dónde se acepta la entrada del usuario. Muchos sitios web dependen de grandes bibliotecas de funciones, que a menudo abarcan cientos de miles de líneas de código y no se han desarrollado internamente. En estos casos, las pruebas de arriba abajo suelen convertirse en la única opción viable, ya que muchas funciones de bajo nivel nunca se utilizan y analizarlas para determinar cuáles son sumideros lleva más tiempo del que suele estar disponible. Lo mismo puede decirse de las pruebas de arriba abajo si no se identifican desde el principio las entradas, o la falta de ellas.

La entrada del usuario adopta principalmente dos formas:

- Entrada que el servidor escribe en la página de una manera que no permite XSS directo, y
- Entrada obtenida de objetos JavaScript del lado del cliente.

Estos son dos ejemplos de cómo el servidor puede insertar datos en JavaScript:

```js
var data = "<escaped data from the server>";
var result = someFunction("<escaped data from the server>");
```

Estos son dos ejemplos de entrada obtenida de objetos JavaScript del lado del cliente:

```js
var data = window.location;
var result = someFunction(window.referrer);
```

Aunque hay poca diferencia en el código JavaScript respecto a cómo se recuperan, es importante tener en cuenta que, cuando la entrada se recibe a través del servidor, este puede aplicar a los datos las transformaciones que desee. En cambio, las transformaciones que realizan los objetos JavaScript se conocen y documentan bastante bien. Si `someFunction` fuera un sumidero en el ejemplo anterior, la posibilidad de explotación en el primer caso dependería del filtrado realizado por el servidor, mientras que en el segundo dependería de la codificación que el navegador aplica al objeto `window.referrer`. Stefano Di Paulo ha escrito un excelente artículo sobre lo que devuelven los navegadores al consultar los distintos elementos de una [URL mediante los atributos document y location](https://github.com/wisec/domxsswiki/wiki/location,-documentURI-and-URL-sources).

Además, JavaScript suele ejecutarse fuera de los bloques `<script>`, como demuestran los numerosos vectores que en el pasado han permitido eludir filtros XSS. Al rastrear la aplicación, es importante observar el uso de scripts en lugares como los manejadores de eventos y los bloques CSS con atributos de expresión. También hay que evaluar cualquier objeto CSS o script externo para determinar qué código se está ejecutando.

Las pruebas automatizadas tienen un éxito muy limitado a la hora de identificar y validar XSS basado en DOM, ya que normalmente identifican XSS enviando un payload específico e intentan observarlo en la respuesta del servidor. Esto puede funcionar bien con el siguiente ejemplo sencillo, en el que el parámetro message se refleja de vuelta al usuario:

```html
<script>
var pos=document.URL.indexOf("message=")+5;
document.write(document.URL.substring(pos,document.URL.length));
</script>
```

Sin embargo, es posible que no se detecte en el siguiente caso hipotético:

```html
<script>
var navAgt = navigator.userAgent;

if (navAgt.indexOf("MSIE")!=-1) {
        document.write("You are using IE as a browser and visiting site: " + document.location.href + ".");
}
else
{
    document.write("You are using an unknown browser.");
}
</script>
```

Por esta razón, las pruebas automatizadas no detectarán las áreas que puedan ser vulnerables a XSS basado en DOM, a menos que la herramienta de prueba pueda realizar análisis adicionales del código del lado del cliente.

Por lo tanto, deben realizarse pruebas manuales, que pueden consistir en examinar las áreas del código donde se hace referencia a parámetros que podrían ser útiles para un atacante. Entre ellas se incluyen los lugares donde se escribe código dinámicamente en la página, así como otros donde se modifica el DOM o incluso se ejecutan scripts directamente.

## Remediación

Para conocer las medidas para prevenir el XSS basado en DOM, consulta la [Hoja de referencia para la prevención de XSS basado en DOM](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

## Referencias

- [DomXSSWiki](https://github.com/wisec/domxsswiki/wiki/)