# Reconocimiento de WebAssembly

|ID          |
|------------|
|WSTG-WASM-01|

## Resumen

WebAssembly (Wasm) es un formato de instrucciones binarias que se entrega al navegador como un módulo compilado. Como a primera vista no es legible para las personas, a menudo se considera opaco y, por tanto, más seguro que JavaScript. En la práctica, al igual que los binarios tradicionales de escritorio, los módulos Wasm se pueden localizar, identificar y analizar.

El reconocimiento es el primer paso de las pruebas de seguridad de WebAssembly. Esta fase se centra en localizar módulos Wasm en la aplicación web, identificar la cadena de herramientas con la que se generaron, comprobar si hay artefactos de depuración expuestos, como mapas de origen, DWARF (Debugging With Attributed Record Formats) y la sección `name`, y mapear la superficie de ataque de la infraestructura (URL, endpoints de API y hosts) que revelan los módulos.

En esta fase solo se **recopila información** sobre el objetivo. Se detiene deliberadamente antes de descompilar el código, leer su lógica o ejecutar el módulo con un depurador. Recopilar esta información correctamente proporciona el mapa fundamental que se necesita para las fases de prueba posteriores.

Este capítulo se centra en los módulos Wasm **entregados al navegador**. El reconocimiento de Wasm que se ejecuta fuera del navegador (entornos de ejecución WASI, plataformas de edge o sin servidor) es distinto, porque los módulos por lo general no se sirven al cliente y deben obtenerse por otros medios. Consulte la [Descripción general de las pruebas de WebAssembly](README.md) para conocer la distinción.

## Objetivos de las pruebas

- Encontrar todos los módulos WebAssembly cargados por la aplicación objetivo, incluidos los que no tienen la extensión `.wasm`, están comprimidos, se cargan de forma diferida, se cargan mediante `fetch`, se instancian manualmente o están incrustados en JavaScript.
- Identificar el compilador, la cadena de herramientas y el lenguaje de origen (p. ej., C/C++, Rust, Go, .NET) a partir de los metadatos de las secciones, las importaciones, las exportaciones y el código auxiliar.
- Detectar artefactos de depuración expuestos: mapas de origen, información DWARF y la sección personalizada `name`.
- Mapear la superficie de ataque de la infraestructura mediante la extracción de URL, endpoints de API y nombres de host o direcciones IP internos incrustados.

## Cómo realizar las pruebas

### Localización de módulos WebAssembly

JavaScript carga los binarios WebAssembly, por lo general de forma asíncrona. La forma más directa de descubrirlos es mediante las herramientas de desarrollo del navegador y la inspección del JavaScript del lado del cliente.

#### Herramientas de desarrollo del navegador

1. Abra las herramientas de desarrollo y vaya a la pestaña **Network**.
2. Vuelva a cargar la aplicación objetivo e interactúe con ella para activar las funciones que se cargan de forma diferida.
3. En Chrome y Edge, seleccione el filtro **Wasm** para mostrar solo las solicitudes de WebAssembly. Firefox no tiene un filtro específico para Wasm, así que busque `.wasm` en la lista de solicitudes y compruebe que la columna de tipo de contenido muestre `application/wasm`.
4. Compruebe también Web Workers y Service Workers. Son lugares habituales donde se carga Wasm, y su tráfico puede aparecer en objetivos separados en DevTools.

#### Inspección del código JavaScript

Si la aplicación usa carga diferida o empaquetado, busque las API de carga de WebAssembly en los archivos JavaScript:

- `WebAssembly.instantiate` / `WebAssembly.instantiateStreaming`
- `WebAssembly.compile` / `WebAssembly.compileStreaming`
- `new WebAssembly.Module` / `new WebAssembly.Instance`

Las aplicaciones no siempre cargan Wasm mediante una URL `.wasm` simple. Es posible que un módulo se obtenga con `fetch` o `XMLHttpRequest` y luego se instancie manualmente desde un `ArrayBuffer`, se descomprima en JavaScript antes de instanciarlo o se arme a partir de varios fragmentos. En estos casos, la pestaña Network solo muestra una solicitud genérica, por lo que buscar las API de carga anteriores es un método más fiable.

Si un módulo se crea en tiempo de ejecución y nunca aparece como una solicitud descargable, se puede capturar en el momento de la instanciación. Por ejemplo, un script inyectado antes de que se ejecute el código de la aplicación (mediante Local Overrides de DevTools, un fragmento de código que se ejecute antes de cargar la página o una extensión de scripts de usuario) puede envolver la API y conservar una copia de los bytes:

```js
const _instantiate = WebAssembly.instantiate;
window.__wasmDumps = [];
WebAssembly.instantiate = function (source, ...rest) {
  if (source instanceof ArrayBuffer || ArrayBuffer.isView(source)) {
    window.__wasmDumps.push(new Uint8Array(source.buffer || source).slice());
  }
  return _instantiate.call(this, source, ...rest);
};
```

El mismo enfoque de envoltura se aplica a `WebAssembly.compile`, `WebAssembly.Module` y las variantes de streaming (que reciben un `Response` en lugar de bytes y requieren clonarlo). A continuación, los bytes capturados se pueden guardar en un archivo para los pasos de análisis siguientes.

#### Módulos incrustados

Los binarios Wasm también se pueden incrustar directamente en JavaScript para evitar una solicitud HTTP adicional. Busque:

- Cadenas Base64 largas que se pasan mediante `atob` o un decodificador personalizado a un `Uint8Array`.
- URI `data:` como `data:application/octet-stream;base64,`.
- Literales de cadena muy largos, que a menudo contienen muchos caracteres escapados o no ASCII, que una función auxiliar decodifica al inicio y pasa a una de las API anteriores.

Las compilaciones de Emscripten con la opción `SINGLE_FILE` hacen esto. Las versiones recientes de Emscripten incrustan el binario de forma predeterminada mediante una codificación de cadena UTF-8 personalizada (Base64 solo se usa si se deshabilita `SINGLE_FILE_BINARY_ENCODE`), por lo que es más probable encontrar los patrones segundo y tercero en las compilaciones actuales. Consulte la [referencia de configuración de Emscripten](https://emscripten.org/docs/tools_reference/settings_reference.html#single-file-binary-encode) para obtener más información. Como la codificación varía según la versión, a menudo es más fácil capturar el módulo en el momento de la instanciación, como se muestra arriba, que decodificar la cadena manualmente.

#### Identificación de archivos Wasm por su contenido

No se base únicamente en la extensión del archivo o en el tipo MIME. Los módulos se pueden servir como `.bin`, `.dat` o sin extensión, o estar comprimidos como `.wasm.gz` o `.wasm.br`. Primero se deben descomprimir los archivos comprimidos (por ejemplo, con `gunzip` o `brotli -d`) antes de comprobar el encabezado. Los navegadores descomprimen de forma transparente las respuestas `Content-Encoding`, por lo que un archivo guardado desde DevTools suele estar ya descomprimido, mientras que uno obtenido con una herramienta de línea de comandos puede no estarlo.

Todos los binarios Wasm comienzan con el mismo encabezado de 8 bytes (los bytes mágicos `\0asm` seguidos de la versión):

```text
00 61 73 6D 01 00 00 00
```

Se puede comprobar un archivo descargado con `file module.bin` o `od -An -tx1 -N8 module.bin`. En el código fuente JavaScript, el mismo encabezado aparece como el prefijo Base64 `AGFzbQ`.

Si un archivo que la aplicación carga como módulo no comienza con estos bytes y no está comprimido, es posible que la aplicación lo codifique o cifre y lo decodifique en JavaScript antes de cargarlo. En ese caso, lea la rutina de decodificación en el código auxiliar y aplíquela al archivo descargado para obtener el binario real.

Una vez confirmado el encabezado, compruebe que el módulo sea válido con `wasm-validate` del WebAssembly Binary Toolkit (WABT):

```sh
wasm-validate module.wasm
```

Si la validación falla porque el módulo utiliza funciones del lenguaje más recientes (por ejemplo, recolección de basura), vuelva a intentarlo con `--enable-all`.

Tenga en cuenta que `WebAssembly.instantiateStreaming` y `WebAssembly.compileStreaming` requieren que el servidor envíe el tipo MIME `application/wasm`, por lo que un archivo Wasm servido con un tipo distinto puede indicar una configuración incorrecta del servidor.

### Inspección de las secciones personalizadas

Una vez descargado un módulo, enumere sus secciones con `wasm-objdump` (parte de WABT). La mayor parte de los pasos de reconocimiento restantes se pueden resolver con esta única salida:

```sh
wasm-objdump -h module.wasm
```

Busque las siguientes secciones personalizadas:

| Sección | Qué revela | Se trata en |
|---------|------------|-------------|
| `producers` | Lenguaje de origen y herramientas utilizadas para compilar el módulo | Identificación del compilador y la cadena de herramientas |
| `sourceMappingURL` | URL de un mapa de origen externo | Mapas de origen |
| `.debug_*`, `external_debug_info` | Información de depuración DWARF, incrustada o en un archivo independiente | Información de depuración DWARF |
| `name` | Nombres originales de funciones y variables | Sección Name |

Las secciones personalizadas son opcionales y se pueden eliminar en cualquier momento. Es común que las compilaciones de producción no tengan ninguna sección personalizada.

### Identificación del compilador y la cadena de herramientas

Saber si un módulo se compiló a partir de C/C++, Rust, Go u otro lenguaje ayuda a quien realiza las pruebas a comprender su modelo de memoria, las convenciones de llamada, el comportamiento en tiempo de ejecución y el código auxiliar de JavaScript previsto.

Esta información también puede ayudar a priorizar las pruebas de seguridad posteriores. Por ejemplo, una cadena de herramientas de lenguaje nativo puede indicar que conviene examinar con más detenimiento la seguridad de la memoria, la gestión de la memoria lineal, las operaciones inseguras y las interacciones entre las interfaces nativas y JavaScript; por su parte, las cadenas de herramientas específicas de un framework pueden apuntar a las superficies de ejecución e integración correspondientes.

La identificación del compilador se debe utilizar para orientar la selección y priorización de las pruebas, no como prueba de que exista una vulnerabilidad específica.

#### Detección automatizada

Herramientas como WASM-Hunter pueden identificar rápidamente el compilador o la cadena de herramientas mediante el análisis de cadenas, símbolos, importaciones y marcadores de ejecución característicos del binario Wasm.

```sh
wasm-hunter -compiler -i target_module.wasm
```

La identificación automatizada es heurística, no definitiva. Los módulos optimizados, despojados de información o ofuscados pueden eliminar o alterar los marcadores que se usan para la detección, por lo que el resultado se debe tratar como una clasificación inicial y verificarse manualmente cuando sea posible.

#### Identificación manual

Cuando la detección automatizada no esté disponible o produzca un resultado incierto, examine varios indicadores en lugar de basarse en una sola firma.

La sección personalizada `producers` suele ser un indicador directo útil, ya que las cadenas de herramientas pueden registrar los lenguajes y herramientas utilizados para compilar el módulo, como `clang`, `rustc` o `wasm-bindgen`. Puede inspeccionarla con `wasm-objdump` de WABT:

```sh
wasm-objdump -s -j producers module.wasm
```

La sección `producers` es opcional y suele faltar en las compilaciones optimizadas o reforzadas. En ese caso, examine las importaciones, exportaciones, cadenas de ejecución, código auxiliar de JavaScript asociado y artefactos de compilación relacionados del módulo. Las importaciones y exportaciones se pueden enumerar con `wasm-objdump -x -j Import module.wasm` y `wasm-objdump -x -j Export module.wasm`. Considere los indicadores individuales como pistas y, cuando sea posible, confirme la identificación con un segundo indicador independiente:

- **Emscripten (C/C++):** Importaciones como `emscripten_resize_heap`, `__syscall_*` o `invoke_*`. Una importación de módulo genérico `env` **no** es prueba suficiente, porque otras cadenas de herramientas también lo utilizan. Las compilaciones de lanzamiento suelen minimizar los nombres de importación (por ejemplo, un módulo llamado `a`), lo que oculta estos patrones.
- **WASI:** Una importación de módulo como `wasi_snapshot_preview1` o `wasi_unstable` indica una compilación que usa llamadas al sistema WASI. No identifica el lenguaje de origen, y los módulos de Emscripten también lo importan, por lo que se deben comprobar marcadores adicionales.
- **Rust (wasm-bindgen):** Importaciones o exportaciones que comienzan con `__wbindgen_` o `__wbg_`, un módulo de importación llamado `wbg` y un archivo generado `*_bg.wasm` junto a un archivo de código auxiliar `*_bg.js`. La sección `producers` suele indicar `rustc` y `wasm-bindgen`.
- **Go:** El archivo de código auxiliar `wasm_exec.js`, un módulo de importación llamado `gojs` (Go 1.21 y posteriores) o `go` (versiones anteriores), y cadenas como `runtime.wasmExit`, `runtime.gopanic` o `Go build ID`. Las versiones recientes de Go también agregan una sección personalizada llamada `go:buildid`.
- **.NET (Blazor WebAssembly):** Un archivo `dotnet.wasm` o `dotnet.native.wasm` cargado desde una ruta `_framework/`. Antes de .NET 10, también está presente un manifiesto `blazor.boot.json`. A partir de .NET 10, la configuración de arranque se integra en `dotnet.js`.
- **AssemblyScript:** Las compilaciones de lanzamiento casi no tienen marcadores: normalmente, una única importación `env.abort` y una exportación `memory`. Las compilaciones de depuración conservan rutas como `~lib/rt/` en la sección `name`, y los módulos compilados con `--exportRuntime` exportan funciones como `__new` y `__pin`.

Ningún marcador por sí solo se debe considerar concluyente. La identificación más sólida se obtiene combinando varios indicadores independientes, como los metadatos `producers`, los símbolos de ejecución, las importaciones y exportaciones, el código auxiliar y los artefactos de implementación.

La eliminación de información y la ofuscación pueden neutralizar la mayoría de los marcadores anteriores. En las compilaciones de lanzamiento es habitual eliminar las secciones `producers` y `name`, minimizar los nombres de importación y exportación y aplicar tree-shaking a los auxiliares de ejecución; por ello, es posible que quien realiza las pruebas no pueda identificar la cadena de herramientas. Un resultado no concluyente es válido en esta fase y se debe registrar como tal, en lugar de hacer conjeturas.

### Pruebas de mapas de origen y símbolos de depuración

A veces, los desarrolladores dejan artefactos de depuración en las compilaciones de producción. Esto constituye un hallazgo por sí mismo y también proporciona a quien realiza las pruebas mucho contexto.

#### Mapas de origen

Al igual que JavaScript, un módulo WebAssembly puede hacer referencia a un mapa de origen externo. La referencia se almacena en una sección personalizada llamada `sourceMappingURL`, cuyo contenido es la URL del archivo de mapa (por ejemplo, `module.wasm.map`).

Si la sección no está presente, solicite directamente ubicaciones probables como `module.wasm.map`. El código auxiliar de JavaScript también puede hacer referencia a su propio mapa de origen mediante un comentario `//# sourceMappingURL=` o un encabezado de respuesta `SourceMap`. Ese mapa describe JavaScript, no el módulo, pero aun así puede revelar los archivos de origen originales.

Si se puede acceder a un mapa, abra la pestaña **Sources** (o **Debugger**) del navegador. El navegador analiza el mapa y muestra la estructura original de los archivos. El nivel de recuperación depende del mapa:

- Si el mapa contiene `sourcesContent` o se pueden descargar los archivos de origen a los que hace referencia, se puede leer el código fuente original.
- De lo contrario, solo se recuperan las rutas de los archivos y las correspondencias con las líneas, lo que aun así revela la estructura del proyecto, los nombres de las dependencias y las rutas de los desarrolladores.

#### Información de depuración DWARF

En C, C++ y Rust, es más habitual distribuir los datos de depuración como DWARF que como mapa de origen. En la fase de reconocimiento, solo hay que comprobar si está presente: ya sea en secciones `.debug_*` dentro del módulo o en una sección `external_debug_info` que apunte a un archivo de depuración independiente («DWARF dividido»), que podría descargarse del servidor.

#### Sección Name

Aunque se eliminen los mapas de origen y DWARF, a veces se conserva la sección personalizada `name`. Contiene los nombres originales de las funciones y, en algunos casos, de las variables locales. Compruebe si está presente en la lista de secciones y vuelque su contenido con:

```sh
wasm-objdump -x -j name module.wasm
```

Los nombres de Rust y C++ suelen estar alterados por name mangling. Los nombres recuperados, como `check_license_key` o `encrypt_password`, proporcionan objetivos inmediatos para análisis posteriores.

### Mapeo de la superficie de ataque (URL, API e IP)

Los módulos WebAssembly suelen contener detalles de infraestructura codificados de forma fija. Algunos ejemplos habituales son:

- URL base y rutas de API de backend (`https://api.example.com/v2/`, `/internal/admin/`).
- WebSocket y otros endpoints que no son HTTP (`wss://realtime.example.com`).
- Nombres de host de pruebas, desarrollo o internos (`build-server.corp.local`, `staging-api.example.com`).
- Direcciones IP privadas y puertos (`10.x.x.x`, `172.16-31.x.x`, `192.168.x.x`).
- URL de servicios de terceros, como servidores de analítica, comprobación de licencias o actualización.

La extracción de esta información de enrutamiento permite mapear la arquitectura de backend de la aplicación y puede revelar endpoints a los que no se hace referencia en ninguna parte del JavaScript visible.

#### Extracción automatizada

`wasm-hunter` puede automatizar esta extracción desde un archivo local o una URL remota:

```sh
wasm-hunter -i target_module.wasm
```

#### Extracción manual

Se puede recopilar la misma información con herramientas genéricas, lo cual resulta útil cuando no se dispone de una herramienta específica. Los literales de cadena se almacenan en los segmentos de datos del módulo, por lo que se pueden buscar directamente en el binario:

```sh
# URLs and endpoints
strings -n 6 module.wasm | grep -Eio '(https?|wss?|ftp)://[^[:space:]"'"'"']+'

# Common internal hostname patterns and API paths
strings -n 6 module.wasm | grep -Ei '\.(internal|local|corp|lan)\b|/api/|/v[0-9]+/'

# Private IPv4 addresses
strings -n 6 module.wasm | grep -Eo '\b(10\.[0-9]{1,3}|172\.(1[6-9]|2[0-9]|3[01])|192\.168)\.[0-9]{1,3}\.[0-9]{1,3}\b'

# Same search for UTF-16 strings (used by some toolchains such as .NET)
strings -n 6 -el module.wasm | grep -Ei 'https?://'
```

También se pueden volcar los segmentos de datos sin procesar para revisarlos manualmente con `wasm-objdump -s -j Data module.wasm`. Las cadenas que estén comprimidas, cifradas o se armen en tiempo de ejecución no aparecerán en estos resultados; también se deben recopilar las cadenas encontradas en el código auxiliar de JavaScript o en los archivos de configuración adjuntos.

(Nota: La extracción de credenciales, tokens y claves criptográficas del binario no forma parte del alcance del reconocimiento).

## Casos de prueba relacionados

- [Descripción general de las pruebas de WebAssembly](README.md)
- [Revisar el contenido de las páginas web para detectar fugas de información (WSTG-INFO-05)](../01-Information_Gathering/05-Review_Web_Page_Content_for_Information_Leakage.md)

## Remediación

- No incluya mapas de origen, datos DWARF ni archivos de depuración divididos en las compilaciones de producción. Mantenga la información de depuración y los símbolos en un servidor de símbolos restringido o en otro entorno controlado.
- Elimine de las compilaciones de lanzamiento los metadatos de depuración y compilación que no sean necesarios. Por ejemplo, `wasm-strip` (WABT) elimina todas las secciones personalizadas, y `wasm-opt --strip-debug --strip-producers` (Binaryen) elimina la información de depuración y la sección `producers`. Cuando sea posible, configure el compilador y el enlazador para que generen artefactos de lanzamiento despojados de información.
- No codifique de forma fija nombres de host internos, direcciones IP, credenciales ni endpoints confidenciales en módulos entregados al cliente. Considere público todo lo que se entrega al navegador.
- Sirva Wasm con el tipo MIME correcto `application/wasm` y verifique que los mapas de origen, los artefactos de depuración y otros archivos de compilación que no sean de producción no sean de acceso público.

## Herramientas

- [WebAssembly Binary Toolkit (WABT)](https://github.com/WebAssembly/wabt) - Incluye `wasm-objdump` para enumerar secciones, importaciones, exportaciones y nombres, y `wasm-strip`. También proporciona `wasm2wat` y `wasm-decompile`, que pertenecen a fases de análisis posteriores.
- [wasm-tools](https://github.com/bytecodealliance/wasm-tools) - Conjunto de herramientas de Bytecode Alliance que incluye un volcado de secciones y estructuras (`wasm-tools objdump`) y un analizador que puede servir como alternativa a WABT.
- [Binaryen](https://github.com/WebAssembly/binaryen) - Proporciona `wasm-opt`, que se utiliza para eliminar la información de depuración y la sección `producers`, y `wasm-dis` para desensamblar.
- [WASM-Hunter](https://github.com/Galaxy-sc/WASM-Hunter) - Herramienta de análisis estático para mapear la superficie de ataque y detectar rápidamente compiladores, con análisis directo de URL remotas.
- `strings`, `od`, `file`, `grep` - Utilidades genéricas suficientes para comprobar encabezados y extraer cadenas básicas.

## Referencias

- [Especificación de WebAssembly: secciones personalizadas y sección Name](https://webassembly.github.io/spec/core/appendix/custom.html)
- [Convenciones de herramientas de WebAssembly: depuración (sourceMappingURL, external_debug_info, DWARF)](https://github.com/WebAssembly/tool-conventions/blob/main/Debugging.md)
- [Convenciones de herramientas de WebAssembly: sección Producers](https://github.com/WebAssembly/tool-conventions/blob/main/ProducersSection.md)
- [Estándar de depuración DWARF de WebAssembly](https://yurydelendik.github.io/webassembly-dwarf/)
- [Referencia de configuración de Emscripten: SINGLE_FILE_BINARY_ENCODE](https://emscripten.org/docs/tools_reference/settings_reference.html#single-file-binary-encode)