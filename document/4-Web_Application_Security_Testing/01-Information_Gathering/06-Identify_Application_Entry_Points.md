# Identificar Puntos de Entrada de la Aplicación

|ID          |
|------------|
|WSTG-INFO-06|

## Resumen

Enumerar la aplicación y su superficie de ataque es un precursor clave antes de que cualquier prueba exhaustiva pueda ser emprendida, ya que permite al probador identificar áreas probables de debilidad. Esta sección apunta a ayudar a identificar y mapear áreas dentro de la aplicación que deberían ser investigadas una vez que la enumeración y mapeo hayan sido completados.

## Objetivos de Prueba

- Identificar posibles puntos de entrada e inyección a través de análisis de solicitud y respuesta.

## Cómo Probar

Antes de que cualquier prueba comience, el probador debería siempre obtener una buena comprensión de la aplicación y cómo el usuario y navegador se comunican con ella. Como el probador camina a través de la aplicación, deberían prestar atención a todas las solicitudes HTTP así como a cada parámetro y campo de formulario que se pasa a la aplicación. Deberían prestar atención especial a cuándo se usan solicitudes GET y cuándo se usan solicitudes POST para pasar parámetros a la aplicación. Además, también necesitan prestar atención a cuándo se usan otros métodos para servicios RESTful.

Nota que para ver los parámetros enviados en el cuerpo de solicitudes como una solicitud POST, el probador puede querer usar una herramienta como un proxy interceptante (ver [herramientas](#herramientas)). Dentro de la solicitud POST, el probador también debería hacer nota especial de cualquier campo de formulario oculto que se esté pasando a la aplicación, ya que estos usualmente contienen información sensible como información de estado, cantidad de items, el precio de items, etc., que el desarrollador nunca pretendió que nadie viera o cambiara.

El uso de un proxy interceptante y una aplicación de toma de notas (por ejemplo, software de hoja de cálculo) para esta etapa de prueba es popular entre probadores. El proxy mantendrá track de cada solicitud y respuesta entre el probador y la aplicación mientras la exploran. Adicionalmente, en este punto, los probadores usualmente atrapan cada solicitud y respuesta para que puedan ver exactamente cada header, parámetro, etc. que se está pasando a la aplicación y qué se está retornando. Esto puede ser bastante tedioso a veces, especialmente en sitios interactivos grandes (piensa en una aplicación bancaria). Sin embargo, la experiencia mostrará qué buscar y esta fase puede ser significativamente optimizada.

Como el probador camina a través de la aplicación, deberían tomar nota de cualquier parámetro interesante en la URL, headers custom, o cuerpo de las solicitudes/respuestas, y guardarlos en una hoja de cálculo. La hoja de cálculo debería incluir la página solicitada (podría ser bueno también agregar el número de solicitud del proxy, para referencia futura), los parámetros interesantes, el tipo de solicitud (GET, POST, etc.), si el acceso es autenticado/no autenticado, si se usa TLS, si es parte de un proceso multi-paso, si se usan WebSockets, y cualquier otra nota relevante. Una vez que tienen cada área de la aplicación mapeada, pueden entonces ir a través de la aplicación y probar cada una de las áreas que han identificado y hacer notas para qué funcionó y qué no funcionó. El resto de esta guía identificará cómo probar cada una de estas áreas de interés, pero esta sección debe ser emprendida antes de que cualquier prueba actual pueda comenzar.

Abajo están algunos puntos de interés para todas las solicitudes y respuestas. Dentro de la sección de solicitudes, enfócate en los métodos GET y POST ya que estos comprenden la mayoría de las solicitudes. Nota que otros métodos, como PUT y DELETE también pueden ser encontrados. A menudo, tales solicitudes más raras, si permitidas, pueden exponer vulnerabilidades. Hay una sección especial en esta guía dedicada para probar estos métodos HTTP.

### Solicitudes

- Identificar dónde se usan GETs y dónde se usan POSTs.
- Identificar todos los parámetros usados en una solicitud POST (estos están en el cuerpo de la solicitud).
- Dentro de la solicitud POST, prestar atención especial a cualquier parámetro oculto. Cuando un POST es enviado, todos los campos de formulario (incluyendo parámetros ocultos) serán enviados en el cuerpo del mensaje HTTP a la aplicación. Estos típicamente no se ven a menos que un proxy sea usado, o el código fuente HTML sea visto. Alterar estos parámetros ocultos puede resultar en cambios a las siguientes páginas que cargan, los datos que contienen, y el grado de acceso concedido.
- Identificar todos los parámetros usados en una solicitud GET (i.e., en la URL), particularmente la query string (usualmente apareciendo después de un ? mark).
- Identificar todos los parámetros de la query string. Estos usualmente están en un formato par, como `foo=bar`. También nota que muchos parámetros pueden estar en una query string separada por un `&`, `\~`, `:`, o cualquier otro carácter especial o encoding.
- Por favor nota, cuando identificando múltiples parámetros en una string o dentro de una solicitud POST, algunos o todos los parámetros serán requeridos para ejecutar los ataques. El probador necesita identificar todos los parámetros (incluso si están encoded o encrypted) e identificar cuáles son procesados por la aplicación. Secciones posteriores de la guía cubrirán cómo probar estos parámetros. En este punto, es importante asegurar que cada uno de ellos es identificado.
- También prestar atención a cualquier header adicional o custom no típicamente visto (como `debug: false`).

### Respuestas

- Identificar dónde nuevas cookies son establecidas (`Set-Cookie` header), modificadas, o agregadas.
- Identificar cualquier redirect (código de estado HTTP 3xx), códigos de estado 400 (particularmente 403 Forbidden), y errores internos del servidor 500 durante respuestas normales (i.e., solicitudes no modificadas).
- También nota dónde cualquier header interesante es usado. Por ejemplo, `Server: BIG-IP` indica que el sitio está load balanced. Así, si un sitio está load balanced y un servidor está incorrectamente configurado, entonces el probador podría tener que hacer múltiples solicitudes para acceder al servidor vulnerable, dependiendo del tipo de load balancing usado.

### OWASP Attack Surface Detector

La herramienta Attack Surface Detector (ASD) investiga el código fuente y descubre los endpoints de una aplicación web, los parámetros que estos endpoints aceptan, y el tipo de dato de esos parámetros. Esto incluye endpoints no enlazados que un spider no sería capaz de encontrar, así como parámetros opcionales que están completamente sin usar en el código del lado del cliente. También tiene la capacidad de calcular los cambios en la superficie de ataque entre dos versiones de una aplicación.

El Attack Surface Detector está disponible como un plugin para ZAP y Burp Suite, y una herramienta command-line también está disponible. La herramienta command-line exporta la superficie de ataque como una salida JSON, que puede entonces ser usada por el plugin de ZAP y Burp Suite. Esto es helpful para casos donde el código fuente no es proporcionado al penetration tester directamente. Por ejemplo, el penetration tester puede obtener el archivo de salida json de un cliente que no quiere proporcionar el código fuente en sí.

#### Cómo Usar

El archivo jar CLI está disponible para descarga desde [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases).

Puedes ejecutar el siguiente comando para ASD para identificar endpoints desde el código fuente de la aplicación web objetivo.

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

Aquí hay un ejemplo de ejecutar el comando contra [OWASP RailsGoat](https://github.com/OWASP/railsgoat).

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

Puedes también generar un archivo de salida JSON usando el flag `-json`, que puede ser usado por el plugin para ZAP y Burp Suite. Ver los siguientes links para más detalles.

- [Home of ASD Plugin for ZAP](https://github.com/secdec/attack-surface-detector-zap/wiki)
- [Home of ASD Plugin for PortSwigger Burp](https://github.com/secdec/attack-surface-detector-burp/wiki)

### Probando para Puntos de Entrada de Aplicación

Los siguientes son dos ejemplos en cómo verificar para puntos de entrada de aplicación.

#### Ejemplo 1

Este ejemplo muestra una solicitud GET que compraría un item de una aplicación de compras online.

```http
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> Todos los parámetros de la solicitud como CUSTOMERID, ITEM, PRICE, IP, y la Cookie, que podría justo ser parámetros encoded o parámetros usados para estado de sesión.

#### Ejemplo 2

Este ejemplo muestra una solicitud POST que te loguearía en una aplicación.

```http
POST /example/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==;CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

Puede ser notado que los parámetros son enviados en varias locaciones:

1. En la query string: `service`
2. En el header Cookie: `SESSIONID`, `CustomCookie`
3. En el cuerpo de la solicitud: `user`, `pass`, `debug`, `fromtrustIP`

Teniendo una variedad de locaciones de inyección proporciona al atacante con posibilidades de chaining que podrían mejorar las chances de encontrar un bug en el código de manejo.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
- [Burp Suite](https://www.portswigger.net/burp/)
- [Fiddler](https://www.telerik.com/fiddler)

## Referencias

- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [OWASP Attack Surface Detector](https://owasp.org/www-project-attack-surface-detector/)