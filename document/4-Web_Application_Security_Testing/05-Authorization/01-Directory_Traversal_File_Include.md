# Traversal de directorios e inclusión de archivos

|ID          |
|------------|
|WSTG-ATHZ-01|

## Resumen

Muchas aplicaciones web utilizan y gestionan archivos como parte de sus operaciones diarias. Si se usan métodos de validación de entradas que no están bien diseñados o implementados, un atacante podría explotar el sistema para leer o escribir archivos que no deberían ser accesibles. En determinadas situaciones, podría ser posible ejecutar código arbitrario o comandos del sistema.

Tradicionalmente, los servidores web y las aplicaciones web implementan mecanismos de autenticación para controlar el acceso a archivos y recursos. Los servidores web intentan confinar los archivos de los usuarios dentro de un «directorio raíz» o «raíz de documentos web», que representa un directorio físico en el sistema de archivos. Los usuarios deben considerar este directorio como el directorio base de la estructura jerárquica de la aplicación web.

Los privilegios se definen mediante listas de control de acceso (ACL), que identifican qué usuarios o grupos pueden acceder a un archivo específico del servidor, modificarlo o ejecutarlo. Estos mecanismos están diseñados para impedir que usuarios maliciosos accedan a archivos sensibles (por ejemplo, el conocido archivo `/etc/passwd` en una plataforma similar a UNIX) o para evitar la ejecución de comandos del sistema.

Muchas aplicaciones web utilizan scripts del lado del servidor para incluir distintos tipos de archivos. Es bastante común usar este método para gestionar imágenes, plantillas, cargar textos estáticos, etc. Desafortunadamente, estas aplicaciones presentan vulnerabilidades de seguridad si los parámetros de entrada (es decir, los parámetros de formularios o los valores de cookies) no se validan correctamente.

En los servidores web y las aplicaciones web, este tipo de problema se presenta en los ataques de traversal de rutas e inclusión de archivos. Al explotar este tipo de vulnerabilidad, un atacante puede leer directorios o archivos que normalmente no podría leer, acceder a datos fuera de la raíz de documentos web o incluir scripts y otros tipos de archivos de sitios externos.

Para los fines de la OWASP Testing Guide, solo se tendrán en cuenta las amenazas de seguridad relacionadas con las aplicaciones web, y no las amenazas a los servidores web (por ejemplo, el infame código de escape `%5c` en el servidor web Microsoft IIS). En la sección de referencias se ofrecerán sugerencias de lecturas adicionales para quienes tengan interés.

Este tipo de ataque también se conoce como ataque de punto-punto-barra (`../`), traversal de directorios, ascenso por directorios o retroceso.

Durante una evaluación, para descubrir fallas de traversal de rutas e inclusión de archivos, los evaluadores deben realizar dos etapas distintas:

1. Enumeración de vectores de entrada (evaluación sistemática de cada vector de entrada)
2. Técnicas de prueba (evaluación metódica de cada técnica de ataque que utiliza un atacante para explotar la vulnerabilidad)

## Objetivos de la prueba

- Identificar los puntos de inyección relacionados con el traversal de rutas.
- Evaluar las técnicas para eludir las validaciones e identificar el alcance del traversal de rutas.

## Cómo realizar las pruebas

### Pruebas de caja negra

#### Enumeración de vectores de entrada

Para determinar qué partes de la aplicación son vulnerables a la elusión de la validación de entradas, el evaluador debe enumerar todas las partes de la aplicación que aceptan contenido del usuario. Esto también incluye las solicitudes HTTP GET y POST, y opciones comunes como la carga de archivos y los formularios HTML.

Estos son algunos ejemplos de las comprobaciones que deben realizarse en esta etapa:

- ¿Hay parámetros de solicitud que podrían utilizarse para operaciones relacionadas con archivos?
- ¿Hay extensiones de archivo inusuales?
- ¿Hay nombres de variables interesantes?
    - `https://example.com/getUserProfile.jsp?item=ikki.html`
    - `https://example.com/index.php?file=content`
    - `https://example.com/main.cgi?home=index.htm`
- ¿Es posible identificar cookies que la aplicación web utilice para generar páginas o plantillas dinámicamente?
    - `Cookie: ID=d9ccd3f4f9f18cc1:TM=2166255468:LM=1162655568:S=3cFpqbJgMSSPKVMV:TEMPLATE=flower`
    - `Cookie: USER=1826cc8f:PSTYLE=GreenDotRed`

#### Técnicas de prueba

La siguiente etapa de las pruebas consiste en analizar las funciones de validación de entradas presentes en la aplicación web. En el ejemplo anterior, la página dinámica llamada `getUserProfile.jsp` carga información estática desde un archivo y muestra su contenido a los usuarios. Un atacante podría insertar la cadena maliciosa `../../../../etc/passwd` para incluir el archivo de hashes de contraseñas de un sistema Linux/UNIX. Obviamente, este tipo de ataque solo es posible si falla el punto de validación; según los privilegios del sistema de archivos, la propia aplicación web debe poder leer el archivo.

> Nota: Para probar correctamente esta falla, el evaluador debe conocer el sistema sometido a pruebas y la ubicación de los archivos solicitados. No tiene sentido solicitar `/etc/passwd` a un servidor web IIS.

```text
https://example.com/getUserProfile.jsp?item=../../../../etc/passwd
```

Otro ejemplo común consiste en incluir contenido de una fuente externa:

```text
https://example.com/index.php?file=https://www.owasp.org/malicioustxt
```

Lo mismo puede aplicarse a las cookies o a cualquier otro vector de entrada que se utilice para generar páginas dinámicamente.

Se pueden encontrar más cargas útiles para la inclusión de archivos en [PayloadsAllTheThings - Inclusión de archivos](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

Es importante tener en cuenta que los distintos sistemas operativos utilizan distintos separadores de rutas.

- Sistemas operativos similares a Unix:
    - directorio raíz: `/`
    - separador de directorios: `/`
- Windows:
    - directorio raíz: `<drive letter>:`
    - separador de directorios: `\` o `/`
- macOS clásico:
    - directorio raíz: `<drive letter>:`
    - separador de directorios: `:`

Un error común de los desarrolladores es no prever todas las formas de codificación y, por lo tanto, validar únicamente el contenido con codificación básica. Si la cadena de prueba no funciona al primer intento, prueba con otro esquema de codificación.

Puedes encontrar técnicas de codificación y cargas útiles de traversal de directorios listas para usar en [PayloadsAllTheThings - Traversal de directorios](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)

#### Consideraciones específicas de Windows

- Shell de Windows: agregar cualquiera de los elementos siguientes a las rutas utilizadas en un comando de shell no cambia su funcionamiento:
    - Corchetes angulares `<` y `>` al final de la ruta
    - Comillas dobles (cerradas correctamente) al final de la ruta
    - Marcadores redundantes del directorio actual, como `./` o `.\\`
    - Marcadores redundantes de directorios superiores con elementos arbitrarios, que pueden existir o no:
        - `file.txt`
        - `file.txt...`
        - `file.txt<spaces>`
        - `file.txt""""`
        - `file.txt<<<>>><`
        - `./././file.txt`
        - `nonexistent/../file.txt`
- API de Windows: los siguientes elementos se descartan cuando se utilizan en cualquier comando de shell o llamada a la API en la que se toma una cadena como nombre de archivo:
    - puntos
    - espacios
- Rutas UNC de Windows: se utilizan para hacer referencia a archivos en recursos compartidos SMB. En ocasiones, se puede hacer que una aplicación haga referencia a archivos en una ruta UNC remota. En ese caso, el servidor SMB de Windows podría enviar las credenciales almacenadas al atacante, quien podría capturarlas y descifrarlas. También pueden utilizarse con una dirección IP o un nombre de dominio autorreferencial para eludir filtros, o para acceder a archivos en recursos compartidos SMB que no son accesibles para el atacante, pero sí para el servidor web.
    - `\\server_or_ip\path\to\file.abc`
    - `\\?\server_or_ip\path\to\file.abc`
- Espacio de nombres de dispositivos NT de Windows: se utiliza para hacer referencia al espacio de nombres de dispositivos de Windows. Algunas referencias permiten acceder a sistemas de archivos mediante una ruta diferente.
    - Puede ser equivalente a una letra de unidad como `c:\`, o incluso a un volumen de unidad sin letra asignada: `\\.\GLOBALROOT\Device\HarddiskVolume1\`
    - Hace referencia a la primera unidad de disco del equipo: `\\.\CdRom0\`

### Pruebas de caja blanca

Cuando el análisis se realiza con un enfoque de caja blanca, los evaluadores deben seguir la misma metodología que en las pruebas de caja negra. Sin embargo, como pueden revisar el código fuente, es posible buscar los vectores de entrada con mayor facilidad y precisión. Durante la revisión del código fuente, pueden utilizar herramientas sencillas (como el comando *grep*) para buscar uno o varios patrones comunes en el código de la aplicación: funciones o métodos de inclusión, operaciones del sistema de archivos, etc.

- `PHP: include(), include_once(), require(), require_once(), fopen(), readfile(), ...`
- `JSP/Servlet: java.io.File(), java.io.FileReader(), ...`
- `ASP: include file, include virtual, ...`

También es posible encontrar fallas de traversal de rutas en software de código abierto publicado en Internet mediante motores de búsqueda de código en línea (por ejemplo, [Searchcode](https://searchcode.com/)).

En PHP, los evaluadores pueden utilizar la siguiente expresión regular:

```text
(include|require)(_once)?\s*['"(]?\s*\$_(GET|POST|COOKIE)
```

Con el método de pruebas de caja blanca, es posible descubrir vulnerabilidades que suelen ser más difíciles de detectar, o incluso imposibles de encontrar durante una evaluación estándar de caja negra.

Algunas aplicaciones web generan páginas dinámicas mediante valores y parámetros almacenados en una base de datos. Es posible que se puedan insertar cadenas de traversal de rutas especialmente diseñadas cuando la aplicación agrega datos a la base de datos. Este tipo de problema de seguridad es difícil de detectar porque los parámetros de las funciones de inclusión parecen internos y **seguros**, pero en realidad no lo son.

Además, al revisar el código fuente es posible analizar las funciones que deberían manejar entradas no válidas: algunos desarrolladores intentan modificar las entradas no válidas para hacerlas válidas y evitar advertencias y errores. Estas funciones suelen ser propensas a fallas de seguridad.

Considera una aplicación web con las siguientes instrucciones:

```php
filename = Request.QueryString("file");
Replace(filename, "/","\");
Replace(filename, "..\","");
```

La falla se puede probar de la siguiente manera:

```text
file=....//....//boot.ini
file=....\\....\\boot.ini
file= ..\..\boot.ini
```

## Herramientas

- [DotDotPwn - El fuzzier de traversal de directorios](https://github.com/wireghoul/dotdotpwn)
- [Cadenas de fuzzing para traversal de rutas (de la herramienta WFuzz)](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/Traversal.txt)
- [ZAP](https://www.zaproxy.org/) [La funcionalidad [Forced Browse de ZAP](https://www.zaproxy.org/docs/desktop/addons/forced-browse/) se basa en la continuación de DirBuster.]
- [Burp Suite](https://portswigger.net)
- Herramientas de codificación y decodificación
- [Buscador de cadenas «grep»](https://www.gnu.org/software/grep/)

## Referencias

- [PayloadsAllTheThings - Traversal de directorios](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
- [PayloadsAllTheThings - Inclusión de archivos](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

### Artículos técnicos

- [Inyección HTTP POST de traversal de directorios en Attachment Mod de phpBB](https://seclists.org/vulnwatch/2004/q4/33)
- [Seudónimos de archivos de Windows: explotación y poesía](https://www.slideshare.net/BaronZor/windows-file-pseudonyms)