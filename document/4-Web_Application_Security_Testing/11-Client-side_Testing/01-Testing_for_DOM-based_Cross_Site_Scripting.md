# Pruebas de Cross Site Scripting Basado en DOM

|ID          |
|------------|
|WSTG-CLNT-01|

## Resumen

[Cross-site scripting basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS) es el nombre de facto para los bugs [XSS](https://owasp.org/www-community/attacks/xss/) que son el resultado de contenido activo del lado del navegador en una página, típicamente JavaScript, obteniendo entrada del usuario a través de una [fuente](https://github.com/wisec/domxsswiki/wiki/sources) y usándola en un [sink](https://github.com/wisec/domxsswiki/wiki/Sinks), llevando a la ejecución de código inyectado. Este documento solo discute bugs de JavaScript que llevan a XSS.

El DOM, o [Modelo de Objeto de Documento](https://en.wikipedia.org/wiki/Document_Object_Model), es el formato estructural usado para representar documentos en un navegador. El DOM habilita scripts dinámicos tales como JavaScript para referenciar componentes del documento tales como un campo de formulario o una cookie de sesión. El DOM también es usado por el navegador para seguridad — por ejemplo, para limitar scripts en diferentes dominios de obtener cookies de sesión para otros dominios. Una vulnerabilidad de XSS basada en DOM puede ocurrir cuando contenido activo, tal como una función JavaScript, es modificado por una solicitud especialmente diseñada de manera que un elemento DOM pueda ser controlado por un atacante.

No todos los bugs XSS requieren que el atacante controle el contenido devuelto por el servidor, sino que pueden abusar de prácticas pobres de codificación JavaScript para lograr los mismos resultados. Las consecuencias son las mismas que en un fallo típico de XSS, solo el medio de entrega es diferente.

En comparación con otros tipos de vulnerabilidades cross site scripting ([reflejado y almacenado](https://owasp.org/www-community/attacks/xss/), donde un parámetro no saneado es pasado por el servidor y luego devuelto al usuario y ejecutado en el contexto del navegador del usuario), una vulnerabilidad XSS basada en DOM controla el flujo del código usando elementos del Modelo de Objeto de Documento (DOM) junto con código diseñado por el atacante para cambiar el flujo.

Debido a su naturaleza, las vulnerabilidades XSS basadas en DOM pueden ser ejecutadas en muchas instancias sin que el servidor pueda determinar qué se está ejecutando realmente. Esto puede hacer que muchas de las técnicas generales de filtrado y detección de XSS sean impotentes contra tales ataques.

Este ejemplo hipotético usa el siguiente código del lado del cliente:

```html
<script>
document.write("Site is at: " + document.location.href + ".");
</script>
```

Un atacante podría añadir `#<script>alert('xss')</script>` a la URL de la página afectada, lo cual, cuando se ejecute, mostrará la caja de alerta. En esta instancia, el código añadido no sería enviado al servidor ya que todo después del carácter `#` no es tratado como parte de la consulta por el navegador, sino como un fragmento. En este ejemplo, el código se ejecuta inmediatamente y una alerta de "xss" es mostrada por la página. A diferencia de los tipos más comunes de cross site scripting ([reflejado y almacenado](https://owasp.org/www-community/attacks/xss/), en los cuales el código se envía al servidor y luego de vuelta al navegador, esto se ejecuta directamente en el navegador del usuario sin contacto con el servidor.

Las [consecuencias](https://owasp.org/www-community/attacks/xss/) de los fallos XSS basados en DOM son tan amplias como las vistas en formas más conocidas de XSS, incluyendo recuperación de cookies, inyección adicional de scripts maliciosos, etc., y por lo tanto deberían ser tratadas con la misma severidad.

## Objetivos de Prueba

- Identificar sinks DOM.
- Construir payloads que pertenezcan a cada tipo de sink.

## Cómo Probar

Las aplicaciones JavaScript difieren significativamente de otros tipos de aplicaciones porque a menudo son generadas dinámicamente por el servidor. Para entender qué código se está ejecutando, el sitio web siendo probado necesita ser rastreado para determinar todas las instancias de JavaScript siendo ejecutado y dónde se acepta la entrada del usuario. Muchos sitios web dependen de grandes bibliotecas de funciones, las cuales a menudo se extienden a cientos de miles de líneas de código y no han sido desarrolladas internamente. En estos casos, las pruebas de arriba hacia abajo a menudo se convierten en la única opción viable, ya que muchas funciones de nivel inferior nunca se usan, y analizarlas para determinar cuáles son sinks consumirá más tiempo del que a menudo está disponible. Lo mismo puede decirse para las pruebas de arriba hacia abajo si las entradas o la falta de ellas no se identifican al principio.

La entrada del usuario viene en dos formas principales:

- Entrada escrita en la página por el servidor de una manera que no permite XSS directo, y
- Entrada obtenida de objetos JavaScript del lado del cliente.

Aquí hay dos ejemplos de cómo el servidor puede insertar datos en JavaScript:

```js
var data = "<escaped data from the server>";
var result = someFunction("<escaped data from the server>");
```

Aquí hay dos ejemplos de entrada de objetos JavaScript del lado del cliente:

```js
var data = window.location;
var result = someFunction(window.referrer);
```

Si bien hay poca diferencia para el código JavaScript en cómo se recuperan, es importante notar que cuando se recibe entrada vía el servidor, el servidor puede aplicar cualquier permutación a los datos que desee. Por otro lado, las permutaciones realizadas por objetos JavaScript están bastante bien entendidas y documentadas. Si `someFunction` en el ejemplo anterior fuera un sink, entonces la explotabilidad en el primer caso dependería del filtrado hecho por el servidor, mientras que en el último caso dependería de la codificación hecha por el navegador en el objeto `window.referrer`. Stefano Di Paulo ha escrito un excelente artículo sobre qué devuelven los navegadores cuando se les pide varios elementos de una [URL usando los atributos document y location](https://github.com/wisec/domxsswiki/wiki/location,-documentURI-and-URL-sources).

Adicionalmente, JavaScript a menudo se ejecuta fuera de los bloques `<script>`, como lo evidencian los muchos vectores que han llevado a evasiones de filtros XSS en el pasado. Al rastrear la aplicación, es importante notar el uso de scripts en lugares tales como manejadores de eventos y bloques CSS con atributos de expresión. También, notar que cualquier objeto CSS o script externo deberá ser evaluado para determinar qué código se está ejecutando.

Las pruebas automatizadas tienen solo un éxito muy limitado en identificar y validar XSS basado en DOM ya que usualmente identifica XSS enviando un payload específico e intenta observarlo en la respuesta del servidor. Esto puede funcionar bien para el ejemplo simple proporcionado a continuación, donde el parámetro message se refleja de vuelta al usuario:

```html
<script>
var pos=document.URL.indexOf("message=")+5;
document.write(document.URL.substring(pos,document.URL.length));
</script>
```

Sin embargo, puede no ser detectado en el siguiente caso artificial:

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

Por esta razón, las pruebas automatizadas no detectarán áreas que puedan ser susceptibles a XSS basado en DOM a menos que la herramienta de prueba pueda realizar análisis adicionales del código del lado del cliente.

Las pruebas manuales deberían, por lo tanto, llevarse a cabo y pueden hacerse examinando áreas en el código donde se hace referencia a parámetros que pueden ser útiles para un atacante. Ejemplos de tales áreas incluyen lugares donde el código se escribe dinámicamente en la página y en otros lugares donde el DOM se modifica o incluso donde los scripts se ejecutan directamente.

## Remediación

Para medidas para prevenir XSS basado en DOM, ver la [Hoja de Referencia de Prevención de XSS Basado en DOM](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

## Referencias

- [DomXSSWiki](https://github.com/wisec/domxsswiki/wiki/)
