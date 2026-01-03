# Prueba de inyección SSI

|ID          |
|------------|
|WSTG-INPV-08|

## Resumen

Los servidores web suelen proporcionar a los desarrolladores la capacidad de agregar pequeñas piezas de código dinámico dentro de páginas HTML estáticas, sin tener que lidiar con lenguajes de servidor o cliente completos. Esta característica es proporcionada por [Server-Side Includes](https://owasp.org/www-community/attacks/Server-Side_Includes_%28SSI%29_Injection)(SSI).

Server-Side Includes son directivas que el servidor web analiza antes de servir la página al usuario. Representan una alternativa a escribir programas CGI o incrustar código usando lenguajes de scripting del lado del servidor, cuando solo hay necesidad de realizar tareas muy simples. Las implementaciones comunes de SSI proporcionan directivas (comandos) para incluir archivos externos, establecer e imprimir variables de entorno CGI del servidor web, o ejecutar scripts CGI externos o comandos del sistema.

SSI puede llevar a una Remote Command Execution (RCE), sin embargo, la mayoría de los servidores web tienen la directiva `exec` deshabilitada por defecto.

Esta es una vulnerabilidad muy similar a una vulnerabilidad clásica de inyección de lenguaje de scripting. Una mitigación es que el servidor web necesita estar configurado para permitir SSI. Por otro lado, las vulnerabilidades de inyección SSI a menudo son más simples de explotar, ya que las directivas SSI son fáciles de entender y, al mismo tiempo, bastante poderosas, por ejemplo, pueden mostrar el contenido de archivos y ejecutar comandos del sistema.

## Objetivos de la prueba

- Identificar puntos de inyección SSI.
- Evaluar la gravedad de la inyección.

## Cómo probar

Para probar inyecciones SSI explotables, inyecte directivas SSI como entrada de usuario. Si SSI están habilitadas y la validación de entrada de usuario no se ha implementado correctamente, el servidor ejecutará la directiva. Esto es muy similar a una vulnerabilidad clásica de inyección de lenguaje de scripting en que ocurre cuando la entrada de usuario no se valida y sanitiza correctamente.

Primero determine si el servidor web soporta directivas SSI. A menudo, la respuesta es sí, ya que el soporte SSI es bastante común. Para determinar si las directivas SSI están soportadas, descubra el tipo de servidor web que el objetivo está ejecutando usando técnicas de recopilación de información (ver [Fingerprint Web Server](../01-Information_Gathering/02-Fingerprint_Web_Server.md)). Si tiene acceso al código, determine si se usan directivas SSI buscando en los archivos de configuración del servidor web palabras clave específicas.

Otra forma de verificar que las directivas SSI están habilitadas es buscando páginas con la extensión `.shtml`, que está asociada con directivas SSI. El uso de la extensión `.shtml` no es obligatorio, por lo que no haber encontrado archivos `.shtml` no significa necesariamente que el objetivo no sea vulnerable a ataques de inyección SSI.

El siguiente paso es determinar todos los posibles vectores de entrada de usuario y probar para ver si la inyección SSI es explotable.

Primero encuentre todas las páginas donde se permite entrada de usuario. Los posibles vectores de entrada también pueden incluir encabezados y cookies. Determine cómo se almacena y usa la entrada, es decir, si la entrada se devuelve como un mensaje de error o elemento de página y si se modificó de alguna manera. El acceso al código fuente puede ayudarle a determinar más fácilmente dónde están los vectores de entrada y cómo se maneja la entrada.

Una vez que tenga una lista de puntos de inyección potenciales, puede determinar si la entrada se valida correctamente. Asegúrese de que sea posible inyectar caracteres usados en directivas SSI como `<!#=/."->` y `[a-zA-Z0-9]`

El siguiente ejemplo devuelve el valor de la variable. La sección [referencias](#referencias) tiene enlaces útiles con documentación específica del servidor para ayudarle a evaluar mejor un sistema particular.

```html
<!--#echo var="VAR" -->
```

Cuando se usa la directiva `include`, si el archivo proporcionado es un script CGI, esta directiva incluirá la salida del script CGI. Esta directiva también puede usarse para incluir el contenido de un archivo o listar archivos en un directorio:

```html
<!--#include virtual="FILENAME" -->
```

Para devolver la salida de un comando del sistema:

```html
<!--#exec cmd="OS_COMMAND" -->
```

Si la aplicación es vulnerable, la directiva se inyecta y sería interpretada por el servidor la próxima vez que se sirva la página.

Las directivas SSI también pueden inyectarse en los encabezados HTTP, si la aplicación web está usando esos datos para construir una página generada dinámicamente:

```text
GET / HTTP/1.1
Host: www.example.com
Referer: <!--#exec cmd="/bin/ps ax"-->
User-Agent: <!--#include virtual="/proc/version"-->
```

## Herramientas

- [Web Proxy Burp Suite](https://portswigger.net/burp/communitydownload)
- [ZAP](https://www.zaproxy.org/)
- [String searcher: grep](https://www.gnu.org/software/grep)

## Referencias

- [Nginx SSI module](https://nginx.org/en/docs/http/ngx_http_ssi_module.html)
- [Apache: Module mod_include](https://httpd.apache.org/docs/current/mod/mod_include.html)
- [IIS: Server-Side Includes directives](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525185%28v=vs.90%29)
- [Apache Tutorial: Introduction to Server-Side Includes](https://httpd.apache.org/docs/current/howto/ssi.html)
- [Apache: Security Tips for Server Configuration](https://httpd.apache.org/docs/current/misc/security_tips.html#ssi)
- [SSI Injection instead of JavaScript Malware](https://jeremiahgrossman.blogspot.com/2006/08/ssi-injection-instead-of-javascript.html)
- [IIS: Notes on Server-Side Includes (SSI) syntax](https://blogs.iis.net/robert_mcmurray/archive/2010/12/28/iis-notes-on-server-side-includes-ssi-syntax-kb-203064-revisited.aspx)
- [Header Based Exploitation](https://www.cgisecurity.com/papers/header-based-exploitation.txt)