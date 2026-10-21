# Autenticación multifactor (MFA)

|ID          |
|------------|
|WSTG-ATHN-11|

## Resumen

Muchas aplicaciones implementan la autenticación multifactor (MFA) como una capa adicional de seguridad para proteger el proceso de inicio de sesión. También se conoce como autenticación de dos factores (2FA) o verificación en dos pasos (2SV), aunque no son exactamente lo mismo. MFA significa pedir al usuario que proporcione *al menos* dos [factores de autenticación](#types-of-mfa) diferentes al iniciar sesión.

MFA añade complejidad tanto a la funcionalidad de autenticación como a otras áreas relacionadas con la seguridad (como la gestión de credenciales y la recuperación de contraseñas), por lo que es fundamental implementarla de forma correcta y robusta.

## Objetivos de la prueba

- Identificar el tipo de MFA que utiliza la aplicación.
- Determinar si la implementación de MFA es robusta y segura.
- Intentar eludir la MFA.

## Cómo probar

### Tipos de MFA

MFA significa que se requieren *al menos* dos de los siguientes factores para la autenticación:

| Factor | Ejemplos |
|--------|----------|
| Algo que sabes | Contraseñas, PIN y preguntas de seguridad. |
| Algo que tienes | Tokens de hardware o software, certificados, correo electrónico*, SMS y llamadas telefónicas. |
| Algo que eres | Huellas dactilares, reconocimiento facial, escaneos del iris, escaneos de la palma de la mano y factores conductuales. |
| Ubicación | Rangos de direcciones IP de origen y geolocalización. |

\* El correo electrónico solo constituye realmente «algo que tienes» si la propia cuenta de correo está protegida con MFA. Por ello, debe considerarse una opción más débil que otras alternativas, como los certificados o TOTP, y algunas definiciones podrían no aceptarlo como MFA.

Cabe señalar que requerir varios ejemplos de un mismo factor (como necesitar tanto una contraseña como un PIN) **no constituye MFA**, aunque puede ofrecer algunas ventajas de seguridad frente a una contraseña simple y puede considerarse verificación en dos pasos (2SV).

Debido a la complejidad de implementar la biometría en un entorno basado en navegador, «Algo que eres» rara vez se usa en aplicaciones web, aunque está empezando a adoptarse mediante estándares como WebAuthn. El segundo factor más común es «Algo que tienes».

### Comprobar si es posible eludir la MFA

El primer paso para probar la MFA es identificar todas las funciones de autenticación de la aplicación, que pueden incluir:

- La página principal de inicio de sesión.
- Funciones críticas para la seguridad (como deshabilitar la MFA o cambiar una contraseña).
- Proveedores de inicio de sesión federado.
- Puntos de conexión de API (tanto desde la interfaz web principal como desde las aplicaciones móviles).
- Protocolos alternativos (no HTTP).
- Funciones de prueba o depuración.

Deben revisarse todos los métodos de inicio de sesión para asegurar que la MFA se aplique de manera uniforme. Si algunos métodos no requieren MFA, podrían proporcionar una forma sencilla de eludirla.

Si la autenticación se realiza en varios pasos, es posible que se pueda eludir completando el primer paso del proceso de autenticación (introducir el nombre de usuario y la contraseña) y, a continuación, accediendo directamente a la aplicación mediante navegación forzada o realizando solicitudes directas a la API sin completar la segunda etapa (introducir el código MFA).

Si la autenticación utiliza un proveedor de OpenID Connect (OIDC) que permite flujos (o políticas) de autenticación personalizados, como Azure B2C, puede haber varios flujos definidos y algunos quizá no requieran MFA. Por ejemplo, si la aplicación se autentica mediante un flujo llamado `B2C_1_SignInWithMFA`, pruebe a manipularlo para que sea `B2C_1_SignIn`, `B2C_1_SignInWithoutMFA` u otro valor similar.

En algunos casos, también puede haber mecanismos intencionales para eludir la MFA, como no exigirla:

- Desde direcciones IP específicas (que pueden falsificarse mediante el encabezado HTTP `X-Forwarded-For`).
- Cuando se establece un encabezado HTTP específico (como un encabezado no estándar, por ejemplo `X-Debug`).
- Para una cuenta específica codificada de forma fija (como una cuenta «root» o «breakglass»).

Cuando una aplicación admite tanto inicios de sesión locales como federados, puede ser posible eludir la MFA si no hay una separación sólida entre estos dos tipos de cuenta. Por ejemplo, si un usuario registra una cuenta local y configura la MFA para ella, pero no tiene MFA configurada en su cuenta del proveedor de inicio de sesión federado, un atacante podría volver a registrar (o vincular) una cuenta federada en la aplicación objetivo con la misma dirección de correo electrónico, tras comprometer la cuenta del usuario en el proveedor de inicio de sesión federado.

Por último, si la MFA se implementa en un sistema distinto de la aplicación principal (por ejemplo, en un proxy inverso para proteger una aplicación heredada que no admite MFA de forma nativa), es posible que se pueda eludir conectándose directamente al servidor de aplicaciones backend, tal como se explica en la guía sobre cómo [mapear la arquitectura de la aplicación](../01-Information_Gathering/10-Map_Application_Architecture.md#content-delivery-network-cdn).

### Comprobar la gestión de MFA

Deben probarse las funciones que se utilizan para gestionar la MFA desde la cuenta del usuario a fin de detectar vulnerabilidades. Por ejemplo:

- ¿Se exige al usuario que vuelva a autenticarse para eliminar o cambiar la configuración de MFA?
- ¿La función de gestión de MFA es vulnerable a la [falsificación de solicitudes entre sitios](../06-Session_Management/05-Cross_Site_Request_Forgery.md)?
- ¿Se puede modificar la configuración de MFA de otros usuarios mediante [vulnerabilidades IDOR](../05-Authorization/04-Insecure_Direct_Object_References.md)?

### Comprobar las opciones de recuperación de MFA

Muchas aplicaciones ofrecen a los usuarios una forma de recuperar el acceso a su cuenta si no pueden autenticarse con el segundo factor (por ejemplo, si han perdido el teléfono). Estos mecanismos pueden representar una debilidad importante en la aplicación, ya que, en la práctica, permiten eludir el segundo factor de autenticación.

#### Códigos de recuperación

Algunas aplicaciones proporcionan al usuario una lista de códigos de recuperación o de respaldo cuando habilita la MFA, que se pueden usar para iniciar sesión. Debe comprobarse que:

- Tengan suficiente longitud y complejidad para proteger contra ataques de fuerza bruta.
- Se generen de forma segura.
- Solo se puedan usar una vez.
- Exista protección contra ataques de fuerza bruta (como el bloqueo de la cuenta).
- Se notifique al usuario (por correo electrónico, SMS, etc.) cuando se use un código.

Consulte la [sección «Códigos de respaldo» de la Hoja de referencia sobre contraseñas olvidadas](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html#backup-codes) para obtener más detalles.

#### Proceso de restablecimiento de MFA

Si la aplicación implementa un proceso de restablecimiento de MFA, debe probarse del mismo modo que se prueba el [proceso de restablecimiento de contraseña](09-Weak_Password_Change_or_Reset_Functionalities.md). Es importante que este proceso sea *al menos* tan sólido como la implementación de MFA de la aplicación.

#### Autenticación alternativa

Algunas aplicaciones permiten que el usuario demuestre su identidad por otros medios, como mediante [preguntas de seguridad](08-Weak_Security_Question_Answer.md). Esto suele representar una debilidad importante, ya que las preguntas de seguridad ofrecen un nivel de seguridad mucho menor que la MFA.

### Contraseñas de un solo uso

La forma más común de MFA es el uso de contraseñas de un solo uso (OTP), que suelen ser códigos numéricos de seis dígitos (aunque pueden ser más largos o más cortos). El servidor y el usuario pueden generarlas por separado (por ejemplo, mediante una aplicación de autenticación), o bien el servidor puede generarlas y enviarlas al usuario. Hay varias formas de proporcionar estas OTP al usuario, entre ellas:

| Tipo | Descripción |
|------|-------------|
| Contraseña de un solo uso basada en HMAC (HOTP) | Genera un código basado en el HMAC de un secreto y un contador compartido. |
| Contraseña de un solo uso basada en el tiempo (TOTP) | Genera un código basado en el HMAC de un secreto y la hora actual. |
| Correo electrónico | Envía un código por correo electrónico. |
| SMS | Envía un código por SMS. |
| Teléfono | Envía un código mediante una llamada de voz a un número de teléfono. |

Normalmente, la OTP se introduce después de que el usuario haya proporcionado su nombre de usuario y contraseña. Deben realizarse varias comprobaciones, entre ellas:

- ¿Se bloquea la cuenta después de varios intentos fallidos de MFA?
- ¿Se bloquea la dirección IP del usuario después de varios intentos fallidos de MFA en distintas cuentas?
- ¿Se registran los intentos fallidos de MFA?
- ¿El formulario es vulnerable a ataques de inyección, incluida la [inyección de comodines SQL](../07-Injection/05-SQL_Injection.md#sql-wildcard-injection)?

Según el tipo de OTP utilizado, también deben realizarse otras comprobaciones específicas:

- ¿Cómo se envían las OTP al usuario (por correo electrónico, SMS, teléfono, etc.)?
    - ¿Hay limitación de tasa para evitar el envío masivo de SMS o llamadas telefónicas que genere costos?
- ¿Qué grado de solidez tienen las OTP (longitud y espacio de claves)?
- ¿Durante cuánto tiempo son válidas las OTP?
- ¿Hay varias OTP válidas al mismo tiempo?
- ¿Se pueden usar las OTP más de una vez?
- ¿Están las OTP vinculadas a la cuenta de usuario correcta o es posible autenticarse con ellas en otras cuentas?

#### HOTP y TOTP

Los códigos HOTP y TOTP se basan en un secreto compartido entre el servidor y el usuario. En el caso de los códigos TOTP, normalmente se proporciona al usuario en forma de código QR para que lo escanee con una aplicación de autenticación (aunque también puede proporcionarse como un secreto de texto que pueda introducir manualmente).

Si el secreto se genera en el servidor, debe comprobarse que tenga suficiente longitud y complejidad ([RFC 4226](https://www.rfc-editor.org/rfc/rfc4226#section-4) recomienda al menos 160 bits) y que se genere mediante una [función aleatoria segura](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation).

Si el usuario puede proporcionar el secreto, debe imponerse una longitud mínima adecuada y comprobar que la entrada no sea vulnerable a los ataques de inyección habituales.

Los códigos TOTP suelen ser válidos durante 30 segundos, pero algunas aplicaciones optan por aceptar varios códigos (por ejemplo, los códigos anterior, actual y siguiente) para tener en cuenta las diferencias entre la hora del sistema del servidor y la del dispositivo del usuario. Algunas aplicaciones pueden aceptar varios códigos a ambos lados del actual, lo que podría facilitar que un atacante adivine o fuerce el código. La siguiente tabla muestra la probabilidad de lograr un ataque de fuerza bruta contra un código OTP si el atacante puede realizar 10 solicitudes por segundo, para aplicaciones que aceptan solo el código actual o varios códigos (consulte [este artículo](https://www.codasecurity.co.uk/articles/mfa-testing#case-study---brute-forcing-totp) para ver los cálculos de la tabla).

| Códigos válidos | Tasa de éxito después de 1 hora | Tasa de éxito después de 4 horas | Tasa de éxito después de 12 horas | Tasa de éxito después de 24 horas |
|-------------|---------------------------|----------------------------|-----------------------------|-----------------------------|
| 1 | 4%  | 13% | 35% | 58% |
| 3 | 10% | 35% | 72% | 92% |
| 5 | 16% | 51% | 88% | 99% |
| 7 | 22% | 63% | 95% | 99% |

#### Correo electrónico, SMS y teléfono

Cuando los códigos se generan en el servidor y se envían al cliente, deben tenerse en cuenta los siguientes aspectos:

- ¿Es el mecanismo de transporte (correo electrónico, SMS o voz) lo suficientemente seguro para la aplicación?
- ¿Tienen los códigos suficiente longitud y complejidad?
- ¿Se generan los códigos mediante una [función aleatoria segura](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation)?
- ¿Durante cuánto tiempo son válidos los códigos?
- ¿Hay varios códigos válidos al mismo tiempo o generar un código nuevo invalida el anterior?
    - ¿Se podría bloquear el acceso a una cuenta solicitando códigos repetidamente?
- ¿Hay suficiente limitación de tasa para evitar que un atacante solicite un gran número de códigos?
    - Una gran cantidad de códigos enviados por correo electrónico podría hacer que se bloquee al servidor por enviar spam.
    - Una gran cantidad de SMS o llamadas de voz puede generar costos o utilizarse para acosar a un usuario.

### Aplicaciones móviles y notificaciones push

Una alternativa a los códigos OTP consiste en enviar una notificación push al teléfono móvil del usuario, que puede aprobarla o rechazarla. Este método es menos común, ya que requiere que el usuario instale una aplicación de autenticación específica.

Para evaluar correctamente su seguridad, es necesario ampliar el alcance de las pruebas para abarcar tanto la aplicación móvil como las API o los servicios de soporte que utiliza; por lo tanto, a menudo quedaría fuera del alcance de una prueba tradicional de aplicaciones web. Sin embargo, se pueden realizar algunas comprobaciones sencillas sin probar la aplicación móvil, entre ellas:

- ¿Proporciona la notificación suficiente contexto (direcciones IP, ubicación, etc.) para que el usuario decida con conocimiento de causa si la aprueba o la rechaza?
- ¿Existe algún tipo de mecanismo de desafío y respuesta (como proporcionar un código en el sitio que el usuario deba introducir en la aplicación, conocido a menudo como «coincidencia de números» o «desafío numérico»)?
- ¿Hay algún mecanismo de limitación de tasa u otra medida para evitar que se envíe una cantidad excesiva de notificaciones al usuario con la esperanza de que acepte alguna a ciegas?

### Filtrado por dirección IP y ubicación

Uno de los factores que a veces se usa con MFA es la ubicación («algún lugar donde estás»), aunque es discutible si constituye un factor de autenticación propiamente dicho. En el contexto de una aplicación web, normalmente significa restringir el acceso a direcciones IP específicas o no pedir al usuario un segundo factor mientras se conecte desde una dirección IP de confianza específica. Un caso habitual sería autenticar a los usuarios solo con su contraseña cuando se conectan desde los rangos de direcciones IP de la oficina y exigir un código OTP cuando se conectan desde otro lugar.

Según la implementación, es posible que un usuario pueda falsificar una dirección IP de confianza estableciendo el encabezado `X-Forwarded-For`, lo que podría permitirle eludir esta comprobación. Tenga en cuenta también que, si la aplicación no sanea correctamente el contenido de este encabezado, quizá sea posible llevar a cabo ataques como la inyección SQL. Si la aplicación admite IPv6, también debe comprobarse que se apliquen las restricciones adecuadas a esas conexiones.

Además, deben revisarse las direcciones IP de confianza para asegurar que no presenten debilidades, por ejemplo, si incluyen:

- Direcciones IP a las que puedan acceder usuarios no confiables (como las redes inalámbricas para invitados de una oficina).
- Direcciones IP asignadas dinámicamente que puedan cambiar.
- Rangos de redes públicas en los que un atacante pueda alojar su propio sistema (como Azure o AWS).

### Certificados y tarjetas inteligentes

La seguridad de la capa de transporte (TLS) se usa habitualmente para cifrar el tráfico entre el cliente y el servidor, y para proporcionar un mecanismo que permita al cliente confirmar la identidad del servidor (comparando el nombre común (CN) o el nombre alternativo del sujeto (SAN) del certificado con el dominio solicitado). Sin embargo, también puede proporcionar un mecanismo para que el servidor confirme la identidad del cliente, conocido como autenticación mediante certificado de cliente o TLS mutuo (mTLS). Una explicación completa de la autenticación mediante certificado de cliente queda fuera del alcance de esta guía, pero el principio fundamental es que el usuario presenta un certificado digital (almacenado en su equipo o en una tarjeta inteligente), que el servidor valida.

El primer paso de las pruebas consiste en determinar si la aplicación objetivo restringe las autoridades de certificación (CA) de confianza que pueden emitir certificados. Esta información puede obtenerse con varias herramientas o mediante el examen manual del protocolo de enlace TLS. La forma más sencilla es usar `s_client` de OpenSSL:

```bash
$ openssl s_client -connect example:443
[...]
Acceptable client certificate CA names
C = US, ST = Example, L = Example, O = Example Org, CN = Example Org Root Certificate Authority
Client Certificate Types: RSA sign, DSA sign, ECDSA sign
```

Si no hay restricciones, es posible que se pueda autenticar con un certificado de otra CA. Si hay restricciones, pero están mal implementadas, quizá sea posible crear una CA local con el nombre correcto («Example Org Root Certificate Authority» en el ejemplo anterior) y utilizar esta nueva CA para firmar certificados de cliente.

Si se puede obtener un certificado válido, también debe comprobarse que solo pueda utilizarse para el usuario al que se expidió (es decir, que no se pueda usar un certificado expedido a Alice para autenticarse en la cuenta de Bob). Además, debe comprobarse que los certificados no hayan caducado ni hayan sido revocados.

## Casos de prueba relacionados

- [Mecanismo de bloqueo débil](03-Weak_Lock_Out_Mechanism.md)
- [Funcionalidades débiles de cambio o restablecimiento de contraseña](09-Weak_Password_Change_or_Reset_Functionalities.md)

## Remediación

Asegúrese de que:

- La MFA esté implementada para todas las cuentas y funciones pertinentes de las aplicaciones.
- Los métodos de MFA admitidos sean adecuados para la aplicación.
- Los mecanismos utilizados para implementar la MFA estén debidamente protegidos contra ataques de fuerza bruta.
- Existan mecanismos adecuados de auditoría y registro para toda la actividad relacionada con la MFA.

Consulte la [Hoja de referencia de OWASP sobre autenticación multifactor](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) para obtener más recomendaciones.

## Referencias

- [Hoja de referencia de OWASP sobre autenticación multifactor](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)