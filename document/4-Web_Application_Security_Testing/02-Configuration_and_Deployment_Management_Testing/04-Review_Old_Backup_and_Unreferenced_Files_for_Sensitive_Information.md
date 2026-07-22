# Revisar Archivos Antiguos de Respaldo y No Referenciados para Información Sensible

|ID          |
|------------|
|WSTG-CONF-04|

## Resumen

Mientras que la mayoría de los archivos dentro de un servidor web son manejados directamente por el servidor mismo, no es inusual encontrar archivos no referenciados o olvidados que pueden ser usados para obtener información importante sobre la infraestructura o las credenciales.

Los escenarios más comunes incluyen la presencia de versiones antiguas renombradas de archivos modificados, archivos de inclusión que se cargan en el lenguaje de elección y descargados como fuente, e incluso respaldos automáticos o manuales en forma de archivos comprimidos. Los archivos de respaldo también pueden ser generados automáticamente por el sistema de archivos subyacente en el que la aplicación está alojada, una característica usualmente referida como "instantáneas".

Todos estos archivos pueden otorgar al probador acceso a trabajos internos, puertas traseras, interfaces administrativas, o incluso credenciales para conectarse a la interfaz administrativa o al servidor de base de datos.

Una fuente importante de vulnerabilidad se encuentra en archivos no relacionados con la aplicación. Estos archivos pueden ser creados al editar archivos de aplicación, creando copias de respaldo sobre la marcha, o dejando archivos antiguos o no referenciados en el árbol web. Realizar edición in situ u otras acciones administrativas en servidores web de producción puede inadvertidamente dejar copias de respaldo, ya sea generadas automáticamente por el editor mientras edita archivos, o por el administrador que está comprimiendo un conjunto de archivos para crear un respaldo.

Es fácil olvidar tales archivos y esto puede plantear una amenaza seria de seguridad a la aplicación. Sucede porque las copias de respaldo pueden ser generadas con extensiones de archivo diferentes de las de los archivos originales. Un archivo `.tar`, `.zip` o `.gz` que generamos (y podríamos olvidar) tiene obviamente una extensión diferente, y lo mismo sucede con copias automáticas creadas por muchos editores (por ejemplo, emacs genera una copia de respaldo nombrada `file~` al editar `file`). Hacer una copia manual puede producir un efecto similar, como cuando 'file' es copiado como 'file.old'. El sistema de archivos subyacente en el que la aplicación está podría estar haciendo `instantáneas` de tu aplicación en diferentes puntos en el tiempo sin tu conocimiento, que también pueden ser accesibles vía web, planteando una amenaza similar pero diferente de estilo "archivo de respaldo" a tu aplicación.

Como resultado, estas actividades generan archivos que no son necesarios por la aplicación y pueden ser manejados de manera diferente que el archivo original por el servidor web. Por ejemplo, si hacemos una copia de login.asp y la nombramos login.asp.old sin medidas de seguridad apropiadas, podría potencialmente permitir a usuarios descargar el código fuente de login.asp. Esto es porque `login.asp.old` será típicamente servido como texto o plano, en lugar de ser ejecutado debido a su extensión. En otras palabras, acceder a `login.asp` causa la ejecución del código del lado servidor de `login.asp`, mientras que acceder a `login.asp.old` causa que el contenido de `login.asp.old` (que es, nuevamente, código del lado servidor) sea plainly retornado al usuario y mostrado en el navegador. Esto puede plantear riesgos de seguridad, ya que información sensible puede ser revelada.

Generalmente, exponer código del lado servidor es una mala idea. No solo estás exponiendo innecesariamente lógica de negocio, sino que podrías estar revelando inconscientemente información relacionada con la aplicación que puede ayudar a un atacante (nombres de rutas, estructuras de datos, etc.). Sin mencionar el hecho de que hay demasiados scripts con nombre de usuario y contraseña embebidos en texto claro (que es una práctica descuidada y extremadamente peligrosa).

Otras causas de archivos no referenciados son debido a elecciones de diseño o configuración cuando permiten diversos tipos de archivos relacionados con la aplicación como archivos de datos, archivos de configuración, archivos de log, para ser almacenados en directorios del sistema de archivos que pueden ser accedidos por el servidor web. Estos archivos normalmente no tienen razón para estar en un espacio del sistema de archivos que podría ser accedido vía web, ya que deberían ser accedidos solo al nivel de aplicación, por la aplicación misma (y no por el usuario casual navegando alrededor).

### Amenazas

Los archivos antiguos, de respaldo y no referenciados presentan varias amenazas a la seguridad de una aplicación web:

- Los archivos no referenciados pueden divulgar información sensible que puede facilitar un ataque enfocado contra la aplicación; por ejemplo, incluir archivos conteniendo credenciales de base de datos, archivos de configuración conteniendo referencias a otro contenido oculto, rutas absolutas de archivos, etc.
- Las páginas no referenciadas pueden contener funcionalidad poderosa que puede ser usada para atacar la aplicación; por ejemplo, una página de administración que no está enlazada desde contenido publicado pero puede ser accedida por cualquier usuario que sepa dónde encontrarla.
- Los archivos antiguos y de respaldo pueden contener vulnerabilidades que han sido arregladas en versiones más recientes; por ejemplo, `viewdoc.old.jsp` puede contener una vulnerabilidad de traversal de directorio que ha sido arreglada en `viewdoc.jsp` pero aún puede ser explotada por cualquiera que encuentre la versión antigua.
- Los archivos de respaldo pueden divulgar el código fuente para páginas diseñadas para ejecutar en el servidor; por ejemplo, solicitar `viewdoc.bak` puede retornar el código fuente para `viewdoc.jsp`, que puede ser revisado para vulnerabilidades que pueden ser difíciles de encontrar haciendo solicitudes ciegas a la página ejecutable. Mientras esta amenaza aplica a lenguajes de scripting como Perl, PHP, ASP, scripts de shell, JSP, etc., no está limitada a ellos, como se muestra en el ejemplo proporcionado en el siguiente punto.
- Los archivos de respaldo pueden contener copias de todos los archivos dentro (o incluso fuera) del webroot. Esto permite a un atacante enumerar rápidamente toda la aplicación, incluyendo páginas no referenciadas, código fuente, archivos de inclusión, etc. Por ejemplo, si olvidas un archivo nombrado `myservlets.jar.old` conteniendo una copia de respaldo de tus clases de implementación de servlet, estás exponiendo mucha información sensible que puede ser decompilada e ingeniería inversa.
- En algunos casos, copiar o editar un archivo modifica el nombre del archivo pero deja la extensión intacta. Esto es común en entornos Windows, donde operaciones de copia de archivos generan nombres de archivo prefijados con "Copy of " o versiones localizadas de esta cadena. Ya que la extensión del archivo se deja sin cambios, este no es un caso donde un archivo ejecutable es retornado como texto plano por el servidor web, y por lo tanto no es un caso de divulgación de código fuente. Sin embargo, estos archivos son peligrosos también porque hay una chance de que incluyan lógica obsoleta e incorrecta que, cuando invocada, podría disparar errores de aplicación, que podrían rendir información valiosa a un atacante si la visualización de mensajes de diagnóstico está habilitada.
- Los archivos de log pueden contener información sensible sobre las actividades de usuarios de la aplicación, por ejemplo, datos sensibles pasados en parámetros URL, IDs de sesión, URLs visitadas (que pueden divulgar contenido adicional no referenciado), etc. Otros archivos de log (ej. logs ftp) pueden contener información sensible sobre el mantenimiento de la aplicación por administradores de sistema.
- Las instantáneas del sistema de archivos pueden contener copias del código que contienen vulnerabilidades que han sido arregladas en versiones más recientes. Por ejemplo, `/.snapshot/monthly.1/view.php` puede contener una vulnerabilidad de traversal de directorio que ha sido arreglada en `/view.php` pero aún puede ser explotada por cualquiera que encuentre la versión antigua.

## Objetivos de Prueba

- Encontrar y analizar archivos no referenciados que podrían contener información sensible.

## Cómo Probar

### Pruebas de Caja Negra

Probar para archivos no referenciados usa técnicas tanto automatizadas como manuales, y típicamente involucra una combinación de lo siguiente:

#### Inferencia del Esquema de Nombres Usado para Contenido Publicado

Enumerar todas las páginas y funcionalidad de la aplicación. Esto puede ser hecho manualmente usando un navegador, o usando una herramienta de spidering de aplicación. La mayoría de las aplicaciones usan un esquema de nombres reconocible, y organizan recursos en páginas y directorios usando palabras que describen su función. A menudo es posible inferir el nombre y ubicación de páginas no referenciadas del esquema de nombres usado para contenido publicado. Por ejemplo, si una página titulada viewuser.asp es encontrada, uno debería también buscar edituser.asp, adduser.asp, y deleteuser.asp. Similarmente, si un directorio /app/user es descubierto, uno debería también buscar /app/admin y /app/manager.

#### Otras Pistas en Contenido Publicado

Muchas aplicaciones web dejan pistas en contenido publicado que pueden llevar al descubrimiento de páginas y funcionalidad ocultas. Estas pistas pueden a menudo ser encontradas en el código fuente de archivos HTML y JavaScript. El código fuente para todo contenido publicado debería ser revisado manualmente para identificar pistas sobre otras páginas y funcionalidad. Por ejemplo:

Comentarios de programadores y secciones comentadas de código fuente pueden referir a contenido oculto:

```html
<!-- <A HREF="uploadfile.jsp">Upload a document to the server</A> -->
<!-- Link removed while bugs in uploadfile.jsp are fixed          -->
```

JavaScript puede contener enlaces de página que son solo renderizados dentro de la GUI del usuario bajo ciertas circunstancias:

```javascript
var adminUser=false;
if (adminUser) menu.add (new menuItem ("Maintain users", "/admin/useradmin.jsp"));
```

Las páginas HTML pueden contener FORMs que han sido ocultadas deshabilitando el elemento SUBMIT:

```html
<form action="forgotPassword.jsp" method="post">
    <input type="hidden" name="userID" value="123">
    <!-- <input type="submit" value="Forgot Password"> -->
</form>
```

Otra fuente de pistas sobre directorios no referenciados es el archivo `/robots.txt` usado para proporcionar instrucciones a robots web:

```html
User-agent: *
Disallow: /Admin
Disallow: /uploads
Disallow: /backup
Disallow: /~jbloggs
Disallow: /include
```

#### Adivinación Ciega

En su forma más simple, esto involucra ejecutar una lista de nombres de archivo comunes a través de un motor de solicitudes en un intento de adivinar archivos y directorios que existen en el servidor. El siguiente script wrapper de netcat leerá una wordlist desde stdin y realizará un ataque de adivinación básico:

```bash
#!/bin/bash

server=example.org
port=80

while read url
do
echo -ne "$url\t"
echo -e "GET /$url HTTP/1.0\nHost: $server\n" | netcat $server $port | head -1
done | tee outputfile
```

Dependiendo del servidor, GET puede ser reemplazado con HEAD para resultados más rápidos. El archivo de salida especificado puede ser grepped para códigos de respuesta "interesantes". El código de respuesta 200 (OK) usualmente indica que un recurso válido ha sido encontrado (proporcionado que el servidor no entrega una página "not found" personalizada usando el código 200). Pero también busca 301 (Moved), 302 (Found), 401 (Unauthorized), 403 (Forbidden) y 500 (Internal error), que también pueden indicar recursos o directorios dignos de mayor investigación.

El ataque de adivinación básico debería ser ejecutado contra el webroot, y también contra todos los directorios que han sido identificados a través de otras técnicas de enumeración. Ataques de adivinación más avanzados/efectivos pueden ser realizados como sigue:

- Identificar las extensiones de archivo en uso dentro de áreas conocidas de la aplicación (ej. JSP, ASPX, HTML), y usar una wordlist básica appendada con cada una de estas extensiones (o usar una lista más larga de extensiones comunes si los recursos lo permiten).
- Para cada archivo identificado a través de otras técnicas de enumeración, crear una wordlist personalizada derivada de ese nombre de archivo. Obtener una lista de extensiones de archivo comunes (incluyendo ~, bak, txt, src, dev, old, inc, orig, copy, tmp, swp, etc.) y usar cada extensión antes, después, y en lugar de, la extensión del nombre de archivo actual.

Nota: Operaciones de copia de archivos de Windows generan nombres de archivo prefijados con "Copy of " o versiones localizadas de esta frase, por lo tanto no cambian extensiones de archivo. Mientras archivos "Copy of " típicamente no divulgan código fuente cuando accedidos, podrían rendir información valiosa en caso de que causen errores cuando invocados.

#### Información Obtenida a Través de Vulnerabilidades y Misconfiguración del Servidor

La manera más obvia en la que un servidor mal configurado puede divulgar páginas no referenciadas es a través de listado de directorios. Solicitar todos los directorios enumerados para identificar cualquiera que proporcione un listado de directorios.

Numerosas vulnerabilidades han sido encontradas en servidores web individuales que permiten a un atacante enumerar contenido no referenciado, por ejemplo:

- Vulnerabilidad de listado de directorios Apache ?M=D.
- Varias vulnerabilidades de divulgación de fuente de script IIS.
- Vulnerabilidades de listado de directorios IIS WebDAV.

#### Uso de Información Disponible Públicamente

Páginas y funcionalidad en aplicaciones web orientadas a internet que no están referenciadas desde dentro de la aplicación misma pueden estar referenciadas desde otras fuentes de dominio público. Hay varias fuentes de estas referencias:

- Páginas que solían estar referenciadas pueden aún aparecer en los archivos de motores de búsqueda de internet. Por ejemplo, `1998results.asp` puede ya no estar enlazada desde el sitio de una compañía, pero puede permanecer en el servidor y en bases de datos de motores de búsqueda. Este script antiguo puede contener vulnerabilidades que podrían ser usadas para comprometer todo el sitio. El operador de búsqueda `site:` de Google puede ser usado para ejecutar una consulta solo contra el dominio de elección, como en: `site:www.example.com`. Usar motores de búsqueda de esta manera ha llevado a una amplia variedad de técnicas que puedes encontrar útiles, y son descritas en la sección `Google Hacking` de esta Guía. Revísala para afilar tus habilidades de prueba vía Google. Los archivos de respaldo no son probables que sean referenciados por cualquier otro archivo y por lo tanto pueden no haber sido indexados por Google, pero si yacen en directorios navegables el motor de búsqueda podría saber sobre ellos.
- Además, Google y Yahoo mantienen versiones cacheadas de páginas encontradas por sus robots. Aún si `1998results.asp` ha sido removido del servidor objetivo, una versión de su salida puede aún estar almacenada por estos motores de búsqueda. La versión cacheada puede contener referencias a, o pistas sobre, contenido oculto adicional que aún permanece en el servidor.
- Contenido que no está referenciado desde dentro de una aplicación objetivo puede estar enlazado por sitios de terceros. Por ejemplo, una aplicación que procesa pagos en línea en nombre de comerciantes de terceros puede contener una variedad de funcionalidad a medida que puede (normalmente) solo ser encontrada siguiendo enlaces dentro de los sitios de sus clientes.

#### Bypass de Filtro de Nombre de Archivo

Porque los filtros de lista de denegación están basados en expresiones regulares, uno puede a veces aprovechar características de expansión de nombre de archivo OS oscuras que funcionan de maneras que el desarrollador no esperaba. El probador puede a veces explotar diferencias en maneras que los nombres de archivo son parseados por la aplicación, servidor web, y OS subyacente y sus convenciones de nombre de archivo.

Ejemplo: Expansión de nombre de archivo Windows 8.3 `c:\\program files` se convierte en `C:\\PROGRA\~1`

- Remover caracteres incompatibles
- Convertir espacios a underscores
- Tomar los primeros seis caracteres del basename
- Agregar `~<digit>` que es usado para distinguir archivos con nombres usando los mismos seis caracteres iniciales
- Esta convención cambia después de la primera colisión de nombre 3
- Truncar extensión de archivo a tres caracteres
- Hacer todos los caracteres mayúsculas

### Pruebas de Caja Gris

Realizar pruebas de caja gris contra archivos antiguos y de respaldo requiere el examen de archivos dentro de directorios que pertenecen al conjunto de directorios web servidos por el(los) servidor(es) web comprendiendo la infraestructura de aplicación web. Teóricamente el examen debería ser realizado a mano para ser thorough. Sin embargo, ya que en la mayoría de los casos copias de archivos o archivos de respaldo tienden a ser creados usando las mismas convenciones de nombres, la búsqueda puede ser fácilmente scriptada. Por ejemplo, editores dejan atrás copias de respaldo nombrándolos con una extensión reconocible o terminación y humanos tienden a dejar atrás archivos con una `.old` o extensión similar predecible. Una estrategia útil sería programar periódicamente un trabajo en background para verificar archivos con extensiones que son probables que sean identificadas como copias o archivos de respaldo, mientras también realizando verificaciones manuales en una base de tiempo más larga.

## Remediación

Para una estrategia de protección efectiva, las pruebas deberían ser combinadas con una política de seguridad que claramente prohíba prácticas peligrosas, incluyendo:

- Editar archivos in-situ en los sistemas de archivos del servidor web o servidor de aplicación. Este es un hábito particularmente malo, ya que es probable que genere archivos de respaldo o temporales por los editores. Es asombroso ver cuán a menudo esto se hace, incluso en organizaciones grandes. Si absolutamente necesitas editar archivos en un sistema de producción, haz asegurar que no dejes atrás nada que no sea explícitamente intencionado, y ten en mente que lo estás haciendo bajo tu propio riesgo.
- Verificar cuidadosamente cualquier otra actividad realizada en sistemas de archivos expuestos por el servidor web, como actividades de administración spot. Por ejemplo, si ocasionalmente necesitas tomar una instantánea de un par de directorios (que no deberías hacer en un sistema de producción), podrías estar tentado a zipearlos primero. Sé cuidadoso de no dejar atrás tales archivos de archivo.
- Políticas de gestión de configuración apropiadas deberían ayudar a prevenir archivos obsoletos y no-referenciados.
- Las aplicaciones deberían ser diseñadas para no crear (o depender de) archivos almacenados bajo los árboles de directorios web servidos por el servidor web. Archivos de datos, archivos de log, archivos de configuración, etc. deberían ser almacenados en directorios no accesibles por el servidor web para contrarrestar la posibilidad de divulgación de información, sin mencionar el potencial para modificación de datos si los permisos de directorio web permiten escritura.
- Las instantáneas del sistema de archivos no deberían ser accesibles vía web si el document root está en un sistema de archivos usando esta tecnología. Configura tu servidor web para denegar acceso a tales directorios, por ejemplo, bajo Apache, una directiva de location como esta debería ser usada:

```xml
<Location ~ ".snapshot">
    Order deny,allow
    Deny from all
</Location>
```

## Herramientas

Las herramientas de evaluación de vulnerabilidades tienden a incluir verificaciones para detectar directorios web teniendo nombres estándar (como "admin", "test", "backup", etc.), y para reportar cualquier directorio web que permita indexación. Si el probador es incapaz de encontrar listado de directorios, deberían intentar verificar extensiones de respaldo probables. Algunas herramientas que pueden ayudar con esto incluyen:

- [Nessus](https://www.tenable.com/products/nessus)
- [Nikto2](https://cirt.net/Nikto2)

Herramientas de spider web

- [wget](https://www.gnu.org/software/wget/)
- [Spike proxy incluye una función de crawler de sitio](https://www.spikeproxy.com/)
- [Xenu](https://home.snafu.de/tilman/xenulink.html)
- [curl](https://curl.haxx.se)

Algunos de ellos también están incluidos en distribuciones Linux estándar. Las herramientas de desarrollo web usualmente incluyen facilidades para identificar enlaces rotos y archivos no referenciados.