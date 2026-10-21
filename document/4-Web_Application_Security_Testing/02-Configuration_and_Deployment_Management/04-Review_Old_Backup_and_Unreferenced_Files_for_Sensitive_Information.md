# Revisar archivos antiguos de respaldo y no referenciados en busca de información sensible

|ID          |
|------------|
|WSTG-CONF-04|

## Resumen

Los servidores web suelen acumular archivos no referenciados u olvidados: versiones antiguas con otro nombre, respaldos de editores o manuales (`file~`, `file.old`, `.bak`), archivos comprimidos e «instantáneas» del sistema de archivos. Como a menudo conservan una extensión o un nombre diferente del original (por ejemplo, `login.asp.old` junto a `login.asp`), suelen servirse como texto sin formato o código fuente, en lugar de ejecutarse, lo que expone código del lado del servidor, credenciales y detalles internos. El mismo problema también se presenta cuando quedan olvidados objetos en el almacenamiento de objetos en la nube (buckets o contenedores) que respaldan una aplicación o su canalización de despliegue.

### Amenazas

- Divulgación de credenciales, rutas internas u otros datos sensibles mediante archivos de inclusión o de configuración no referenciados.
- Acceso a funcionalidades potentes no referenciadas (por ejemplo, una página de administración sin enlaces).
- Exposición de vulnerabilidades ya corregidas en la versión actual a través de una copia antigua o de respaldo que todavía las contiene.
- Divulgación del código fuente cuando una copia de respaldo de un archivo ejecutable se sirve como texto sin formato, en lugar de ejecutarse.
- Divulgación masiva mediante archivos comprimidos de respaldo que contienen muchos archivos a la vez (código fuente, archivos de inclusión, clases compiladas que se pueden descompilar).
- Datos sensibles en archivos de registro (ID de sesión, parámetros de URL, actividad de administración).
- Artefactos olvidados de respaldo o despliegue (volcados de bases de datos, archivos `.tfstate` y `.env`) que quedan en el almacenamiento de objetos en la nube (buckets o contenedores) y son accesibles directamente, sin pasar por la aplicación; consulte [Artefactos en el almacenamiento de objetos en la nube](#artefactos-en-el-almacenamiento-de-objetos-en-la-nube).

## Objetivos de prueba

- Encontrar y analizar archivos antiguos, de respaldo y no referenciados —incluidos los que quedan expuestos solo por su extensión y los objetos almacenados en la nube— que podrían contener información sensible.
- Validar que no existan formas de eludir las reglas establecidas para impedir que se sirvan estos archivos, ya sea a nivel del sistema o del framework.

## Cómo probar

### Pruebas de caja negra

Las pruebas para detectar archivos antiguos, de respaldo y no referenciados utilizan técnicas automatizadas y manuales, y suelen combinar las siguientes. Como principio general, considere el descubrimiento automatizado una forma económica de generar a escala una lista de posibles candidatos, y verifique siempre manualmente una muestra de los resultados. Las herramientas automatizadas informan sobre códigos de estado y huellas, no sobre si un archivo contiene realmente datos sensibles; tanto los falsos positivos (páginas personalizadas de «no encontrado» que devuelven `200`) como los falsos negativos (contenido sensible tras una respuesta inesperada) son habituales.

#### Extensiones de archivos sensibles

Envíe solicitudes con distintas extensiones y verifique cómo se gestionan, por directorio, ya que cada directorio puede estar configurado de manera diferente (por ejemplo, un directorio puede ejecutar archivos `.php` mientras otro los sirve como texto sin formato). Las herramientas de análisis que buscan directorios conocidos pueden identificar los directorios del servidor web, y replicar la estructura del sitio ayuda a reconstruir el árbol de directorios que sirve la aplicación.

Si la arquitectura de la aplicación web utiliza balanceo de carga o es heterogénea por otros motivos (por ejemplo, una combinación de nodos IIS y Apache), evalúe todos los servidores, ya que la configuración puede variar ligeramente entre nodos e introducir comportamientos asimétricos o vulnerabilidades.

##### Ejemplo

El probador ha identificado la existencia de un archivo llamado `connection.inc`. Al intentar acceder directamente a él, obtiene el siguiente contenido:

```php
<?
    mysql_connect("127.0.0.1", "root", "password")
        or die("Could not connect");
?>
```

El probador determina que hay un DBMS MySQL en el backend y que la aplicación web utiliza credenciales débiles para acceder a él.

Las extensiones y rutas que vale la pena comprobar se agrupan en distintas categorías, lo cual es importante porque conllevan diferentes niveles de riesgo y requieren un tratamiento distinto al encontrarlas. Un resultado de cualquier categoría es solo un posible hallazgo: confirme que la respuesta devuelve realmente el contenido del archivo (y no una página 404 personalizada o de acceso denegado) y lea suficiente contenido para verificar que sea realmente sensible antes de informar del hallazgo. Esto es especialmente importante para las categorías «Puede servirse legítimamente» y «La sensibilidad depende del contenido» que aparecen a continuación, en las que la mera presencia de la extensión no constituye por sí misma un hallazgo.

| Tipo | Categoría | Ejemplos |
|------|----------|----------|
| Credenciales y secretos | Por lo general, nunca deberían servirse públicamente | `.env`, `.pem`, `.key`, `.p12`, `.pfx`, `.tfstate`, `.tfvars`, `.kdbx`, `.htpasswd`, `.npmrc`, `.dockercfg`/`.docker/config.json`, `.asa`, `.inc`, `.config` |
| Código fuente y configuración | Por lo general, nunca deberían servirse públicamente | Archivos de código fuente `.java`, `.yaml`/`.yml` y `.json` fuera de las rutas de API previstas (manifiestos de Kubernetes, definiciones de canalizaciones de CI, `docker-compose.yml`) |
| Respaldos y archivos comprimidos | Por lo general, nunca deberían servirse públicamente | `.bak`, `.old`, `.orig`, `.save`, `.swp`, `~` (respaldos de editores), `.zip`, `.tar`, `.gz`, `.tgz`, `.rar`, `.7z` |
| Artefactos de datos | La sensibilidad depende en gran medida del contenido | `.db`, `.sqlite`, `.sqlite3`, `.parquet`, `.avro`, `.csv`, `.ipynb` (en particular, los notebooks pueden contener credenciales codificadas de forma fija en la salida de las celdas, aunque a menudo no las contienen) |
| Puede servirse legítimamente | Puede servirse legítimamente | `.txt`, `.md`, `.log`, `.pdf`, `.docx`, `.rtf`, `.xlsx`, `.pptx` |
| Rutas especiales (no son extensiones de archivo) | Nombres de directorios y rutas, no extensiones | `.git/`, `.docker/` |

La lista anterior contiene solo algunos ejemplos, ya que hay demasiadas extensiones y nombres de archivo que convendría comprobar para tratarlos exhaustivamente aquí. Consulte [FILExt](https://filext.com/) para obtener una base de datos de extensiones más completa y consulte las [listas de palabras de raft/common backup/config de SecLists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content) para obtener listas de nombres de archivo mantenidas y actualizadas que permitan realizar el descubrimiento automatizado.

Para identificar archivos con una extensión determinada, combine herramientas de análisis de vulnerabilidades, herramientas de rastreo y replicación de sitios, consultas en motores de búsqueda (consulte [Pruebas: rastreo y búsquedas en Google](../01-Information_Gathering/01-Conduct_Search_Engine_Reconnaissance_for_Information_Leakage.md)) e inspección manual, ya que esta última permite superar las limitaciones del rastreo automatizado (por ejemplo, enlaces que solo aparecen después de que se ejecuta JavaScript del lado del cliente).

##### Gestión de nombres de archivo heredados 8.3 de Windows

En los sistemas basados en Windows, la generación de nombres de archivo cortos heredados 8.3 puede afectar a la forma en que el servidor web resuelve y permite acceder a los archivos, independientemente de cualquier lógica de carga de archivos. En los entornos donde está habilitada la generación de nombres de archivo 8.3, es posible que aún se pueda acceder a archivos sensibles que no son accesibles directamente mediante su nombre largo a través de su equivalente abreviado. Esto puede divulgar código fuente o archivos de configuración, incluso cuando los controles de acceso bloquean correctamente el nombre largo.

Ejemplos de comportamientos de resolución de nombres de archivo 8.3 que pueden provocar una exposición no deseada de archivos:

1. Un archivo como `file.phtml` puede procesarse como código PHP.
2. Es posible acceder a un nombre de archivo abreviado correspondiente (por ejemplo, `FILE~1.PHT`), según la configuración del servidor y del controlador.
3. Los archivos con nombres engañosos o extendidos aún pueden resolverse hacia controladores ejecutables una vez que el sistema operativo los expande.

Las pruebas deben centrarse en determinar si la gestión heredada de nombres de archivo permite acceder a archivos sensibles que no deberían servirse de esta manera; las pruebas de los propios mecanismos de carga de archivos se tratan por separado en los casos de prueba de carga de archivos y lógica de negocio.

#### Inferencia a partir del esquema de nombres utilizado para el contenido publicado

Enumere todas las páginas y funciones de la aplicación, manualmente o con una herramienta de rastreo. La mayoría de las aplicaciones utiliza un esquema de nombres reconocible y organiza los recursos en páginas y directorios con palabras que describen su función. Por ello, a menudo es posible inferir el nombre y la ubicación de páginas no referenciadas a partir del esquema de nombres del contenido publicado. Por ejemplo, si se encuentra una página titulada `viewuser.asp`, también se deberían buscar `edituser.asp`, `adduser.asp` y `deleteuser.asp`. De manera similar, si se descubre el directorio `/app/user`, también se deberían buscar `/app/admin` y `/app/manager`.

#### Otras pistas en el contenido publicado

Muchas aplicaciones web dejan pistas en el contenido publicado que pueden llevar al descubrimiento de páginas y funcionalidades ocultas. A menudo, estas pistas se encuentran en el código fuente de archivos HTML y JavaScript, que se debería revisar manualmente para identificarlas. Por ejemplo:

Los comentarios de los programadores y las secciones comentadas del código fuente pueden hacer referencia a contenido oculto:

```html
<!-- <A HREF="uploadfile.jsp">Upload a document to the server</A> -->
<!-- Link removed while bugs in uploadfile.jsp are fixed          -->
```

JavaScript puede contener enlaces a páginas que solo se muestran en la interfaz gráfica del usuario en determinadas circunstancias:

```javascript
var adminUser=false;
if (adminUser) menu.add (new menuItem ("Maintain users", "/admin/useradmin.jsp"));
```

Las páginas HTML pueden contener formularios ocultos al deshabilitar el elemento SUBMIT:

```html
<form action="forgotPassword.jsp" method="post">
    <input type="hidden" name="userID" value="123">
    <!-- <input type="submit" value="Forgot Password"> -->
</form>
```

Otra fuente de pistas sobre directorios no referenciados es el archivo `/robots.txt`, que se utiliza para proporcionar instrucciones a los robots web:

```html
User-agent: *
Disallow: /Admin
Disallow: /uploads
Disallow: /backup
Disallow: /~jbloggs
Disallow: /include
```

#### Adivinación a ciegas

En su forma más simple, consiste en enviar una lista de nombres de archivo comunes mediante un motor de solicitudes para intentar adivinar los archivos y directorios que existen en el servidor. El siguiente script de envoltura de netcat leerá una lista de palabras desde stdin y realizará un ataque básico de adivinación:

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

Según el servidor, se puede sustituir GET por HEAD para obtener resultados más rápidos. Se puede buscar en el archivo de salida especificado los códigos de respuesta «interesantes». El código de respuesta 200 (OK) suele indicar que se ha encontrado un recurso válido (siempre que el servidor no entregue una página personalizada de «no encontrado» con el código 200). Sin embargo, también se deben tener en cuenta los códigos 301 (Moved), 302 (Found), 401 (Unauthorized), 403 (Forbidden) y 500 (Internal error), que también pueden indicar recursos o directorios que conviene investigar más a fondo.

El ataque básico de adivinación se debería ejecutar contra la raíz web y también contra todos los directorios identificados mediante otras técnicas de enumeración. Se pueden realizar ataques de adivinación más avanzados o eficaces de la siguiente manera:

- Identifique las extensiones de archivo utilizadas en las áreas conocidas de la aplicación (por ejemplo, JSP, ASPX, HTML) y añada cada una de estas extensiones a una lista de palabras básica (o utilice una lista más extensa de extensiones comunes si los recursos lo permiten).
- Para cada archivo identificado mediante otras técnicas de enumeración, cree una lista de palabras personalizada basada en ese nombre de archivo. Obtenga una lista de extensiones de archivo comunes (incluidas `~`, `bak`, `txt`, `src`, `dev`, `old`, `inc`, `orig`, `copy`, `tmp`, `swp`, `env`, `save`, etc.) y utilice cada extensión antes, después y en lugar de la extensión del nombre de archivo real.

> Nota: Las operaciones de copia de archivos de Windows generan nombres de archivo con el prefijo «Copy of » o con versiones localizadas de esta frase, por lo que no cambian las extensiones de archivo. Aunque los archivos «Copy of » normalmente no divulgan código fuente al acceder a ellos, podrían proporcionar información valiosa si provocan errores al invocarlos.

#### Información obtenida mediante vulnerabilidades y configuraciones incorrectas del servidor

La forma más obvia en que un servidor mal configurado puede divulgar páginas no referenciadas es mediante el listado de directorios. Solicite todos los directorios enumerados para identificar cuáles muestran un listado de directorios.

Se han encontrado numerosas vulnerabilidades en servidores web concretos que permiten a un atacante enumerar contenido no referenciado, por ejemplo:

- Vulnerabilidad de listado de directorios de Apache `?M=D`.
- Diversas vulnerabilidades de divulgación del código fuente de scripts de IIS.
- Vulnerabilidades de listado de directorios de IIS WebDAV.

#### Uso de información disponible públicamente

Es posible que páginas y funcionalidades de aplicaciones web expuestas a Internet que no están referenciadas dentro de la propia aplicación aparezcan en otras fuentes de dominio público. Estas referencias pueden encontrarse en diversas fuentes:

- Las páginas que solían estar referenciadas pueden seguir apareciendo en los archivos de los motores de búsqueda de Internet. Por ejemplo, es posible que `1998results.asp` ya no esté enlazada desde el sitio de una empresa, pero que siga en el servidor y en las bases de datos de los motores de búsqueda. El operador de búsqueda `site:` de Google se puede utilizar para realizar una consulta únicamente en el dominio elegido, como en: `site:www.example.com`. El uso de motores de búsqueda de esta manera ha dado lugar a una amplia variedad de técnicas descritas en la sección `Google Hacking` de esta guía. Es poco probable que otros archivos hagan referencia a los archivos de respaldo, por lo que tal vez no se hayan indexado; sin embargo, si están en directorios navegables, el motor de búsqueda podría conocerlos.
- Google y otros motores de búsqueda conservan versiones en caché de las páginas que encuentran sus rastreadores, que aún pueden hacer referencia a contenido oculto adicional o dar pistas sobre él, incluso después de que se haya eliminado la página original.
- Los sitios de terceros pueden enlazar a contenido que no está referenciado dentro de una aplicación objetivo. Por ejemplo, una aplicación que procesa pagos en línea en nombre de comerciantes externos puede contener diversas funciones personalizadas que normalmente solo se encuentran siguiendo enlaces en los sitios de sus clientes.
- Las búsquedas en repositorios públicos de código y en registros de compilación de CI/CD (búsqueda de código en GitHub/GitLab y registros de CI en caché) pueden revelar referencias a nombres de archivo, rutas o nombres de buckets internos que se hayan confirmado por error.

#### Elusión de filtros de nombres de archivo

Como los filtros de listas de denegación se basan en expresiones regulares, a veces se pueden aprovechar características poco conocidas de expansión de nombres de archivo del sistema operativo que funcionan de formas que el desarrollador no previó. En ocasiones, el probador puede aprovechar las diferencias entre la forma en que la aplicación, el servidor web y el sistema operativo subyacente analizan los nombres de archivo y aplican sus convenciones.

Ejemplo: la expansión de nombres de archivo 8.3 de Windows convierte `c:\program files` en `C:\PROGRA~1`

- Elimine los caracteres incompatibles.
- Convierta los espacios en guiones bajos.
- Tome los seis primeros caracteres del nombre base.
- Añada `~<digit>`, que se utiliza para distinguir archivos cuyos nombres comparten los mismos seis caracteres iniciales.
- Esta convención cambia después de las tres primeras colisiones de nombres.
- Trunque la extensión del archivo a tres caracteres.
- Convierta todos los caracteres a mayúsculas.

#### Artefactos en el almacenamiento de objetos en la nube

El mismo problema de los «archivos olvidados» se repite en el almacenamiento de objetos en la nube que respalda una aplicación web, sus recursos estáticos o su canalización de despliegue (AWS S3, Azure Blob Storage, Google Cloud Storage). Esta subsección abarca únicamente el descubrimiento de artefactos olvidados; para profundizar en las ACL de buckets o contenedores, las políticas de IAM y las URL firmadas, consulte [Almacenamiento en la nube](11-Cloud_Storage.md).

- Identifique los buckets o contenedores asociados al objetivo mediante convenciones de nombres (`<company>-backup`, `<company>-assets`, `<company>-logs`, `<company>-dev`, `<app>-uploads`), registros DNS (CNAME que apuntan a endpoints de almacenamiento; consulte también [Secuestro de subdominios](10-Subdomain_Takeover.md)) y referencias encontradas en paquetes de JS, mensajes de error o configuraciones de CI.
- Los tipos habituales de artefactos olvidados en buckets reflejan la lista de extensiones anterior, con algunas adiciones específicas del almacenamiento: archivos `terraform.tfstate`, archivos `.env` cargados como parte de un paso de despliegue, exportaciones de instantáneas de bases de datos y artefactos de compilación de CI/CD (binarios compilados o archivos comprimidos que contienen secretos incrustados).
- Una vez que se encuentre un objeto o listado candidato, verifique manualmente su contenido antes de informar del hallazgo. Un objeto llamado `backup.sql` que resulta estar vacío o ser un marcador de posición no constituye un hallazgo; confirme que los datos presentes sean realmente sensibles.
- Herramientas como [S3Scanner](https://github.com/sa7mon/S3Scanner) y [cloud_enum](https://github.com/initstring/cloud_enum) automatizan el descubrimiento en los principales proveedores. Sin embargo, al igual que en el descubrimiento en el servidor, considere los resultados como candidatos que requieren confirmación manual, no como hallazgos por sí mismos.

### Pruebas de caja gris

Las pruebas de caja gris de archivos antiguos y de respaldo, así como de la gestión de extensiones de archivo, consisten en examinar directamente la configuración del servidor o de la aplicación, en lugar de limitarse a realizar sondeos desde el exterior:

- Examine los archivos de los directorios que sirven los servidores web de la infraestructura de la aplicación. Como los archivos copiados o de respaldo suelen seguir convenciones de nombres predecibles (sufijos de respaldo propios de los editores, copias creadas manualmente con nombres del tipo `.old` o `.bak`), esta búsqueda se puede automatizar con un script. Una estrategia útil es programar periódicamente un trabajo que busque estas extensiones, junto con comprobaciones manuales menos frecuentes.
- Revise la configuración del servidor para confirmar qué tipo de contenido o controlador se utiliza para servir cada extensión, por directorio, y verifique que corresponda a lo previsto (por ejemplo, que un directorio que contiene archivos `.inc` o `.config` no sea accesible desde la web).
- Si la aplicación depende de una infraestructura con balanceo de carga o heterogénea, determine si esto genera comportamientos incoherentes entre los nodos.
- Si el probador tiene acceso a la configuración de la infraestructura en la nube, revise directamente las políticas y ACL de buckets o contenedores (consulte [Almacenamiento en la nube](11-Cloud_Storage.md) para las comprobaciones específicas de ACL, IAM y URL firmadas), en lugar de depender únicamente de las pruebas externas.

## Mitigación

Para contar con una estrategia de protección eficaz, las pruebas se deben combinar con una política de seguridad que prohíba claramente las prácticas peligrosas, entre ellas:

- Editar archivos directamente en los sistemas de archivos del servidor web o del servidor de aplicaciones. Se trata de una práctica especialmente perjudicial, ya que es probable que los editores generen archivos de respaldo o temporales. Si es absolutamente necesario editar archivos en un sistema de producción, asegúrese de no dejar nada que no esté previsto explícitamente y tenga en cuenta que lo hace bajo su propia responsabilidad.
- Revisar cuidadosamente cualquier otra actividad realizada en los sistemas de archivos expuestos por el servidor web, como las tareas de administración improvisadas (por ejemplo, comprimir directorios para crear rápidamente un respaldo y dejar el archivo comprimido en el servidor).
- Adoptar políticas de gestión de la configuración que impidan la acumulación de archivos obsoletos y no referenciados en los directorios servidos.
- Diseñar las aplicaciones para que no creen ni dependan de archivos almacenados en los árboles de directorios web servidos por el servidor web. Los archivos de datos, de registro, de configuración, etc., se deben almacenar en directorios a los que no pueda acceder el servidor web, con el fin de evitar tanto la divulgación de información como la modificación de datos cuando los permisos lo permitan.
- Asegurarse de que las instantáneas del sistema de archivos no sean accesibles desde la web si la raíz de documentos está en un sistema de archivos que utiliza esta tecnología. Por ejemplo, en Apache se debería utilizar una directiva de ubicación como la siguiente:

```xml
<Location ~ ".snapshot">
    Order deny,allow
    Deny from all
</Location>
```

- Configurar los servidores web y los directorios para que cada extensión de archivo se sirva con el controlador previsto (o no se sirva), por directorio, y validar esta configuración después de realizar cambios en la infraestructura en entornos con balanceo de carga o heterogéneos.
- En el almacenamiento de objetos en la nube, deshabilitar de forma predeterminada los listados públicos y la lectura pública, y utilizar políticas de IAM con permisos acotados en lugar de conceder acceso público a todo el bucket. Tratar como sensibles de forma predeterminada todos los buckets utilizados para respaldos, registros o artefactos de despliegue.

## Herramientas

Las herramientas de evaluación de vulnerabilidades suelen incluir comprobaciones para detectar directorios web con nombres habituales (como «admin», «test», «backup», etc.) y notificar los directorios web que permiten la indexación. Si el probador no logra encontrar listados de directorios, debería intentar comprobar extensiones de respaldo probables. Algunas herramientas útiles para ello son:

- [Nessus](https://www.tenable.com/products/nessus)
- [Nikto2](https://cirt.net/Nikto2)

### Herramientas de rastreo web

- [wget](https://www.gnu.org/software/wget)
- [curl](https://curl.se/)

### Alternativas modernas

- [Burp Suite](https://portswigger.net/burp) - proxy de pruebas de seguridad web de uso extendido
- [ZAP](https://www.zaproxy.org/) - herramienta de código abierto para probar la seguridad de aplicaciones web, que incluye exploración forzada y rastreo para este tipo de problema
- [ffuf](https://github.com/ffuf/ffuf) - fuzzing web rápido, adecuado para el descubrimiento basado en listas de palabras de respaldos y extensiones
- [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/) - rastreador de sitios web y comprobador de enlaces rotos

### Descubrimiento de almacenamiento de objetos en la nube

- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)

Algunas de estas herramientas también se incluyen en las distribuciones estándar de Linux. Las herramientas de desarrollo web suelen incluir funciones para identificar enlaces rotos y archivos no referenciados.

## Referencias

- [FILExt - Base de datos de extensiones de archivo](https://filext.com/)
- [SecLists - Listas de palabras de Discovery/Web-Content](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)