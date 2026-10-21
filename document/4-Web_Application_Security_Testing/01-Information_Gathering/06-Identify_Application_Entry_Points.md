# Identificar los puntos de entrada de la aplicación

|ID          |
|------------|
|WSTG-INFO-06|

## Resumen

Enumerar la aplicación y su superficie de ataque es un paso clave antes de realizar pruebas exhaustivas, ya que permite al evaluador identificar las áreas con mayor probabilidad de presentar debilidades. Esta sección tiene como objetivo ayudar a identificar y delimitar las áreas de la aplicación que deberían investigarse una vez completadas la enumeración y el mapeo.

## Objetivos de las pruebas

- Identificar posibles puntos de entrada e inyección mediante el análisis de solicitudes y respuestas.

## Cómo realizar las pruebas

Antes de comenzar cualquier prueba, el evaluador siempre debería comprender bien la aplicación y cómo se comunican con ella el usuario y el navegador. Al recorrer la aplicación, debería prestar atención a todas las solicitudes HTTP, así como a cada parámetro y campo de formulario que se envíe a la aplicación. Debería prestar especial atención a cuándo se usan solicitudes GET y cuándo se usan solicitudes POST para enviar parámetros a la aplicación. Además, también debe prestar atención a cuándo se usan otros métodos para los servicios RESTful.

Para ver los parámetros enviados en el cuerpo de solicitudes como las solicitudes POST, el evaluador puede usar una herramienta como un proxy de interceptación (consulta [herramientas](#tools)). Dentro de la solicitud POST, también debería prestar especial atención a los campos de formulario ocultos que se envían a la aplicación, ya que suelen contener información sensible, como información sobre el estado, la cantidad de artículos, el precio de los artículos, etc., que el desarrollador nunca tuvo la intención de que alguien viera o modificara.

El uso de un proxy de interceptación y de una aplicación para tomar notas (por ejemplo, un programa de hojas de cálculo) es habitual entre los evaluadores en esta fase de las pruebas. El proxy registrará todas las solicitudes y respuestas que intercambian el evaluador y la aplicación mientras la explora. Además, en esta etapa los evaluadores suelen interceptar todas las solicitudes y respuestas para poder ver exactamente cada encabezado, parámetro, etc., que se envía a la aplicación y lo que esta devuelve. Esto puede resultar bastante tedioso, especialmente en sitios interactivos grandes (por ejemplo, una aplicación bancaria). Sin embargo, la experiencia permite saber qué buscar y optimizar considerablemente esta fase.

Al recorrer la aplicación, el evaluador debería anotar los parámetros interesantes que encuentre en la URL, los encabezados personalizados o el cuerpo de las solicitudes y respuestas, y guardarlos en una hoja de cálculo. La hoja de cálculo debería incluir la página solicitada (también puede ser útil agregar el número de solicitud del proxy como referencia futura), los parámetros interesantes, el tipo de solicitud (GET, POST, etc.), si el acceso está autenticado o no, si se usa TLS, si forma parte de un proceso de varios pasos, si se usan WebSockets y cualquier otra nota pertinente. Una vez que se hayan mapeado todas las áreas de la aplicación, se podrá recorrerla y probar cada una de las áreas identificadas, y anotar qué funcionó y qué no. El resto de esta guía explica cómo probar cada una de estas áreas de interés, pero esta sección debe completarse antes de comenzar las pruebas propiamente dichas.

A continuación se indican algunos aspectos de interés para todas las solicitudes y respuestas. En la sección de solicitudes, céntrate en los métodos GET y POST, ya que constituyen la mayoría de las solicitudes. Ten en cuenta que también pueden aparecer otros métodos, como PUT y DELETE. A menudo, estas solicitudes menos frecuentes, si están permitidas, pueden exponer vulnerabilidades. Esta guía incluye una sección específica dedicada a probar estos métodos HTTP.

### Solicitudes

- Identifica dónde se usan solicitudes GET y dónde se usan solicitudes POST.
- Identifica todos los parámetros que se usan en una solicitud POST (se encuentran en el cuerpo de la solicitud).
- En la solicitud POST, presta especial atención a los parámetros ocultos. Cuando se envía una solicitud POST, todos los campos del formulario (incluidos los parámetros ocultos) se envían a la aplicación en el cuerpo del mensaje HTTP. Por lo general, no se ven a menos que se use un proxy o se consulte el código fuente HTML. Modificar estos parámetros ocultos puede cambiar las páginas que se cargan a continuación, los datos que contienen y el nivel de acceso concedido.
- Identifica todos los parámetros que se usan en una solicitud GET (es decir, en la URL), especialmente la cadena de consulta (que suele aparecer después de un signo ?).
- Identifica todos los parámetros de la cadena de consulta. Suelen presentarse en pares, como `foo=bar`. Ten en cuenta también que una cadena de consulta puede contener varios parámetros separados por `&`, `\~`, `:`, u otros caracteres especiales o codificaciones.
- Ten en cuenta que, al identificar varios parámetros en una cadena o en una solicitud POST, algunos o todos pueden ser necesarios para ejecutar los ataques. El evaluador debe identificar todos los parámetros (aunque estén codificados o cifrados) y determinar cuáles procesa la aplicación. En secciones posteriores de la guía se explica cómo probar estos parámetros. En esta etapa, es importante asegurarse de identificar cada uno de ellos.
- Presta también atención a los encabezados adicionales o personalizados que no suelen aparecer (como `debug: false`).

### Respuestas

- Identifica dónde se establecen (`Set-Cookie` header), modifican o agregan nuevas cookies.
- Identifica las redirecciones (código de estado HTTP 3xx), los códigos de estado 400 (en particular, 403 Forbidden) y los errores internos del servidor 500 que se produzcan durante respuestas normales (es decir, ante solicitudes sin modificar).
- Anota también dónde se usan encabezados interesantes. Por ejemplo, `Server: BIG-IP` indica que el sitio usa balanceo de carga. Por lo tanto, si el sitio tiene balanceo de carga y uno de los servidores está mal configurado, es posible que el evaluador deba realizar varias solicitudes para acceder al servidor vulnerable, según el tipo de balanceo de carga utilizado.

### OWASP Attack Surface Detector

La herramienta Attack Surface Detector (ASD) analiza el código fuente y descubre los endpoints de una aplicación web, los parámetros que aceptan y el tipo de datos de esos parámetros. Esto incluye endpoints sin enlaces que un rastreador no podría encontrar, así como parámetros opcionales que no se usan en absoluto en el código del lado del cliente. También puede calcular los cambios en la superficie de ataque entre dos versiones de una aplicación.

Attack Surface Detector está disponible como complemento para ZAP y Burp Suite, y también cuenta con una herramienta de línea de comandos. Esta herramienta exporta la superficie de ataque en formato JSON, que luego puede utilizarse con los complementos de ZAP y Burp Suite. Esto resulta útil cuando no se proporciona directamente el código fuente al evaluador de penetración. Por ejemplo, el evaluador de penetración puede recibir el archivo de salida JSON de un cliente que no desea proporcionar el código fuente en sí.

#### Cómo usarlo

El archivo jar de CLI se puede descargar desde [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases).

Puedes ejecutar el siguiente comando de ASD para identificar endpoints a partir del código fuente de la aplicación web objetivo.

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

Este es un ejemplo de ejecución del comando contra [OWASP RailsGoat](https://github.com/OWASP/railsgoat).

```text
$ java -jar attack-surface-detector-cli-1.3.5.jar railsgoat/
Beginning endpoint detection for '<...>/railsgoat' with 1 framework types
Using framework=RAILS
[0] GET: /login (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_contro
ller.rb (lines '6'-'9')
[1] GET: /logout (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[2] POST: /forgot_password (0 variants): PARAMETERS={email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/
password_resets_controller.rb (lines '29'-'38')
[3] GET: /password_resets (0 variants): PARAMETERS={token=name=token, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/p
assword_resets_controller.rb (lines '19'-'27')
[4] POST: /password_resets (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user=name=user, paramType=QUERY_STRING, dataType=STRING, confirm_password=name=confirm_password, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '5'-'17')
[5] GET: /sessions/new (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[6] POST: /sessions (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user_id=name=user_id, paramType=SESSION, dataType=STRING, remember_me=name=remember_me, paramType=QUERY_STRING, dataType=STRING, url=name=url, paramType=QUERY_STRING, dataType=STRING, email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '11'-'31')
[7] DELETE: /sessions/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[8] GET: /users (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '9'-'11')
[9] GET: /users/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '13'-'15')
... snipped ...
[38] GET: /api/v1/mobile/{id} (0 variants): PARAMETERS={id=name=id, paramType=QUERY_STRING, dataType=STRING, class=name=class, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/api/v1/mobile_controller.rb (lines '8'-'13')
[39] GET: / (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
Generated 40 distinct endpoints with 0 variants for a total of 40 endpoints
Successfully validated serialization for these endpoints
0 endpoints were missing code start line
0 endpoints were missing code end line
0 endpoints had the same code start and end line
Generated 36 distinct parameters
Generated 36 total parameters
- 36/36 have their data type
- 0/36 have a list of accepted values
- 36/36 have their parameter type
--- QUERY_STRING: 35
--- SESSION: 1
Finished endpoint detection for '<...>/railsgoat'
----------
-- DONE --
0 projects had duplicate endpoints
Generated 40 distinct endpoints
Generated 40 total endpoints
Generated 36 distinct parameters
Generated 36 total parameters
1/1 projects had endpoints generated
To enable logging include the -debug argument
```

También puedes generar un archivo de salida JSON con la opción `-json`, que pueden utilizar los complementos de ZAP y Burp Suite. Consulta los siguientes enlaces para obtener más detalles.

- [Página principal del complemento ASD para ZAP](https://github.com/secdec/attack-surface-detector-zap/wiki)
- [Página principal del complemento ASD para PortSwigger Burp](https://github.com/secdec/attack-surface-detector-burp/wiki)

### OWASP Noir

[OWASP Noir](https://owasp.org/projects/noir) es una herramienta de línea de comandos que encuentra los endpoints de una aplicación web mediante el análisis de su código fuente. Para cada endpoint, informa el método HTTP y los parámetros leídos de la cadena de consulta, el cuerpo de la solicitud, la ruta, los encabezados y las cookies. Esto incluye endpoints a los que no enlaza ninguna página y parámetros que el código del lado del cliente nunca envía, que un rastreador no detectaría. Noir admite numerosos lenguajes y marcos de trabajo, como Rails, Django, Express, Spring y Laravel. Consulta la lista completa de [lenguajes y marcos de trabajo compatibles](https://owasp-noir.github.io/noir/usage/supported/language_and_frameworks/).

Noir genera resultados en numerosos [formatos de salida](https://owasp-noir.github.io/noir/usage/output_formats/), incluidos texto sin formato, JSON, OpenAPI y comandos curl. También puede volver a enviar los endpoints descubiertos a través de un proxy de interceptación como ZAP o Burp Suite, para que aparezcan en el historial del proxy junto con el resto del tráfico capturado.

#### Cómo usarlo

Instala Noir siguiendo la [guía de instalación](https://owasp-noir.github.io/noir/get_started/installation/) y, a continuación, ejecuta el comando `scan` en el directorio raíz del código fuente:

`noir scan <source-code-path> [flags]`

Este es un ejemplo de ejecución del comando contra [OWASP RailsGoat](https://github.com/OWASP/railsgoat), la misma aplicación que se usó en el ejemplo anterior de Attack Surface Detector. Si no se usa `--no-log`, Noir también imprime un registro de progreso con las tecnologías detectadas (en este caso, `ruby_rails`) y el tiempo de ejecución.

```text
$ noir scan railsgoat --no-log
GET /api/v1/mobile/1?class=
  ○ path: id

GET /api/v1/mobile?class=

GET /api/v1/users

GET /api/v1/users/1

GET /users/1/benefit_forms

GET /users/1/messages

GET /users/1/messages/1
  ○ path: id

DELETE /users/1/messages/1
  ○ path: id

POST /users/1/messages
  ○ body: {"creator_id":"","message":"","read":"","receiver_id":""}

GET /users/1/paid_time_off

GET /users/1/pay

GET /users/1/pay/1

DELETE /users/1/pay/1
  ○ path: id

GET /users/1/performance

GET /users/1/retirement

POST /schedule
  ○ body: {"schedule":"","date_range1":"","date_begin":"","date_end":"","event_desc":"","event_name":"","event_type":""}

POST /sessions
  ○ body: 
    ├── url
    ├── email
    ├── password
    └── remember_me

... snipped ...

POST /admin/1/delete_user
  ○ body: 
    └── admin_id

PATCH /admin/1/update_user
  ○ body: 
    ├── admin_id
    └── user

GET /admin/1/get_all_users

GET /admin/1/analytics?field=&ip=

GET /dashboard/home?font=
  ○ cookies: 
    └── font

GET /dashboard/change_graph?graph=

GET /?url=

GET /404.html

GET /422.html

GET /500.html

GET /robots.txt
```

Noir identificó 56 endpoints. Cada entrada indica el método, la ruta y los parámetros agrupados por ubicación (`path`, `body`, `cookies`). Algunos son fáciles de pasar por alto al navegar, como la cookie `font` que lee `/dashboard/home`.

Cambia el formato de salida con `-f` para pasar los resultados a otras herramientas. El formato `curl` imprime una solicitud por endpoint, y `-u` establece la URL base:

```text
$ noir scan railsgoat --no-log -f curl -u http://localhost:3000
curl -i -X 'GET' 'http://localhost:3000/api/v1/mobile/1?class='
curl -i -X 'GET' 'http://localhost:3000/api/v1/mobile?class='
curl -i -X 'GET' 'http://localhost:3000/api/v1/users'
curl -i -X 'GET' 'http://localhost:3000/api/v1/users/1'
curl -i -X 'GET' 'http://localhost:3000/users/1/benefit_forms'
curl -i -X 'GET' 'http://localhost:3000/users/1/messages'
... snipped ...
```

Para enviar todos los endpoints a través de un proxy de interceptación y probarlos manualmente, agrega `--probe-via` con la dirección del proxy:

```bash
noir scan railsgoat -u http://localhost:3000 --probe-via http://127.0.0.1:8080
```

Noir también puede exportar un documento OpenAPI para importarlo en ZAP o Burp Suite. Consulta [Reconocimiento de API](../12-API_Testing/01-API_Reconnaissance.md#server-side-source-code) para ver un ejemplo.

### Puntos de entrada de la aplicación

A continuación se muestran dos ejemplos de cómo comprobar los puntos de entrada de una aplicación.

#### Solicitudes GET

Este ejemplo muestra una solicitud GET que compraría un artículo en una aplicación de compras en línea.

```http
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> Todos los parámetros de la solicitud, como CUSTOMERID, ITEM, PRICE, IP y la cookie, que podrían ser simplemente parámetros codificados o parámetros utilizados para el estado de la sesión.

#### Solicitudes POST

Este ejemplo muestra una solicitud POST que permitiría iniciar sesión en una aplicación.

```http
POST /example/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==;CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

Se puede observar que los parámetros se envían en varios lugares:

1. En la cadena de consulta: `service`
2. En el encabezado Cookie: `SESSIONID`, `CustomCookie`
3. En el cuerpo de la solicitud: `user`, `pass`, `debug`, `fromtrustIP`

La variedad de ubicaciones de inyección ofrece al atacante posibilidades de encadenamiento que podrían aumentar las probabilidades de encontrar un error en el código que las procesa.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
- [Burp Suite](https://www.portswigger.net/burp/)
- [Fiddler](https://www.telerik.com/fiddler)
- [OWASP Noir](https://github.com/owasp-noir/noir)
- Descubrimiento de contenido y parámetros: [ffuf](https://github.com/ffuf/ffuf), [feroxbuster](https://github.com/epi052/feroxbuster), [Arjun](https://github.com/s0md3v/Arjun)
- Rastreadores: [katana](https://github.com/projectdiscovery/katana), [hakrawler](https://github.com/hakluke/hakrawler)
- Fuentes de URL históricas: [gau](https://github.com/lc/gau), [waybackurls](https://github.com/tomnomnom/waybackurls), [waymore](https://github.com/xnl-h4ck3r/waymore)
- Consulta también las herramientas enumeradas en [Reconocimiento de API](../12-API_Testing/01-API_Reconnaissance.md) (Kiterunner, LinkFinder, jsluice, xnLinkFinder, Param Miner, etc.)

## Referencias

- [RFC 2616 - Protocolo de transferencia de hipertexto - HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [OWASP Attack Surface Detector](https://owasp.org/www-project-attack-surface-detector/)