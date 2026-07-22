# Pruebas de JSON Web Tokens

|ID          |
|------------|
|WSTG-SESS-10|

## Resumen

Los JSON Web Tokens (JWTs) son tokens JSON firmados criptográficamente, destinados a compartir claims entre sistemas. Se usan frecuentemente como tokens de autenticación o sesión, particularmente en APIs REST.

Los JWTs son una fuente común de vulnerabilidades, tanto en cómo se implementan en aplicaciones, como en las bibliotecas subyacentes. Como se usan para autenticación, una vulnerabilidad puede fácilmente resultar en un compromiso completo de la aplicación.

## Objetivos de Prueba

- Determinar si los JWTs exponen información sensible.
- Determinar si los JWTs pueden ser manipulados o modificados.

## Cómo Probar

### Visión General

Los JWTs están compuestos de tres componentes:

- El encabezado (header)
- El payload (o cuerpo)
- La firma (signature)

Cada componente está codificado en base64, y se separan por puntos (`.`). Tener en cuenta que la codificación base64 usada en un JWT elimina los signos igual (`=`), por lo que podría necesitar añadirlos de vuelta para decodificar las secciones.

### Analizar los Contenidos

#### Encabezado

El encabezado define el tipo de token (típicamente `JWT`), y el algoritmo usado para la firma. Un encabezado decodificado de ejemplo se muestra a continuación:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Hay tres tipos principales de algoritmos que se usan para calcular las firmas:

| Algoritmo | Descripción |
|-----------|-------------|
| HSxxx     | HMAC usando una clave secreta y SHA-xxx. |
| RSxxx y PSxxx | Firma de clave pública usando RSA. |
| ESxxx     | Firma de clave pública usando ECDSA. |

También hay una amplia gama de [otros algoritmos](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms) que podrían usarse para tokens cifrados (JWEs), aunque estos son menos comunes.

#### Payload

El payload del JWT contiene los datos reales. Un payload de ejemplo se muestra a continuación:

```json
{
  "username": "administrator",
  "is_admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}
```

El payload usualmente no está cifrado, así que revisarlo para determinar si hay datos sensibles o potencialmente inapropiados incluidos dentro.

Este JWT incluye el nombre de usuario y estado administrativo del usuario, así como dos claims estándar (`iat` y `exp`). Estos claims están definidos en [RFC 7519](https://tools.ietf.org/html/rfc7519#section-4.1), un breve resumen se da en la tabla a continuación:

| Claim | Nombre Completo | Descripción |
|-------|-----------------|-------------|
| `iss` | Issuer          | La identidad de la parte que emitió el token. |
| `iat` | Issued At       | El timestamp Unix de cuándo se emitió el token. |
| `nbf` | Not Before      | El timestamp Unix de la fecha más temprana en que el token puede usarse. |
| `exp` | Expires         | El timestamp Unix de cuándo expira el token. |

#### Firma

La firma se calcula usando el algoritmo definido en el encabezado JWT, y luego se codifica en base64 y se añade al token. Modificar cualquier parte del JWT debería causar que la firma sea inválida, y el token sea rechazado por el servidor.

### Revisar Uso

Así como ser criptográficamente seguro en sí mismo, el JWT también necesita almacenarse y enviarse de manera segura. Esto debería incluir verificaciones de que:

- Siempre se [envía sobre conexiones cifradas (HTTPS)](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- Si se almacena en una cookie, entonces debería estar [marcado con atributos apropiados](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

La validez del JWT también debería revisarse, basada en los claims `iat`, `nbf` y `exp`, para determinar que:

- El JWT tiene una vida útil razonable para la aplicación.
- Los tokens expirados son rechazados por la aplicación.

### Verificación de Firma

Una de las vulnerabilidades más serias encontradas con JWTs es cuando la aplicación falla en validar que la firma sea correcta. Esto usualmente ocurre cuando un desarrollador usa una función tal como la función `jwt.decode()` de Node.js, que simplemente decodifica el cuerpo del JWT, en lugar de `jwt.verify()`, que verifica la firma antes de decodificar el JWT.

Esto puede probarse fácilmente modificando el cuerpo del JWT sin cambiar nada en el encabezado o firma, enviándolo en una solicitud para ver si la aplicación lo acepta.

#### El Algoritmo None

Además de los algoritmos basados en clave pública y HMAC, la especificación JWT también define un algoritmo de firma llamado `none`. Como el nombre sugiere, esto significa que no hay firma para el JWT, permitiendo que sea modificado.

Esto puede probarse modificando el algoritmo de firma (`alg`) en el encabezado JWT a `none`, como se muestra en el ejemplo a continuación:

```json
{
        "alg": "none",
        "typ": "JWT"
}
```

El encabezado y payload entonces se recodifican con base64, y la firma se remueve (dejando el punto trailing). Usando el encabezado anterior, y el payload listado en la sección [payload](#payload), esto daría el siguiente JWT:

```txt
eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0K.eyJ1c2VybmFtZSI6ImFkbWluaW5pc3RyYXRvciIsImlzX2FkbWluIjp0cnVlLCJpYXQiOjE1MTYyMzkwMjIsImV4cCI6MTUxNjI0MjYyMn0.
```

Algunas implementaciones intentan evitar esto bloqueando explícitamente el uso del algoritmo `none`. Si esto se hace de manera insensible a mayúsculas, podría ser posible evadir especificando un algoritmo tal como `NoNe`.

#### ECDSA "Firmas Psíquicas"

Se identificó una vulnerabilidad en Java versión 15 a 18 donde no validaban correctamente las firmas ECDSA en algunas circunstancias ([CVE-2022-21449](https://neilmadden.blog/2022/04/19/psychic-signatures-in-java/), conocida como "psychic signatures"). Si una de estas versiones vulnerables se usa para analizar un JWT usando el algoritmo `ES256`, esto puede usarse para evadir completamente la verificación de firma manipulando el cuerpo y luego reemplazando la firma con el siguiente valor:

```txt
MAYCAQACAQA
```

Resultando en un JWT que se ve algo así:

```txt
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.MAYCAQACAQA
```

### Claves HMAC Débiles

Si el JWT está firmado usando un algoritmo basado en HMAC (tal como HS256), la seguridad de la firma depende enteramente de la fuerza de la clave secreta usada en el HMAC.

Si la aplicación está usando software comercial o de código abierto, el primer paso debería ser investigar el código, y ver si hay una clave de firma HMAC por defecto que se use.

Si no hay un valor predeterminado, entonces podría ser posible crackear, adivinar o forzar bruscamente la clave. La manera más simple de hacer esto es usar el script [crackjwt.py](https://github.com/Sjord/jwtcrack), que simplemente requiere el JWT y un archivo de diccionario.

Una opción más poderosa es convertir el JWT a un formato que pueda ser usado por [John the Ripper](https://github.com/openwall/john) usando el script [jwt2john.py](https://github.com/Sjord/jwtcrack/blob/master/jwt2john.py). John puede entonces ser usado para llevar a cabo ataques mucho más avanzados contra la clave.

Si el JWT es grande, podría exceder el tamaño máximo soportado por John. Esto puede trabajarse aumentando el valor de la variable `SALT_LIMBS` en `/src/hmacSHA256_fmt_plug.c` (o el archivo equivalente para otros formatos HMAC) y recompilando John, como se discute en el siguiente [issue de GitHub](https://github.com/openwall/john/issues/1904).

Si esta clave puede obtenerse, entonces es posible crear y firmar JWTs arbitrarios, lo cual usualmente resulta en un compromiso completo de la aplicación.

### Confusión HMAC vs Clave Pública

Si la aplicación usa JWTs con firmas basadas en clave pública, pero no verifica que el algoritmo sea correcto, esto potencialmente puede explotarse en un ataque de confusión de tipo de firma. Para que esto sea exitoso, las siguientes condiciones necesitan cumplirse:

1. La aplicación debe esperar que el JWT esté firmado con un algoritmo basado en clave pública (i.e., `RSxxx` o `ESxxx`).
2. La aplicación no debe verificar qué algoritmo está realmente usando el JWT para la firma.
3. La clave pública usada para verificar el JWT debe estar disponible para el atacante.

Si todas estas condiciones son verdaderas, entonces un atacante puede usar la clave pública para firmar el JWT usando un algoritmo basado en HMAC (tal como `HS256`). Por ejemplo, la biblioteca [Node.js jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) usa la misma función tanto para tokens basados en clave pública como HMAC, como se muestra en el ejemplo a continuación:

```javascript
// Verificar un JWT firmado usando RS256
jwt.verify(token, publicKey);

// Verificar un JWT firmado usando HS256
jwt.verify(token, secretKey);
```

Esto significa que si el JWT se firma usando `publicKey` como clave secreta para el algoritmo `HS256`, la firma se considerará válida.

Para explotar este problema, la clave pública debe obtenerse. La manera más común en que esto puede suceder es si la aplicación reutiliza la misma clave tanto para firmar JWTs como parte del certificado TLS. En este caso, la clave puede descargarse del servidor usando un comando tal como el siguiente:

```sh
openssl s_client -connect example.org:443 | openssl x509 -pubkey -noout
```

Alternativamente, la clave podría estar disponible desde un archivo público en el sitio en una ubicación común tal como `/.well-known/jwks.json`.

Para probar esto, modificar los contenidos del JWT, y luego usar la clave pública previamente obtenida para firmar el JWT usando el algoritmo `HS256`. Esto a menudo es difícil de realizar al probar sin acceso al código fuente o detalles de implementación, porque el formato de la clave debe ser idéntico al usado por el servidor, así que problemas tales como espacio vacío o codificación CRLF podrían resultar en que las claves no coincidan.

### Clave Pública Proporcionada por el Atacante

El [estándar JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) (que define el encabezado y firmas usadas por JWTs) permite que la clave usada para firmar el token se embeba en el encabezado. Si la biblioteca usada para validar el token soporta esto, y no verifica la clave contra una lista de claves aprobadas, esto permite a un atacante firmar un JWT con una clave arbitraria que proporciona.

Hay una variedad de scripts que pueden usarse para hacer esto, tales como [jwk-node-jose.py](https://github.com/zi0Black/POC-CVE-2018-0114) o [jwt_tool](https://github.com/ticarpi/jwt_tool).

### Manipulación del Key ID (kid)

El parámetro de encabezado `kid` típicamente se usa para recuperar la clave necesaria para verificar la firma desde un sistema de archivos o base de datos. Puede ser vulnerable a varios ataques de inyección.

#### Directory Traversal

Si la aplicación usa el parámetro `kid` para leer un archivo de clave del sistema de archivos, un atacante podría especificar una ruta a un archivo vacío conocido, tal como `../../../../dev/null` (en Linux) o `nul` (en Windows).

Por ejemplo, un atacante puede modificar el encabezado para apuntar a un archivo vacío:

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "../../../../../dev/null"
}
```

Como el contenido de `/dev/null` está vacío, el atacante puede entonces firmar el token malicioso usando una **cadena vacía** como clave secreta. Si el servidor es vulnerable, leerá el archivo vacío, usará la cadena vacía para verificar la firma, y aceptará el token forjado.

#### Inyección de Comandos/SQL

Si el `kid` se pasa sin saneamiento a una consulta de base de datos o un comando del sistema para recuperar la clave, podría ser vulnerable a Inyección SQL o Inyección de Comandos.

Por ejemplo, un atacante puede inyectar un payload SQL en el parámetro `kid` para controlar la clave devuelta por la base de datos:

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "invalid-key' UNION SELECT 'attacker-controlled-key'--"
}
```

Esto permite a un atacante forzar a la aplicación a usar una clave conocida (por ejemplo, "attacker-controlled-key") para verificación, permitiéndoles forjar tokens válidos.

## Casos de Prueba Relacionados

- [Pruebas de Información Sensible Enviada vía Canales No Cifrados](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- [Pruebas de Atributos de Cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).
- [Pruebas de Almacenamiento del Navegador](../11-Client-side_Testing/12-Testing_Browser_Storage.md).

## Remediación

- Usar una biblioteca segura y actualizada para manejar JWTs.
- Asegurar que la firma sea válida, y que esté usando el algoritmo esperado.
- Usar una clave HMAC fuerte o una clave privada única para firmarlos.
- Asegurar que no haya información sensible expuesta en el payload.
- Asegurar que los JWTs se almacenen y transmitan de manera segura.
- Ver la [Hoja de Referencia de JSON Web Tokens de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html).

## Herramientas

- [John the Ripper](https://github.com/openwall/john)
- [jwt2john](https://github.com/Sjord/jwtcrack)
- [jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [JSON Web Tokens Burp Extension](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [ZAP JWT Add-on](https://github.com/SasanLabs/owasp-zap-jwt-addon)

## Referencias

- [RFC 7515 JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [RFC 7519 JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [Hoja de Referencia de JSON Web Token de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
