# Enumerar Interfaces de Administración de Infraestructura y Aplicación

|ID          |
|------------|
|WSTG-CONF-05|

## Resumen

Las interfaces de administrador pueden estar presentes en la aplicación o en el servidor de aplicaciones para permitir que ciertos usuarios realicen actividades privilegiadas en el sitio. Se deben realizar pruebas para revelar si y cómo esta funcionalidad privilegiada puede ser accedida por un usuario no autorizado o estándar.

Una aplicación puede requerir una interfaz de administrador para permitir que un usuario privilegiado acceda a funcionalidades que pueden hacer cambios en cómo funciona el sitio. Tales cambios pueden incluir:

- aprovisionamiento de cuentas de usuario
- diseño y disposición del sitio
- manipulación de datos
- cambios de configuración

En muchas instancias, tales interfaces no tienen controles suficientes para protegerlas del acceso no autorizado. Las pruebas están destinadas a descubrir estas interfaces de administrador y acceder a funcionalidades destinadas a los usuarios privilegiados.

## Objetivos de la Prueba

- Identificar interfaces y funcionalidades de administrador ocultas.

## Cómo Probar

### Pruebas de Caja Negra

La siguiente sección describe vectores que pueden usarse para probar la presencia de interfaces administrativas. Estas técnicas también pueden usarse para probar problemas relacionados, incluyendo escalada de privilegios, y se describen en mayor detalle en otras partes de esta guía (por ejemplo, [Pruebas para Bypass de Esquema de Autorización](../05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md) y [Pruebas para Referencias Directas de Objetos Inseguras](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)).

- Enumeración de directorios y archivos: Una interfaz administrativa puede estar presente pero no visiblemente disponible para el probador. La ruta de la interfaz administrativa puede adivinarse mediante solicitudes simples como /admin o /administrator. En algunos escenarios, estas rutas pueden revelarse en segundos usando técnicas avanzadas de búsqueda de Google - [Google dorks](https://www.exploit-db.com/google-hacking-database). Hay muchas herramientas disponibles para realizar fuerza bruta de contenidos del servidor, vea la sección de herramientas a continuación para más información. Un probador puede tener que identificar también el nombre del archivo de la página de administración. Navegar forzosamente a la página identificada puede proporcionar acceso a la interfaz.
- Comentarios y enlaces en el código fuente: Muchos sitios usan código común que se carga para todos los usuarios del sitio. Al examinar todo el código fuente enviado al cliente, pueden descubrirse enlaces a funcionalidades administrativas y deben investigarse.
- Revisión de documentación del servidor y aplicación: Si el servidor de aplicaciones o la aplicación se despliega en su configuración por defecto, puede ser posible acceder a la interfaz administrativa usando información descrita en la documentación de configuración o ayuda. Deben consultarse listas de contraseñas por defecto si se encuentra una interfaz administrativa y se requieren credenciales.
- Información públicamente disponible: Muchas aplicaciones, como WordPress, tienen interfaces administrativas que están disponibles por defecto.
- Puerto alternativo del servidor: Las interfaces administrativas pueden verse en un puerto diferente en el host que la aplicación principal. Por ejemplo, la interfaz de Administración de Apache Tomcat puede verse a menudo en el puerto 8080.
- Manipulación de parámetros: Un parámetro GET o POST, o una cookie puede ser requerido para habilitar la funcionalidad administrativa. Pistas para esto incluyen la presencia de campos ocultos como:

```html
<input type="hidden" name="admin" value="no">
```

o en una cookie:

`Cookie: session_cookie; useradmin=0`

Una vez que se ha descubierto una interfaz administrativa, una combinación de las técnicas anteriores puede usarse para intentar bypass de autenticación. Si esto falla, el probador puede desear intentar un ataque de fuerza bruta. En tal instancia, el probador debe ser consciente del potencial de bloqueo de cuenta administrativa si tal funcionalidad está presente.

### Pruebas de Caja Gris

Se debe realizar un examen más detallado de los componentes del servidor y aplicación para asegurar el endurecimiento (es decir, páginas de administrador no accesibles a todos mediante el uso de filtrado IP u otros controles), y donde sea aplicable, verificación de que todos los componentes no usan credenciales o configuraciones por defecto.
El código fuente debe revisarse para asegurar que el modelo de autorización y autenticación asegura una clara separación de deberes entre usuarios normales y administradores del sitio. Las funciones de interfaz de usuario compartidas entre usuarios normales y administradores deben revisarse para asegurar una clara separación entre el renderizado de tales componentes y la fuga de información de tal funcionalidad compartida.

Cada framework web puede tener sus propias páginas o rutas de admin por defecto, como en los siguientes ejemplos:

PHP:

```html
/phpinfo
/phpmyadmin/
/phpMyAdmin/
/mysqladmin/
/MySQLadmin
/MySQLAdmin
/login.php
/logon.php
/xmlrpc.php
/dbadmin
```

WordPress:

```html
wp-admin/
wp-admin/about.php
wp-admin/admin-ajax.php
wp-admin/admin-db.php
wp-admin/admin-footer.php
wp-admin/admin-functions.php
wp-admin/admin-header.php
```

Joomla:

```html
/administrator/index.php
/administrator/index.php?option=com_login
/administrator/index.php?option=com_content
/administrator/index.php?option=com_users
/administrator/index.php?option=com_menus
/administrator/index.php?option=com_installer
/administrator/index.php?option=com_config
```

Tomcat:

```html
/manager/html
/host-manager/html
/manager/text
/tomcat-users.xml
```

Apache:

```html
/index.html
/httpd.conf
/apache2.conf
/server-status
```

Nginx:

```html
/index.html
/index.htm
/index.php
/nginx_status
/index.php
/nginx.conf
/html/error
```

## Herramientas

Varias herramientas pueden asistir en identificar interfaces y funcionalidades administrativas ocultas, incluyendo:

- [ZAP - Forced Browse](https://www.zaproxy.org/docs/desktop/addons/forced-browse/) es un uso actualmente mantenido del proyecto anterior de OWASP DirBuster.
- [THC-HYDRA](https://github.com/vanhauser-thc/thc-hydra) es una herramienta que permite fuerza bruta de muchas interfaces, incluyendo autenticación HTTP basada en formularios.
- Un forzador de fuerza bruta es mucho más efectivo cuando usa un buen diccionario, como el diccionario [Netsparker](https://www.netsparker.com/blog/web-security/svn-digger-better-lists-for-forced-browsing/).

## Referencias

- [Cirt: Lista de contraseñas por defecto](https://cirt.net/passwords)
- [FuzzDB puede usarse para hacer fuerza bruta de navegación de ruta de login admin](https://github.com/fuzzdb-project/fuzzdb/blob/master/discovery/predictable-filepaths/login-file-locations/Logins.txt)
- [Parámetros comunes de admin o debugging](https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/business-logic/CommonDebugParamNames.txt)