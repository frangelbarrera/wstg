# Probar la Subida de Tipos de Archivo Inesperados

|ID          |
|------------|
|WSTG-BUSL-08|

## Resumen

Los procesos de negocio de muchas aplicaciones permiten la subida y manipulación de datos que se envían vía archivos. Pero el proceso de negocio debe verificar los archivos y solo permitir ciertos tipos de archivo "aprobados". Decidir qué archivos están "aprobados" lo determina la lógica de negocio y es específico de la aplicación/sistema. El riesgo es que al permitir a los usuarios subir archivos, los atacantes podrían enviar un tipo de archivo inesperado que podría ser ejecutado y impactar adversamente la aplicación o sistema a través de ataques que podrían desfigurar el sitio, realizar comandos remotos, navegar los archivos del sistema, navegar los recursos locales, atacar otros servidores, o explotar las vulnerabilidades locales, solo por nombrar algunos.

Las vulnerabilidades relacionadas con la subida de tipos de archivo inesperados son únicas en que la subida debería rechazar rápidamente un archivo si no tiene una extensión específica. Adicionalmente, esto es diferente de subir archivos maliciosos en que en la mayoría de los casos un formato de archivo incorrecto podría no por sí mismo ser inherentemente "malicioso" pero podría ser perjudicial para los datos guardados. Por ejemplo, si una aplicación acepta archivos de Excel de Windows, si se sube un archivo de base de datos similar podría ser leído pero los datos extraídos podrían ser movidos a ubicaciones incorrectas.

La aplicación podría estar esperando que solo ciertos tipos de archivo sean subidos para procesamiento, tales como archivos `.csv` o `.txt`. La aplicación podría no validar el archivo subido por extensión (para validación de archivo de baja seguridad) o contenido (para validación de archivo de alta seguridad). Esto podría resultar en resultados inesperados del sistema o base de datos dentro de la aplicación/sistema o dar a los atacantes métodos adicionales para explotar la aplicación/sistema.

### Ejemplo

Suponer que una aplicación de compartir imágenes permite a los usuarios subir un archivo gráfico `.gif` o `.jpg` al sitio. ¿Qué pasa si un atacante es capaz de subir un archivo HTML con una etiqueta `<script>` en él o un archivo PHP? El sistema podría mover el archivo de una ubicación temporal a la ubicación final donde el código PHP puede ahora ser ejecutado contra la aplicación o sistema.

## Objetivos de Prueba

- Revisar la documentación del proyecto para tipos de archivo que son rechazados por el sistema.
- Verificar que los tipos de archivo no bienvenidos sean rechazados y manejados de manera segura.
- Verificar que las subidas por lote de archivos sean seguras y no permitan ninguna evasión contra las medidas de seguridad establecidas.

## Cómo Probar

### Método de Prueba Específico

- Estudiar los requisitos lógicos de la aplicación.
- Preparar una biblioteca de archivos que están "no aprobados" para subida que podrían contener archivos tales como: jsp, exe, o archivos HTML conteniendo script.
- En la aplicación navegar al mecanismo de envío o subida de archivos.
- Enviar el archivo "no aprobado" para subida y verificar que se les prevenga apropiadamente de subir
- Verificar si el sitio solo hace verificaciones de tipo de archivo en JavaScript del lado del cliente
- Verificar si el sitio solo verifica el tipo de archivo por "Content-Type" en la solicitud HTTP.
- Verificar si el sitio solo verifica el tipo de archivo por la extensión del archivo.
- Verificar si otros archivos subidos pueden ser accedidos directamente por URL especificada.
- Verificar si el archivo subido puede incluir inyección de código o script.
- Verificar si hay alguna verificación de ruta de archivo para archivos subidos. Especialmente, los hackers podrían comprimir archivos con ruta especificada en ZIP para que los archivos extraídos puedan ser subidos a la ruta prevista después de subir y descomprimir.

## Casos de Prueba Relacionados

- [Pruebas de Manejo de Extensiones de Archivo para Información Sensible](../02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)
- [Probar la Subida de Archivos Maliciosos](09-Test_Upload_of_Malicious_Files.md)

## Remediación

Las aplicaciones deberían ser desarrolladas con mecanismos para solo aceptar y manipular archivos "aceptables" que el resto de la funcionalidad de la aplicación esté lista para manejar y esperando. Algunos ejemplos específicos incluyen: listas de bloqueo o listas blancas de extensiones de archivo, usar "Content-Type" del encabezado, o usar un reconocedor de tipo de archivo, todo para solo permitir tipos de archivo especificados en el sistema.

## Referencias

- [OWASP - Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [File upload security best practices: Block a malicious file upload](https://www.computerweekly.com/answer/File-upload-security-best-practices-Block-a-malicious-file-upload)
- [Stop people uploading malicious PHP files via forms](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
