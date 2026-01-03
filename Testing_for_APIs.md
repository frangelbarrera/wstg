# Pruebas de API

Las APIs Web han ganado mucha popularidad ya que permiten a programas de terceros interactuar con sitios de una manera más eficiente y fácil. En esta guía, discutiremos algunos conceptos básicos sobre APIs y la manera de probar seguridad para APIs.

## Conceptos de Fondo

REST (Representational State Transfer) es una arquitectura que es implementada mientras desarrolladores diseñan APIs.
Aplicaciones web APIs siguiendo el estilo REST son llamadas REST API.
REST APIs usan URIs (Uniform Resource Identifiers) para acceder recursos. La sintaxis URI genérica como definida en [RFC3986](https://tools.ietf.org/html/rfc3986) como abajo:

> URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]

Estamos interesados en el path de URI como la relación entre usuario y recursos.
Por ejemplo, `https://api.test.xyz/admin/testing/report`, esto muestra reporte de testing, hay relación entre usuario admin y sus reportes.

El path de cualquier URI definirá modelo de recurso REST API, recursos son separados por una slash forward y basados en diseño Top-Down.
Por ejemplo:

- `https://api.test.xyz/admin/testing/report`
- `https://api.test.xyz/admin/testing/`
- `https://api.test.xyz/admin/`

REST API requests siguen los [HTTP Request Methods](https://tools.ietf.org/html/rfc7231#section-4) definidos en [RFC7231](https://tools.ietf.org/html/rfc7231)

| Methods | Description                                   |
|---------|-----------------------------------------------|
| GET     | Get the representation of resource’s state    |
| POST    | Create a new resource                         |
| PUT     | Update a resource                             |
| DELETE  | Remove a resource                             |
| HEAD    | Get metadata associated with resource’s state |
| OPTIONS | List available methods                        |

REST APIs usan el código de estado de respuesta del mensaje de respuesta HTTP para notificar al cliente sobre el resultado de su request.

| Response Code | Response Message      | Description                                                                                            |
|---------------|-----------------------|--------------------------------------------------------------------------------------------------------|
| 200           | OK                    | Success while processing client's request                                                              |
| 201           | Created               | New resource created                                                                                   |
| 301           | Moved Permanently     | Permanent redirection                                                                                  |
| 304           | Not Modified          | Caching related response that returned when the client has the same copy of the resource as the server |
| 307           | Temporary Redirect    | Temporary redirection of resource                                                                      |
| 400           | Bad Request           | Malformed request by the client                                                                        |
| 401           | Unauthorized          | Client is not allowed to make requests or access a particular resource                                 |
| 402           | Forbidden             | Client is forbidden to access the resource                                                             |
| 404           | Not Found             | Resource doesn't exist or incorrect based on the request                                               |
| 405           | Method Not Allowed    | Invalid method or unknown method used                                                                  |
| 500           | Internal Server Error | Server failed to process request due to an internal error                                              |

Headers HTTP son usados en requests y responses.
Mientras haciendo requests API, header Content-Type es usado y es set a `application/json` porque el body del mensaje contiene formato de datos JSON.

Tipos de autenticación web están basados en:

- Bearer Tokens: Identificado por el header `Authorization: Bearer <token>`. Una vez un usuario logs in, son proporcionados con un bearer token que es enviado en cada request para autenticar y autorizar al usuario a acceder recursos protegidos OAuth 2.0.
- HTTP Cookies: Identificado por el header `Cookie: <name>=<unique value>`. En éxito de login de usuario, el servidor replies con un header `Set-Cookie` especificando su nombre y valor único. En cada request, el browser automáticamente lo appends a los requests yendo a ese servidor, siguiendo [SOP](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy).
- Basic HTTP authentication: Identificado por el header `Authorization: Basic <base64 value>`. Una vez un usuario está tratando de login, el request es enviado con el header mencionado conteniendo un valor base64, teniendo su contenido como `username:password`. Esta es una de las formas más débiles de autenticación ya que transmite el username y password en cada request en una manera encoded, que puede ser fácilmente retrieved.

## Cómo Probar

### Método de Prueba Genérico

Paso 1: List endpoint y make different request method: Login con perfil de usuario y usa una herramienta spider para listar los endpoints de este rol.
Para examinar los endpoints, necesitarás hacer diferentes métodos de request y observar cómo la API se comporta.

Paso 2: Exploit bugs - Como sabes cómo listar endpoints y examinar endpoints con métodos HTTP en paso 1, encontraremos alguna manera de exploit bug. Algunas estrategias de testing están abajo:

- IDOR testing
- Privilege escalation

### Prueba Específica – (Token-Based) Authentication

Token-based authentication es implementada enviando un token signed (verificado por el servidor) con cada HTTP request.

El formato de token más comúnmente usado es el JSON Web Token (JWT), definido en [RFC7519](https://tools.ietf.org/html/rfc7519). La guía [Testing JSON Web Tokens](/document/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md) contiene detalles adicionales sobre cómo testear JWTs.

## Casos de Prueba Relacionados

- [IDOR](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)
- [Privilege escalation](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/03-Testing_for_Privilege_Escalation.md)
- All [Session Management](https://github.com/OWASP/wstg/tree/master/document/4-Web_Application_Security_Testing/06-Session_Management_Testing) test cases
- [Testing JSON Web Tokens](/document/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## Herramientas

- ZAP
- Burp suite

## Referencias

- [REST HTTP Methods](https://restfulapi.net/http-methods/)
- [RFC3986 URI](https://tools.ietf.org/html/rfc3986)
- [JWT](https://jwt.io/)
- [Cracking JWT](https://www.sjoerdlangkemper.nl/2016/09/28/attacking-jwt-authentication/)