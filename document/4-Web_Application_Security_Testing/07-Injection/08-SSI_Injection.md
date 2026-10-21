# Inyección SSI

|ID          |
|------------|
|WSTG-INJT-08|

## Resumen

Los servidores web suelen ofrecer a los desarrolladores la capacidad de añadir pequeños fragmentos de código dinámico dentro de páginas HTML estáticas, sin tener que recurrir a lenguajes completos del lado del servidor o del cliente. Esta característica la proporcionan los [Server-Side Includes](https://owasp.org/www-community/attacks/Server-Side_Includes_%28SSI%29_Injection) (SSI).

Los Server-Side Includes son directivas que el servidor web analiza antes de servir la página al usuario. Representan una alternativa a escribir programas CGI o incrustar código mediante lenguajes de scripting del lado del servidor cuando solo se necesitan realizar tareas muy sencillas. Las implementaciones comunes de SSI proporcionan directivas (comandos) para incluir archivos externos, establecer e imprimir variables del entorno CGI del servidor web, o ejecutar scripts CGI externos o comandos del sistema.

SSI puede provocar la ejecución remota de comandos (RCE); sin embargo, la mayoría de los servidores web tienen la directiva `exec` deshabilitada de forma predeterminada.

Esta vulnerabilidad es muy similar a una vulnerabilidad clásica de inyección en un lenguaje de scripting. Una medida de mitigación consiste en configurar el servidor web para permitir SSI. Por otro lado, las vulnerabilidades de inyección SSI suelen ser más fáciles de explotar, ya que las directivas SSI son fáciles de entender y, al mismo tiempo, bastante potentes; por ejemplo, pueden mostrar el contenido de archivos y ejecutar comandos del sistema.

## Objetivos de la prueba

- Identificar puntos de inyección SSI.
- Evaluar la gravedad de la inyección.

## Cómo probar

Para comprobar si es posible explotar SSI, inyecte directivas SSI como entrada del usuario. Si SSI está habilitado y la validación de la entrada del usuario no se ha implementado correctamente, el servidor ejecutará la directiva. Esto es muy similar a una vulnerabilidad clásica de inyección en un lenguaje de scripting, ya que se produce cuando la entrada del usuario no se valida ni se sanea correctamente.

Primero determine si el servidor web admite directivas SSI. A menudo, la respuesta es sí, ya que el soporte para SSI es bastante común. Para determinar si se admiten directivas SSI, averigüe qué tipo de servidor web ejecuta el objetivo mediante técnicas de recopilación de información (consulte [Identificación del servidor web](../01-Information_Gathering/02-Fingerprint_Web_Server.md)). Si tiene acceso al código, determine si se utilizan directivas SSI buscando palabras clave específicas en los archivos de configuración del servidor web.

Otra forma de comprobar si las directivas SSI están habilitadas es buscar páginas con la extensión `.shtml`, asociada con las directivas SSI. El uso de la extensión `.shtml` no es obligatorio, por lo que no haber encontrado archivos `.shtml` no significa necesariamente que el objetivo no sea vulnerable a ataques de inyección SSI.

El siguiente paso consiste en determinar todos los posibles vectores de entrada del usuario y comprobar si es posible explotar la inyección SSI.

Primero encuentre todas las páginas que admiten entradas del usuario. Los posibles vectores de entrada también pueden incluir encabezados y cookies. Determine cómo se almacena y utiliza la entrada; es decir, si se devuelve como mensaje de error o elemento de página y si se modificó de alguna manera. El acceso al código fuente puede ayudarle a determinar con mayor facilidad dónde están los vectores de entrada y cómo se gestionan las entradas.

Una vez que tenga una lista de posibles puntos de inyección, puede determinar si la entrada se valida correctamente. Asegúrese de que sea posible inyectar caracteres utilizados en directivas SSI, como `<!#=/."->` y `[a-zA-Z0-9]`

El siguiente ejemplo devuelve el valor de la variable. La sección de [referencias](#references) contiene enlaces útiles a documentación específica de cada servidor que pueden ayudarle a evaluar mejor un sistema determinado.

```html
<!--#echo var="VAR" -->
```

Al utilizar la directiva `include`, si el archivo proporcionado es un script CGI, esta directiva incluirá la salida del script CGI. También puede utilizarse para incluir el contenido de un archivo o enumerar los archivos de un directorio:

```html
<!--#include virtual="FILENAME" -->
```

Para devolver la salida de un comando del sistema:

```html
<!--#exec cmd="OS_COMMAND" -->
```

Si la aplicación es vulnerable, la directiva se inyecta y el servidor la interpretará la próxima vez que se sirva la página.

Las directivas SSI también pueden inyectarse en los encabezados HTTP si la aplicación web utiliza esos datos para construir una página generada dinámicamente:

```text
GET / HTTP/1.1
Host: www.example.com
Referer: <!--#exec cmd="/bin/ps ax"-->
User-Agent: <!--#include virtual="/proc/version"-->
```

## Herramientas

- [Proxy web Burp Suite](https://portswigger.net/burp/communitydownload)
- [ZAP](https://www.zaproxy.org/)
- [Buscador de cadenas: grep](https://www.gnu.org/software/grep)

## Referencias

- [Módulo SSI de Nginx](https://nginx.org/en/docs/http/ngx_http_ssi_module.html)
- [Apache: módulo mod_include](https://httpd.apache.org/docs/current/mod/mod_include.html)
- [IIS: directivas Server-Side Includes](https://learn.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525185%28v=vs.90%29)
- [Tutorial de Apache: introducción a Server-Side Includes](https://httpd.apache.org/docs/current/howto/ssi.html)
- [Apache: recomendaciones de seguridad para la configuración del servidor](https://httpd.apache.org/docs/current/misc/security_tips.html#ssi)
- [Inyección SSI en lugar de malware JavaScript](https://blog.jeremiahgrossman.com/2006/08/ssi-injection-instead-of-javascript.html)
- [Explotación basada en encabezados](https://www.cgisecurity.com/papers/header-based-exploitation.txt)