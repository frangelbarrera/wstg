# Enumerar interfaces de administración de infraestructura y aplicaciones

|ID          |
|------------|
|WSTG-CONF-05|

## Resumen

Puede haber interfaces de administración en la aplicación o en el servidor de aplicaciones para permitir que ciertos usuarios realicen actividades privilegiadas en el sitio. Se deben realizar pruebas para determinar si un usuario no autorizado o estándar puede acceder a esta funcionalidad privilegiada y cómo puede hacerlo.

Una aplicación puede requerir una interfaz de administración para que un usuario privilegiado acceda a funcionalidades que permiten modificar el funcionamiento del sitio. Estos cambios pueden incluir:

- Aprovisionamiento de cuentas de usuario
- Diseño y disposición del sitio
- Manipulación de datos
- Cambios de configuración

En muchos casos, estas interfaces no cuentan con controles suficientes para protegerlas del acceso no autorizado. Las pruebas tienen como objetivo descubrir estas interfaces de administración y acceder a las funcionalidades destinadas a los usuarios privilegiados.

## Objetivos de la prueba

- Identificar interfaces de administración ocultas y funcionalidades asociadas, incluidas las consolas de administración de la nube, de orquestación de contenedores, de gestión de API y de gestión de CI/CD.
- Comprobar si alguna interfaz descubierta usa credenciales predeterminadas, débiles o reutilizadas, y si está expuesta a redes o usuarios distintos de los previstos.

## Cómo probar

### Pruebas de caja negra

En la siguiente sección se describen vectores que pueden utilizarse para comprobar la presencia de interfaces administrativas. Estas técnicas también pueden utilizarse para probar problemas relacionados, incluida la escalada de privilegios, que se describen con más detalle en otras partes de esta guía (por ejemplo, [eludir el esquema de autorización](../05-Authorization/02-Bypassing_Authorization_Schema.md) y [referencias directas a objetos inseguras](../05-Authorization/04-Insecure_Direct_Object_References.md)).

- Enumeración de directorios y archivos: Puede haber una interfaz administrativa que no esté visiblemente disponible para el evaluador. La ruta de la interfaz administrativa puede adivinarse mediante solicitudes sencillas, como /admin o /administrator. En algunos casos, estas rutas pueden descubrirse en segundos mediante técnicas avanzadas de búsqueda en Google: [Google dorks](https://www.exploit-db.com/google-hacking-database). Hay muchas herramientas para realizar fuerza bruta sobre el contenido del servidor; consulte la sección de herramientas a continuación para obtener más información. También puede ser necesario identificar el nombre del archivo de la página de administración. Acceder directamente a la página identificada puede proporcionar acceso a la interfaz.
- Comentarios y enlaces en el código fuente: Muchos sitios utilizan código común que se carga para todos los usuarios. Al examinar todo el código fuente enviado al cliente, pueden descubrirse enlaces a funcionalidades de administración, que deben investigarse.
- Revisión de la documentación del servidor y de la aplicación: Si el servidor de aplicaciones o la aplicación se despliega con su configuración predeterminada, es posible que se pueda acceder a la interfaz de administración mediante información descrita en la documentación de configuración o de ayuda. Si se encuentra una interfaz administrativa que requiere credenciales, deben consultarse listas de contraseñas predeterminadas.
- Información disponible públicamente: Muchas aplicaciones, como WordPress, tienen interfaces de administración disponibles de forma predeterminada.
- Puerto alternativo del servidor: Las interfaces de administración pueden estar en un puerto distinto del de la aplicación principal en el host. Por ejemplo, la interfaz de administración de Apache Tomcat suele estar disponible en el puerto 8080.
- Manipulación de parámetros: Puede ser necesario un parámetro GET o POST, o una cookie, para habilitar la funcionalidad de administración. Algunas pistas son la presencia de campos ocultos como:

```html
<input type="hidden" name="admin" value="no">
```

o en una cookie:

`Cookie: session_cookie; useradmin=0`

Una vez descubierta una interfaz de administración, puede utilizarse una combinación de las técnicas anteriores para intentar eludir la autenticación. Si esto falla, el evaluador puede intentar un ataque de fuerza bruta. En ese caso, debe tener en cuenta la posibilidad de que se bloquee la cuenta administrativa, si existe tal funcionalidad.

### Pruebas de caja gris

Se debe realizar un examen más detallado de los componentes del servidor y de la aplicación para comprobar que estén protegidos (es decir, que las páginas de administración no sean accesibles para todo el mundo mediante filtrado de IP u otros controles) y, cuando corresponda, verificar que ningún componente utilice credenciales o configuraciones predeterminadas.
Se debe revisar el código fuente para comprobar que el modelo de autorización y autenticación garantice una separación clara de funciones entre los usuarios normales y los administradores del sitio. También se deben revisar las funciones de la interfaz de usuario compartidas por usuarios normales y administradores para comprobar que haya una separación clara entre la representación de esos componentes y la filtración de información derivada de dichas funciones compartidas.

Cada framework web puede tener sus propias páginas o rutas de administración predeterminadas, como en los siguientes ejemplos:

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

### Interfaces modernas de administración, gestión de API, contenedores y CI/CD

Además de las rutas clásicas de administración de aplicaciones y frameworks, en los despliegues modernos suelen exponerse otras interfaces privilegiadas. Los evaluadores deben enumerarlas específicamente y comprobar si usan credenciales predeterminadas o si están expuestas de forma involuntaria:

#### Consolas de proveedores de nube

- Las propias consolas web de proveedores de nube (AWS, Azure, GCP) no son directamente accesibles desde la superficie de ataque de la aplicación, pero las credenciales filtradas o las configuraciones incorrectas suelen detectarse a través de la aplicación: compruebe si hay credenciales de nube expuestas en archivos JavaScript del cliente, archivos de configuración o mensajes de error, y confirme que no permiten acceder a la consola o API más allá de lo que necesita la aplicación.
- Las interfaces de administración autohospedadas de servicios relacionados con la nube suelen encontrarse expuestas: navegadores de almacenamiento de objetos, consolas de administración de bases de datos (por ejemplo, proxies de RDS/Cloud SQL con interfaces de administración expuestas) y paneles de supervisión proporcionados por IaaS. Trate cualquier instancia descubierta como una interfaz de administración y sométala a las mismas comprobaciones de credenciales y exposición que se describen a continuación.

#### Paneles de Kubernetes y de orquestación de contenedores

- Kubernetes Dashboard: históricamente se ha expuesto sin autenticación o con un inicio de sesión mediante un `service-account-token` con privilegios excesivos, lo que permite controlar todo el clúster. Compruebe si existen `/api/v1/namespaces/kubernetes-dashboard/...` o un ingreso/NodePort dedicado al panel, y confirme que se requiere autenticación y que se utiliza una cuenta de servicio con privilegios mínimos, en lugar de `cluster-admin` (`kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>`, según [Permisos de archivos](09-File_Permissions.md#kubernetes-volume-mounts-and-securitycontext)).
- API de solo lectura de `kubelet` (puerto `10255`, obsoleta) y API autenticada de kubelet (puerto `10250`): compruebe si alguna es accesible desde fuera de la red del clúster y si está habilitado el acceso anónimo.
- etcd (puerto `2379`/`2380`): si es accesible, el acceso sin autenticación expone todo el estado del clúster, incluidos los Secrets.
- Interfaces de usuario de plataformas de contenedores, como Portainer, Rancher o un proxy de la consola de Kubernetes administrada por un proveedor de nube: compruebe si tienen credenciales de administrador predeterminadas y si los puntos de conexión de configuración inicial/registro siguen siendo accesibles después del despliegue (algunas herramientas permiten que cualquiera reclame la primera cuenta de administrador si no se completó la configuración).

#### Puntos de conexión de administración/Actuator de aplicaciones

- Spring Boot Actuator: `/actuator`, `/actuator/env`, `/actuator/heapdump`, `/actuator/httptrace` y `/actuator/mappings` son, por sí mismos, interfaces de administración, no simples datos de depuración: `/actuator/env` puede exponer credenciales, y algunas versiones o configuraciones de Actuator permiten reconfigurar el sistema (por ejemplo, cambiar los niveles de registro mediante `/actuator/loggers`) en lugar de limitarse al acceso de lectura. Consulte [Puntos de conexión y modos de depuración](02-Application_Platform_Configuration.md#debug-endpoints-and-debug-modes) para ver la lista completa de puntos de conexión; cualquier punto de conexión de Actuator accesible sin autenticación debe considerarse aquí un hallazgo de exposición de una interfaz de administración, no solo una filtración de información.
- Puntos de conexión de administración equivalentes de otros frameworks: middleware de comprobación de estado/diagnóstico de ASP.NET Core, administración de Django (`/admin/` con `DEBUG=True` o un superusuario débil o predeterminado) y módulos o blueprints de administración nativos similares que se habilitan de forma predeterminada.

#### Consolas de gestión de API

- Las interfaces de administración de puertas de enlace/API (por ejemplo, Kong Admin API, Apigee, acceso a la consola de AWS API Gateway mediante claves filtradas y Tyk Dashboard) suelen escuchar en un puerto o una ruta distintos de los de la API pública y pueden no tener autenticación habilitada de forma predeterminada (por ejemplo, históricamente, Kong Admin API permitía el acceso sin autenticación de forma predeterminada en el puerto `8001`).
- Compruebe si hay documentación de API o portales para desarrolladores expuestos que también ofrezcan funcionalidades de administración (generación de claves, cambios de cuotas), en lugar de limitarse a la documentación, y confirme que estas funcionalidades exijan una autenticación proporcional a las acciones disponibles.

#### Interfaces de administración de CI/CD

- Algunas consolas de CI/CD autohospedadas habituales que deben enumerarse son Jenkins (`/login`, consola `/script`, `/asynchPeople/`), el área de administración de CI/CD de GitLab, TeamCity, Argo CD y los puntos de conexión de administración de ejecutores autohospedados de GitHub Actions/GitLab Actions.
- Estas interfaces tienen un impacto especialmente alto si quedan expuestas, ya que los sistemas de CI/CD suelen disponer de credenciales de despliegue, acceso al código fuente y la capacidad de ejecutar pasos de compilación arbitrarios. Compruebe si hay:
    - Credenciales predeterminadas o documentadas que nunca se cambiaron (por ejemplo, el archivo de contraseña inicial de administrador de Jenkins que quedó legible o los asistentes de configuración que no se completaron).
    - Acceso anónimo o de invitado habilitado (históricamente, Jenkins permite esto mediante una configuración de «lectura/compilación anónimas»).
    - Consolas de scripts o creación arbitraria de canalizaciones/trabajos accesibles sin autenticación completa, ya que estas funcionalidades suelen equivaler a la ejecución remota de código.

#### Comprobaciones de credenciales predeterminadas y exposición

Para cualquier interfaz de administración encontrada mediante las técnicas anteriores (clásica o moderna), realice las mismas comprobaciones básicas:

- Pruebe las credenciales predeterminadas documentadas para ese producto y versión específicos (consulte la documentación del proveedor y listas como [Cirt: lista de contraseñas predeterminadas](https://cirt.net/passwords) o [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)); no intente la fuerza bruta más allá de una lista pequeña y documentada de credenciales predeterminadas sin autorización.
- Confirme que no se pueda acceder a la interfaz desde la Internet pública si está destinada únicamente al uso interno o por parte de operadores (compruébelo mediante un análisis directo de IP/puerto y también mediante el nombre de host, ya que puede ser accesible por IP aunque no haya enlaces a ella).
- Confirme que TLS sea obligatorio y que la interfaz no acepte HTTP sin cifrar para enviar credenciales.
- Si la interfaz ofrece un asistente de configuración/inicio, confirme que no se pueda volver a activar ni acceder a él después del despliegue para reclamar una cuenta de administrador.

## Herramientas

Varias herramientas pueden ayudar a identificar interfaces y funcionalidades de administración ocultas, entre ellas:

- [ZAP - Forced Browse](https://www.zaproxy.org/docs/desktop/addons/forced-browse/) es una aplicación actualmente mantenida del proyecto anterior DirBuster de OWASP.
- [THC-HYDRA](https://github.com/vanhauser-thc/thc-hydra) es una herramienta que permite realizar ataques de fuerza bruta contra muchas interfaces, incluida la autenticación HTTP basada en formularios.
- Una herramienta de fuerza bruta resulta mucho más eficaz si utiliza un buen diccionario, como el diccionario de [Netsparker](https://www.invicti.com/blog/web-security/svn-digger-better-lists-for-forced-browsing).
- [nuclei](https://github.com/projectdiscovery/nuclei), con los conjuntos de plantillas `exposed-panels` y `default-logins`, puede identificar mediante huellas muchas de las interfaces modernas de administración, CI/CD y orquestación descritas anteriormente, a gran escala.
- [kube-hunter](https://github.com/aquasecurity/kube-hunter): identifica puntos de conexión expuestos del servidor de API de Kubernetes, kubelet y el panel.
- [SecLists - Credenciales predeterminadas](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)

## Referencias

- [Cirt: lista de contraseñas predeterminadas](https://cirt.net/passwords)
- [FuzzDB puede utilizarse para realizar fuerza bruta en la búsqueda de rutas de inicio de sesión de administración](https://github.com/fuzzdb-project/fuzzdb/blob/master/discovery/predictable-filepaths/login-file-locations/Logins.txt)
- [Parámetros comunes de administración o depuración](https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/business-logic/CommonDebugParamNames.txt)
- [Seguridad de Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
- [Kong Admin API: proteger Admin API](https://developer.konghq.com/gateway/secure-the-admin-api/)
- [Jenkins: proteger Jenkins](https://www.jenkins.io/doc/book/security/)