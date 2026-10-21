# E. Inyección codificada {#encoded-injection}

## Antecedentes

La codificación de caracteres es el proceso de asignar caracteres, números y otros símbolos a un formato estándar. Por lo general, esto se hace para crear un mensaje listo para transmitirse entre un emisor y un receptor. En términos sencillos, consiste en convertir caracteres (pertenecientes a distintos idiomas, como el inglés, el chino, el griego o cualquier otro idioma conocido) en bytes. Un ejemplo de esquema de codificación de caracteres ampliamente utilizado es el Código Estándar Estadounidense para el Intercambio de Información (ASCII), que inicialmente usaba códigos de 7 bits. Entre los ejemplos más recientes de esquemas de codificación se encuentran los estándares de la industria informática Unicode `UTF-8` y `UTF-16`.

En el ámbito de la seguridad de las aplicaciones, y debido a la gran variedad de esquemas de codificación disponibles, la codificación de caracteres se utiliza con frecuencia de forma indebida. Se usa para codificar cadenas de inyección maliciosas de manera que queden ofuscadas. Esto puede permitir evadir filtros de validación de entradas o aprovechar formas específicas en que los navegadores muestran texto codificado.

## Codificación de entradas - Evasión de filtros

Las aplicaciones web suelen emplear distintos tipos de mecanismos de filtrado de entradas para limitar lo que los usuarios pueden enviar. Si estos filtros no están implementados adecuadamente, es posible hacer pasar uno o dos caracteres a través de ellos. Por ejemplo, en ASCII, un `/` puede representarse como `2F` (hexadecimal), mientras que en Unicode el mismo carácter (`/`) se codifica como `C0` `AF` (secuencia de 2 bytes). Por lo tanto, es importante que el control de filtrado de entradas tenga en cuenta el esquema de codificación utilizado. Si se determina que el filtro solo detecta inyecciones codificadas en `UTF-8`, se puede emplear otro esquema de codificación para evadirlo.

## Codificación de salida - Consenso entre servidor y navegador

Los navegadores web necesitan conocer el esquema de codificación utilizado para mostrar correctamente una página web. Lo ideal es que esta información se proporcione al navegador en el campo del encabezado HTTP (`Content-Type`), como se muestra a continuación:

```http
Content-Type: text/html; charset=UTF-8
```

o mediante una etiqueta META de HTML (`META HTTP-EQUIV`), como se muestra a continuación:

``` html
<META http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
```

Estas declaraciones de codificación de caracteres permiten que el navegador entienda qué conjunto de caracteres debe usar al convertir bytes en caracteres. Ten en cuenta que el tipo de contenido indicado en el encabezado HTTP tiene precedencia sobre la declaración de la etiqueta META.

CERT lo describe de la siguiente manera:

Muchas páginas web dejan sin definir la codificación de caracteres (el parámetro `charset` en HTTP). En las primeras versiones de HTML y HTTP, se suponía que la codificación de caracteres predeterminada era `ISO-8859-1` si no se definía. De hecho, muchos navegadores tenían un valor predeterminado diferente, por lo que no era posible confiar en que el valor predeterminado fuera `ISO-8859-1`. La versión 4 de HTML legitima esto: si no se especifica la codificación de caracteres, se puede usar cualquier codificación de caracteres.

Si el servidor web no especifica qué codificación de caracteres se está usando, no puede determinar qué caracteres son especiales. Las páginas web con una codificación de caracteres no especificada funcionan la mayor parte del tiempo porque la mayoría de los conjuntos de caracteres asignan los mismos caracteres a los valores de bytes inferiores a 128. Pero ¿cuáles de los valores superiores a 128 son especiales? Algunos esquemas de `codificación de caracteres` de 16 bits tienen representaciones adicionales de varios bytes para caracteres especiales, como `<`. Algunos navegadores reconocen esta codificación alternativa y actúan en consecuencia. Este comportamiento es «correcto», pero dificulta mucho más la prevención de ataques con scripts maliciosos. El servidor simplemente no sabe qué secuencias de bytes representan los caracteres especiales.

Por lo tanto, si no recibe del servidor la información sobre la codificación de caracteres, el navegador intenta adivinar el esquema de codificación o recurre a un esquema predeterminado. En algunos casos, el usuario establece explícitamente en el navegador una codificación predeterminada diferente. Cualquier discrepancia entre el esquema de codificación usado por la página web (servidor) y el del navegador puede hacer que este interprete la página de una manera no deseada o inesperada.

### Inyecciones codificadas

Todos los escenarios que se presentan a continuación constituyen solo un subconjunto de las distintas formas de lograr la ofuscación para evadir filtros de entradas. Además, el éxito de las inyecciones codificadas depende del navegador utilizado. Por ejemplo, anteriormente las inyecciones codificadas en `US-ASCII` solo funcionaban en el navegador IE, pero no en Firefox. Por lo tanto, cabe señalar que las inyecciones codificadas dependen en gran medida del navegador.

### Codificación básica

Considera un filtro básico de validación de entradas que protege contra la inyección del carácter de comilla simple. En este caso, la siguiente inyección evadiría fácilmente este filtro:

``` html
<script>alert(String.fromCharCode(88,83,83))</script>
```

La función JavaScript `String.fromCharCode` toma los valores Unicode proporcionados y devuelve la cadena correspondiente. Esta es una de las formas más básicas de inyección codificada. Otro vector que puede usarse para evadir este filtro es:

``` html
<IMG src="" onerror=javascript:alert(&quot;XSS&quot;)>
```

O usando los [códigos de caracteres HTML](https://www.rapidtables.com/code/text/unicode-characters.html) correspondientes:

``` html
<IMG src="" onerror="javascript:alert(&#34;XSS&#34;)">
```

Lo anterior usa entidades HTML para construir la cadena de inyección. La codificación de entidades HTML se usa para mostrar caracteres que tienen un significado especial en HTML. Por ejemplo, `>` funciona como corchete de cierre de una etiqueta HTML. Para mostrar realmente este carácter en la página web, deben insertarse entidades de caracteres HTML en el código fuente de la página. Las inyecciones mencionadas anteriormente son una forma de codificación. Existen muchas otras maneras de codificar (ofuscar) una cadena para evadir el filtro descrito.

### Codificación hexadecimal

Hex, abreviatura de hexadecimal, es un sistema de numeración en base 16; es decir, tiene 16 valores distintos, de `0` a `9` y de `A` a `F`, para representar distintos caracteres. La codificación hexadecimal es otra forma de ofuscación que a veces se usa para evadir filtros de validación de entradas. Por ejemplo, la versión codificada en hexadecimal de la cadena `<IMG SRC=javascript:alert('XSS')>` es:

``` html
<IMG SRC=%6A%61%76%61%73%63%72%69%70%74%3A%61%6C%65%72%74%28%27%58%53%53%27%29>
```

A continuación se muestra una variación de la cadena anterior. Puede usarse en caso de que se filtre `%`:

``` html
<IMG SRC=&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x27&#x58&#x53&#x53&#x27&#x29>
```

Existen otros esquemas de codificación, como Base64 y octal, que pueden usarse para ofuscar. Aunque no todos los esquemas de codificación funcionarán en todos los casos, un poco de prueba y error, combinado con manipulaciones inteligentes, sin duda revelará la vulnerabilidad de un filtro de validación de entradas mal construido.

### Codificación UTF-7

La codificación UTF-7 de

``` html
<SCRIPT>
    alert(‘XSS’);
</SCRIPT>
```

es la siguiente:

`+ADw-SCRIPT+AD4-alert('XSS');+ADw-/SCRIPT+AD4-`

Para que el script anterior funcione, el navegador debe interpretar la página web como codificada en `UTF-7`.

### Codificación multibyte

La codificación de ancho variable es otro tipo de esquema de codificación de caracteres que usa códigos de distintas longitudes para codificar caracteres. La codificación multibyte es un tipo de codificación de ancho variable que usa una cantidad variable de bytes para representar un carácter. Se usa principalmente para codificar caracteres que pertenecen a conjuntos de caracteres grandes, como el chino, el japonés y el coreano.

En el pasado, la codificación multibyte se ha usado para evadir funciones estándar de validación de entradas y llevar a cabo ataques de cross-site scripting e inyección SQL.

## Referencias

- [Codificación (semiótica)](https://en.wikipedia.org/wiki/Encoding_(semiotics))
- [Entidades HTML](https://www.w3schools.com/HTML/html_entities.asp)
- [Unicode y conjuntos de caracteres](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)