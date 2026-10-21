# Mapear la arquitectura de la aplicación

|ID          |
|------------|
|WSTG-INFO-10|

## Resumen

Para probar una aplicación de manera eficaz y poder proporcionar recomendaciones útiles sobre cómo abordar los problemas identificados, es importante entender qué se está probando realmente. Además, puede ser útil determinar si algunos componentes específicos deben considerarse fuera del alcance de las pruebas.

Las aplicaciones web modernas pueden variar significativamente en complejidad: desde un script sencillo que se ejecuta en un único servidor hasta una aplicación muy compleja distribuida en decenas de sistemas, lenguajes y componentes diferentes. También puede haber componentes adicionales a nivel de red, como firewalls o sistemas de protección contra intrusiones, que pueden tener un impacto significativo en las pruebas.

## Objetivos de las pruebas

- Entender la arquitectura de la aplicación y las tecnologías que utiliza.

## Cómo probar

Al probar desde una perspectiva de caja negra, es importante intentar construir una imagen clara de cómo funciona la aplicación y qué tecnologías y componentes utiliza. En algunos casos, es posible probar componentes específicos, como un firewall de aplicaciones web, mientras que otros pueden identificarse inspeccionando el comportamiento de la aplicación.

Las secciones siguientes proporcionan una descripción general de alto nivel de los componentes arquitectónicos comunes, junto con detalles sobre cómo identificarlos.

### Componentes de la aplicación

#### Servidor web

Las aplicaciones sencillas pueden ejecutarse en un único servidor, que puede identificarse siguiendo los pasos descritos en la sección [Obtener la huella digital del servidor web](02-Fingerprint_Web_Server.md) de la guía.

#### Plataforma como servicio (PaaS)

En un modelo de plataforma como servicio (PaaS), el proveedor del servicio administra el servidor web y la infraestructura subyacente, y el cliente solo es responsable de la aplicación que se despliega en dicha infraestructura. Desde la perspectiva de las pruebas, hay dos diferencias principales:

- El propietario de la aplicación no tiene acceso a la infraestructura subyacente, lo que significa que no podrá corregir directamente ningún problema.
- Es probable que las pruebas de infraestructura estén fuera del alcance de cualquier encargo.

En algunos casos, es posible identificar el uso de PaaS, ya que la aplicación puede utilizar un nombre de dominio específico (por ejemplo, las aplicaciones desplegadas en Azure App Services tendrán un dominio `*.azurewebsites.net`, aunque también pueden utilizar dominios personalizados). En otros casos, es difícil determinar si se utiliza PaaS.

#### Sin servidor

En un modelo sin servidor, los desarrolladores proporcionan código que se ejecuta directamente en una plataforma de alojamiento como funciones individuales, en lugar de ejecutar una aplicación web tradicional de mayor tamaño desplegada en un directorio raíz web. Esto lo hace adecuado para una arquitectura basada en microservicios. Al igual que en un entorno PaaS, es probable que las pruebas de infraestructura estén fuera del alcance.

En algunos casos, la presencia de encabezados HTTP específicos puede indicar el uso de código sin servidor. Por ejemplo, las funciones de AWS Lambda suelen devolver los siguientes encabezados:

```http
X-Amz-Invocation-Type
X-Amz-Log-Type
X-Amz-Client-Context
```

Azure Functions son menos fáciles de identificar. Suelen devolver el encabezado `Server: Kestrel`, pero esto por sí solo no basta para determinar que se trata de una función de Azure App, ya que podría ser otro código que se ejecuta en Kestrel.

#### Microservicios

En una arquitectura basada en microservicios, la API de la aplicación está compuesta por varios servicios independientes, en lugar de ejecutarse como una aplicación monolítica. Los servicios suelen ejecutarse en contenedores (normalmente con Kubernetes) y pueden utilizar distintos sistemas operativos y lenguajes. Aunque suelen estar detrás de una única puerta de enlace de API y un único dominio, el uso de varios lenguajes (a menudo indicado en mensajes de error detallados) puede sugerir que se utilizan microservicios.

#### Almacenamiento estático

Muchas aplicaciones almacenan contenido estático en plataformas de almacenamiento dedicadas, en lugar de alojarlo directamente en el servidor web principal. Las dos plataformas más comunes son los buckets S3 de Amazon y las cuentas de almacenamiento de Azure, que pueden identificarse fácilmente por sus nombres de dominio:

- `BUCKET.s3.amazonaws.com` o `s3.REGION.amazonaws.com/BUCKET` para los buckets S3 de Amazon
- `ACCOUNT.blob.core.windows.net` para las cuentas de almacenamiento de Azure

Estas cuentas de almacenamiento pueden exponer archivos sensibles, como se explica en la sección [Guía de pruebas de almacenamiento en la nube](../02-Configuration_and_Deployment_Management/11-Cloud_Storage.md).

#### Base de datos

La mayoría de las aplicaciones web no triviales utilizan algún tipo de base de datos para almacenar contenido dinámico. En algunos casos, es posible determinar qué base de datos se utiliza. A menudo puede hacerse de las siguientes maneras:

- Escanear los puertos del servidor y buscar puertos abiertos asociados a bases de datos específicas.
- Provocar mensajes de error relacionados con SQL (o NoSQL) o encontrar errores existentes mediante un [motor de búsqueda](../01-Information_Gathering/01-Conduct_Search_Engine_Reconnaissance_for_Information_Leakage.md)

Cuando no es posible determinar la base de datos de forma concluyente, el evaluador a menudo puede hacer una conjetura fundamentada a partir de otros aspectos de la aplicación:

- Windows, IIS y ASP.NET suelen utilizar Microsoft SQL Server.
- Los sistemas integrados suelen utilizar SQLite.
- PHP suele utilizar MySQL o PostgreSQL.
- APEX suele utilizar Oracle.

Estas no son reglas estrictas, pero pueden proporcionar un punto de partida razonable si no se dispone de mejor información.

#### Autenticación

La mayoría de las aplicaciones cuentan con autenticación de usuarios. Pueden utilizar varios sistemas de autenticación, como los siguientes:

- Configuración del servidor web (incluidos los archivos `.htaccess`) o contraseñas codificadas directamente en scripts.
    - Suele presentarse como autenticación HTTP Basic, indicada por una ventana emergente en el navegador y un encabezado HTTP `WWW-Authenticate: Basic`.
- Cuentas de usuario locales en una base de datos.
    - Suelen integrarse en un formulario o punto de conexión de la API de la aplicación.
- Una fuente de autenticación central existente, como Active Directory o un servidor LDAP.
    - Puede utilizar autenticación NTLM, indicada por un encabezado HTTP `WWW-Authenticate: NTLM`.
    - Puede integrarse en la aplicación web mediante un formulario.
    - Puede requerir que el nombre de usuario se introduzca con el formato "DOMAIN\username" o proporcionar un menú desplegable con los dominios disponibles.
- Inicio de sesión único (SSO) con un proveedor interno o externo.
    - Suele utilizar OAuth, OpenID Connect o SAML.

Las aplicaciones pueden ofrecer varias opciones para que los usuarios se autentiquen (como registrar una cuenta local o utilizar su cuenta existente de Facebook) y pueden utilizar distintos mecanismos para los usuarios habituales y los administradores.

#### Servicios y API de terceros

Casi todas las aplicaciones web incluyen recursos de terceros que se cargan o con los que interactúa el cliente. Estos pueden incluir:

- [Contenido activo](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Mixed_content#mixed_active_content) (como scripts, hojas de estilo, fuentes e iframes)
- [Contenido pasivo](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Mixed_content#mixed_passivedisplay_content) (como imágenes y videos)
- API externas
- Botones de redes sociales
- Redes publicitarias
- Pasarelas de pago

El navegador del usuario solicita estos recursos directamente, por lo que son más fáciles de identificar mediante las herramientas para desarrolladores o un proxy de interceptación. Aunque es importante identificarlos (ya que pueden afectar la seguridad de la aplicación), hay que tener en cuenta que *suelen estar fuera del alcance de las pruebas*, pues pertenecen a terceros.

### Componentes de red

#### Proxy inverso

Un proxy inverso se sitúa delante de uno o varios servidores de backend y redirige las solicitudes al destino adecuado. Puede utilizarse para implementar varias funciones, como las siguientes:

- Actuar como un [balanceador de carga](#load-balancer) o un [firewall de aplicaciones web](#web-application-firewall-waf)
- Permitir que varias aplicaciones se alojen en una única dirección IP o dominio (en subdirectorios)
- Implementar el filtrado de IP u otras restricciones
- Almacenar en caché contenido del backend para mejorar el rendimiento

No siempre es posible detectar un proxy inverso (especialmente si solo hay una aplicación detrás de él), pero a veces puede identificarse por lo siguiente:

- Una discrepancia entre el servidor de frontend y la aplicación de backend (por ejemplo, un encabezado `Server: nginx` con una aplicación ASP.NET).
    - Esto a veces puede dar lugar a [vulnerabilidades de contrabando de solicitudes](https://portswigger.net/web-security/request-smuggling).
- Encabezados duplicados (especialmente el encabezado `Server`).
- Varias aplicaciones alojadas en la misma dirección IP o dominio (especialmente si utilizan lenguajes diferentes).

#### Balanceador de carga

Un balanceador de carga se sitúa delante de varios servidores de backend y distribuye las solicitudes entre ellos para proporcionar mayor redundancia y capacidad de procesamiento a la aplicación.

Los balanceadores de carga pueden ser difíciles de detectar, pero a veces es posible identificarlos enviando varias solicitudes y examinando las respuestas en busca de diferencias, como las siguientes:

- Horas del sistema incoherentes.
- Direcciones IP internas o nombres de host diferentes en mensajes de error detallados.
- Direcciones diferentes devueltas mediante [falsificación de solicitudes del lado del servidor (SSRF)](../07-Injection/19-Server-Side_Request_Forgery.md).

La presencia de cookies específicas también puede ser un indicio (por ejemplo, los balanceadores de carga F5 BIG-IP crearán una cookie llamada `BIGipServer`.

#### Red de distribución de contenido (CDN)

Una red de distribución de contenido (CDN) es un conjunto de servidores proxy de caché distribuidos geográficamente, diseñado para mejorar el rendimiento del sitio.

Normalmente se configura apuntando el dominio de acceso público a los servidores de la CDN y, después, configurando la CDN para que se conecte a los servidores de backend correctos (a veces llamados «origen»).

La forma más sencilla de detectar una CDN es realizar una consulta WHOIS para las direcciones IP a las que resuelve el dominio. Si pertenecen a una empresa de CDN (como Akamai, Cloudflare o Fastly; consulta [Wikipedia](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers) para ver una lista más completa), es probable que se esté utilizando una CDN.

Al probar un sitio detrás de una CDN, hay que tener en cuenta lo siguiente:

- Las direcciones IP y los servidores pertenecen al proveedor de la CDN y es probable que estén fuera del alcance de las pruebas de infraestructura.
- Muchas CDN también incluyen funciones como la detección de bots, la limitación de la tasa de solicitudes y los firewalls de aplicaciones web.
- Las CDN suelen almacenar contenido en caché. Por lo tanto, es posible que los cambios realizados en el backend no aparezcan inmediatamente en el sitio.

Si el sitio está detrás de una CDN, puede ser útil identificar los servidores de backend. Si no se aplica un control de acceso adecuado, el evaluador podría eludir la CDN (y las protecciones que ofrece) accediendo directamente a los servidores de backend. Existen varios métodos que pueden permitir identificar el sistema de backend:

- Los correos electrónicos enviados por la aplicación pueden provenir directamente del servidor de backend, lo que podría revelar su dirección IP.
- El análisis exhaustivo de DNS, las transferencias de zona o las listas de transparencia de certificados de un dominio pueden revelarlo en un subdominio.
- Escanear los rangos de IP que se sabe que utiliza la empresa puede ayudar a identificar el servidor de backend.
- Explotar la [falsificación de solicitudes del lado del servidor (SSRF)](../07-Injection/19-Server-Side_Request_Forgery.md) puede revelar la dirección IP.
- Los mensajes de error detallados de la aplicación pueden exponer direcciones IP o nombres de host.

### Componentes de seguridad

#### Firewall de red

La mayoría de los servidores web están protegidos por un firewall de filtrado de paquetes o de inspección con estado, que bloquea el tráfico de red que no es necesario. Para detectarlo, hay que escanear los puertos del servidor y examinar los resultados.

Si la mayoría de los puertos aparecen como "closed" (es decir, devuelven un paquete `RST` en respuesta al paquete inicial `SYN`), esto sugiere que el servidor podría no estar protegido por un firewall. Si los puertos aparecen como "filtered" (es decir, no se recibe respuesta al enviar un paquete `SYN` a un puerto sin uso), lo más probable es que haya un firewall.

Además, si hay servicios inadecuados expuestos al exterior (como SMTP, IMAP o MySQL), esto sugiere que no hay un firewall o que está mal configurado.

#### Sistema de detección y prevención de intrusiones de red

Un sistema de detección de intrusiones de red (IDS) está diseñado para detectar actividad sospechosa o maliciosa a nivel de red, como el escaneo de puertos o de vulnerabilidades, y generar alertas. Un sistema de prevención de intrusiones (IPS) funciona de forma similar, pero también toma medidas para impedir la actividad, normalmente bloqueando la dirección IP de origen.

Por lo general, un IPS puede detectarse ejecutando herramientas automatizadas de escaneo (como un escáner de puertos) contra el objetivo y comprobando si se bloquea la IP de origen. Sin embargo, es posible que un IPS no detecte muchas herramientas a nivel de aplicación (especialmente si no descifra TLS).

#### Firewall de aplicaciones web (WAF)

Un firewall de aplicaciones web (WAF) inspecciona el contenido de las solicitudes HTTP y bloquea aquellas que parecen sospechosas o maliciosas. También puede utilizarse para aplicar dinámicamente otros controles, como CAPTCHA o la limitación de la tasa de solicitudes. Por lo general, utiliza un conjunto de firmas conocidas como maliciosas y expresiones regulares, como el [OWASP Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/), para identificar el tráfico malicioso. Los WAF pueden proteger eficazmente contra ciertos tipos de ataques, como la inyección SQL o el cross-site scripting, pero son menos eficaces contra otros, como los problemas relacionados con el control de acceso o la lógica de negocio.

Un WAF puede desplegarse en varios lugares, entre ellos:

- En el propio servidor web
- En una máquina virtual independiente o un dispositivo de hardware
- En la nube, delante del servidor de backend

Como un WAF bloquea las solicitudes maliciosas, es posible detectarlo añadiendo cadenas de ataque habituales a los parámetros y observando si se bloquean. Por ejemplo, intenta añadir un parámetro llamado `foo` con un valor como `' UNION SELECT 1` o `><script>alert(1)</script>`. Si se bloquean estas solicitudes, es probable que haya un WAF. Además, el contenido de las páginas de bloqueo puede proporcionar información sobre la tecnología específica utilizada. Por último, algunos WAF pueden añadir cookies o encabezados HTTP a las respuestas que revelen su presencia.

Si se utiliza un WAF basado en la nube, puede ser posible eludirlo accediendo directamente al servidor de backend mediante los mismos métodos descritos en la sección [Red de distribución de contenido](#content-delivery-network-cdn).