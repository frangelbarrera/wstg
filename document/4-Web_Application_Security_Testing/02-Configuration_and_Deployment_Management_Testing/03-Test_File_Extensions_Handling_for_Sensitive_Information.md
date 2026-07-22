# Probar Manejo de Extensiones de Archivo para Información Sensible

|ID          |
|------------|
|WSTG-CONF-03|

## Resumen

Los servidores web comúnmente usan extensiones de archivo para determinar qué tecnologías, lenguajes y plugins deben usarse para cumplir solicitudes web. Mientras que este comportamiento es consistente con RFCs y Estándares Web, usar extensiones de archivo estándar proporciona al probador de penetración información útil sobre las tecnologías subyacentes usadas en un appliance web y simplifica enormemente la tarea de determinar el escenario de ataque a usar en tecnologías particulares. Además, mis-configuración de servidores web podría fácilmente revelar información confidencial sobre credenciales de acceso.

Las verificaciones de extensión de archivo a menudo se hacen para validar archivos antes de subirlos al servidor. Subidas de archivo no restringidas pueden llevar a resultados imprevistos porque el contenido puede no ser lo esperado, o debido a manejo inesperado de nombre de archivo del OS.

Entender cómo los servidores web manejan solicitudes para archivos con diferentes extensiones puede clarificar el comportamiento del servidor basado en los tipos de archivos accedidos. Por ejemplo, puede ayudar a entender qué extensiones de archivo son devueltas como texto o plano versus aquellas que causan ejecución del lado del servidor. Las últimas son indicativas de tecnologías, lenguajes o plugins usados por servidores web o servidores de aplicación. Esta información puede proporcionar insight adicional en cómo la aplicación web está ingenierizada. Por ejemplo, mientras que una extensión ".pl" es típicamente asociada con soporte Perl del lado del servidor, la extensión de archivo sola puede ser engañosa y no enteramente indicativa de la tecnología subyacente. Tome, por instancia, recursos del lado del servidor escritos en Perl, que podrían ser renombrados para disfrazar el uso de Perl. Ver la siguiente sección en "componentes de servidor web" para más en identificar tecnologías y componentes del lado del servidor.

## Objetivos de Prueba

- Fuerza bruta extensiones de archivo sensibles que podrían contener datos crudos como scripts, credenciales, etc.
- Validar que no existen bypasses de framework del sistema para las reglas que han sido establecidas

## Cómo Probar

### Navegación Forzada

Enviar solicitudes con diferentes extensiones de archivo y verificar cómo son manejadas. La verificación debería ser por directorio web. Verificar directorios que permitan ejecución de script. Directorios de servidor web pueden ser identificados por herramientas de escaneo que buscan la presencia de directorios bien conocidos. Adicionalmente, mirroring la estructura del sitio ayuda a los probadores a reconstruir el árbol de directorio servido por la aplicación.

Si la arquitectura de aplicación web está load-balanced, es importante evaluar todos los servidores web. La facilidad de esta tarea depende de la configuración de la infraestructura de balancing. En una infraestructura con componentes redundantes, puede haber ligeras variaciones en la configuración de servidores web o aplicación individuales. Esto puede suceder si la arquitectura web emplea tecnologías heterogéneas (piense en un set de servidores web IIS y Apache en una configuración load-balancing, que puede introducir comportamiento ligeramente asimétrico entre ellos, y posiblemente diferentes vulnerabilidades).

#### Ejemplo

El probador ha identificado la existencia de un archivo nombrado `connection.inc`. Intentar accederlo directamente da de vuelta su contenido, que es:

```php
<?
    mysql_connect("127.0.0.1", "root", "password")
        or die("Could not connect");
?>
```

El probador determina la existencia de un backend DBMS MySQL y las credenciales débiles usadas por la aplicación web para accederlo.

Las siguientes extensiones de archivo nunca deberían ser devueltas por un servidor web, ya que pertenecen a archivos que podrían contener información sensible o archivos que no tienen razón válida para ser servidos.

- `.asa`
- `.inc`
- `.config`

Las siguientes extensiones de archivo están relacionadas con archivos que, cuando accedidos, son ya sea mostrados o descargados por el navegador. Por lo tanto, archivos con estas extensiones deben ser verificados para asegurar que son de hecho supuestos a ser servidos (y no leftovers), y que no contienen información sensible.

- `.zip`, `.tar`, `.gz`, `.tgz`, `.rar`, etc.: archivos de archivo (comprimidos)
- `.java`: No hay razón para proporcionar acceso a archivos fuente Java
- `.txt`: Archivos de texto
- `.pdf`: Documentos PDF
- `.docx`, `.rtf`, `.xlsx`, `.pptx`, etc.: Documentos de Office
- `.bak`, `.old` y otras extensiones indicativas de archivos de respaldo (por ejemplo: `~` para archivos de respaldo Emacs)

La lista dada arriba detalla solo unos pocos ejemplos, ya que extensiones de archivo son demasiadas para ser tratadas comprehensivamente aquí. Referir a [FILExt](https://filext.com/) para una base de datos más thorough de extensiones.

Para identificar archivos con una extensión dada, una mezcla de técnicas puede ser empleada. Estas técnicas pueden incluir usar escáneres de vulnerabilidad, herramientas de spidering y mirroring, y querying motores de búsqueda (ver [Testing: Spidering and googling](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md)). Inspección manual de la aplicación también puede ser beneficiosa, ya que supera limitaciones en spidering automático. Ver también [Testing for Old, Backup and Unreferenced Files](04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md) que trata con los temas de seguridad relacionados con archivos "olvidados".

### Subida de Archivo

Manejo de archivo legacy Windows 8.3 puede a veces ser usado para derrotar filtros de subida de archivo.

Ejemplos de uso:

1. `file.phtml` gets processed as PHP code.
2. `FILE~1.PHT` is served, but not processed by the PHP ISAPI handler.
3. `shell.phPWND` can be uploaded.
4. `SHELL~1.PHP` will be expanded and returned by the OS shell, then processed by the PHP ISAPI handler.

### Pruebas de Caja Gris

Pruebas white-box de manejo de extensión de archivo involucran verificar las configuraciones del servidor en la arquitectura de aplicación web y verificando las reglas para servir diferentes extensiones de archivo.

Si la aplicación web confía en una infraestructura load-balanced, heterogénea, determinar si esto puede introducir diferente comportamiento.

## Herramientas

Escáneres de vulnerabilidad, como Nessus y Nikto, verifican la existencia de directorios web bien conocidos. Pueden permitir al probador descargar la estructura del sitio, que es útil cuando tratando de determinar la configuración de directorios web y cómo extensiones de archivo individuales son servidas. Otras herramientas que pueden ser usadas para este propósito incluyen:

- [wget](https://www.gnu.org/software/wget)
- [curl](https://curl.haxx.se)
- Realizar una búsqueda Google para "web mirroring tools"