# Probar Configuración de Infraestructura de Red

|ID          |
|------------|
|WSTG-CONF-01|

## Resumen

La complejidad intrínseca de la infraestructura de servidor web interconectada y heterogénea, que puede incluir cientos de aplicaciones web, hace que la gestión y revisión de configuración sea un paso fundamental en las pruebas y despliegue de cada aplicación individual. Solo se necesita una sola vulnerabilidad para socavar la seguridad de toda la infraestructura, e incluso problemas pequeños y aparentemente insignificantes pueden evolucionar a riesgos graves para otra aplicación en el mismo servidor. Para abordar estos problemas, es de suma importancia realizar una revisión exhaustiva de la configuración y problemas de seguridad conocidos, después de haber mapeado toda la arquitectura.

La gestión adecuada de la configuración de la infraestructura del servidor web es muy importante para preservar la seguridad de la aplicación en sí. Si elementos como el software del servidor web, los servidores de base de datos backend o los servidores de autenticación no se revisan y aseguran adecuadamente, podrían introducir riesgos no deseados o introducir nuevas vulnerabilidades que podrían comprometer la aplicación en sí.

Por ejemplo, una vulnerabilidad del servidor web que permitiría a un atacante remoto divulgar el código fuente de la aplicación en sí (una vulnerabilidad que ha surgido varias veces tanto en servidores web como en servidores de aplicaciones) podría comprometer la aplicación, ya que usuarios anónimos podrían usar la información divulgada en el código fuente para aprovechar ataques contra la aplicación o sus usuarios.

Los siguientes pasos deben tomarse para probar la infraestructura de gestión de configuración:

- Los diferentes elementos que componen la infraestructura deben determinarse para entender cómo interactúan con una aplicación web y cómo afectan su seguridad.
- Todos los elementos de la infraestructura deben revisarse para asegurar que no contengan vulnerabilidades conocidas.
- Debe hacerse una revisión de las herramientas administrativas utilizadas para mantener todos los diferentes elementos.
- Los sistemas de autenticación deben revisarse para asegurar que satisfagan las necesidades de la aplicación y que no puedan ser manipulados por usuarios externos para aprovechar el acceso.
- Debe mantenerse una lista de puertos definidos que sean requeridos para la aplicación y mantenerse bajo control de cambios.

Después de haber mapeado los diferentes elementos que componen la infraestructura (ver [Mapear Arquitectura de Red y Aplicación](../01-Information_Gathering/10-Map_Application_Architecture.md)), es posible revisar la configuración de cada elemento encontrado y probar cualquier vulnerabilidad conocida.

## Objetivos de Prueba

- Revisar las configuraciones de aplicaciones establecidas en la red y validar que no sean vulnerables.
- Validar que los frameworks y sistemas utilizados sean seguros y no susceptibles a vulnerabilidades conocidas debido a software no mantenido o configuraciones y credenciales predeterminadas.

## Cómo Probar

### Vulnerabilidades Conocidas del Servidor

Vulnerabilidades en varias áreas de la arquitectura de la aplicación, ya sea en el servidor web o en la base de datos backend, pueden comprometer severamente la aplicación. Por ejemplo, considere una vulnerabilidad del servidor que permite a un usuario remoto no autenticado subir archivos al servidor web o incluso reemplazar archivos existentes. Esta vulnerabilidad podría comprometer la aplicación, ya que un usuario malicioso podría reemplazar la aplicación en sí o introducir código que afectaría los servidores backend, ya que su código de aplicación se ejecutaría como cualquier otra aplicación.

Revisar vulnerabilidades del servidor puede ser difícil de hacer si la prueba necesita hacerse a través de una prueba de penetración ciega. En estos casos, las vulnerabilidades necesitan probarse desde un sitio remoto, típicamente usando una herramienta automatizada. Sin embargo, probar algunas vulnerabilidades puede tener resultados impredecibles en el servidor web, y probar otras (como las directamente involucradas en ataques de denegación de servicio) podría no ser posible debido al tiempo de inactividad del servicio involucrado si la prueba fue exitosa.

Algunas herramientas automatizadas marcarán vulnerabilidades dependiendo de la versión del servidor web que recuperen. Esto lleva a falsos positivos y falsos negativos. Por un lado, si la versión del servidor web ha sido removida u oscurecida por el administrador del sitio local, la herramienta de escaneo no marcará el servidor como vulnerable incluso si lo es. Por otro lado, si el proveedor del software no actualiza la versión del servidor web cuando se corrigen vulnerabilidades, la herramienta de escaneo marcará vulnerabilidades que no existen. El último caso es en realidad muy común ya que algunos proveedores de sistemas operativos retroportan parches de vulnerabilidades de seguridad al software que proporcionan en el sistema operativo, pero no hacen una carga completa a la última versión del software. Esto sucede en la mayoría de las distribuciones GNU/Linux como Debian, Red Hat y SuSE. En la mayoría de los casos, el escaneo de vulnerabilidades de una arquitectura de aplicación solo encontrará vulnerabilidades asociadas con los "elementos expuestos" de la arquitectura (como el servidor web) y usualmente será incapaz de encontrar vulnerabilidades asociadas a elementos que no están directamente expuestos, como el backend de autenticación, la base de datos backend o proxies inversos [1] en uso.

Finalmente, no todos los proveedores de software divulgan públicamente vulnerabilidades, lo que significa que estas debilidades pueden no estar registradas en bases de datos de vulnerabilidades conocidas [2]. Esta información solo se divulga a clientes o se publica a través de correcciones que no tienen avisos acompañantes. Esto reduce la efectividad de las herramientas de escaneo de vulnerabilidades. Típicamente, la cobertura de vulnerabilidades de estas herramientas será muy buena para productos comunes (como el servidor web Apache, Microsoft IIS o IBM's Lotus Domino) pero carecerá para productos menos conocidos.

Es por esto que revisar vulnerabilidades se hace mejor cuando el probador es proporcionado con información interna sobre el software, incluyendo versiones, lanzamientos y parches aplicados. Con esta información, el probador puede recuperar datos del proveedor y analizar vulnerabilidades potenciales en la arquitectura, así como su impacto potencial en la aplicación. Cuando sea posible, estas vulnerabilidades pueden probarse para determinar sus efectos reales y detectar si podría haber elementos externos (como sistemas de detección o prevención de intrusiones) que podrían reducir o negar la posibilidad de explotación exitosa. Los probadores podrían incluso determinar a través de una revisión de configuración que la vulnerabilidad no está realmente presente ya que afecta un componente de software que no está en uso.

También vale la pena notar que los proveedores a veces corrigen silenciosamente vulnerabilidades y hacen las correcciones disponibles con nuevos lanzamientos de software. Diferentes proveedores tienen ciclos de lanzamiento variables que determinan el soporte que pueden proporcionar para lanzamientos más antiguos. Un probador con información detallada sobre las versiones de software utilizadas por la arquitectura puede analizar el riesgo asociado con el uso de lanzamientos de software antiguos que podrían no ser soportados en el corto plazo o ya no son soportados. Esto es crítico porque si emerge una vulnerabilidad en una versión de software antiguo no soportada, el personal de sistemas puede no estar directamente al tanto de ello. No se harán parches disponibles para ello y los avisos podrían no listar esa versión como vulnerable ya que ya no es soportada. Incluso si están al tanto de la vulnerabilidad y los riesgos asociados del sistema, una actualización completa a un nuevo lanzamiento de software será necesaria, potencialmente introduciendo un tiempo de inactividad significativo en la arquitectura de la aplicación o necesitando recodificación de la aplicación debido a incompatibilidades con la última versión del software.

### Herramientas Administrativas

Cualquier infraestructura de servidor web requiere la existencia de herramientas administrativas para mantener y actualizar la información utilizada por la aplicación. Esta información incluye contenido estático (páginas web, archivos gráficos), código fuente de la aplicación, bases de datos de autenticación de usuarios, etc. El tipo de herramientas administrativas utilizadas puede variar dependiendo del sitio específico, tecnología o software en uso. Por ejemplo, algunos servidores web serán gestionados usando interfaces administrativas que son ellas mismas servidores web (como el servidor web iPlanet) o serán administradas por archivos de configuración de texto plano (como en el caso de Apache [3]) o usan herramientas GUI del sistema operativo (como cuando se usa el servidor IIS de Microsoft o ASP.Net).

En la mayoría de los casos, la configuración del servidor se gestiona con varias herramientas de mantenimiento de archivos, administradas a través de servidores FTP, WebDAV, sistemas de archivos de red (NFS, CIFS) u otros mecanismos. Obviamente, el sistema operativo de los elementos que componen la arquitectura de la aplicación también será gestionado usando otras herramientas. Las aplicaciones también pueden contener interfaces administrativas embebidas para gestionar datos de la aplicación (usuarios, contenido, etc.).

Después de mapear las interfaces administrativas utilizadas para gestionar diferentes partes de la arquitectura, es importante revisarlas. Si un atacante gana acceso a cualquiera de estas interfaces, podrían potencialmente comprometer o dañar la arquitectura de la aplicación. Para lograr esto, es importante:

- Determinar los mecanismos que controlan el acceso a estas interfaces y sus susceptibilidades asociadas. Esta información puede estar disponible en línea.
- Asegurar que el nombre de usuario y contraseña predeterminados sean cambiados.

Algunas compañías eligen no gestionar todos los aspectos de sus aplicaciones de servidor web y pueden delegar la gestión de contenido a otras partes. Esta compañía externa podría proporcionar solo ciertas partes del contenido (como actualizaciones de noticias o promociones), o podría gestionar completamente el servidor web (incluyendo contenido y código). Es común encontrar interfaces administrativas disponibles desde internet en estas situaciones, ya que usar internet es más barato que proporcionar una línea dedicada que conectará la compañía externa a la infraestructura de la aplicación a través de una interfaz de gestión únicamente. En tales situaciones, es crucial probar si las interfaces administrativas son vulnerables a ataques.

## Referencias

- [1] WebSEAL, también conocido como Tivoli Authentication Manager, es un proxy inverso de IBM que es parte del framework Tivoli.
- [2] Como Bugtraq de Symantec, X-Force de ISS o la Base de Datos Nacional de Vulnerabilidades (NVD) de NIST.
- [3] Hay algunas herramientas de administración basadas en GUI para Apache (como NetLoony) pero no están en uso generalizado aún.