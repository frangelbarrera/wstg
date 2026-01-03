# Probar Configuración de Plataforma de Aplicación

|ID          |
|------------|
|WSTG-CONF-02|

## Resumen

La configuración adecuada de los elementos individuales que componen una arquitectura de aplicación es importante para prevenir errores que podrían comprometer la seguridad de toda la arquitectura.

Revisar y probar configuraciones son tareas críticas en crear y mantener una arquitectura. Esto es porque varios sistemas a menudo vienen con configuraciones genéricas, que pueden no alinearse bien con las tareas que se supone que deben realizar en los sitios específicos donde están instalados.

Mientras que la instalación típica de servidor web y de aplicación contendrá mucha funcionalidad (como ejemplos de aplicaciones, documentación, páginas de prueba), lo que no es esencial debería removerse antes del despliegue para evitar explotación post-instalación.

## Objetivos de Prueba

- Asegurar que archivos predeterminados y conocidos hayan sido removidos.
- Validar que no se deje código de depuración o extensiones en los entornos de producción.
- Revisar los mecanismos de logging establecidos para la aplicación.

## Cómo Probar

### Pruebas de Caja Negra

#### Archivos y Directorios de Muestra y Conocidos

En una instalación predeterminada, muchos servidores web y de aplicación proporcionan aplicaciones y archivos de muestra para el beneficio del desarrollador, para probar si el servidor está funcionando correctamente justo después de la instalación. Sin embargo, muchas aplicaciones de servidor web predeterminadas han sido conocidas por ser vulnerables más tarde. Este fue el caso, por ejemplo, para CVE-1999-0449 (Denegación de Servicio en IIS cuando el sitio de muestra Exair había sido instalado), CAN-2002-1744 (Vulnerabilidad de traversal de directorio en CodeBrws.asp en Microsoft IIS 5.0), CAN-2002-1630 (Uso de sendmail.jsp en Oracle 9iAS), o CAN-2003-1172 (Traversal de directorio en la muestra view-source en Cocoon de Apache).

Escáneres CGI, que incluyen una lista detallada de archivos y directorios de muestra conocidos proporcionados por diferentes servidores web o de aplicación, podrían ser una forma rápida de determinar si estos archivos están presentes. Sin embargo, la única forma de estar realmente seguro es hacer una revisión completa del contenido del servidor web o de aplicación, y determinar si están relacionados con la aplicación en sí o no.

#### Revisión de Comentarios

Es muy común para programadores agregar comentarios al desarrollar aplicaciones web grandes. Sin embargo, comentarios incluidos inline en código HTML podrían revelar información interna que no debería estar disponible para un atacante. A veces, una parte del código fuente se comenta cuando una funcionalidad ya no es requerida, pero este comentario se filtra inadvertidamente a las páginas HTML devueltas a los usuarios.

La revisión de comentarios debería hacerse para determinar si alguna información está siendo filtrada a través de comentarios. Esta revisión solo puede hacerse exhaustivamente a través de un análisis del contenido estático y dinámico del servidor web, y a través de búsquedas de archivos. Puede ser útil navegar el sitio de manera automática o guiada, y almacenar todo el contenido recuperado. Este contenido recuperado puede entonces buscarse para analizar cualquier comentario HTML disponible en el código.

#### Configuración del Sistema

Varias herramientas, documentos o checklists pueden usarse para dar a profesionales de TI y seguridad una evaluación detallada de la conformidad de los sistemas objetivo a varias líneas base de configuración o benchmarks. Tales herramientas incluyen, pero no se limitan a, las siguientes:

- [CIS-CAT Lite](https://www.cisecurity.org/blog/introducing-cis-cat-lite/)
- [Microsoft's Attack Surface Analyzer](https://github.com/microsoft/AttackSurfaceAnalyzer)
- [NIST's National Checklist Program](https://nvd.nist.gov/ncp/repository)

### Pruebas de Caja Gris

#### Revisión de Configuración

La configuración del servidor web o de aplicación toma un rol importante en proteger el contenido del sitio y debe revisarse cuidadosamente para detectar errores comunes de configuración. Obviamente, la configuración recomendada varía dependiendo de la política del sitio, y la funcionalidad que debería proporcionar el software del servidor. En la mayoría de los casos, sin embargo, las directrices de configuración (ya sea proporcionadas por el proveedor del software o partes externas) deberían seguirse para determinar si el servidor ha sido adecuadamente asegurado.

Es imposible decir genéricamente cómo debería configurarse un servidor, sin embargo, algunas directrices comunes deberían tomarse en cuenta:

- Solo habilitar módulos del servidor (extensiones ISAPI en el caso de IIS) que sean necesarios para la aplicación. Esto reduce la superficie de ataque ya que el servidor se reduce en tamaño y complejidad como módulos de software son deshabilitados. También previene vulnerabilidades que podrían aparecer en el software del proveedor de afectar el sitio si solo están presentes en módulos que ya han sido deshabilitados.
- Manejar errores del servidor (40x o 50x) con páginas hechas a medida en lugar de con las páginas predeterminadas del servidor web. Específicamente asegurar que cualquier error de aplicación no sea devuelto al usuario final y que no se filtre código a través de estos errores ya que ayudará a un atacante. Es en realidad muy común olvidar este punto ya que los desarrolladores necesitan esta información en entornos pre-producción.
- Asegurar que el software del servidor se ejecute con privilegios minimizados en el sistema operativo. Esto previene un error en el software del servidor de comprometer directamente todo el sistema, aunque un atacante podría elevar privilegios una vez ejecutando código como el servidor web.
- Asegurar que el software del servidor registre adecuadamente tanto acceso legítimo como errores.
- Asegurar que el servidor esté configurado para manejar sobrecargas adecuadamente y prevenir ataques de Denegación de Servicio. Asegurar que el servidor haya sido ajustado en rendimiento adecuadamente.
- Nunca otorgar identidades no administrativas (con la excepción de `NT SERVICE\WMSvc`) acceso a applicationHost.config, redirection.config, y administration.config (ya sea Lectura o Escritura). Esto incluye `Network Service`, `IIS_IUSRS`, `IUSR`, o cualquier identidad personalizada usada por pools de aplicación IIS. Procesos de trabajo IIS no están destinados a acceder a ninguno de estos archivos directamente.
- Nunca compartir applicationHost.config, redirection.config, y administration.config en la red. Al usar Configuración Compartida, preferir exportar applicationHost.config a otra ubicación (ver la sección titulada "Setting Permissions for Shared Configuration").
- Tener en mente que todos los usuarios pueden leer archivos .NET Framework `machine.config` y root `web.config` por defecto. No almacenar información sensible en estos archivos si debería ser solo para ojos de administrador.
- Encriptar información sensible que debería ser leída solo por procesos de trabajo IIS y no por otros usuarios en la máquina.
- No otorgar acceso de Escritura a la identidad que el servidor Web usa para acceder al `applicationHost.config` compartido. Esta identidad debería tener solo acceso de Lectura.
- Usar una identidad separada para publicar applicationHost.config al share. No usar esta identidad para configurar acceso a la configuración compartida en los servidores Web.
- Usar una contraseña fuerte al exportar las claves de encriptación para uso con configuración compartida.
- Mantener acceso restringido al share conteniendo la configuración compartida y claves de encriptación. Si este share es comprometido, un atacante será capaz de leer y escribir cualquier configuración IIS para tus servidores Web, redirigir tráfico de tu sitio a fuentes maliciosas, y en algunos casos ganar control de todos los servidores web cargando código arbitrario en procesos de trabajo IIS.
- Considerar proteger este share con reglas de firewall y políticas IPsec para permitir solo a los servidores web miembros conectarse.

#### Logging

El logging es un activo importante de la seguridad de una arquitectura de aplicación, ya que puede usarse para detectar fallos en aplicaciones (usuarios constantemente tratando de recuperar un archivo que realmente no existe) así como ataques sostenidos de usuarios maliciosos. Los logs son típicamente generados adecuadamente por software de servidor web y otros. No es común encontrar aplicaciones que registren adecuadamente sus acciones a un log y, cuando lo hacen, la intención principal de los logs de aplicación es producir salida de depuración que podría usarse por el programador para analizar un error particular.

En ambos casos (logs de servidor y aplicación) varios temas deberían probarse y analizarse basados en el contenido del log:

1. ¿Los logs contienen información sensible?
2. ¿Los logs se almacenan en un servidor dedicado?
3. ¿Puede el uso de log generar una condición de Denegación de Servicio?
4. ¿Cómo se rotan? ¿Se mantienen logs por el tiempo suficiente?
5. ¿Cómo se revisan los logs? ¿Pueden administradores usar estas revisiones para detectar ataques dirigidos?
6. ¿Cómo se preservan los backups de log?
7. ¿Los datos siendo logueados son validados (longitud min/máx, chars etc) antes de ser logueados?

##### Información Sensible en Logs

Algunas aplicaciones podrían, por ejemplo, usar solicitudes GET para reenviar datos de formulario que pueden verse en los logs del servidor. Esto significa que logs del servidor podrían contener información sensible (como nombres de usuario y contraseñas, o detalles de cuenta bancaria). Esta información sensible puede ser mal usada por un atacante si obtuvieron los logs, por ejemplo, a través de interfaces administrativas o vulnerabilidades conocidas de servidor web o misconfiguración (como la bien conocida misconfiguración `server-status` en servidores HTTP basados en Apache).

Los logs de eventos a menudo contendrán datos que son útiles para un atacante (fuga de información) o pueden usarse directamente en exploits:

- Información de depuración
- Trazas de pila
- Nombres de usuario
- Nombres de componentes del sistema
- Direcciones IP internas
- Datos personales menos sensibles (ej. direcciones de email, direcciones postales y números de teléfono asociados con individuos nombrados)
- Datos de negocio

También, en algunas jurisdicciones, almacenar alguna información sensible en archivos de log, como datos personales, podría obligar a la empresa a aplicar las leyes de protección de datos que aplicarían a sus bases de datos backend a archivos de log también. Y fallar en hacerlo, incluso inconscientemente, podría llevar penalidades bajo las leyes de protección de datos que aplican.

Una lista más amplia de información sensible es:

- Código fuente de aplicación
- Valores de identificación de sesión
- Tokens de acceso
- Datos personales sensibles y algunas formas de información personalmente identificable (PII)
- Contraseñas de autenticación
- Cadenas de conexión de base de datos
- Claves de encriptación
- Datos de cuenta bancaria o titular de tarjeta de pago
- Datos de una clasificación de seguridad más alta que la que el sistema de logging está permitido almacenar
- Información comercialmente sensible
- Información ilegal de recolectar en la jurisdicción relevante
- Información que un usuario ha optado por no recolectar, o no consentido ej. uso de no rastrear, o donde el consentimiento para recolectar ha expirado

#### Ubicación de Log

Típicamente servidores generarán logs locales de sus acciones y errores, consumiendo el disco del sistema en el que el servidor está corriendo. Sin embargo, si el servidor es comprometido, sus logs pueden ser borrados por el intruso para limpiar todas las trazas de su ataque y métodos. Si esto sucediera el administrador del sistema no tendría conocimiento de cómo ocurrió el ataque o dónde estaba localizada la fuente del ataque. En realidad, la mayoría de kits de herramientas de atacante incluyen un "log zapper" que es capaz de limpiar cualquier log que contenga información dada (como la dirección IP del atacante) y son rutinariamente usados en root kits de nivel sistema de atacante.

Por lo tanto, es sabio mantener logs en una ubicación separada y no en el servidor web en sí. Esto también hace más fácil agregar logs de diferentes fuentes que se refieren a la misma aplicación (como aquellos de una granja de servidores web) y también hace más fácil hacer análisis de log (que puede ser intensivo en CPU) sin afectar el servidor en sí.

#### Almacenamiento de Log

Almacenamiento impropio de logs puede introducir una condición de Denegación de Servicio. Cualquier atacante con recursos suficientes podría ser capaz de producir un número suficiente de solicitudes que llenarían el espacio asignado a archivos de log, si no son específicamente prevenidos de hacerlo. Sin embargo, si el servidor no está configurado adecuadamente, los archivos de log se almacenarán en la misma partición de disco que la usada para el software del sistema operativo o la aplicación en sí. Esto significa que si el disco se llena, el sistema operativo o la aplicación podrían fallar debido a la incapacidad de escribir en el disco.

Típicamente en sistemas UNIX logs estarán localizados en /var (aunque algunas instalaciones de servidor podrían residir en /opt o /usr/local) y es importante asegurar que los directorios en los que se almacenan logs estén en una partición separada. En algunos casos, y para prevenir que los logs del sistema sean afectados, el directorio de log del software del servidor en sí (como /var/log/apache en el servidor web Apache) debería almacenarse en una partición dedicada.

Esto no es decir que logs deberían permitirse crecer para llenar el sistema de archivos en el que residen. El crecimiento de logs de servidor debería monitorearse para detectar esta condición ya que puede ser indicativo de un ataque.

Probar esta condición, que puede ser riesgosa en entornos de producción, puede hacerse disparando un número suficiente y sostenido de solicitudes para ver si estas solicitudes son logueadas y si hay posibilidad de llenar la partición de log a través de estas solicitudes. En algunos entornos donde parámetros QUERY_STRING también son logueados independientemente de si son producidos a través de GET o solicitudes POST, queries grandes pueden simularse que llenarán los logs más rápido ya que, típicamente, una sola solicitud causará solo una pequeña cantidad de datos a ser logueada, como fecha y hora, dirección IP fuente, URI solicitada, y resultado del servidor.

#### Rotación de Log

La mayoría de servidores (pero pocas aplicaciones personalizadas) rotarán logs para prevenir que llenen el sistema de archivos en el que residen. La suposición durante rotación de log es que la información dentro de ellos es solo necesaria por una duración limitada.

Esta característica debería probarse para asegurar que:

- Logs se mantienen por el tiempo definido en la política de seguridad, no más y no menos.
- Logs se comprimen una vez rotados (esto es una conveniencia, ya que significará que más logs se almacenarán para el mismo espacio de disco disponible).
- Permisos de sistema de archivos para archivos de log rotados deberían ser los mismos que (o más estrictos que) aquellos para los archivos de log en sí. Por ejemplo, servidores web necesitarán escribir a los logs que usan pero no necesitan realmente escribir a logs rotados, lo que significa que los permisos de los archivos pueden cambiarse al rotar para prevenir el proceso de servidor web de modificar estos.

Algunos servidores podrían rotar logs cuando alcanzan un tamaño dado. Si esto sucede, debe asegurarse que un atacante no pueda forzar logs a rotar para ocultar sus huellas.

#### Control de Acceso a Log

La información de log de eventos nunca debería ser visible para usuarios finales. Incluso administradores web no deberían tener acceso a tales logs ya que viola controles de separación de deberes. Asegurar que cualquier esquema de control de acceso que se use para proteger acceso a logs crudos, y cualquier aplicación proporcionando capacidades para ver o buscar los logs no estén vinculados con esquemas de control de acceso para otros roles de usuario de aplicación. Tampoco debería ningún dato de log ser visible para usuarios no autenticados.

#### Revisión de Log

Revisar logs puede usarse no solo para extraer estadísticas de uso de archivos en servidores web (lo que es típicamente en lo que se enfocan la mayoría de aplicaciones basadas en log) sino también para determinar si ataques están ocurriendo en el servidor web.

Para analizar ataques de servidor web, los archivos de log de error del servidor necesitan analizarse. La revisión debería concentrarse en:

- Mensajes de error 40x (no encontrado). Una gran cantidad de estos de la misma fuente podrían ser indicativos de una herramienta de escáner CGI siendo usada contra el servidor web
- Mensajes de error 50x (error de servidor). Estos pueden ser una indicación de un atacante abusando partes de la aplicación que fallan inesperadamente. Por ejemplo, las primeras fases de un ataque de inyección SQL producirán estos mensajes de error cuando la query SQL no está construida adecuadamente y su ejecución falla en la base de datos backend.

Estadísticas de log o análisis no deberían generarse o almacenarse en el mismo servidor que produce los logs. De lo contrario, un atacante podría, a través de una vulnerabilidad de servidor web o configuración impropia, ganar acceso a ellos y recuperar información similar como sería divulgada por los archivos de log en sí.

## Referencias

- Apache
    - Apache Security, by Ivan Ristic, O’reilly, March 2005.
    - [Apache Security Secrets: Revealed (Again), Mark Cox, November 2003](https://awe.com/mark/talks/apachecon2003us.html)
    - [Apache Security Secrets: Revealed, ApacheCon 2002, Las Vegas, Mark J Cox, October 2002](https://awe.com/mark/talks/apachecon2002us.html)
    - [Performance Tuning](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
- Lotus Domino
    - Lotus Security Handbook, William Tworek et al., April 2004, available in the IBM Redbooks collection
    - Lotus Domino Security, an X-force white-paper, Internet Security Systems, December 2002
    - Hackproofing Lotus Domino Web Server, David Litchfield, October 2001
- Microsoft IIS
    - [Security Best Practices for IIS 8](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635855(v=ws.11))
    - [CIS Microsoft IIS Benchmarks](https://www.cisecurity.org/benchmark/microsoft_iis/)
    - Securing Your Web Server (Patterns and Practices), Microsoft Corporation, January 2004
    - IIS Security and Programming Countermeasures, by Jason Coombs
    - From Blueprint to Fortress: A Guide to Securing IIS 5.0, by John Davis, Microsoft Corporation, June 2001
    - Secure IIS 5 Checklist, by Michael Howard, Microsoft Corporation, June 2000
- Red Hat’s (formerly Netscape’s) iPlanet
    - Guide to the Secure Configuration and Administration of iPlanet Web Server, Enterprise Edition 4.1, by James M Hayes, The Network Applications Team of the Systems and Network Attack Center (SNAC), NSA, January 2001
- WebSphere
    - IBM WebSphere V5.0 Security, WebSphere Handbook Series, by Peter Kovari et al., IBM, December 2002.
    - IBM WebSphere V4.0 Advanced Edition Security, by Peter Kovari et al., IBM, March 2002.
- General
    - [Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Guide to Computer Security Log Management, NIST
    - [PCI DSS v3.2.1](https://www.pcisecuritystandards.org/document_library) Requirement 10 and PA-DSS v3.2 Requirement 4, PCI Security Standards Council

- Generic:
    - [CERT Security Improvement Modules: Securing Public Web Servers](https://resources.sei.cmu.edu/asset_files/SecurityImprovementModule/2000_006_001_13637.pdf)