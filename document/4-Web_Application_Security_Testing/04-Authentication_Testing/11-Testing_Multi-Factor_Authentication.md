# Pruebas de Autenticación Multifactor (MFA)

|ID          |
|------------|
|WSTG-ATHN-11|

## Resumen

Muchas aplicaciones implementan la Autenticación Multifactor (MFA) como una capa adicional de seguridad para proteger el proceso de inicio de sesión. Esto también se conoce como autenticación de dos factores (2FA) o verificación en dos pasos (2SV), aunque estos no son estrictamente lo mismo. MFA significa pedir al usuario que proporcione *al menos* dos [factores de autenticación](#tipos-de-mfa) diferentes al iniciar sesión.

MFA añade complejidad adicional tanto a la funcionalidad de autenticación como a otras áreas relacionadas con la seguridad (como la gestión de credenciales y la recuperación de contraseñas), lo que significa que es crítico que se implemente de manera correcta y robusta.

## Objetivos de la Prueba

- Identificar el tipo de MFA utilizado por la aplicación.
- Determinar si la implementación de MFA es robusta y segura.
- Intentar eludir la MFA.

## Cómo Probar

### Tipos de MFA

MFA significa que *al menos* dos de los siguientes factores son requeridos para la autenticación:

| Factor | Ejemplos |
|--------|----------|
| Algo que sabes | Contraseñas, PIN y preguntas de seguridad. |
| Algo que tienes | Tokens de hardware o software, certificados, correo electrónico*, SMS y llamadas telefónicas. |
| Algo que eres | Huellas dactilares, reconocimiento facial, escaneos de iris, escaneos de palma y factores conductuales. |
| Ubicación | Rangos de IP de origen y geolocalización. |

\* El correo electrónico solo constituye realmente "algo que tienes" si la cuenta de correo electrónico en sí está protegida con MFA. Como tal, debe considerarse más débil que otras alternativas como certificados o TOTP, y puede no aceptarse como MFA bajo algunas definiciones.

Tenga en cuenta que requerir múltiples ejemplos de un solo factor (como necesitar tanto una contraseña como un PIN) **no constituye MFA**, aunque puede proporcionar algunos beneficios de seguridad sobre una simple contraseña, y puede considerarse verificación en dos pasos (2SV).

Debido a la complejidad de implementar biometría en un entorno basado en navegador, "Algo que eres" rara vez se usa para aplicaciones web, aunque está comenzando a adoptarse usando estándares como WebAuthn. El segundo factor más común es "Algo que tienes".

### Verificar Elusiones de MFA

El primer paso para probar MFA es identificar toda la funcionalidad de autenticación en la aplicación, que puede incluir:

- La página de inicio de sesión principal.
- Funcionalidad crítica para la seguridad (como deshabilitar MFA o cambiar una contraseña).
- Proveedores de inicio de sesión federados.
- Puntos finales de API (desde tanto la interfaz web principal como aplicaciones móviles).
- Protocolos alternativos (no HTTP).
- Funcionalidad de prueba o depuración.

Todos los diferentes métodos de inicio de sesión deben revisarse para asegurar que MFA se aplique de manera consistente. Si algunos métodos no requieren MFA, entonces estos pueden proporcionar un método simple para eludirlos.

Si la autenticación se hace en múltiples pasos, entonces puede ser posible eludirla completando el primer paso del proceso de autenticación (ingresando el nombre de usuario y la contraseña), y luego navegando forzosamente a la aplicación o haciendo solicitudes directas a la API sin completar la segunda etapa (ingresando el código MFA).

Si la autenticación usa un proveedor de OpenID Connect (OIDC) que permite flujos de autenticación personalizados (o políticas) como Azure B2C, puede haber múltiples flujos definidos, algunos de los cuales pueden no requerir MFA. Por ejemplo, si la aplicación se autentica con un flujo llamado `B2C_1_SignInWithMFA`, entonces intente manipularlo a `B2C_1_SignIn`, `B2C_1_SignInWithoutMFA` u otros valores similares.

En algunos casos, también puede haber elusiones intencionales de MFA implementadas, como no requerir MFA:

- Desde direcciones IP específicas (que pueden ser falsificables usando el encabezado HTTP `X-Forwarded-For`).
- Cuando se establece un encabezado HTTP específico (como un encabezado no estándar como `X-Debug`).
- Para una cuenta específica codificada (como una cuenta "root" o "breakglass").

Donde una aplicación soporta tanto inicios de sesión locales como federados, puede ser posible eludir la MFA si no hay una separación fuerte entre estos dos tipos de cuentas. Por ejemplo, si un usuario registra una cuenta local y configura MFA para ella, pero no tiene MFA configurado en su cuenta en el proveedor de inicio de sesión federado, puede ser posible para un atacante volver a registrar (o vincular) una cuenta federada en la aplicación objetivo con la misma dirección de correo electrónico comprometiendo la cuenta del usuario en el proveedor de inicio de sesión federado.

Finalmente, si la MFA se implementa en un sistema diferente a la aplicación principal (como en un proxy inverso, para proteger una aplicación heredada que no soporta MFA de forma nativa), entonces puede ser posible eludirla conectando directamente al servidor de aplicación backend, como se discute en la guía sobre cómo [mapear la arquitectura de la aplicación](../01-Information_Gathering/10-Map_Application_Architecture.md#content-delivery-network-cdn).

### Verificar Gestión de MFA

La funcionalidad usada para gestionar MFA desde dentro de la cuenta del usuario debe probarse para vulnerabilidades, incluyendo:

- ¿Se requiere que el usuario se reautentique para eliminar o cambiar configuraciones de MFA?
- ¿Es la funcionalidad de gestión de MFA vulnerable a [falsificación de solicitud entre sitios](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)?
- ¿Pueden modificarse las configuraciones de MFA de otros usuarios a través de vulnerabilidades [IDOR](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)?

### Verificar Opciones de Recuperación de MFA

Muchas aplicaciones proporcionarán a los usuarios una forma de recuperar el acceso a su cuenta si no pueden autenticarse con su segundo factor (por ejemplo, si han perdido su teléfono). Estos mecanismos pueden representar a menudo una debilidad significativa en la aplicación, ya que efectivamente permiten eludir el segundo factor de autenticación.

#### Códigos de Recuperación

Algunas aplicaciones proporcionarán al usuario una lista de códigos de recuperación o de respaldo cuando habiliten MFA, que pueden usarse para iniciar sesión. Estos deben verificarse para asegurar:

- Que son suficientemente largos y complejos para proteger contra ataques de fuerza bruta.
- Que se generan de manera segura.
- Que solo pueden usarse una vez.
- Que hay protección contra fuerza bruta (como bloqueo de cuenta).
- Que el usuario es notificado (vía correo electrónico, SMS, etc.) cuando se usa un código.

Vea la ["sección de Códigos de Respaldo" en la Hoja de Trucos de Contraseña Olvidada](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html#backup-codes) para más detalles.

#### Proceso de Restablecimiento de MFA

Si la aplicación implementa un proceso de restablecimiento de MFA, este debe probarse de la misma manera que se prueba el [proceso de restablecimiento de contraseña](09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md). Es importante que este proceso sea *al menos* tan fuerte como la implementación de MFA para la aplicación.

#### Autenticación Alternativa

Algunas aplicaciones permitirán al usuario probar su identidad a través de otros medios, como el uso de [preguntas de seguridad](08-Testing_for_Weak_Security_Question_Answer.md). Esto usualmente representa una debilidad significativa, ya que las preguntas de seguridad proporcionan un nivel de seguridad mucho menor que MFA.

### Contraseñas de un Solo Uso

La forma más común de MFA es la de Contraseñas de un Solo Uso (OTP), que son típicamente códigos numéricos de seis dígitos (aunque pueden ser más largos o más cortos). Estos pueden generarse tanto por el servidor como por el usuario (por ejemplo, con una aplicación autenticadora), o pueden generarse en el servidor y enviarse al usuario. Hay varias formas en que esta OTP puede proporcionarse al usuario, incluyendo:

| Tipo | Descripción |
|------|-------------|
| Contraseña de un Solo Uso basada en HMAC (HOTP) | Genera un código basado en el HMAC de un secreto y un contador compartido. |
| Contraseña de un Solo Uso basada en Tiempo (TOTP) | Genera un código basado en HMAC de un secreto y la hora actual. |
| Correo electrónico | Envía un código vía correo electrónico. |
| SMS | Envía un código vía SMS. |
| Teléfono | Envía un código vía llamada de voz a un número de teléfono. |

La OTP se ingresa típicamente después de que el usuario ha proporcionado su nombre de usuario y contraseña. Hay varias verificaciones que deben realizarse, incluyendo:

- ¿Se bloquea la cuenta después de múltiples intentos fallidos de MFA?
- ¿Se bloquea la dirección IP del usuario después de múltiples intentos fallidos de MFA en diferentes cuentas?
- ¿Se registran los intentos fallidos de MFA?
- ¿Es el formulario vulnerable a ataques de inyección, incluyendo [inyección de comodines SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection)?

Dependiendo del tipo de OTP usadas, también hay algunas otras verificaciones específicas que deben realizarse:

- Cómo se envían las OTP al usuario (correo electrónico, SMS, teléfono, etc.)
    - ¿Hay limitación de tasa para prevenir spam de SMS/teléfono que cueste dinero?
- ¿Qué tan fuertes son las OTP (longitud y espacio de claves)?
- ¿Cuánto tiempo son válidas las OTP?
- ¿Son múltiples OTP válidas a la vez?
- ¿Pueden usarse las OTP más de una vez?
- ¿Están las OTP vinculadas a la cuenta de usuario correcta o es posible autenticarse con ellas en otras cuentas?

#### HOTP y TOTP

Los códigos HOTP y TOTP están basados en un secreto compartido entre el servidor y el usuario. Para códigos TOTP, esto se proporciona usualmente al usuario en forma de un código QR que escanean con una aplicación autenticadora (aunque también puede proporcionarse como un secreto de texto para que lo ingresen manualmente).

Donde el secreto se genera en el servidor, debe verificarse para asegurar que es suficientemente largo y complejo ([RFC 4226](https://www.rfc-editor.org/rfc/rfc4226#section-4) recomienda al menos 160 bits), y que se genera usando una [función aleatoria segura](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation).

Donde el secreto puede proporcionarse por el usuario, debe aplicarse una longitud mínima apropiada, y la entrada debe verificarse para los ataques de inyección usuales.

Los códigos TOTP son típicamente válidos por 30 segundos, pero algunas aplicaciones eligen aceptar múltiples códigos (como los anteriores, actuales y siguientes) para lidiar con diferencias entre la hora del sistema en el servidor y en el dispositivo del usuario. Algunas aplicaciones pueden permitir múltiples códigos en cualquier lado del actual, lo que puede hacer más fácil para un atacante adivinar o forzar bruta el código. La tabla a continuación muestra la probabilidad de forzar bruta exitosamente un código OTP basado en un atacante siendo capaz de hacer 10 solicitudes por segundo, para aplicaciones que aceptan ya sea solo el código actual, o múltiples códigos (vea [este artículo](https://www.codasecurity.co.uk/articles/mfa-testing#case-study---brute-forcing-totp) para los cálculos detrás de la tabla).

| Códigos Válidos | Tasa de éxito después de 1 hora | Tasa de éxito después de 4 horas | Tasa de éxito después de 12 horas | Tasa de éxito después de 24 horas |
|-----------------|---------------------------------|----------------------------------|-----------------------------------|-----------------------------------|
| 1 | 4%  | 13% | 35% | 58% |
| 3 | 10% | 35% | 72% | 92% |
| 5 | 16% | 51% | 88% | 99% |
| 7 | 22% | 63% | 95% | 99% |

#### Correo Electrónico, SMS y Teléfono

Donde los códigos se generan en el servidor y se envían al cliente, las siguientes áreas deben considerarse:

- ¿Es el mecanismo de transporte (correo electrónico, SMS o voz) lo suficientemente seguro para la aplicación?
- ¿Son los códigos suficientemente largos y complejos?
- ¿Se generan los códigos usando una [función aleatoria segura](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation)?
- ¿Cuánto tiempo son válidos los códigos?
- ¿Son múltiples códigos válidos a la vez, o genera un nuevo código invalida el anterior?
    - ¿Podría esto usarse para bloquear el acceso a una cuenta solicitando códigos repetidamente?
- ¿Hay suficiente limitación de tasa para prevenir que un atacante solicite grandes números de códigos?
    - Grandes números de códigos por correo electrónico pueden hacer que el servidor sea bloqueado por enviar spam.
    - Grandes números de SMS o llamadas de voz pueden costar dinero, o usarse para acosar a un usuario.

### Aplicaciones Móviles y Notificaciones Push

Un enfoque alternativo a los códigos OTP es enviar una notificación push al teléfono móvil del usuario, que pueden aprobar o denegar. Este método es menos común, ya que requiere que el usuario instale una aplicación autenticadora específica.

Evaluar correctamente la seguridad de esto requiere expandir el alcance de las pruebas para cubrir tanto la aplicación móvil como cualquier API o servicios de soporte usados por ella; lo que significa que a menudo estaría fuera del alcance de una prueba de aplicación web tradicional. Sin embargo, hay un par de verificaciones simples que pueden realizarse sin probar la aplicación móvil, incluyendo:

- ¿Proporciona la notificación suficiente contexto (direcciones IP, ubicación, etc.) para que el usuario tome una decisión informada sobre aprobar o denegar?
- ¿Hay algún tipo de mecanismo de desafío y respuesta (como proporcionar un código en el sitio que el usuario necesita ingresar en la aplicación - a menudo llamado "coincidencia de números" o "desafío de números")?
- ¿Hay alguna limitación de tasa o mecanismos para prevenir que el usuario sea spameado con notificaciones con la esperanza de que simplemente acepte una ciegamente?

### Filtrado de Dirección IP y Ubicación

Uno de los factores que a veces se usa con MFA es la ubicación ("algún lugar donde estás"), aunque si esto constituye un factor de autenticación apropiado es debatible. En el contexto de una aplicación web, esto típicamente significa restringir el acceso a direcciones IP específicas, o no pedir al usuario un segundo factor mientras se conectan desde una dirección IP específica de confianza. Un escenario común para esto sería autenticar usuarios con solo su contraseña cuando se conectan desde los rangos de IP de la oficina, pero requiriendo un código OTP cuando se conectan desde otro lugar.

Dependiendo de la implementación, puede ser posible para un usuario falsificar una dirección IP de confianza estableciendo el encabezado `X-Forwarded-For`, lo que podría permitirles eludir esta verificación. Tenga en cuenta que si la aplicación no sanitiza correctamente el contenido de este encabezado, también puede ser posible llevar a cabo ataques como inyección SQL aquí. Si la aplicación soporta IPv6, entonces esto también debe verificarse para asegurar que se aplican restricciones apropiadas a esas conexiones.

Adicionalmente, las direcciones IP de confianza deben revisarse para asegurar que no presentan debilidades, como si incluyen:

- Direcciones IP que podrían ser accesibles por usuarios no confiables (como las redes inalámbricas de invitados en una oficina).
- Direcciones IP asignadas dinámicamente que podrían cambiar.
- Rangos de red pública donde un atacante podría alojar su propio sistema (como Azure o AWS).

### Certificados y Tarjetas Inteligentes

La Seguridad de la Capa de Transporte (TLS) se usa comúnmente para cifrar el tráfico entre el cliente y el servidor, y para proporcionar un mecanismo para que el cliente confirme la identidad del servidor (comparando el Nombre Común (CN) o Nombre Alternativo del Sujeto (SAN) en el certificado con el dominio solicitado). Sin embargo, también puede proporcionar un mecanismo para que el servidor confirme la identidad del cliente, conocido como autenticación de certificado de cliente o TLS mutuo (mTLS). Una discusión completa de la autenticación de certificado de cliente está fuera del alcance de esta guía, pero el principio clave es que el usuario presenta un certificado digital (almacenado ya sea en su máquina o en una tarjeta inteligente), que es validado por el servidor.

El primer paso al probar es determinar si la aplicación objetivo restringe las Autoridades de Certificación (CA) que son confiables para emitir certificados. Esta información puede obtenerse usando varias herramientas, o examinando manualmente el handshake TLS. La forma más simple es usar el `s_client` de OpenSSL:

```bash
$ openssl s_client -connect example:443
[...]
Acceptable client certificate CA names
C = US, ST = Example, L = Example, O = Example Org, CN = Example Org Root Certificate Authority
Client Certificate Types: RSA sign, DSA sign, ECDSA sign
```

Si no hay restricciones, entonces puede ser posible autenticarse usando un certificado de una CA diferente. Si hay restricciones pero están mal implementadas, puede ser posible crear una CA local con el nombre correcto ("Example Org Root Certificate Authority" en el ejemplo anterior), y usar esta nueva CA para firmar certificados de cliente.

Si se puede obtener un certificado válido, entonces también debe verificarse que el certificado solo puede usarse para el usuario al que se emite (es decir, que no se puede usar un certificado emitido a Alice para autenticarse en la cuenta de Bob). Adicionalmente, los certificados deben verificarse para asegurar que no han expirado ni han sido revocados.

## Casos de Prueba Relacionados

- [Pruebas de Mecanismo de Bloqueo Débil](03-Testing_for_Weak_Lock_Out_Mechanism.md)
- [Pruebas de Funcionalidades de Cambio o Restablecimiento de Contraseña Débil](09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md)

## Remediación

Asegúrese de que:

- MFA se implementa para todas las cuentas y funcionalidad relevantes en las aplicaciones.
- Los métodos de soporte de MFA son apropiados para la aplicación.
- Los mecanismos usados para implementar MFA están apropiadamente asegurados y protegidos contra ataques de fuerza bruta.
- Hay auditoría y registro apropiados para toda actividad relacionada con MFA.

Vea la [Hoja de Trucos de Autenticación Multifactor de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) para más recomendaciones.

## Referencias

- [Hoja de Trucos de Autenticación Multifactor de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)