# Inyección Codificada

## Antecedentes

La codificación de caracteres es el proceso de mapear caracteres, números y otros símbolos a un formato estándar. Típicamente, esto se hace para crear un mensaje listo para transmisión entre remitente y receptor. Es, en términos simples, la conversión de caracteres (pertenecientes a diferentes lenguajes como inglés, chino, griego o cualquier otro lenguaje conocido) en bytes. Un ejemplo de un esquema de codificación de caracteres ampliamente usado es el Código Estándar Americano para Intercambio de Información (ASCII) que inicialmente usó códigos de 7 bits. Ejemplos más recientes de esquemas de codificación serían los estándares de la industria de computación Unicode `UTF-8` y `UTF-16`.

En el espacio de seguridad de aplicaciones y debido a la plétora de esquemas de codificación disponibles, la codificación de caracteres tiene un mal uso popular. Se está usando para codificar cadenas de inyección maliciosas de una manera que las ofusca. Esto puede llevar al bypass de filtros de validación de entrada, o aprovechar formas particulares en las que los navegadores renderizan texto codificado.

## Codificación de Entrada – Evasión de Filtros

Las aplicaciones web usualmente emplean diferentes tipos de mecanismos de filtrado de entrada para limitar la entrada que puede ser enviada por el usuario. Si estos filtros de entrada no están implementados suficientemente bien, es posible deslizar un carácter o dos a través de estos filtros. Por instancia, un `/` puede ser representado como `2F` (hex) en ASCII, mientras que el mismo carácter (`/`) está codificado como `C0` `AF` en Unicode (secuencia de 2 bytes). Por lo tanto, es importante para el control de filtrado de entrada estar al tanto del esquema de codificación usado. Si el filtro se encuentra detectando solo inyecciones codificadas en `UTF-8`, un esquema de codificación diferente puede ser empleado para bypass este filtro.

## Codificación de Salida – Consenso de Servidor y Navegador

Los navegadores web necesitan estar al tanto del esquema de codificación usado para mostrar coherentemente una página web. Idealmente, esta información debería ser proporcionada al navegador en el campo de encabezado HTTP (`Content-Type`), como se muestra abajo:

```http
Content-Type: text/html; charset=UTF-8
```

o a través de la etiqueta HTML META (`META HTTP-EQUIV`), como se muestra abajo:

``` html
<META http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
```

Es a través de estas declaraciones de codificación de caracteres que el navegador entiende qué conjunto de caracteres usar cuando convirtiendo bytes a caracteres. Nota que el tipo de contenido mencionado en el encabezado HTTP tiene precedencia sobre la declaración de etiqueta META.

CERT lo describe aquí como sigue:

Muchas páginas web dejan la codificación de caracteres (`charset` parámetro en HTTP) indefinida. En versiones anteriores de HTML y HTTP, la codificación de caracteres se suponía que defaultaba a `ISO-8859-1` si no estaba definida. De hecho, muchos navegadores tenían un default diferente, así que no era posible confiar en el default siendo `ISO-8859-1`. HTML versión 4 legitima esto - si la codificación de caracteres no está especificada, cualquier codificación de caracteres puede ser usada.

Si el servidor web no especifica qué codificación de caracteres está en uso, no puede decir cuáles caracteres son especiales. Las páginas web con codificación de caracteres no especificada funcionan la mayor parte del tiempo porque la mayoría de los conjuntos de caracteres asignan los mismos caracteres a valores de byte por debajo de 128. Pero cuáles de los valores por encima de 128 son especiales? Algunos esquemas de `codificación de caracteres` de 16 bits tienen representaciones multi-byte adicionales para caracteres especiales como `<`. Algunos navegadores reconocen esta codificación alternativa y actúan sobre ella. Este es comportamiento "correcto", pero hace ataques usando scripts maliciosos mucho más difíciles de prevenir. El servidor simplemente no sabe cuáles secuencias de bytes representan los caracteres especiales.

Por lo tanto en el evento de no recibir la información de codificación de caracteres del servidor, el navegador o intenta adivinar el esquema de codificación o revierte a un esquema default. En algunos casos, el usuario establece explícitamente el default de codificación en el navegador a un esquema diferente. Cualquier tal mismatch en el esquema de codificación usado por la página web (servidor) y el navegador puede causar que el navegador interprete la página de una manera que es no intencionada o inesperada.

### Inyecciones Codificadas

Todos los escenarios dados abajo forman solo un subconjunto de las varias formas en que la ofuscación puede lograrse para bypass filtros de entrada. También, el éxito de inyecciones codificadas depende del navegador en uso. Por ejemplo, inyecciones codificadas en `US-ASCII` fueron previamente exitosas solo en navegador IE pero no en Firefox. Por lo tanto, puede notarse que inyecciones codificadas, en gran medida, son dependientes del navegador.

### Codificación Básica

Considera un filtro básico de validación de entrada que protege contra inyección de carácter de comilla simple. En este caso la siguiente inyección fácilmente bypassaría este filtro:

``` html
<script>alert(String.fromCharCode(88,83,83))</script>
```

La función JavaScript `String.fromCharCode` toma los valores Unicode dados y retorna la cadena correspondiente. Esta es una de las formas más básicas de inyecciones codificadas. Otro vector que puede usarse para bypass este filtro es:

``` html
<IMG src="" onerror=javascript:alert("XSS")>
```

O usando los respectivos [códigos de caracteres HTML](https://www.rapidtables.com/code/text/unicode-characters.html):

``` html
<IMG src="" onerror="javascript:alert(&#34;XSS&#34;)">
```

Lo arriba usa codificación de Entidades HTML para construir la cadena de inyección. La codificación de Entidades HTML se usa para mostrar caracteres que tienen un significado especial en HTML. Por instancia, `>` funciona como un bracket de cierre para una etiqueta HTML. Para mostrar este carácter en la página web las entidades de caracteres HTML deberían insertarse en la fuente de la página. Las inyecciones mencionadas arriba son una forma de codificación. Hay numerosas otras formas en las que una cadena puede ser codificada (ofuscada) para bypass el filtro arriba.

### Codificación Hex

Hex, corto para Hexadecimal, es un sistema de numeración base 16 i.e tiene 16 valores diferentes de `0` a `9` y `A` a `F` para representar varios caracteres. La codificación hex es otra forma de ofuscación que a veces se usa para bypass filtros de validación de entrada. Por instancia, la versión codificada en hex de la cadena `<IMG SRC=javascript:alert('XSS')>` es

``` html
<IMG SRC=%6A%61%76%61%73%63%72%69%70%74%3A%61%6C%65%72%74%28%27%58%53%53%27%29>
```

Una variación de la cadena arriba se da abajo. Puede usarse en caso ‘%’ esté siendo filtrado:

``` html
<IMG SRC=&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x27&#x58&#x53&#53&#x27&#x29>
```

Hay otros esquemas de codificación, como Base64 y Octal, que pueden usarse para ofuscación. Aunque, cada esquema de codificación puede no funcionar cada vez, un poco de prueba y error acoplado con manipulaciones inteligentes definitivamente revelaría el loophole en un filtro de validación de entrada débilmente construido.

### Codificación UTF-7

La codificación UTF-7 de

``` html
<SCRIPT>
    alert(‘XSS’);
</SCRIPT>
```

es como abajo

`+ADw-SCRIPT+AD4-alert('XSS');+ADw-/SCRIPT+AD4-`

Para que el script arriba funcione, el navegador tiene que interpretar la página web como codificada en `UTF-7`.

### Codificación Multi-byte

La codificación de ancho variable es otro tipo de esquema de codificación de caracteres que usa códigos de longitudes variables para codificar caracteres. La Codificación Multi-Byte es un tipo de codificación de ancho variable que usa un número variable de bytes para representar un carácter. La codificación multi-byte se usa principalmente para codificar caracteres que pertenecen a un conjunto de caracteres grande e.g. chino, japonés y coreano.

La codificación multibyte ha sido usada en el pasado para bypass funciones estándar de validación de entrada y llevar a cabo ataques de cross site scripting e inyección SQL.

## Referencias

- [Codificación (Semiotics)](https://en.wikipedia.org/wiki/Encoding_(semiotics))
- [Entidades HTML](https://www.w3schools.com/HTML/html_entities.asp)
- [Cómo prevenir ataques de validación de entrada](https://searchsecurity.techtarget.com/answer/How-to-prevent-input-validation-attacks)
- [Unicode y Conjuntos de Caracteres](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)