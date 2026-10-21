# JSON Web Tokens

|ID          |
|------------|
|WSTG-SESS-10|

## Resumen

Los JSON Web Tokens (JWT) son tokens JSON firmados criptográficamente, diseñados para compartir claims entre sistemas. Se usan con frecuencia como tokens de autenticación o de sesión, especialmente en API REST.

Los JWT son una fuente habitual de vulnerabilidades, tanto por la forma en que se implementan en las aplicaciones como por las bibliotecas subyacentes. Como se usan para la autenticación, una vulnerabilidad puede provocar fácilmente el compromiso completo de la aplicación.

## Objetivos de prueba

- Determinar si los JWT exponen información sensible.
- Determinar si los JWT pueden manipularse o modificarse.

## Cómo probar

### Visión general

Los JWT constan de tres componentes:

- El encabezado
- El payload (o cuerpo)
- La firma

Cada componente está codificado en base64 y los componentes están separados por puntos (`.`). Ten en cuenta que la codificación base64 utilizada en un JWT elimina los signos igual (`=`), por lo que quizá debas volver a añadirlos para decodificar las secciones.

### Analizar el contenido

#### Encabezado

El encabezado define el tipo de token (normalmente `JWT`) y el algoritmo utilizado para la firma. A continuación se muestra un ejemplo de encabezado decodificado:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Hay tres tipos principales de algoritmos utilizados para calcular las firmas:

| Algoritmo | Descripción |
|-----------|-------------|
| HSxxx | HMAC con una clave secreta y SHA-xxx. |
| RSxxx y PSxxx | Firma con clave pública mediante RSA. |
| ESxxx | Firma con clave pública mediante ECDSA. |

También existe una amplia variedad de [otros algoritmos](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms) que pueden utilizarse para tokens cifrados (JWE), aunque son menos comunes.

#### Payload

El payload del JWT contiene los datos reales. A continuación se muestra un ejemplo de payload:

```json
{
  "username": "administrator",
  "is_admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}
```

El payload normalmente no está cifrado, así que revísalo para determinar si contiene información sensible o potencialmente inapropiada.

Este JWT incluye el nombre de usuario y el estado administrativo del usuario, además de dos claims estándar (`iat` y `exp`). Estos claims se definen en [RFC 7519](https://tools.ietf.org/html/rfc7519#section-4.1); la tabla siguiente ofrece un breve resumen:

| Claim | Nombre completo | Descripción |
|-------|-----------------|-------------|
| `iss` | Emisor | La identidad de la parte que emitió el token. |
| `iat` | Emitido en | La marca de tiempo Unix del momento en que se emitió el token. |
| `nbf` | No antes de | La marca de tiempo Unix de la fecha más temprana en que se puede utilizar el token. |
| `exp` | Expira | La marca de tiempo Unix del momento en que expira el token. |

#### Firma

La firma se calcula mediante el algoritmo definido en el encabezado del JWT, se codifica en base64 y se añade al token. Modificar cualquier parte del JWT debería invalidar la firma y hacer que el servidor rechace el token.

Para leer las tres partes de un token, decodifícalo con una herramienta para JWT. La pestaña JWT de Gori decodifica el token al pegarlo y permite editar los claims para volver a firmarlo. `gori run jwt` muestra la misma decodificación:

```text
$ gori run jwt "$TOKEN"
// header
{
  "typ": "JWT",
  "alg": "RS256"
}

// payload
{
  "data": {
    "id": 25,
    "email": "alice@wstg.test",
    "password": "646e613efcfc1317061b1df9340e3726",
    "role": "customer",
    // ... profile fields omitted
  },
  "bid": 6,
  "iat": 1789655485
}

// signature (not verified)
wcaY3SC6taMCJUH8F4CEcF2a1rUa8bjYetKAo3s8DO3yjt3D4GFmz2SyJR3151np…
```

Este token fue emitido por OWASP Juice Shop. Su payload contiene el rol del usuario y un hash de contraseña. Como el payload no está cifrado, considera que cualquier persona que tenga el token puede leer todos sus valores.

### Revisar el uso

Además de ser seguro desde el punto de vista criptográfico, el JWT también debe almacenarse y enviarse de forma segura. Esto debe incluir comprobar que:

- Siempre se [envíe a través de conexiones cifradas (HTTPS)](../09-Weak_Cryptography/03-Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- Si se almacena en una cookie, esta debe estar [marcada con los atributos apropiados](../06-Session_Management/02-Cookies_Attributes.md).

También debe revisarse la validez del JWT, en función de los claims `iat`, `nbf` y `exp`, para determinar que:

- El JWT tenga una duración razonable para la aplicación.
- La aplicación rechace los tokens expirados.

### Verificación de la firma

Una de las vulnerabilidades más graves que se encuentran en los JWT se produce cuando la aplicación no valida que la firma sea correcta. Esto suele ocurrir cuando un desarrollador utiliza una función como `jwt.decode()` de Node.js, que simplemente decodifica el cuerpo del JWT, en lugar de `jwt.verify()`, que verifica la firma antes de decodificar el JWT.

Esto se puede comprobar fácilmente modificando el cuerpo del JWT sin cambiar nada en el encabezado ni en la firma y enviándolo en una solicitud para ver si la aplicación lo acepta.

La pestaña JWT de Gori también muestra payloads listos para enviar que corresponden a las vulnerabilidades descritas a continuación. `gori run jwt --attacks` muestra el mismo conjunto: variantes del algoritmo `none`, intentos con secretos débiles y payloads de inyección en encabezados.

```text
$ gori run jwt "$TOKEN" --attacks
[none]      alg=none                  unsigned; accepted if the server trusts alg=none from the token
    eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJkYXRhIjp7ImlkIjoyNSwidXNlcm5hbWUiOiIiLCJlbWFpbCI6…
... snipped: alg=None, alg=NONE, signature stripped ...
[weak-secret] HS256 secret=(empty)      verifies if the server's HMAC key is empty
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImlkIjoyNSwidXNlcm5hbWUiOiIiLCJlbWFpbCI…
[weak-secret] HS256 secret=secret       verifies if the server's HMAC key is "secret"
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImlkIjoyNSwidXNlcm5hbWUiOiIiLCJlbWFpbCI…
... snipped: more weak-secret candidates ...
[header-inject] kid=/dev/null             kid path-traversal to an empty file → HMAC with an empty key verifies
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii4uLy4uLy4uLy4uLy4uLy4uLy4uLy4uL2Rldi9udWxs…
[header-inject] kid SQLi                  kid used in a SQL key lookup; craft the UNION to return a known key
    eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IngnIFVOSU9OIFNFTEVDVCAnYXR0YWNrZXIifQ.eyJkY…
... snipped: jku, x5u, jwk header injections ...
```

Envía cada candidato a la aplicación y observa si alguno es aceptado.

#### El algoritmo none

Además de los algoritmos basados en clave pública y HMAC, la especificación JWT también define un algoritmo de firma llamado `none`. Como su nombre indica, significa que el JWT no tiene firma, lo que permite modificarlo.

Esto se puede probar modificando a `none` el algoritmo de firma (`alg`) en el encabezado del JWT, como se muestra en el ejemplo siguiente:

```json
{
        "alg": "none",
        "typ": "JWT"
}
```

A continuación, se vuelve a codificar el encabezado y el payload en base64 y se elimina la firma (se deja el punto final). Con el encabezado anterior y el payload de la sección [payload](#payload), se obtendría el siguiente JWT:

```txt
eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0K.eyJ1c2VybmFtZSI6ImFkbWluaW5pc3RyYXRvciIsImlzX2FkbWluIjp0cnVlLCJpYXQiOjE1MTYyMzkwMjIsImV4cCI6MTUxNjI0MjYyMn0.
```

Algunas implementaciones intentan evitarlo bloqueando explícitamente el uso del algoritmo `none`. Si se hace sin distinguir entre mayúsculas y minúsculas, quizá sea posible eludir la protección especificando un algoritmo como `NoNe`.

#### Firmas «psíquicas» de ECDSA

Se identificó una vulnerabilidad en las versiones 15 a 18 de Java, en las que las firmas ECDSA no se validaban correctamente en algunas circunstancias ([CVE-2022-21449](https://neilmadden.blog/2022/04/19/psychic-signatures-in-java/), conocida como «firmas psíquicas»). Si se utiliza una de estas versiones vulnerables para analizar un JWT que use el algoritmo `ES256`, se puede eludir por completo la verificación de la firma manipulando el cuerpo y reemplazando la firma por el siguiente valor:

```txt
MAYCAQACAQA
```

El JWT resultante tendría un aspecto similar al siguiente:

```txt
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.MAYCAQACAQA
```

### Claves HMAC débiles

Si el JWT está firmado mediante un algoritmo basado en HMAC (como HS256), la seguridad de la firma depende por completo de la solidez de la clave secreta utilizada en el HMAC.

Si la aplicación utiliza software comercial listo para usar o de código abierto, el primer paso debería ser investigar el código para determinar si utiliza una clave de firma HMAC predeterminada.

Si no hay una clave predeterminada, quizá sea posible descifrarla, adivinarla o probarla por fuerza bruta. La forma más sencilla de hacerlo es utilizar el script [crackjwt.py](https://github.com/Sjord/jwtcrack), que solo requiere el JWT y un archivo de diccionario.

Una opción más potente es convertir el JWT a un formato compatible con [John the Ripper](https://github.com/openwall/john) mediante el script [jwt2john.py](https://github.com/Sjord/jwtcrack/blob/master/jwt2john.py). Después, se puede utilizar John para llevar a cabo ataques mucho más avanzados contra la clave.

Si el JWT es grande, podría superar el tamaño máximo compatible con John. Se puede solucionar aumentando el valor de la variable `SALT_LIMBS` en `/src/hmacSHA256_fmt_plug.c` (o en el archivo equivalente para otros formatos HMAC) y recompilando John, como se explica en el siguiente [issue de GitHub](https://github.com/openwall/john/issues/1904).

Si se obtiene esta clave, es posible crear y firmar JWT arbitrarios, lo que suele provocar el compromiso completo de la aplicación.

### Confusión entre HMAC y clave pública

Si la aplicación utiliza JWT con firmas basadas en clave pública, pero no comprueba que el algoritmo sea correcto, es posible aprovecharlo para realizar un ataque de confusión de tipo de firma. Para que el ataque tenga éxito, deben cumplirse las siguientes condiciones:

1. La aplicación debe esperar que el JWT esté firmado con un algoritmo basado en clave pública (es decir, `RSxxx` o `ESxxx`).
2. La aplicación no debe comprobar qué algoritmo utiliza realmente el JWT para la firma.
3. La clave pública utilizada para verificar el JWT debe estar disponible para el atacante.

Si se cumplen todas estas condiciones, un atacante puede utilizar la clave pública para firmar el JWT con un algoritmo basado en HMAC (como `HS256`). Por ejemplo, la biblioteca [Node.js jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) utiliza la misma función tanto para tokens basados en clave pública como para tokens basados en HMAC, como se muestra en el ejemplo siguiente:

```javascript
// Verify a JWT signed using RS256
jwt.verify(token, publicKey);

// Verify a JWT signed using HS256
jwt.verify(token, secretKey);
```

Esto significa que, si el JWT se firma con `publicKey` como clave secreta para el algoritmo `HS256`, la firma se considerará válida.

Para explotar este problema, es necesario obtener la clave pública. La forma más habitual de que esto ocurra es que la aplicación reutilice la misma clave para firmar los JWT y como parte del certificado TLS. En ese caso, se puede descargar la clave del servidor mediante un comando como el siguiente:

```sh
openssl s_client -connect example.org:443 | openssl x509 -pubkey -noout
```

Como alternativa, la clave puede estar disponible en un archivo público del sitio, en una ubicación habitual como `/.well-known/jwks.json`.

Para probarlo, modifica el contenido del JWT y luego utiliza la clave pública obtenida previamente para firmar el JWT con el algoritmo `HS256`. A menudo es difícil realizar esta prueba sin acceso al código fuente o a los detalles de implementación, porque el formato de la clave debe ser idéntico al que utiliza el servidor; por tanto, problemas como espacios en blanco o la codificación CRLF pueden hacer que las claves no coincidan.

### Clave pública proporcionada por el atacante (JWK integrado)

El [estándar JSON Web Signature (JWS) (RFC 7515 §4.1.3)](https://tools.ietf.org/html/rfc7515#section-4.1.3) permite integrar directamente en el encabezado, mediante el parámetro `jwk`, la clave pública utilizada para verificar la firma. Si la biblioteca de verificación del lado del servidor acepta sin más esta clave integrada, sin compararla con un almacén de confianza o una lista de claves públicas conocidas permitidas, un atacante puede integrar su propia clave pública y firmar tokens falsificados con la clave privada correspondiente.

Por ejemplo, un atacante puede integrar su clave pública directamente en el encabezado:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "use": "sig",
    "kid": "attacker-inline-key",
    "n": "u1SucjA342W..."
  }
}
```

Hay diversos scripts y extensiones que se pueden utilizar para generar e inyectar encabezados `jwk` integrados, como [jwt_tool](https://github.com/ticarpi/jwt_tool) o la [extensión JSON Web Tokens para Burp](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6).

### Manipulación del ID de clave (kid)

El parámetro de encabezado `kid` se utiliza normalmente para recuperar del sistema de archivos o de una base de datos la clave necesaria para verificar la firma. Puede ser vulnerable a varios ataques de inyección.

#### Recorrido de directorios

Si la aplicación utiliza el parámetro `kid` para leer un archivo de clave del sistema de archivos, un atacante podría especificar la ruta de un archivo vacío conocido, como `../../../../dev/null` (en Linux) o `nul` (en Windows).

Por ejemplo, un atacante puede modificar el encabezado para que apunte a un archivo vacío:

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "../../../../../dev/null"
}
```

Como el contenido de `/dev/null` está vacío, el atacante puede firmar el token malicioso utilizando una **cadena vacía** como clave secreta. Si el servidor es vulnerable, leerá el archivo vacío, utilizará la cadena vacía para verificar la firma y aceptará el token falsificado.

#### Inyección de comandos/SQL

Si el valor de `kid` se pasa sin sanear a una consulta de base de datos o a un comando del sistema para recuperar la clave, puede ser vulnerable a inyección SQL o inyección de comandos.

Por ejemplo, un atacante puede inyectar un payload SQL en el parámetro `kid` para controlar la clave que devuelve la base de datos:

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "invalid-key' UNION SELECT 'attacker-controlled-key'--"
}
```

Esto permite al atacante obligar a la aplicación a utilizar una clave conocida (por ejemplo, «attacker-controlled-key») para la verificación y, así, falsificar tokens válidos.

### Inyección en los encabezados de URL de JWKS (jku) y URL de X.509 (x5u)

La [especificación JSON Web Signature (JWS) (RFC 7515 §4.1.2, §4.1.5)](https://tools.ietf.org/html/rfc7515#section-4.1.2) admite parámetros de encabezado que apuntan a almacenes externos de claves públicas:

- `jku` (URL del conjunto JWK): URI que hace referencia a un recurso con un conjunto de claves públicas codificadas en JSON.
- `x5u` (URL de X.509): URI que hace referencia a un recurso con un conjunto de certificados de clave pública X.509.

Si la biblioteca de verificación obtiene automáticamente claves públicas de la URI especificada en el encabezado sin aplicar una lista estricta de dominios permitidos, un atacante puede hacer que `jku`/`x5u` apunte a un servidor bajo su control, alojar allí su propia clave y firmar tokens falsificados con la clave privada correspondiente. El servidor aceptará esos tokens como válidos.

#### Procedimiento de prueba

1. Genera un par de claves RSA en una máquina de prueba.
2. Aloja la clave pública correspondiente en un servidor público, con formato de conjunto de claves web JSON (JWKS):

    ```json
    {
      "keys": [
        {
          "kty": "RSA",
          "e": "AQAB",
          "use": "sig",
          "kid": "test-key-01",
          "alg": "RS256",
          "n": "u1SucjA342W..."
        }
      ]
    }
    ```

3. Modifica el payload del JWT objetivo para alterar los claims (por ejemplo, modifica `user_id` o `role`).
4. Actualiza el encabezado del JWT con el parámetro `jku` que apunta al endpoint JWKS alojado y con el valor `kid` correspondiente:

    ```json
    {
      "alg": "RS256",
      "typ": "JWT",
      "jku": "https://attacker.example.com/.well-known/jwks.json",
      "kid": "test-key-01"
    }
    ```

5. Firma el token falsificado con la clave privada y envíalo en una solicitud autenticada.
6. Si se bloquean los nombres de host externos directos, evalúa formas de eludir el filtrado de URL:
    - Busca vulnerabilidades de redirección abierta en dominios permitidos (por ejemplo, `https://trusted.example.com/oauth/redirect?url=https://attacker.example.com/jwks.json`).
    - Busca discrepancias en el análisis de URL o fallos de normalización de rutas (por ejemplo, `https://trusted.example.com@attacker.example.com/jwks.json`).
    - Comprueba si el servidor intenta resolver direcciones de la red interna (SSRF) cuando `jku` apunta a una dirección de bucle local (`http://127.0.0.1/`) o a endpoints de metadatos de la nube (`http://169.254.169.254/`).

El parámetro `x5u` se puede probar de la misma forma: en lugar de alojar un documento JWKS, aloja en la URL objetivo una cadena de certificados X.509 codificada en PEM y firmada con una clave controlada por el atacante. Muchas bibliotecas extraen la clave pública del certificado hoja sin comprobar si la cadena es de confianza o termina en una CA raíz fijada; por tanto, comprueba si se acepta una cadena autofirmada o que no sea de confianza por otros motivos.

### Confusión de audiencia (aud) y emisor (iss) del token

En arquitecturas distribuidas de microservicios y multiusuario, varios servicios distintos suelen compartir un proveedor de identidad (IdP) centralizado. Un fallo de autorización habitual se produce cuando un servicio posterior valida la firma criptográfica del token frente al IdP, pero no comprueba que el token se haya emitido explícitamente para ese servicio concreto.

- `aud` (Audiencia): Identifica los destinatarios a los que va dirigido el JWT ([RFC 7519 §4.1.3](https://tools.ietf.org/html/rfc7519#section-4.1.3)).
- `iss` (Emisor): Identifica la entidad principal que emitió el JWT ([RFC 7519 §4.1.1](https://tools.ietf.org/html/rfc7519#section-4.1.1)).

Si el Servicio B confía en la firma del IdP, pero omite la validación de `aud`, un atacante con credenciales válidas para el Servicio A, de pocos privilegios, puede obtener su token y reproducirlo contra el Servicio B.

#### Procedimiento de prueba

1. Autentícate en una aplicación o un tenant de pocos privilegios registrado con el proveedor de identidad de la organización objetivo.
2. Intercepta y extrae el JWT válido emitido.
3. Inspecciona los claims del payload para identificar el `aud` objetivo (por ejemplo, `aud: "client-portal"`) y el `iss` (por ejemplo, `iss: "https://auth.example.com/"`).
4. Reproduce el token sin modificarlo en una solicitud a un servicio de altos privilegios, una API administrativa o un tenant separado (por ejemplo, `https://internal-admin.example.com/api/v1/users`).
5. Si el servicio de altos privilegios acepta el token y realiza la acción, es vulnerable a la reproducción de tokens entre servicios debido a la falta de validación de la audiencia.
6. Prueba por separado la confusión del emisor: si el IdP es multiusuario o da soporte a varios entornos (por ejemplo, pruebas y producción) mediante un endpoint JWKS compartido, obtén un token emitido para otro tenant o aplicación con menor nivel de confianza y reprodúcelo contra el servicio objetivo. Si se acepta, significa que `iss` no se valida o se valida con una comparación débil (como una coincidencia de prefijo o subcadena) en lugar de una comparación exacta con el emisor de confianza esperado.

## Casos de prueba relacionados

- [Información sensible enviada a través de canales sin cifrar](../09-Weak_Cryptography/03-Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- [Atributos de las cookies](../06-Session_Management/02-Cookies_Attributes.md).
- [Pruebas del almacenamiento del navegador](../11-Client-side/12-Browser_Storage.md).
- [Falsificación de solicitudes del lado del servidor](../07-Injection/19-Server-Side_Request_Forgery.md).

## Remediación

- Utiliza una biblioteca segura y actualizada para gestionar los JWT.
- Asegúrate de que la firma sea válida y de que se utilice el algoritmo esperado.
- No confíes en parámetros de encabezado no verificados, como `jku`, `x5u` o `jwk`, sin compararlos con una lista estricta de dominios/URI permitidos.
- Valida siempre los claims estándar: asegúrate de que `exp` esté en el futuro, `nbf` esté en el pasado, `iss` coincida con el servidor de autorización de confianza y `aud` coincida exactamente con el identificador esperado del servicio.
- Utiliza una clave HMAC robusta o una clave privada única para firmar los tokens.
- Asegúrate de que el payload no exponga información sensible.
- Asegúrate de que los JWT se almacenen y transmitan de forma segura.
- Consulta la [hoja de referencia de OWASP sobre JSON Web Tokens](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html).

## Herramientas

- [John the Ripper](https://github.com/openwall/john)
- [jwt2john](https://github.com/Sjord/jwtcrack)
- [jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [Extensión JSON Web Tokens para Burp](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [Complemento JWT para ZAP](https://github.com/SasanLabs/owasp-zap-jwt-addon)
- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [Gori](https://github.com/hahwul/gori)
    - La pestaña JWT decodifica tokens y genera payloads para ataques contra firmas; también se puede usar mediante scripts con `gori run jwt`.

## Referencias

- [RFC 7515 JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [RFC 7519 JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [RFC 8725 Prácticas recomendadas actuales para JSON Web Token](https://tools.ietf.org/html/rfc8725)
- [Hoja de referencia de OWASP sobre JSON Web Token](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html)