# Mapear Arquitectura de la Aplicación

|ID          |
|------------|
|WSTG-INFO-10|

## Resumen

Para probar efectivamente una aplicación, y poder proporcionar recomendaciones significativas sobre cómo abordar cualquiera de los problemas identificados, es importante entender qué se está probando realmente. Adicionalmente, podría ser helpful determinar si componentes específicos deberían considerarse fuera de alcance para pruebas.

Las aplicaciones web modernas pueden variar significativamente en complejidad, desde un script simple ejecutándose en un solo servidor hasta una aplicación altamente compleja distribuida a través de docenas de diferentes sistemas, lenguajes y componentes. También puede haber componentes adicionales a nivel de red como firewalls o sistemas de protección de intrusiones que pueden tener un impacto significativo en las pruebas.

## Objetivos de Prueba

- Entender la arquitectura de la aplicación y las tecnologías en uso.

## Cómo Probar

Cuando se prueba desde una perspectiva black box, es importante intentar construir una imagen clara de cómo funciona la aplicación, y cuáles tecnologías y componentes están en lugar. En algunos casos, es posible probar componentes específicos como un firewall de aplicación web, mientras otros componentes pueden identificarse inspeccionando el comportamiento de la aplicación.

Las secciones abajo proporcionan una visión general de alto nivel de componentes arquitectónicos comunes, junto con detalles sobre cómo pueden identificarse.

### Componentes de Aplicación

#### Servidor Web

Aplicaciones simples pueden ejecutarse en un solo servidor, que puede identificarse usando los pasos discutidos en la sección [Huella Digital del Servidor Web](02-Fingerprint_Web_Server-es.md) de la guía.

#### Platform-as-a-Service (PaaS)

En un modelo Platform-as-a-Service (PaaS), el servidor web y la infraestructura subyacente son manejados por el proveedor de servicio, y el cliente solo es responsable de la aplicación que se despliega en ellos. Desde una perspectiva de pruebas, hay dos diferencias principales:

- El propietario de la aplicación no tiene acceso a la infraestructura subyacente, lo que significa que no podrán remediate directamente cualquier problema
- Las pruebas de infraestructura son likely a estar fuera de alcance para cualquier compromiso

En algunos casos, es posible identificar el uso de PaaS, ya que la aplicación puede usar un nombre de dominio específico (por ejemplo, aplicaciones desplegadas en Azure App Services tendrán un dominio `*.azurewebsites.net` - aunque también pueden usar dominios custom). En otros casos, es difícil determinar si PaaS está en uso.

#### Serverless

En un modelo Serverless, los desarrolladores proporcionan código que se ejecuta directamente en una plataforma de hosting como funciones individuales, en lugar de ejecutar una aplicación web tradicional más grande desplegada en un webroot. Esto lo hace bien suited para arquitectura basada en microservicios. Como con un entorno PaaS, las pruebas de infraestructura son likely a estar fuera de alcance.

En algunos casos, el uso de código Serverless puede indicarse por la presencia de headers HTTP específicos. Por ejemplo, funciones AWS Lambda típicamente retornan los siguientes headers:

```http
X-Amz-Invocation-Type
X-Amz-Log-Type
X-Amz-Client-Context
```

Azure Functions son menos obvias. Típicamente retornan el header `Server: Kestrel` - pero esto por sí solo no es suficiente para determinar que es una función Azure App, ya que podría ser algún otro código ejecutándose en Kestrel.

#### Microservicios

En una arquitectura basada en microservicios, la API de la aplicación está hecha de múltiples servicios discretos, en lugar de ejecutarse como una aplicación monolítica. Los servicios en sí a menudo ejecutan dentro de contenedores (usualmente con Kubernetes), y pueden usar una variedad de diferentes sistemas operativos y lenguajes. Aunque típicamente están detrás de una sola puerta de enlace API y dominio, el uso de múltiples lenguajes (a menudo indicado en mensajes de error detallados) puede sugerir que microservicios están en uso.

#### Almacenamiento Estático

Muchas aplicaciones almacenan contenido estático en plataformas de almacenamiento dedicadas, en lugar de alojarlo directamente en el servidor web principal. Las dos plataformas más comunes son Amazon's S3 Buckets, y Azure's Storage Accounts, y pueden identificarse fácilmente por los nombres de dominio:

- `BUCKET.s3.amazonaws.com` o `s3.REGION.amazonaws.com/BUCKET` para Amazon S3 Buckets
- `ACCOUNT.blob.core.windows.net` para Azure Storage Accounts

Estas cuentas de almacenamiento a menudo pueden exponer archivos sensibles, como discutido en la sección [Testing Cloud Storage Guide](../02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage.md).

#### Base de Datos

La mayoría de aplicaciones web no triviales usan algún tipo de base de datos para almacenar contenido dinámico. En algunos casos, es posible determinar la base de datos. Esto a menudo puede hacerse por:

- Escaneando puertos del servidor y buscando cualquier puerto abierto asociado con bases de datos específicas
- Triggering mensajes de error relacionados con SQL (o NoSQL) (o encontrando errores existentes de un [motor de búsqueda](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage-es.md)

Cuando no es posible determinar concluyentemente la base de datos, el probador a menudo puede hacer una suposición educada basada en otros aspectos de la aplicación:

- Windows, IIS y ASP.NET a menudo usan Microsoft SQL server
- Sistemas embebidos a menudo usan SQLite
- PHP a menudo usa MySQL o PostgreSQL
- APEX a menudo usa Oracle

Estas no son reglas duras, pero ciertamente pueden darte un punto de partida razonable si no hay mejor información disponible.

#### Autenticación

La mayoría de aplicaciones tienen autenticación de usuario. Hay múltiples backends de autenticación que pueden usarse, como:

- Configuración de servidor web (incluyendo archivos `.htaccess`) o hard-coding passwords en scripts
    - Usualmente se muestra como autenticación HTTP Basic, indicada por un pop-up en el navegador y un header HTTP `WWW-Authenticate: Basic`
- Cuentas de usuario locales en una base de datos
    - Usualmente integrado en un formulario o endpoint API en la aplicación
- Una fuente de autenticación central existente como Active Directory o un servidor LDAP
    - Puede usar autenticación NTLM, indicada por un header HTTP `WWW-Authenticate: NTLM`
    - Puede estar integrado en la aplicación web en un formulario
    - Puede requerir que el username se ingrese en el formato "DOMAIN\username", o puede dar un dropdown de dominios disponibles
- Single Sign-On (SSO) con ya sea un proveedor interno o externo
    - Típicamente usa OAuth, OpenID Connect, o SAML

Las aplicaciones pueden proporcionar múltiples opciones para que el usuario se autentique (como registrando una cuenta local, o usando su cuenta existente de Facebook), y pueden usar diferentes mecanismos para usuarios normales y administradores.

#### Servicios y APIs de Terceros

Casi todas las aplicaciones web incluyen recursos de terceros que se cargan o con los que el cliente interactúa. Estos pueden incluir:

- [Contenido activo](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content) (como scripts, hojas de estilo, fonts, e iframes)
- [Contenido pasivo](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content) (como imágenes y videos)
- APIs externas
- Botones de redes sociales
- Redes de publicidad
- Puertas de pago

Estos recursos son solicitados directamente por el navegador del usuario, haciéndolos más fáciles de identificar usando las herramientas de desarrollador, o un proxy interceptante. Mientras es importante identificarlos (ya que pueden impactar la seguridad de la aplicación), recuerda que *usualmente están fuera de alcance para pruebas*, ya que pertenecen a terceros.

### Componentes de Red

#### Proxy Inverso

Un proxy inverso se sienta enfrente de uno o más servidores backend y redirige solicitudes al destino apropiado. Pueden usarse para implementar varias funcionalidades, como:

- Actuando como un [load balancer](#load-balancer) o [web application firewall](#web-application-firewall-waf)
- Permitiendo múltiples aplicaciones ser alojadas en una sola dirección IP o dominio (en subcarpetas)
- Implementando filtrado IP u otras restricciones
- Caching contenido del backend para mejorar rendimiento

No siempre es posible detectar un proxy inverso (especialmente si solo hay una sola aplicación detrás de él), pero a veces puedes identificarlo por:

- Un mismatch entre el servidor frontend y la aplicación backend (como un header `Server: nginx` con una aplicación ASP.NET)
    - Esto a veces puede llevar a [vulnerabilidades de request smuggling](https://portswigger.net/web-security/request-smuggling)
- Headers duplicados (especialmente el header `Server`)
- Múltiples aplicaciones alojadas en la misma dirección IP o dominio (especialmente si usan diferentes lenguajes)

#### Load Balancer

Un load balancer se sienta enfrente de múltiples servidores backend y asigna solicitudes entre ellos para proporcionar mayor redundancia y capacidad de procesamiento para la aplicación.

Los load balancers pueden ser difíciles de detectar, pero a veces pueden identificarse haciendo múltiples solicitudes y examinando las respuestas por diferencias, como:

- Tiempos de sistema inconsistentes
- Diferentes direcciones IP internas o hostnames en mensajes de error detallados
- Diferentes direcciones retornadas de [Server-Side Request Forgery (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md)

También pueden indicarse por la presencia de cookies específicas (por ejemplo, load balancers F5 BIG-IP crearán una cookie llamada `BIGipServer`.

#### Content Delivery Network (CDN)

Una Content Delivery Network (CDN) es un conjunto geográficamente distribuido de servidores proxy de caching diseñados para mejorar el rendimiento del sitio.

Típicamente se configura apuntando el dominio público-facing a los servidores de la CDN, y luego configurando la CDN para conectarse a los servidores backend correctos (a veces conocidos como el "origin").

La manera más fácil de detectar una CDN es realizar una búsqueda WHOIS para las direcciones IP que el dominio resuelve. Si pertenecen a una compañía CDN (como Akamai, Cloudflare o Fastly - ver [Wikipedia](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers) para una lista más completa), entonces es likely que una CDN esté en uso.

Cuando pruebas un sitio detrás de una CDN, deberías tener en mente los siguientes puntos:

- Las direcciones IP y servidores pertenecen al proveedor CDN, y son likely a estar fuera de alcance para pruebas de infraestructura
- Muchas CDNs también incluyen features como detección de bots, rate limiting, y web application firewalls
- Las CDNs usualmente cachean contenido. Por lo tanto, cambios hechos en el backend pueden no aparecer inmediatamente en el sitio.

Si el sitio está detrás de una CDN, podría ser útil identificar los servidores backend. Si el control de acceso apropiado no se enforce, el probador puede ser capaz de bypass la CDN (y cualquier protección que ofrezca) accediendo directamente a los servidores backend. Hay una variedad de diferentes métodos que pueden permitir identificar el sistema backend:

- Emails enviados por la aplicación pueden venir direct del servidor backend, que podría revelar su dirección IP
- DNS grinding, zone transfers o listas de transparencia de certificados para un dominio pueden revelarlo en un subdomain
- Escaneando los rangos IP conocidos por ser usados por la compañía puede ayudar a identificar el servidor backend
- Exploiting [Server-Side Request Forgery (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md) puede revelar la dirección IP
- Mensajes de error detallados de la aplicación pueden exponer direcciones IP o hostnames

### Componentes de Seguridad

#### Firewall de Red

La mayoría de servidores web estarán protegidos por un firewall de filtrado de paquetes o inspección stateful, que bloquea cualquier tráfico de red que no sea requerido. Para detectarlo, realiza un port scan del servidor y examina los resultados.

Si la mayoría de los puertos se muestran como "closed" (i.e, retornan un paquete `RST` en respuesta al paquete inicial `SYN`), esto sugiere que el servidor puede no estar protegido por un firewall. Si los puertos se muestran como "filtered" (i.e, no se recibe respuesta cuando enviando un paquete `SYN` a un puerto no usado), entonces un firewall es most likely a estar en lugar.

Adicionalmente, si servicios inapropiados están expuestos al mundo (como SMTP, IMAP, MySQL, etc), esto sugiere que ya sea no hay firewall en lugar, o que el firewall está mal configurado.

#### Sistema de Detección y Prevención de Intrusiones de Red

Un Sistema de Detección de Intrusiones de Red (IDS) está diseñado para detectar actividad sospechosa o maliciosa a nivel de red, como scanning de puertos o vulnerabilidades, y levantar alertas. Un Sistema de Prevención de Intrusiones (IPS) funciona similarmente, pero también toma acción para prevenir la actividad, usualmente bloqueando la dirección IP fuente.

Un IPS usualmente puede detectarse ejecutando herramientas de scanning automatizadas (como un port scanner) contra el target, y viendo si la IP fuente es bloqueada. Sin embargo, muchas herramientas a nivel de aplicación pueden no ser detectadas por un IPS (especialmente si no desencripta TLS).

#### Web Application Firewall (WAF)

Un Web Application Firewall (WAF) inspecciona el contenido de solicitudes HTTP y bloquea aquellas que aparecen sospechosas o maliciosas. También pueden usarse para aplicar dinámicamente otros controles como CAPTCHA o rate limiting. Usualmente utilizan un conjunto de firmas conocidas malas y expresiones regulares, como el [OWASP Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/), para identificar tráfico malicioso. Los WAFs pueden ser efectivos protegiendo contra ciertos tipos de ataques como SQL injection o cross-site scripting, pero son menos efectivos contra otros tipos como control de acceso o problemas relacionados con lógica de negocio.

Un WAF puede desplegarse en múltiples locaciones, incluyendo:

- En el servidor web en sí
- En una máquina virtual separada o appliance de hardware
- En la cloud, enfrente del servidor backend

Porque un WAF bloquea solicitudes maliciosas, puede detectarse agregando strings de ataque comunes a parámetros y observando si son bloqueados o no. Por ejemplo, intenta agregar un parámetro llamado `foo` con un valor como `' UNION SELECT 1` o `><script>alert(1)</script>`. Si estas solicitudes son bloqueadas, es likely que haya un WAF en lugar. Adicionalmente, el contenido de las páginas de bloqueo puede proporcionar información sobre la tecnología específica que está en uso. Finalmente, algunos WAFs pueden agregar cookies o headers HTTP a respuestas que pueden revelar su presencia.

Si un WAF basado en cloud está en uso, entonces puede ser posible bypassarlo accediendo directamente al servidor backend, usando los mismos métodos discutidos en la sección [Content Delivery Network](#content-delivery-network-cdn).