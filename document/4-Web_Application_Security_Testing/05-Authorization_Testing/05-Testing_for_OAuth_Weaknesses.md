# Pruebas de Debilidades de OAuth

|ID          |
|------------|
|WSTG-ATHZ-05|

## Resumen

[OAuth2.0](https://oauth.net/2/) (en adelante denominado OAuth) es un framework de autorización que permite a un cliente acceder a recursos en nombre de su usuario.

Para lograr esto, OAuth depende en gran medida de tokens para comunicarse entre las diferentes entidades, cada entidad teniendo un [rol](https://datatracker.ietf.org/doc/html/rfc6749#section-1.1) diferente:

- **Propietario del Recurso (Resource Owner):** La entidad que otorga acceso a un recurso, el propietario, y en la mayoría de los casos es el usuario mismo
- **Cliente:** La aplicación que está solicitando acceso a un recurso en nombre del Propietario del Recurso. Estos clientes vienen en dos [tipos](https://oauth.net/2/client-types/):
    - **Público:** clientes que no pueden proteger un secreto (*por ejemplo* aplicaciones enfocadas en frontend, tales como SPAs, aplicaciones móviles, etc.)
    - **Confidencial:** clientes que son capaces de autenticarse de manera segura con el servidor de autorización manteniendo sus secretos registrados a salvo (*por ejemplo* servicios backend)
- **Servidor de Autorización:** El servidor que tiene información de autorización y otorga el acceso
- **Servidor de Recursos:** La aplicación que sirve el contenido accedido por el cliente

Dado que la responsabilidad de OAuth es delegar derechos de acceso del propietario al cliente, este es un objetivo muy atractivo para atacantes, y las malas implementaciones llevan a acceso no autorizado a los recursos e información de los usuarios.

Para proporcionar acceso a una aplicación cliente, OAuth depende de varios [tipos de grant de autorización](https://oauth.net/2/grant-types/) para generar un token de acceso:

- [Código de Autorización (Authorization Code)](https://oauth.net/2/grant-types/authorization-code/): usado por clientes tanto confidenciales como públicos para intercambiar un código de autorización por un token de acceso, pero recomendado solo para clientes confidenciales
- [Proof Key for Code Exchange (PKCE)](https://oauth.net/2/pkce/): PKCE se construye sobre el grant de Código de Autorización, proporcionando seguridad más fuerte para que sea usado por clientes públicos, y mejorando la postura de los confidenciales
- [Credenciales de Cliente (Client Credentials)](https://oauth.net/2/grant-types/client-credentials/): usado para comunicación máquina a máquina, donde el "usuario" aquí es la máquina solicitando acceso a sus propios recursos desde el Servidor de Recursos
- [Código de Dispositivo (Device Code)](https://oauth.net/2/grant-types/device-code/): usado para dispositivos con capacidades de entrada limitadas.
- [Token de Refresco (Refresh Token)](https://oauth.net/2/grant-types/refresh-token/): tokens proporcionados por el servidor de autorización para permitir a los clientes refrescar los tokens de acceso de los usuarios una vez que se vuelven inválidos o expiran. Este tipo de grant se usa en conjunto con otro tipo de grant.

Dos flujos serán obsoletos en el release de [OAuth2.1](https://oauth.net/2.1/), y su uso no se recomienda:

- [Implicit Flow*](https://oauth.net/2/grant-types/implicit/): La implementación segura de PKCE vuelve este flujo obsoleto. Antes de PKCE, el flujo implícito era usado por aplicaciones del lado del cliente tales como [aplicaciones de página única](https://en.wikipedia.org/wiki/Single-page_application) ya que [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) relajó la [política del mismo origen](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) para que los sitios se comunicaran entre sí. Para más información sobre por qué el grant implícito no se recomienda, revisar esta [sección](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.1.2).
- [Credenciales de Contraseña del Propietario del Recurso (ROPC)](https://oauth.net/2/grant-types/password/): usadas para intercambiar credenciales de usuarios directamente con el cliente, quien luego las envía a la autorización para intercambiarlas por un token de acceso. Para información sobre por qué este flujo no se recomienda, revisar esta [sección](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.4).

*: El flujo implícito en OAuth solo está obsoleto, sin embargo es todavía una solución viable dentro de Open ID Connect (OIDC) para recuperar `id_tokens`. Tener cuidado de entender cómo se está usando el flujo implícito, lo cual puede identificarse si solo se está usando el endpoint `/authorization` para ganar un token de acceso, sin depender del endpoint `/token` de ninguna manera. Un ejemplo de esto puede encontrarse [aquí](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post).

*Por favor notar que los flujos de OAuth son un tema complejo, y lo anterior incluye solo un resumen de las áreas clave. Las referencias inline contienen más información sobre los flujos específicos.*

## Objetivos de Prueba

- Determinar si la implementación de OAuth2 es vulnerable o usa una implementación obsoleta o personalizada.

## Cómo Probar

### Probar Tipos de Grant Obsoletos

Los tipos de grant obsoletos fueron obsoletos por razones de seguridad y funcionalidad. Identificar si se están usando nos permite revisar rápidamente si son susceptibles a cualquiera de las amenazas pertenecientes a su uso. Algunos podrían estar fuera del alcance del atacante, tal como la manera en que un cliente podría estar usando las credenciales de los usuarios. Esto debería documentarse y elevarse a los equipos de ingeniería internos.

Para clientes públicos, generalmente es posible identificar el tipo de grant en la solicitud al endpoint `/token`. Se indica en el intercambio de token con el parámetro `grant_type`.

El siguiente ejemplo muestra el grant de Código de Autorización con PKCE.

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

Los valores para el parámetro `grant_type` y el tipo de grant que indican son:

- `password`: Indica el grant ROPC.
- `client_credentials`: Indica el grant de Client Credential.
- `authorization_code`: Indica el grant de Authorization Code.

El tipo de Implicit Flow no se indica por el parámetro `grant_type` ya que el token se presenta en la respuesta a la solicitud del endpoint `/authorization`, y en su lugar puede identificarse a través del `response_type`. Abajo hay un ejemplo.

```http
GET /authorize
  ?client_id=<some_client_id>
  &response_type=token 
  &redirect_uri=https%3A%2F%2Fclient.example.com%2F
  &scope=openid%20profile%20email
  &state=<random_state>
```

Los siguientes parámetros de URL indican el flujo de OAuth siendo usado:

- `response_type=token`: Indica Implicit Flow, ya que el cliente está solicitando directamente del servidor de autorización devolver un token.
- `response_type=code`: Indica flujo de Authorization Code, ya que el cliente está solicitando del servidor de autorización devolver un código, que se intercambiará posteriormente con un token.
- `code_challenge=sha256(xyz)`: Indica la extensión PKCE, ya que ningún otro flujo usa este parámetro.

La siguiente es una solicitud de autorización de ejemplo para el flujo de Authorization Code con PKCE:

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

#### Clientes Públicos

El grant de Authorization Code con extensión PKCE se recomienda para clientes públicos. Una solicitud de autorización para el flujo de Authorization Code con PKCE debería contener `response_type=code` y `code_challenge=sha256(xyz)`.

El intercambio de token debería contener el tipo de grant `authorization_code` y un `code_verifier`.

Los tipos de grant impropios para clientes públicos son:

- Grant de Authorization Code sin la extensión PKCE
- Client Credentials
- Implicit Flow
- ROPC

#### Clientes Confidenciales

El grant de Authorization Code se recomienda para clientes confidenciales. La extensión PKCE podría usarse también.

Los tipos de grant impropios para clientes confidenciales son:

- Client Credentials (Excepto para máquina-a-máquina -- ver abajo)
- Implicit Flow
- ROPC

##### Máquina-a-Máquina

En situaciones donde no ocurre interacción del usuario y los clientes son solo clientes confidenciales, el grant de Client Credentials podría usarse.

Si se conocen el `client_id` y `client_secret`, es posible obtener un token pasando el tipo de grant `client_credentials`.

```bash
$ curl --request POST \
  --url https://as.example.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<some_client_id>","client_secret":"<some_client_secret>","grant_type":"client_credentials"}' --proxy https://localhost:8080/ -k
```

### Fuga de Credenciales

Dependiendo del flujo, OAuth transporta varios tipos de credenciales en parámetros de URL.

Los siguientes tokens pueden considerarse credenciales filtradas:

- token de acceso
- token de refresco
- código de autorización
- PKCE code challenge / code verifier

Debido a cómo funciona OAuth, el `code` de autorización así como el `code_challenge`, y `code_verifier` podrían ser parte de la URL. El flujo implícito transporta el token de autorización como parte de la URL si el `response_mode` no está establecido a [`form_post`](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html). Esto podría llevar a fuga del token o código solicitado en el encabezado referrer, en archivos de log, y proxies debido a que estos parámetros se pasan ya sea en la consulta o el fragmento.

El riesgo que conlleva el flujo implícito filtrando los tokens es mucho mayor que filtrar el `code` o cualquier otro parámetro `code_*`, ya que están vinculados a clientes específicos y son más difíciles de abusar en caso de fuga.

Para probar este escenario, hacer uso de un proxy de intercepción HTTP tal como ZAP e interceptar el tráfico de OAuth.

- Paso a paso a través del proceso de autorización e identificar cualquier credencial presente en la URL.
- Si cualquier recurso externo está incluido en una página involucrada con el flujo de OAuth, analizar la solicitud hecha a ellos. Las credenciales podrían filtrarse en el encabezado referrer.

Después de pasar por el flujo de OAuth y usar la aplicación, unas pocas solicitudes se capturan en el historial de solicitudes de un proxy de intercepción HTTP. Buscar el encabezado referrer HTTP (por ejemplo `Referer: https://idp.example.com/`) que contiene el servidor de autorización y URL del cliente en el historial de solicitudes.

Revisar las etiquetas meta HTML (aunque esta etiqueta no está [soportada](https://caniuse.com/mdn-html_elements_meta_name_referrer) en todos los navegadores), o la [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) podría ayudar a evaluar si alguna fuga de credencial está ocurriendo a través del encabezado referrer.

## Casos de Prueba Relacionados

- [Pruebas de JSON Web Tokens](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## Remediación

- Al implementar OAuth, siempre considerar la tecnología usada y si la aplicación es una aplicación del lado del servidor que puede evitar revelar secretos, o una aplicación del lado del cliente que no puede.
- En casi cualquier caso, usar el flujo de Authorization Code con PKCE. Una excepción podría ser flujos máquina-a-máquina.
- Usar parámetros POST o valores de encabezado para transportar secretos.
- Cuando no existen otras posibilidades (por ejemplo, en aplicaciones heredadas que no pueden migrarse), implementar encabezados de seguridad adicionales tales como una `Referrer-Policy`.

## Herramientas

- [BurpSuite](https://portswigger.net/burp/releases)
- [EsPReSSO](https://github.com/portswigger/espresso)
- [ZAP](https://www.zaproxy.org/)

## Referencias

- [User Authentication with OAuth 2.0](https://oauth.net/articles/authentication/)
- [The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://datatracker.ietf.org/doc/html/rfc6750)
- [OAuth 2.0 Threat Model and Security Considerations](https://datatracker.ietf.org/doc/html/rfc6819)
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-16)
- [Authorization Code Flow with Proof Key for Code Exchange](https://auth0.com/docs/authorization/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce)
