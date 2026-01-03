# Pruebas de Funcionalidades Débiles de Cambio o Restablecimiento de Contraseña

|ID          |
|------------|
|WSTG-ATHN-09|

## Resumen

Para cualquier aplicación que requiera que el usuario se autentique con una contraseña, debe existir un mecanismo mediante el cual el usuario pueda recuperar el acceso a su cuenta si olvida su contraseña. Aunque esto a veces puede ser un proceso manual que implica contactar al propietario del sitio web o un equipo de soporte, los usuarios suelen poder realizar un restablecimiento de contraseña de autoservicio y recuperar el acceso a su cuenta proporcionando alguna otra evidencia de su identidad.

Dado que esta funcionalidad proporciona una ruta directa para comprometer la cuenta del usuario, es crucial que se implemente de forma segura.

## Objetivos de la Prueba

- Determinar si la funcionalidad de cambio y restablecimiento de contraseña permite comprometer cuentas.

## Cómo Probar

### Recopilación de Información

El primer paso es recopilar información sobre qué mecanismos están disponibles para permitir que el usuario restablezca su contraseña en la aplicación. Si hay múltiples interfaces en el mismo sitio (como una interfaz web, aplicación móvil y API), entonces todas deben revisarse, en caso de que proporcionen funcionalidad diferente.

Una vez establecido esto, determinar qué información se requiere para que un usuario inicie un restablecimiento de contraseña. Esto puede ser el nombre de usuario o dirección de correo electrónico (ambos de los cuales pueden obtenerse de información pública), pero también podría ser un ID de usuario generado internamente.

### Preocupaciones Generales

Independientemente de los métodos específicos utilizados para restablecer contraseñas, hay una serie de áreas comunes que deben considerarse:

- ¿Es el proceso de restablecimiento de contraseña más débil que el proceso de autenticación?

  El proceso de restablecimiento de contraseña proporciona un mecanismo alternativo para acceder a la cuenta de un usuario, y por lo tanto debería ser al menos tan seguro como el proceso de autenticación habitual. Sin embargo, puede proporcionar una forma más fácil de comprometer la cuenta, especialmente si utiliza factores de autenticación más débiles como preguntas de seguridad.

  Además, el proceso de restablecimiento de contraseña puede omitir el requisito de usar Autenticación Multifactor (MFA), lo que puede reducir sustancialmente la seguridad de la aplicación.

- ¿Hay limitación de tasa u otra protección contra ataques automatizados?

  Al igual que con cualquier mecanismo de autenticación, el proceso de restablecimiento de contraseña debería tener protección contra ataques automatizados o de fuerza bruta. Hay una variedad de métodos diferentes que pueden usarse para lograr esto, como limitación de tasa o el uso de CAPTCHA. Estos son particularmente importantes en funcionalidad que activa acciones externas (como enviar un correo electrónico o SMS), o cuando el usuario está ingresando un token de restablecimiento de contraseña.

  También es posible proteger contra ataques de fuerza bruta bloqueando la cuenta del proceso de restablecimiento de contraseña después de un cierto número de intentos consecutivos. Sin embargo, esto también podría impedir que un usuario legítimo pueda restablecer su contraseña y recuperar el acceso a su cuenta.

- ¿Es vulnerable a ataques comunes?

  Además de las áreas específicas discutidas en esta guía, también es importante verificar otras vulnerabilidades comunes como inyección SQL o scripting entre sitios (XSS).

- ¿Permite el proceso de restablecimiento enumeración de usuarios?

  Consulte la guía [Pruebas de Enumeración de Cuentas y Cuentas de Usuario Adivinables](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md) para más información.

### Correo Electrónico - Nueva Contraseña Enviada

En este modelo, se envía una nueva contraseña al usuario por correo electrónico una vez que han probado su identidad. Esto se considera menos seguro por dos razones principales:

- La contraseña se envía al usuario en forma no encriptada.
- La contraseña de la cuenta se cambia cuando se hace la solicitud, bloqueando efectivamente al usuario de su cuenta hasta que reciba el correo electrónico. Al hacer solicitudes repetidas, es posible impedir que un usuario pueda acceder a su cuenta.

Cuando se utiliza este enfoque, las siguientes áreas deben revisarse:

- ¿Se fuerza al usuario a cambiar la contraseña en el inicio de sesión inicial?

  La nueva contraseña se envía por correo electrónico no encriptado, y puede permanecer en la bandeja de entrada del usuario indefinidamente si no elimina el correo electrónico. Como tal, el usuario debería ser requerido a cambiar la contraseña tan pronto como inicie sesión por primera vez.

- ¿Se genera la contraseña de forma segura?

  La contraseña debería generarse utilizando un Generador de Números Pseudoaleatorios Criptográficamente Seguro (CSPRNG), y debería ser lo suficientemente larga para prevenir adivinación de contraseña o ataques de fuerza bruta. Para una experiencia segura y amigable para el usuario, debería generarse utilizando un enfoque de estilo frase segura (es decir, combinando múltiples palabras), en lugar de una cadena de caracteres aleatorios.

- ¿Se envía la contraseña existente del usuario?

  En lugar de generar una nueva contraseña para el usuario, algunas aplicaciones enviarán al usuario su contraseña existente. Este es un enfoque muy inseguro, ya que expone su contraseña actual por correo electrónico no encriptado. Además, si el sitio puede recuperar la contraseña existente, esto implica que las contraseñas se almacenan utilizando encriptación reversible, o (más probablemente) en texto plano no encriptado, ambas representan una debilidad de seguridad grave.

- ¿Se envían los correos electrónicos desde un dominio con protección anti-suplantación?

  El dominio debería implementar SPF, DKIM y DMARC para prevenir que los atacantes suplanten correos electrónicos desde él, lo que podría usarse como parte de un ataque de ingeniería social.

- ¿Se considera el correo electrónico lo suficientemente seguro?

  Los correos electrónicos se envían típicamente no encriptados, y en muchos casos la cuenta de correo electrónico del usuario no estará protegida por MFA. También puede compartirse entre múltiples individuos, particularmente en un entorno corporativo.

  Considere si la funcionalidad de restablecimiento de contraseña basada en correo electrónico es apropiada, basada en el contexto de la aplicación que se está probando.

### Correo Electrónico - Enlace Enviado

En este modelo, se envía por correo electrónico al usuario un enlace que contiene un token. Pueden hacer clic en este enlace y se les solicita ingresar una nueva contraseña en el sitio. Este es el enfoque más común utilizado para restablecimiento de contraseña, pero es más complejo de implementar que el enfoque discutido anteriormente. Las áreas clave a probar son:

- ¿Utiliza el enlace HTTPS?

  Si el token se envía por HTTP no encriptado, puede ser posible para un atacante interceptarlo.

- ¿Puede usarse el enlace múltiples veces?

  Los enlaces deberían expirar después de usarse, de lo contrario proporcionan una puerta trasera persistente para la cuenta.

- ¿Expira el enlace si permanece sin usar?

  Los enlaces deberían tener límite de tiempo. Exactamente cuánto es apropiado dependerá del sitio, pero rara vez debería ser más de una hora.

- ¿Es el token lo suficientemente largo y aleatorio?

  La seguridad del proceso depende enteramente de que un atacante no pueda adivinar o forzar brutalmente un token. Los tokens deberían generarse con un Generador de Números Pseudoaleatorios Criptográficamente Seguro (CSPRNG), y deberían ser lo suficientemente largos que sea impráctico para un atacante adivinar o forzar brutalmente. Al menos 128 bits (o 32 caracteres hexadecimales) es un mínimo suficiente para hacer tal ataque en línea impráctico.

  Los tokens nunca deberían generarse basados en valores conocidos, como tomando el hash MD5 del correo electrónico del usuario con `md5($email)`, o usando GUID que pueden usar funciones PRNG inseguras, o pueden no ser aleatorios dependiendo del tipo.

  Un enfoque alternativo a tokens aleatorios es usar un token firmado criptográficamente como un JWT. En este caso, las verificaciones habituales de JWT deberían llevarse a cabo (¿se verifica la firma, puede usarse el algoritmo "nONe", puede forzarse brutalmente la clave HMAC, etc.). Consulte la guía [Pruebas de Tokens Web JSON](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md) para más información.

- ¿Contiene el enlace un ID de usuario?

  A veces el enlace de restablecimiento de contraseña puede incluir un ID de usuario además de un token, como `reset.php?userid=1&token=123456`. En este caso, puede ser posible modificar el parámetro `userid` para restablecer las contraseñas de otros usuarios.

- ¿Puede inyectarse un encabezado de host diferente?

  Si la aplicación confía en el valor del encabezado `Host` y lo usa para generar el enlace de restablecimiento de contraseña, puede ser posible robar tokens inyectando un encabezado `Host` modificado en la solicitud. Consulte la guía [Pruebas de Inyección de Encabezado de Host](../07-Input_Validation_Testing/17-Testing_for_Host_Header_Injection.md) para más información.

- ¿Se expone el enlace a terceros?

  Si la página a la que se lleva al usuario incluye contenido de otras partes (como cargar scripts de otros dominios), entonces el token de restablecimiento en la URL puede exponerse en el encabezado HTTP `Referer` enviado en estas solicitudes. El encabezado HTTP `Referrer-Policy` puede usarse para proteger contra esto, así que verifique si se define uno para la página.

  Además, si la página incluye cualquier script de seguimiento, análisis o publicidad, el token también se expondrá a ellos.

- ¿Se envían los correos electrónicos desde un dominio con protección anti-suplantación?

  El dominio debería implementar SPF, DKIM y DMARC para prevenir que los atacantes suplanten correos electrónicos desde él, lo que podría usarse como parte de un ataque de ingeniería social.

- ¿Se considera el correo electrónico lo suficientemente seguro?

  Los correos electrónicos se envían típicamente no encriptados, y en muchos casos la cuenta de correo electrónico del usuario no estará protegida por MFA. También puede compartirse entre múltiples individuos, particularmente en un entorno corporativo.

  Considere si la funcionalidad de restablecimiento de contraseña basada en correo electrónico es apropiada, basada en el contexto de la aplicación que se está probando.

### Tokens Enviados por SMS o Llamada Telefónica

En lugar de enviar un token en un correo electrónico, un enfoque alternativo es enviarlo por SMS o una llamada telefónica automatizada, que el usuario luego ingresará en la aplicación. Las áreas clave a probar son:

- ¿Es el token lo suficientemente largo y aleatorio?

  Los tokens enviados de esta manera son típicamente más cortos, ya que están destinados a ser escritos manualmente por el usuario, en lugar de estar incrustados en un enlace. Es bastante común que las aplicaciones usen seis dígitos numéricos, lo que solo proporciona ~20 bits de seguridad (factible para un ataque de fuerza bruta en línea), en lugar del token de correo electrónico típicamente más largo.

  Esto hace que sea mucho más importante que la funcionalidad de restablecimiento de contraseña esté protegida contra ataques de fuerza bruta.

- ¿Puede usarse el token múltiples veces?

  Los tokens deberían invalidarse después de usarse, de lo contrario proporcionan una puerta trasera persistente para la cuenta.

- ¿Expira el token si permanece sin usar?

  Como los tokens más cortos son más susceptibles a ataques de fuerza bruta, debería implementarse un tiempo de expiración más corto para limitar la ventana disponible para que un atacante lleve a cabo un ataque.

- ¿Están en vigor limitaciones de tasa y restricciones apropiadas?

  Enviar un SMS o activar una llamada telefónica automatizada a un usuario es significativamente más disruptivo que enviar un correo electrónico, y podría usarse para acosar a un usuario, o incluso llevar a cabo un ataque de denegación de servicio contra su teléfono. La aplicación debería implementar limitación de tasa para prevenir esto.

  Además, los mensajes SMS y las llamadas telefónicas a menudo incurren en costos financieros para la parte emisora. Si un atacante puede causar que se envíen un gran número de mensajes, esto podría resultar en costos significativos para el operador del sitio web. Esto es especialmente cierto si se envían a números internacionales o de tarifa premium. Sin embargo, permitir números internacionales puede ser un requisito de la aplicación.

- ¿Se considera SMS o una llamada telefónica lo suficientemente segura?

  [Una variedad de ataques](https://www.ncsc.gov.uk/guidance/protecting-sms-messages-used-in-critical-business-processes#section_4) se han demostrado que permitirían a un atacante secuestrar efectivamente mensajes SMS, hay opiniones contradictorias sobre si SMS es lo suficientemente seguro para usarse como factor de autenticación.

  Usualmente es posible responder a una llamada telefónica automatizada con acceso físico a un dispositivo, sin necesidad de ningún tipo de PIN o huella digital para desbloquear el teléfono. En algunas circunstancias (como un entorno de oficina compartida), esto podría permitir a un atacante interno restablecer trivialmente la contraseña de otro usuario caminando a su escritorio cuando están fuera de la oficina.

  Considere si SMS o llamadas telefónicas automatizadas son apropiadas, basadas en el contexto de la aplicación que se está probando.

### Preguntas de Seguridad

En lugar de enviarles un enlace o nueva contraseña, las preguntas de seguridad pueden usarse como mecanismo para autenticar al usuario. Esto se considera un enfoque débil, y no debería usarse si hay mejores opciones disponibles.

Consulte la guía [Pruebas de Respuestas Débiles a Preguntas de Seguridad](08-Testing_for_Weak_Security_Question_Answer.md) para más información.

### Identidad Autenticada y Cambios de Configuración

Si la aplicación soporta la capacidad de modificar el identificador primario de una cuenta (como una dirección de correo electrónico o número de teléfono) que se utiliza en las funcionalidades de cambio y restablecimiento de contraseña, el usuario debería ser forzado a reautenticarse. Cuando el identificador primario utilizado en la funcionalidad de cambio de contraseña puede modificarse sin reautenticación, permite que la reautenticación en la funcionalidad de cambio de contraseña sea omitida. En general, cualquier cosa que impacte la seguridad de la cuenta (correo electrónico, MFA, configuraciones de respaldo, etc.) debería requerir reautenticación antes de poder modificarse.

Por ejemplo: Una aplicación tiene un flujo de restablecimiento de contraseña que envía un enlace de restablecimiento a la dirección de correo electrónico de la cuenta. La aplicación también requiere reautenticación si se intenta cambiar la contraseña desde la perspectiva de un usuario autenticado. Si un atacante gana acceso a la cuenta (a través de una cookie robada, acceso físico a la computadora, etc.) y cambia la dirección de correo electrónico de la cuenta sin necesidad de reautenticarse, entonces el flujo de restablecimiento de contraseña puede usarse para cambiar la contraseña, omitiendo el flujo de cambio de contraseña autenticado.

### Cambios de Contraseña Autenticados

Una vez que el usuario ha probado su identidad (ya sea a través de un enlace de restablecimiento de contraseña, un código de recuperación, o iniciando sesión en la aplicación), deberían poder cambiar su contraseña. Las áreas clave a probar son:

- Al establecer la contraseña, ¿puede especificar el ID de usuario?

  Si el ID de usuario se incluye en la solicitud de restablecimiento de contraseña y no se valida, puede ser posible modificarlo y cambiar las contraseñas de otros usuarios.

- ¿Se requiere que el usuario se reautentique?

  Si un usuario conectado intenta cambiar su contraseña, deberían ser preguntados para reautenticarse con su contraseña actual para proteger contra un atacante ganando acceso temporal a una sesión desatendida. Si el usuario tiene MFA habilitado, entonces típicamente se reautenticarían con eso, en lugar de su contraseña.

- ¿Es el formulario de cambio de contraseña vulnerable a CSRF?

  Si el usuario no es requerido a reautenticarse, entonces puede ser posible llevar a cabo un ataque CSRF contra el formulario de restablecimiento de contraseña, permitiendo que su cuenta sea comprometida. Consulte la guía [Pruebas de Falsificación de Solicitud entre Sitios](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md) para más información.

- ¿Se aplica una política de contraseña fuerte y efectiva?

  La política de contraseña debería ser consistente en la funcionalidad de registro, cambio de contraseña y restablecimiento de contraseña. Consulte la guía [Pruebas de Métodos de Autenticación Débiles](07-Testing_for_Weak_Authentication_Methods.md) para más información.

## Referencias

- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)