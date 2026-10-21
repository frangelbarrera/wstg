# Pruebas de funcionalidades débiles de cambio o restablecimiento de contraseña

|ID          |
|------------|
|WSTG-ATHN-09|

## Resumen

Para cualquier aplicación que requiera que el usuario se autentique con una contraseña, debe existir un mecanismo mediante el cual el usuario pueda recuperar el acceso a su cuenta si olvida su contraseña. Aunque a veces puede tratarse de un proceso manual que implique contactar al propietario del sitio web o a un equipo de soporte, con frecuencia se permite a los usuarios restablecer su contraseña por sí mismos y recuperar el acceso a su cuenta proporcionando alguna otra prueba de su identidad.

Dado que esta funcionalidad proporciona una vía directa para comprometer la cuenta del usuario, es crucial que se implemente de forma segura.

## Objetivos de la prueba

- Determinar si la funcionalidad de cambio y restablecimiento de contraseña permite comprometer cuentas.

## Cómo probar

### Recopilación de información

El primer paso consiste en recopilar información sobre los mecanismos disponibles para que el usuario restablezca su contraseña en la aplicación. Si hay varias interfaces en el mismo sitio (como una interfaz web, una aplicación móvil y una API), deben revisarse todas, ya que podrían proporcionar distintas funcionalidades.

Una vez establecido esto, determine qué información se requiere para que un usuario inicie un restablecimiento de contraseña. Puede ser el nombre de usuario o la dirección de correo electrónico (ambos pueden obtenerse de información pública), pero también podría ser un ID de usuario generado internamente.

### Consideraciones generales

Independientemente de los métodos específicos utilizados para restablecer contraseñas, hay varias cuestiones comunes que deben tenerse en cuenta:

- ¿Es el proceso de restablecimiento de contraseña más débil que el proceso de autenticación?

  El proceso de restablecimiento de contraseña proporciona un mecanismo alternativo para acceder a la cuenta de un usuario y, por lo tanto, debería ser al menos tan seguro como el proceso de autenticación habitual. Sin embargo, puede ofrecer una forma más fácil de comprometer la cuenta, especialmente si utiliza factores de autenticación más débiles, como preguntas de seguridad.

  Además, el proceso de restablecimiento de contraseña puede omitir el requisito de usar la autenticación multifactor (MFA), lo que puede reducir considerablemente la seguridad de la aplicación.

- ¿Hay limitación de tasa u otra protección contra ataques automatizados?

  Al igual que cualquier mecanismo de autenticación, el proceso de restablecimiento de contraseña debería estar protegido contra ataques automatizados o de fuerza bruta. Hay varios métodos para conseguirlo, como la limitación de tasa o el uso de CAPTCHA. Estas medidas son especialmente importantes en funcionalidades que desencadenan acciones externas (como enviar un correo electrónico o un SMS) o cuando el usuario introduce un token de restablecimiento de contraseña.

  También es posible protegerse contra ataques de fuerza bruta bloqueando el acceso de la cuenta al proceso de restablecimiento de contraseña después de cierto número de intentos consecutivos. Sin embargo, esto también podría impedir que un usuario legítimo restablezca su contraseña y recupere el acceso a su cuenta.

- ¿Es vulnerable a ataques comunes?

  Además de las cuestiones específicas que se analizan en esta guía, también es importante comprobar otras vulnerabilidades comunes, como la inyección SQL o el scripting entre sitios (XSS).

- ¿Permite el proceso de restablecimiento enumerar usuarios?

  Consulte la guía [Enumeración de cuentas](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md) para obtener más información.

### Correo electrónico: envío de una nueva contraseña

En este modelo, se envía una nueva contraseña al usuario por correo electrónico una vez que ha demostrado su identidad. Se considera menos seguro por dos razones principales:

- La contraseña se envía al usuario sin cifrar.
- La contraseña de la cuenta se cambia al realizar la solicitud, lo que, en la práctica, impide que el usuario acceda a su cuenta hasta que reciba el correo electrónico. Si se realizan solicitudes repetidas, es posible impedir que el usuario acceda a su cuenta.

Si se utiliza este enfoque, deben revisarse los siguientes aspectos:

- ¿Se obliga al usuario a cambiar la contraseña al iniciar sesión por primera vez?

  La nueva contraseña se envía por correo electrónico sin cifrar y puede permanecer indefinidamente en la bandeja de entrada del usuario si este no elimina el mensaje. Por ello, se debería exigir al usuario que cambie la contraseña en cuanto inicie sesión por primera vez.

- ¿Se genera la contraseña de forma segura?

  La contraseña debería generarse mediante un generador de números pseudoaleatorios criptográficamente seguro (CSPRNG) y ser lo suficientemente larga para impedir que se adivine o se obtenga mediante ataques de fuerza bruta. Para ofrecer una experiencia segura y fácil de usar, debería generarse mediante un método seguro basado en frases de contraseña (es decir, combinando varias palabras), en lugar de una cadena de caracteres aleatorios.

- ¿Se envía al usuario su contraseña actual?

  En lugar de generar una contraseña nueva, algunas aplicaciones envían al usuario su contraseña actual. Se trata de un enfoque muy inseguro, ya que expone la contraseña actual del usuario mediante un correo electrónico sin cifrar. Además, si el sitio puede recuperar la contraseña actual, esto implica que las contraseñas se almacenan con cifrado reversible o, lo que es más probable, como texto sin cifrar; ambas opciones representan una grave debilidad de seguridad.

- ¿Se envían los correos electrónicos desde un dominio con protección contra la suplantación?

  El dominio debería implementar SPF, DKIM y DMARC para impedir que los atacantes suplanten correos electrónicos de ese dominio, lo que podría utilizarse como parte de un ataque de ingeniería social.

- ¿Se considera suficientemente seguro el correo electrónico?

  Los correos electrónicos suelen enviarse sin cifrar y, en muchos casos, la cuenta de correo electrónico del usuario no está protegida por MFA. También puede compartirse entre varias personas, sobre todo en un entorno corporativo.

  Considere si la funcionalidad de restablecimiento de contraseña por correo electrónico es apropiada teniendo en cuenta el contexto de la aplicación que se está probando.

### Correo electrónico: envío de un enlace

En este modelo, se envía al usuario por correo electrónico un enlace que contiene un token. El usuario puede hacer clic en el enlace y se le solicita que introduzca una nueva contraseña en el sitio. Este es el método más utilizado para restablecer contraseñas, pero es más complejo de implementar que el método descrito anteriormente. Los aspectos clave que se deben probar son:

- ¿El enlace utiliza HTTPS?

  Si el token se envía mediante HTTP sin cifrar, un atacante podría interceptarlo.

- ¿Se puede utilizar el enlace varias veces?

  Los enlaces deberían caducar después de usarse; de lo contrario, proporcionan una puerta trasera persistente a la cuenta.

- ¿Caduca el enlace si no se utiliza?

  Los enlaces deberían tener una vigencia limitada. La duración apropiada dependerá del sitio, pero rara vez debería ser superior a una hora.

- ¿El token es lo suficientemente largo y aleatorio?

  La seguridad del proceso depende por completo de que un atacante no pueda adivinar el token ni obtenerlo mediante fuerza bruta. Los tokens deberían generarse con un generador de números pseudoaleatorios criptográficamente seguro (CSPRNG) y ser lo suficientemente largos para que resulte inviable adivinarlos u obtenerlos mediante fuerza bruta. Un mínimo de 128 bits (o 32 caracteres hexadecimales) es suficiente para que un ataque en línea de este tipo resulte inviable.

  Los tokens nunca deberían generarse a partir de valores conocidos, por ejemplo, calculando el hash MD5 del correo electrónico del usuario con `md5($email)`, ni utilizando GUID que puedan emplear funciones PRNG inseguras o que quizá ni siquiera sean aleatorios, según el tipo.

  Los evaluadores también deberían verificar cómo se almacenan los tokens de restablecimiento de contraseña en el servidor. Si se almacenan como texto sin cifrar en la base de datos, un atacante que obtenga acceso a esta podría reutilizarlos para restablecer las contraseñas de los usuarios. Una implementación más segura almacena una versión hasheada del token y compara el hash durante la validación.

  Una alternativa a los tokens aleatorios es utilizar un token firmado criptográficamente, como un JWT. En ese caso, deberían realizarse las comprobaciones habituales de JWT (¿se verifica la firma?, ¿se puede usar el algoritmo "nONe"?, ¿se puede obtener la clave HMAC mediante fuerza bruta?, etc.). Consulte la guía [Pruebas de JSON Web Tokens](../06-Session_Management/10-JSON_Web_Tokens.md) para obtener más información.

- ¿El enlace contiene un ID de usuario?

  A veces, el enlace de restablecimiento de contraseña puede incluir un ID de usuario además de un token, como `reset.php?userid=1&token=123456`. En ese caso, podría ser posible modificar el parámetro `userid` para restablecer las contraseñas de otros usuarios.

- ¿Se puede inyectar un encabezado `Host` diferente?

  Si la aplicación confía en el valor del encabezado `Host` y lo utiliza para generar el enlace de restablecimiento de contraseña, podría ser posible robar tokens inyectando un encabezado `Host` modificado en la solicitud. Consulte la guía [Inyección del encabezado Host](../07-Injection/17-Host_Header_Injection.md) para obtener más información.

- ¿Se expone el enlace a terceros?

  Compruebe si el token de restablecimiento puede quedar expuesto en el encabezado HTTP `Referer` cuando la página de restablecimiento carga recursos de terceros.

  Cuando un enlace de restablecimiento de contraseña contiene el token en la URL, como `https://example.com/reset?token=ABC123`, el token puede incluirse en el encabezado HTTP `Referer` cuando la página carga recursos externos, como scripts de análisis, imágenes o JavaScript de dominios de terceros.

  Compruebe si el token de restablecimiento se filtra a dominios externos inspeccionando las solicitudes de red que realiza la página de restablecimiento.

  1. Inicie una solicitud de restablecimiento de contraseña y abra el enlace de restablecimiento.
  2. Inspeccione las solicitudes de red con las herramientas para desarrolladores del navegador o con un proxy de interceptación, como Burp Suite o ZAP.
  3. Identifique las solicitudes realizadas a dominios externos.
  4. Compruebe si la URL completa de restablecimiento, incluido el token, aparece en el encabezado `Referer` de esas solicitudes.
  5. Compruebe si la página de restablecimiento establece un encabezado de respuesta `Referrer-Policy` adecuado.

  Si los dominios de terceros reciben el token, un atacante que controle esos recursos puede capturarlo y restablecer la contraseña de la víctima.

- ¿Se envían los correos electrónicos desde un dominio con protección contra la suplantación?

  El dominio debería implementar SPF, DKIM y DMARC para impedir que los atacantes suplanten correos electrónicos de ese dominio, lo que podría utilizarse como parte de un ataque de ingeniería social.

- ¿Se considera suficientemente seguro el correo electrónico?

  Los correos electrónicos suelen enviarse sin cifrar y, en muchos casos, la cuenta de correo electrónico del usuario no está protegida por MFA. También puede compartirse entre varias personas, sobre todo en un entorno corporativo.

  Considere si la funcionalidad de restablecimiento de contraseña por correo electrónico es apropiada teniendo en cuenta el contexto de la aplicación que se está probando.

### Tokens enviados por SMS o llamada telefónica

En lugar de enviar un token por correo electrónico, otra opción es enviarlo por SMS o mediante una llamada telefónica automatizada, para que el usuario lo introduzca en la aplicación. Los aspectos clave que se deben probar son:

- ¿El token es lo suficientemente largo y aleatorio?

  Los tokens enviados de esta manera suelen ser más cortos, ya que están pensados para que el usuario los escriba manualmente, en lugar de incluirlos en un enlace. Es bastante común que las aplicaciones utilicen seis dígitos numéricos, lo que solo proporciona ~20 bits de seguridad (una cantidad viable para un ataque de fuerza bruta en línea), en lugar del token de correo electrónico, que suele ser más largo.

  Esto hace que sea mucho más importante proteger la funcionalidad de restablecimiento de contraseña contra ataques de fuerza bruta.

- ¿Se puede utilizar el token varias veces?

  Los tokens deberían invalidarse después de usarse; de lo contrario, proporcionan una puerta trasera persistente a la cuenta.

- ¿Caduca el token si no se utiliza?

  Como los tokens más cortos son más vulnerables a los ataques de fuerza bruta, debería establecerse un tiempo de expiración más breve para limitar el tiempo disponible para que un atacante lleve a cabo un ataque.

- ¿Se aplican límites de tasa y restricciones apropiados?

  Enviar un SMS o iniciar una llamada telefónica automatizada a un usuario resulta mucho más perturbador que enviar un correo electrónico, y podría utilizarse para acosar al usuario o incluso llevar a cabo un ataque de denegación de servicio contra su teléfono. La aplicación debería implementar limitación de tasa para impedirlo.

  Además, los mensajes SMS y las llamadas telefónicas suelen generar costos para quien los envía. Si un atacante puede provocar el envío de una gran cantidad de mensajes, esto podría generar costos importantes para el operador del sitio web. Esto es especialmente cierto si se envían a números internacionales o de tarifa premium. Sin embargo, permitir números internacionales puede ser un requisito de la aplicación.

- ¿Se consideran suficientemente seguros los SMS o las llamadas telefónicas?

  [Se ha demostrado una variedad de ataques](https://www.ncsc.gov.uk/guidance/protecting-sms-messages-used-in-critical-business-processes#section_4) que permitirían a un atacante secuestrar mensajes SMS de forma efectiva; hay opiniones contradictorias sobre si los SMS son suficientemente seguros para utilizarse como factor de autenticación.

  Normalmente es posible contestar una llamada telefónica automatizada si se tiene acceso físico al dispositivo, sin necesidad de ningún PIN ni huella dactilar para desbloquear el teléfono. En ciertas circunstancias (como en una oficina compartida), esto podría permitir a un atacante interno restablecer fácilmente la contraseña de otro usuario acercándose a su escritorio cuando este se encuentre fuera de la oficina.

  Considere si los SMS o las llamadas telefónicas automatizadas son apropiados teniendo en cuenta el contexto de la aplicación que se está probando.

### Preguntas de seguridad

En lugar de enviar un enlace o una contraseña nueva, se pueden utilizar preguntas de seguridad como mecanismo para autenticar al usuario. Se considera un método débil y no debería utilizarse si hay opciones mejores disponibles.

Consulte la guía [Preguntas y respuestas de seguridad débiles](08-Weak_Security_Question_Answer.md) para obtener más información.

### Identidad autenticada y cambios de configuración

Si la aplicación permite modificar el identificador principal de una cuenta (como una dirección de correo electrónico o un número de teléfono) que se utiliza en las funcionalidades de cambio y restablecimiento de contraseña, se debería obligar al usuario a volver a autenticarse. Si el identificador principal utilizado en la funcionalidad de cambio de contraseña puede modificarse sin volver a autenticarse, se puede eludir la reautenticación requerida para cambiar la contraseña. En general, cualquier cambio que afecte a la seguridad de la cuenta (correo electrónico, MFA, configuración de copias de seguridad, etc.) debería requerir una reautenticación antes de poder realizarse.

Por ejemplo: Una aplicación tiene un flujo de restablecimiento de contraseña que envía un enlace de restablecimiento a la dirección de correo electrónico de la cuenta. La aplicación también exige la reautenticación si un usuario autenticado intenta cambiar su contraseña. Si un atacante obtiene acceso a la cuenta (mediante una cookie robada, acceso físico al equipo, etc.) y cambia la dirección de correo electrónico de la cuenta sin tener que volver a autenticarse, podrá utilizar el flujo de restablecimiento de contraseña para cambiar la contraseña y eludir el flujo autenticado de cambio de contraseña.

### Cambios de contraseña autenticados

Una vez que el usuario ha demostrado su identidad (ya sea mediante un enlace de restablecimiento de contraseña, un código de recuperación o iniciando sesión en la aplicación), debería poder cambiar su contraseña. Los aspectos clave que se deben probar son:

- Al establecer la contraseña, ¿se puede especificar el ID de usuario?

  Si el ID de usuario se incluye en la solicitud de restablecimiento de contraseña y no se valida, podría ser posible modificarlo y cambiar las contraseñas de otros usuarios.

- ¿Se exige al usuario que vuelva a autenticarse?

  Si un usuario con sesión iniciada intenta cambiar su contraseña, se le debería pedir que vuelva a autenticarse con su contraseña actual para protegerse frente a un atacante que obtenga acceso temporal a una sesión desatendida. Si el usuario tiene MFA habilitado, normalmente debería volver a autenticarse mediante ese factor, en lugar de con su contraseña.

- ¿El formulario de cambio de contraseña es vulnerable a CSRF?

  Si no se exige al usuario que vuelva a autenticarse, podría ser posible llevar a cabo un ataque CSRF contra el formulario de restablecimiento de contraseña y comprometer su cuenta. Consulte la guía [Falsificación de solicitudes entre sitios](../06-Session_Management/05-Cross_Site_Request_Forgery.md) para obtener más información.

- ¿Se aplica una política de contraseñas sólida y eficaz?

  La política de contraseñas debería ser coherente en las funcionalidades de registro, cambio de contraseña y restablecimiento de contraseña. Consulte la guía [Métodos de autenticación débiles](07-Weak_Authentication_Methods.md) para obtener más información.

## Remediación

Asegúrese de que:

- Los flujos de cambio y restablecimiento de contraseña sean al menos tan seguros como la autenticación habitual y no permitan eludir MFA.
- Los tokens de restablecimiento tengan alta entropía, sean de un solo uso y de corta duración, y se almacenen como hashes; los enlaces de restablecimiento utilicen HTTPS y no se construyan a partir de un encabezado `Host` no confiable.
- Las páginas de restablecimiento no filtren tokens a terceros a través del encabezado `Referer`; establezca una `Referrer-Policy` apropiada y evite los recursos de terceros en esas páginas. Consulte [Otras configuraciones incorrectas de encabezados de seguridad HTTP](../02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations.md).
- Se apliquen limitación de tasa, reautenticación para los cambios sensibles de la cuenta, protección contra CSRF y una política de contraseñas coherente.

## Referencias

- [Hoja de referencia de OWASP para contraseñas olvidadas](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)