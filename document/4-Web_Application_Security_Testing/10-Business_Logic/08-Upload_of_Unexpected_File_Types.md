# Carga de tipos de archivo inesperados

|ID          |
|------------|
|WSTG-BUSL-08|

## Resumen

Los procesos de negocio de muchas aplicaciones permiten cargar y manipular datos enviados mediante archivos. Sin embargo, el proceso de negocio debe verificar los archivos y permitir únicamente ciertos tipos de archivo «aprobados». La lógica de negocio determina qué archivos están «aprobados», y esto depende de cada aplicación o sistema. El riesgo es que, al permitir que los usuarios carguen archivos, los atacantes puedan enviar un tipo de archivo inesperado que se ejecute y afecte negativamente a la aplicación o al sistema mediante ataques que podrían desfigurar el sitio, ejecutar comandos remotos, explorar los archivos del sistema o los recursos locales, atacar otros servidores o explotar vulnerabilidades locales, entre otras posibilidades.

Las vulnerabilidades relacionadas con la carga de tipos de archivo inesperados son particulares porque el sistema debería rechazar rápidamente un archivo si no tiene una extensión específica. Además, esto difiere de la carga de archivos maliciosos en que, en la mayoría de los casos, un formato de archivo incorrecto no es «malicioso» por sí mismo, pero puede perjudicar los datos guardados. Por ejemplo, si una aplicación acepta archivos de Microsoft Excel y se carga un archivo de base de datos similar, es posible que se lea, pero que los datos extraídos se trasladen a ubicaciones incorrectas.

Es posible que la aplicación espere que solo se carguen ciertos tipos de archivo para su procesamiento, como archivos `.csv` o `.txt`. La aplicación podría no validar el archivo cargado según su extensión (validación de archivos de baja garantía) o su contenido (validación de archivos de alta garantía). Esto podría producir resultados inesperados en el sistema o la base de datos de la aplicación o el sistema, o proporcionar a los atacantes métodos adicionales para explotar la aplicación o el sistema.

### Ejemplo

Supongamos que una aplicación para compartir imágenes permite a los usuarios cargar en el sitio archivos gráficos `.gif` o `.jpg`. ¿Qué ocurriría si un atacante pudiera cargar un archivo HTML con una etiqueta `<script>` o un archivo PHP? El sistema podría trasladar el archivo desde una ubicación temporal hasta la ubicación final, donde el código PHP podría ejecutarse contra la aplicación o el sistema.

## Objetivos de prueba

- Revisar la documentación del proyecto para identificar los tipos de archivo que el sistema rechaza.
- Verificar que los tipos de archivo no permitidos se rechacen y se manejen de forma segura.
- Verificar que las cargas por lotes de archivos sean seguras y no permitan eludir las medidas de seguridad establecidas.

## Cómo probar

### Método de prueba específico

- Estudiar los requisitos lógicos de la aplicación.
- Preparar una colección de archivos «no aprobados» para la carga, que podría incluir archivos como jsp, exe o archivos HTML que contengan scripts.
- Navegar en la aplicación hasta el mecanismo de envío o carga de archivos.
- Enviar el archivo «no aprobado» para su carga y verificar que se impida correctamente su carga.
- Comprobar si el sitio solo verifica el tipo de archivo mediante JavaScript del lado del cliente.
- Comprobar si el sitio solo verifica el tipo de archivo mediante el encabezado «Content-Type» de la solicitud HTTP.
- Comprobar si el sitio solo verifica el tipo de archivo mediante la extensión del archivo.
- Comprobar si se puede acceder directamente a otros archivos cargados mediante una URL específica.
- Comprobar si el archivo cargado puede incluir código o inyección de scripts.
- Comprobar si se verifica la ruta de los archivos cargados. En particular, los atacantes podrían comprimir archivos en un ZIP con una ruta especificada para que, después de cargar y descomprimir el archivo, los archivos extraídos se carguen en la ruta prevista.

## Casos de prueba relacionados

- [Gestión de extensiones de archivo para información confidencial](../02-Configuration_and_Deployment_Management/03-File_Extensions_Handling_for_Sensitive_Information.md)
- [Probar la carga de archivos maliciosos](09-Upload_of_Malicious_Files.md)

## Remediación

Las aplicaciones deben desarrollarse con mecanismos que solo acepten y manipulen archivos «aceptables», es decir, aquellos que el resto de las funciones de la aplicación esté preparado para manejar y espere recibir. Algunos ejemplos específicos son las listas de bloqueo o de permitidos de extensiones de archivo, el uso del valor «Content-Type» del encabezado o el uso de un identificador de tipos de archivo. Todos estos mecanismos deben permitir que solo se incorporen al sistema los tipos de archivo especificados.

## Referencias

- [OWASP: carga de archivos sin restricciones](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Buenas prácticas de seguridad para la carga de archivos: bloquear la carga de archivos maliciosos](https://www.computerweekly.com/answer/File-upload-security-best-practices-Block-a-malicious-file-upload)
- [Evitar que se carguen archivos PHP maliciosos mediante formularios](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [CWE-434: carga sin restricciones de archivos de tipo peligroso](https://cwe.mitre.org/data/definitions/434.html)