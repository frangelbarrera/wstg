# Debilidades de OAuth

|ID          |
|------------|
|WSTG-ATHZ-05|

## Resumen

[OAuth2.0](https://oauth.net/2/) (en adelante, OAuth) es un marco de autorización que permite a un cliente acceder a recursos en nombre de su usuario.

Para lograrlo, OAuth depende en gran medida de tokens para la comunicación entre las diferentes entidades, cada una con un [rol](https://datatracker.ietf.org/doc/html/rfc6749#section-1.1) distinto:

- **Propietario del recurso (Resource Owner):** La entidad que otorga acceso a un recurso y es su propietaria; en la mayoría de los casos, es el propio usuario.
- **Cliente:** La aplicación que solicita acceso a un recurso en nombre del propietario del recurso. Hay dos [tipos](https://oauth.net/2/client-types/) de clientes:
    - **Públicos:** clientes que no pueden proteger un secreto (*por ejemplo*, aplicaciones centradas en el frontend, como las SPA y las aplicaciones móviles)
    - **Confidenciales:** clientes capaces de autenticarse de forma segura con el servidor de autorización, manteniendo a salvo los secretos que tienen registrados (*por ejemplo*, servicios de backend)
- **Servidor de autorización:** El servidor que contiene la información de autorización y concede el acceso
- **Servidor de recursos:** La aplicación que sirve el contenido al que accede el cliente

Como la responsabilidad de OAuth es delegar al cliente los derechos de acceso del propietario, es un objetivo muy atractivo para los atacantes; las implementaciones deficientes pueden permitir el acceso no autorizado a los recursos y la información de los usuarios.

Para proporcionar acceso a una aplicación cliente, OAuth se basa en varios [tipos de concesión de autorización](https://oauth.net/2/grant-types/) para generar un token de acceso:

- [Código de autorización](https://oauth.net/2/grant-types/authorization-code/): lo usan tanto los clientes confidenciales como los públicos para intercambiar un código de autorización por un token de acceso, pero se recomienda solo para clientes confidenciales
- [Proof Key for Code Exchange (PKCE)](https://oauth.net/2/pkce/): PKCE se basa en la concesión de código de autorización y la refuerza para que los clientes públicos puedan usarla de forma más segura; también mejora la seguridad de los clientes confidenciales
- [Credenciales de cliente (Client Credentials)](https://oauth.net/2/grant-types/client-credentials/): se usa para la comunicación de máquina a máquina, donde el «usuario» es la máquina que solicita acceso a sus propios recursos desde el servidor de recursos
- [Código de dispositivo (Device Code)](https://oauth.net/2/grant-types/device-code/): se usa en dispositivos con capacidades de entrada limitadas.
- [Token de actualización (Refresh Token)](https://oauth.net/2/grant-types/refresh-token/): tokens que proporciona el servidor de autorización para permitir que los clientes actualicen los tokens de acceso de los usuarios cuando estos dejan de ser válidos o caducan. Este tipo de concesión se usa junto con otro tipo de concesión.

Con la publicación de [OAuth2.1](https://oauth.net/2.1/), se dejarán de utilizar dos flujos, cuyo uso no se recomienda:

- [Flujo implícito*](https://oauth.net/2/grant-types/implicit/): La implementación segura de PKCE vuelve obsoleto este flujo. Antes de PKCE, las aplicaciones del lado del cliente, como las [aplicaciones de página única](https://en.wikipedia.org/wiki/Single-page_application), usaban el flujo implícito, ya que [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) flexibilizaba la [política del mismo origen](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) para permitir la comunicación entre sitios. Para obtener más información sobre por qué no se recomienda la concesión implícita, consulta esta [sección](https://datatracker.ietf.org/doc/html/rfc9700#section-2.1.2).
- [Credenciales de contraseña del propietario del recurso](https://oauth.net/2/grant-types/password/): se usa para intercambiar las credenciales de los usuarios directamente con el cliente, que luego las envía al servidor de autorización para intercambiarlas por un token de acceso. Para obtener información sobre por qué no se recomienda este flujo, consulta esta [sección](https://datatracker.ietf.org/doc/html/rfc9700#section-2.4).

*: El flujo implícito solo está obsoleto en OAuth; sin embargo, sigue siendo una solución viable en OpenID Connect (OIDC) para obtener `id_tokens`. Asegúrate de comprender cómo se usa el flujo implícito. Puedes identificarlo si solo se usa el endpoint `/authorization` para obtener un token de acceso, sin recurrir de ninguna manera al endpoint `/token`. Puedes encontrar un ejemplo [aquí](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post).

*Ten en cuenta que los flujos de OAuth son un tema complejo y que lo anterior solo resume los aspectos clave. Las referencias incluidas en el texto contienen más información sobre los flujos específicos.*

## Objetivos de prueba

- Determinar si la implementación de OAuth2 es vulnerable o utiliza una implementación obsoleta o personalizada.

## Cómo probar

### Tipos de concesión obsoletos

Los tipos de concesión obsoletos dejaron de usarse por motivos de seguridad y funcionalidad. Identificar si se utilizan permite comprobar rápidamente si son susceptibles a alguna de las amenazas relacionadas con su uso. Algunos aspectos podrían quedar fuera del alcance del atacante, como la forma en que un cliente utiliza las credenciales de los usuarios. Esto debe documentarse y comunicarse a los equipos internos de ingeniería.

En el caso de los clientes públicos, generalmente es posible identificar el tipo de concesión en la solicitud al endpoint `/token`. Se indica en el intercambio del token mediante el parámetro `grant_type`.

El siguiente ejemplo muestra la concesión de código de autorización con PKCE.

```http
POST /oauth/token HTTP/1.1
Host: as.example.com
[...]

{
  "client_id":"example-client",
  "code_verifier":"example",
  "grant_type":"authorization_code",
  "code":"example",
  "redirect_uri":"https://client.example.com"
}
```

Los valores del parámetro `grant_type` y los tipos de concesión que indican son:

- `password`: indica la concesión ROPC.
- `client_credentials`: indica la concesión de credenciales de cliente.
- `authorization_code`: indica la concesión de código de autorización.

El tipo de flujo implícito no se indica mediante el parámetro `grant_type`, ya que el token se incluye en la respuesta a la solicitud del endpoint `/authorization`; en su lugar, puede identificarse mediante `response_type`. A continuación se muestra un ejemplo.

```http
GET /authorize
  ?client_id=<some_client_id>
  &response_type=token 
  &redirect_uri=https%3A%2F%2Fclient.example.com%2F
  &scope=openid%20profile%20email
  &state=<random_state>
```

Los siguientes parámetros de URL indican el flujo de OAuth que se utiliza:

- `response_type=token`: indica el flujo implícito, ya que el cliente solicita directamente al servidor de autorización que devuelva un token.
- `response_type=code`: indica el flujo de código de autorización, ya que el cliente solicita al servidor de autorización que devuelva un código, que luego se intercambiará por un token.
- `code_challenge=sha256(xyz)`: indica la extensión PKCE, ya que ningún otro flujo utiliza este parámetro.

A continuación se muestra un ejemplo de solicitud de autorización para el flujo de código de autorización con PKCE:

```http
GET /authorize
    ?redirect_uri=https%3A%2F%2Fclient.example.com%2F
    &client_id=<some_client_id>
    &scope=openid%20profile%20email
    &response_type=code
    &response_mode=query
    &state=<random_state>
    &nonce=<random_nonce>
    &code_challenge=<random_code_challenge>
    &code_challenge_method=S256 HTTP/1.1
Host: as.example.com
[...]
```

#### Clientes públicos

La concesión de código de autorización con la extensión PKCE se recomienda para los clientes públicos. Una solicitud de autorización para el flujo de código de autorización con PKCE debe incluir `response_type=code` y `code_challenge=sha256(xyz)`.

El intercambio del token debe incluir el tipo de concesión `authorization_code` y un `code_verifier`.

Los tipos de concesión inadecuados para clientes públicos son:

- Concesión de código de autorización sin la extensión PKCE
- Credenciales de cliente
- Flujo implícito
- ROPC

#### Clientes confidenciales

La concesión de código de autorización se recomienda para los clientes confidenciales. También se puede usar la extensión PKCE.

Los tipos de concesión inadecuados para clientes confidenciales son:

- Credenciales de cliente (excepto para máquina a máquina; consulta más abajo)
- Flujo implícito
- ROPC

##### Máquina a máquina

En situaciones en las que no hay interacción con el usuario y los clientes son únicamente clientes confidenciales, se puede usar la concesión de credenciales de cliente.

Si conoces `client_id` y `client_secret`, es posible obtener un token indicando el tipo de concesión `client_credentials`.

```bash
$ curl --request POST \
  --url https://as.example.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<some_client_id>","client_secret":"<some_client_secret>","grant_type":"client_credentials"}' --proxy https://localhost:8080/ -k
```

### Fuga de credenciales

Según el flujo, OAuth transmite varios tipos de credenciales como parámetros de URL.

Los siguientes tokens pueden considerarse credenciales filtradas:

- token de acceso
- token de actualización
- código de autorización
- PKCE code challenge / code verifier

Debido al funcionamiento de OAuth, tanto el código de autorización `code` como `code_challenge` y `code_verifier` pueden formar parte de la URL. El flujo implícito transmite el token de autorización como parte de la URL si `response_mode` no está configurado como [`form_post`](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html). Esto puede provocar que el token o el código solicitados se filtren en el encabezado de referencia, en archivos de registro y en proxies, ya que estos parámetros se envían en la consulta o en el fragmento.

El riesgo de que el flujo implícito filtre los tokens es mucho mayor que el de filtrar `code` o cualquier otro parámetro `code_*`, ya que estos están vinculados a clientes específicos y son más difíciles de aprovechar en caso de filtración.

Para probar este escenario, usa un proxy de interceptación HTTP, como ZAP, e intercepta el tráfico de OAuth.

- Recorre el proceso de autorización e identifica las credenciales presentes en la URL.
- Si una página que participa en el flujo de OAuth incluye recursos externos, analiza las solicitudes que se les hacen. Las credenciales podrían filtrarse en el encabezado de referencia.

Después de recorrer el flujo de OAuth y usar la aplicación, el historial de solicitudes de un proxy de interceptación HTTP capturará algunas solicitudes. Busca en el historial de solicitudes el encabezado de referencia HTTP (por ejemplo, `Referer: https://idp.example.com/`) que contiene la URL del servidor de autorización y del cliente.

Revisar las etiquetas meta HTML (aunque esta etiqueta no está [admitida](https://caniuse.com/mdn-html_elements_meta_name_referrer) en todos los navegadores) o la [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy) podría ayudar a determinar si se están filtrando credenciales mediante el encabezado de referencia.

## Casos de prueba relacionados

- [Pruebas de JSON Web Tokens](../06-Session_Management/10-JSON_Web_Tokens.md)

## Remediación

- Al implementar OAuth, ten siempre en cuenta la tecnología utilizada y si la aplicación es del lado del servidor, capaz de evitar revelar secretos, o del lado del cliente, que no puede hacerlo.
- En casi todos los casos, usa el flujo de código de autorización con PKCE. Los flujos de máquina a máquina pueden ser una excepción.
- Usa parámetros POST o valores de encabezado para transmitir secretos.
- Cuando no haya otras posibilidades (por ejemplo, en aplicaciones heredadas que no se pueden migrar), implementa encabezados de seguridad adicionales, como `Referrer-Policy`.

## Herramientas

- [BurpSuite](https://portswigger.net/burp/releases)
- [EsPReSSO](https://github.com/portswigger/espresso)
- [ZAP](https://www.zaproxy.org/)

## Referencias

- [User Authentication with OAuth 2.0](https://oauth.net/articles/authentication/)
- [The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://datatracker.ietf.org/doc/html/rfc6750)
- [OAuth 2.0 Threat Model and Security Considerations](https://datatracker.ietf.org/doc/html/rfc6819)
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/rfc9700)
- [Authorization Code Flow with Proof Key for Code Exchange](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)