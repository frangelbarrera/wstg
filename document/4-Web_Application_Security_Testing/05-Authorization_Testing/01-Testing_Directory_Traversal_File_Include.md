# Pruebas de Traversal de Directorio e Inclusión de Archivos

|ID          |
|------------|
|WSTG-ATHZ-01|

## Resumen

Muchas aplicaciones web usan y manejan archivos como parte de su operación diaria. Usando métodos de validación de input que no han sido bien diseñados o deployed, un agresor podría exploit el sistema para leer o escribir archivos que no están destinados a ser accesibles. En situaciones particulares, podría ser posible ejecutar código arbitrario o comandos del sistema.

Tradicionalmente, servidores web y aplicaciones web implementan mecanismos de autenticación para controlar acceso a archivos y recursos. Servidores web intentan confinar archivos de usuarios dentro de un "directorio raíz" o "web document root", que representa un directorio físico en el sistema de archivos. Los usuarios tienen que considerar este directorio como el directorio base en la estructura jerárquica de la aplicación web.

La definición de los privilegios se hace usando Access Control Lists (ACL) que identifican qué usuarios o grupos se supone que pueden acceder, modificar, o ejecutar un archivo específico en el servidor. Estos mecanismos están diseñados para prevenir que usuarios maliciosos accedan a archivos sensibles (por ejemplo, el común archivo `/etc/passwd` en una plataforma UNIX-like) o para evitar la ejecución de comandos del sistema.

Muchas aplicaciones web usan scripts del lado del servidor para incluir diferentes tipos de archivos. Es bastante común usar este método para manejar imágenes, templates, cargar textos estáticos, y así on. Desafortunadamente, estas aplicaciones exponen vulnerabilidades de seguridad si parámetros de input (i.e., parámetros de formulario, valores de cookie) no son correctamente validados.

En servidores web y aplicaciones web, este tipo de problema surge en ataques de path traversal/file include. By exploiting este tipo de vulnerabilidad, un atacante es capaz de leer directorios o archivos que normalmente no podrían leer, acceder a datos fuera del web document root, o incluir scripts y otros tipos de archivos de sitios externos.

Para el propósito de la OWASP Testing Guide, solo las amenazas de seguridad relacionadas con aplicaciones web serán consideradas y no amenazas a servidores web (e.g., el infame código de escape `%5c` en Microsoft IIS web server). Sugerencias de lectura adicional serán proporcionadas en la sección de referencias para lectores interesados.

Este tipo de ataque también se conoce como el ataque dot-dot-slash (`../`), directory traversal, directory climbing, o backtracking.

Durante una assessment, para descubrir flaws de path traversal y file include, testers necesitan realizar dos diferentes stages:

1. Input Vectors Enumeration (una evaluación sistemática de cada vector de input)
2. Testing Techniques (una evaluación metódica de cada técnica de ataque usada por un atacante para exploit la vulnerabilidad)

## Objetivos de Prueba

- Identificar puntos de inyección que pertenezcan a path traversal.
- Evaluar técnicas de bypassing e identificar la extent de path traversal.

## Cómo Probar

### Black-Box Testing

#### Input Vectors Enumeration

Para determinar qué parte de la aplicación es vulnerable a input validation bypassing, el tester necesita enumerar todas las partes de la aplicación que aceptan content del usuario. Esto también incluye HTTP GET y POST queries y opciones comunes como file uploads y HTML forms.

Aquí hay algunos ejemplos de los checks a realizar en esta stage:

- ¿Hay parámetros de solicitud que podrían ser usados para operaciones relacionadas con archivos?
- ¿Hay extensiones de archivo inusuales?
- ¿Hay nombres de variable interesantes?
    - `https://example.com/getUserProfile.jsp?item=ikki.html`
    - `https://example.com/index.php?file=content`
    - `https://example.com/main.cgi?home=index.htm`
- ¿Es posible identificar cookies usadas por la aplicación web para la generación dinámica de páginas o templates?
    - `Cookie: ID=d9ccd3f4f9f18cc1:TM=2166255468:LM=1162655568:S=3cFpqbJgMSSPKVMV:TEMPLATE=flower`
    - `Cookie: USER=1826cc8f:PSTYLE=GreenDotRed`

#### Testing Techniques

La siguiente stage de testing es analyzing las funciones de validación de input presentes en la aplicación web. Usando el ejemplo anterior, la página dinámica llamada `getUserProfile.jsp` carga información estática desde un archivo y muestra el content a usuarios. Un atacante podría insertar la string maliciosa `../../../../etc/passwd` para incluir el archivo de hash de passwords de un sistema Linux/UNIX. Obviamente, este tipo de ataque es posible solo si el checkpoint de validación falla; según los privilegios del sistema de archivos, la aplicación web en sí debe ser capaz de leer el archivo.

**Note:** Para probar exitosamente este flaw, el tester necesita tener conocimiento del sistema siendo tested y la location de los archivos siendo requested. No hay point requesting `/etc/passwd` desde un IIS web server.

```text
https://example.com/getUserProfile.jsp?item=../../../../etc/passwd
```

Otro ejemplo común es incluyendo content desde una fuente externa:

```text
https://example.com/index.php?file=https://www.owasp.org/malicioustxt
```

Lo mismo puede aplicarse a cookies o cualquier otro vector de input que se use para generación de página dinámica.

Más payloads de file inclusion pueden encontrarse en [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

Es importante notar que diferentes sistemas operativos usan diferentes separadores de path

- Unix-like OS:
    - root directory: `/`
    - directory separator: `/`
- Windows OS:
    - root directory: `<drive letter>:`
    - directory separator: `\` or `/`
- Classic macOS:
    - root directory: `<drive letter>:`
    - directory separator: `:`

Es un error común por developers no esperar cada forma de encoding y por lo tanto solo hacer validation para content encoded básico. Si al principio la test string no es exitosa, intenta otro esquema de encoding.

Puedes encontrar técnicas de encoding y payloads de directory traversal listos para usar en [PayloadsAllTheThings - Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)

#### Windows Specific Considerations

- Windows shell: Appending cualquiera de los siguientes a paths usados en un comando shell resulta en no difference en function:
    - Angle brackets `<` and `>` at the end of the path
    - Double quotes (closed properly) at the end of the path
    - Extraneous current directory markers such as `./` or `.\\`
    - Extraneous parent directory markers with arbitrary items that may or may not exist:
        - `file.txt`
        - `file.txt...`
        - `file.txt<spaces>`
        - `file.txt""""`
        - `file.txt<<<>>><`
        - `./././file.txt`
        - `nonexistant/../file.txt`
- Windows API: Los siguientes items son discarded cuando usados en cualquier comando shell o API call donde una string es tomada como un filename:
    - periods
    - spaces
- Windows UNC Filepaths: Used to reference files on SMB shares. Sometimes, an application can be made to refer to files on a remote UNC filepath. If so, the Windows SMB server may send stored credentials to the attacker, which can be captured and cracked. These may also be used with a self-referential IP address or domain name to evade filters, or used to access files on SMB shares inaccessible to the attacker, but accessible from the web server.
    - `\\server_or_ip\path\to\file.abc`
    - `\\?\server_or_ip\path\to\file.abc`
- Windows NT Device Namespace: Used to refer to the Windows device namespace. Certain references will allow access to file systems using a different path.
    - May be equivalent to a drive letter such as `c:\`, or even a drive volume without an assigned letter: `\\.\GLOBALROOT\Device\HarddiskVolume1\`
    - Refers to the first disc drive on the machine: `\\.\CdRom0\`

### Gray-Box Testing

Cuando el analysis se realiza con un gray-box testing approach, testers tienen que follow la misma methodology como en black-box testing. However, since they can review the source code, it is possible to search the input vectors more easily and accurately. During a source code review, they can use simple tools (such as the *grep* command) to search for one or more common patterns within the application code: inclusion functions/methods, filesystem operations, and so on.

- `PHP: include(), include_once(), require(), require_once(), fopen(), readfile(), ...`
- `JSP/Servlet: java.io.File(), java.io.FileReader(), ...`
- `ASP: include file, include virtual, ...`

Using online code search engines (e.g., [Searchcode](https://searchcode.com/)), it may also be possible to find path traversal flaws in Open Source software published on the internet.

For PHP, testers can use the following regex:

```text
(include|require)(_once)?\s*['"(]?\s*\$_(GET|POST|COOKIE)
```

Using the gray-box testing method, it is possible to discover vulnerabilities that are usually harder to discover, or even impossible to find during a standard black-box assessment.

Some web applications generate dynamic pages using values and parameters stored in a database. It may be possible to insert specially crafted path traversal strings when the application adds data to the database. This kind of security problem is difficult to discover due to the fact the parameters inside the inclusion functions seem internal and **safe** but are not in reality.

Additionally, by reviewing the source code it is possible to analyze the functions that are supposed to handle invalid input: some developers try to change invalid input to make it valid, avoiding warnings and errors. These functions are usually prone to security flaws.

Consider a web application with these instructions:

```php
filename = Request.QueryString("file");
Replace(filename, "/","\");
Replace(filename, "..\","");
```

Testing for the flaw is achieved by:

```text
file=....//....//boot.ini
file=....\\....\\boot.ini
file= ..\..\boot.ini
```

## Herramientas

- [DotDotPwn - The Directory Traversal Fuzzer](https://github.com/wireghoul/dotdotpwn)
- [Path Traversal Fuzz Strings (from WFuzz Tool)](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/Traversal.txt)
- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net)
- Encoding/Decoding tools
- [String searcher "grep"](https://www.gnu.org/software/grep/)
- [DirBuster](https://wiki.owasp.org/index.php/Category:OWASP_DirBuster_Project)

## Referencias

- [PayloadsAllTheThings - Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
- [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

### Whitepapers

- [phpBB Attachment Mod Directory Traversal HTTP POST Injection](https://seclists.org/vulnwatch/2004/q4/33)
- [Windows File Pseudonyms: Pwnage and Poetry](https://www.slideshare.net/BaronZor/windows-file-pseudonyms)