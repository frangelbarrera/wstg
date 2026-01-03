# Prueba de Mecanismos de Bloqueo Débiles

|ID          |
|------------|
|WSTG-ATHN-03|

## Resumen

Los mecanismos de bloqueo de cuentas se utilizan para mitigar ataques de fuerza bruta. Algunos de los ataques que pueden ser derrotados mediante el uso de mecanismos de bloqueo:

- Ataque de adivinanza de contraseña o nombre de usuario de inicio de sesión.
- Adivinanza de código en cualquier funcionalidad de 2FA o preguntas de seguridad.

Los mecanismos de bloqueo de cuentas requieren un equilibrio entre proteger las cuentas del acceso no autorizado y proteger a los usuarios de ser denegados el acceso autorizado. Las cuentas suelen bloquearse después de 3 a 5 intentos fallidos y solo pueden desbloquearse después de un período de tiempo predeterminado, mediante un mecanismo de desbloqueo de autoservicio o intervención de un administrador.

A pesar de que es fácil realizar ataques de fuerza bruta, el resultado de un ataque exitoso es peligroso ya que el atacante tendrá acceso completo a la cuenta del usuario y con ella toda la funcionalidad y servicios a los que tiene acceso.

## Objetivos de la Prueba

- Evaluar la capacidad del mecanismo de bloqueo de cuentas para mitigar la adivinanza de contraseñas por fuerza bruta.
- Evaluar la resistencia del mecanismo de desbloqueo al desbloqueo no autorizado de cuentas.

## Cómo Probar

### Mecanismo de Bloqueo

Para probar la fortaleza de los mecanismos de bloqueo, necesitará acceso a una cuenta que esté dispuesto o pueda permitirse bloquear. Si solo tiene una cuenta con la que puede iniciar sesión en la aplicación web, realice esta prueba al final de su plan de pruebas para evitar perder tiempo de pruebas al ser bloqueado.

Para evaluar la capacidad del mecanismo de bloqueo de cuentas para mitigar la adivinanza de contraseñas por fuerza bruta, intente un inicio de sesión inválido utilizando la contraseña incorrecta un número de veces, antes de usar la contraseña correcta para verificar que la cuenta fue bloqueada. Un ejemplo de prueba puede ser el siguiente:

1. Intente iniciar sesión con una contraseña incorrecta 3 veces.
2. Inicie sesión exitosamente con la contraseña correcta, mostrando así que el mecanismo de bloqueo no se activa después de 3 intentos de autenticación incorrectos.
3. Intente iniciar sesión con una contraseña incorrecta 4 veces.
4. Inicie sesión exitosamente con la contraseña correcta, mostrando así que el mecanismo de bloqueo no se activa después de 4 intentos de autenticación incorrectos.
5. Intente iniciar sesión con una contraseña incorrecta 5 veces.
6. Intente iniciar sesión con la contraseña correcta. La aplicación devuelve "Su cuenta está bloqueada.", confirmando así que la cuenta está bloqueada después de 5 intentos de autenticación incorrectos.
7. Intente iniciar sesión con la contraseña correcta 5 minutos después. La aplicación devuelve "Su cuenta está bloqueada.", mostrando así que el mecanismo de bloqueo no desbloquea automáticamente después de 5 minutos.
8. Intente iniciar sesión con la contraseña correcta 10 minutos después. La aplicación devuelve "Su cuenta está bloqueada.", mostrando así que el mecanismo de bloqueo no desbloquea automáticamente después de 10 minutos.
9. Inicie sesión exitosamente con la contraseña correcta 15 minutos después, mostrando así que el mecanismo de bloqueo desbloquea automáticamente después de un período de 10 a 15 minutos.

#### Mecanismos de Bloqueo Únicos

Hay implementaciones más únicas de mecanismos de bloqueo en uso que aún son aceptables. Una de ellas es el [Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/authentication.html#authentication-flow-lockout-behavior) de AWS. Utiliza un algoritmo de escalado simple que mitiga ataques de fuerza bruta mientras no permite ataques de denegación de servicio a largo plazo contra el usuario afectado. Después de 5 intentos fallidos de inicio de sesión con una contraseña, el usuario se bloquea por un segundo. La duración del bloqueo se duplica con cada intento fallido, con un máximo de 15 minutos. Los intentos de inicio de sesión realizados durante el período de bloqueo son excepciones denominadas `Intentos de contraseña excedidos`, y en la mayoría de las implementaciones no se devuelven al usuario que inició el inicio de sesión. Si no se muestran al usuario, un probador puede pensar que no se está utilizando ningún mecanismo de bloqueo, ya que 200 intentos rápidos de inicio de sesión generarían muchas de las excepciones, y muy pocos intentos fallidos legítimos de inicio de sesión.
  
La matemática detrás del mecanismo es: `2^(n-5)`, con `n` siendo el número de intentos fallidos de inicio de sesión. El número resultante es la cantidad de segundos que el usuario está bloqueado. Para restablecer el conteo de bloqueo a cero, el usuario debe iniciar sesión exitosamente o esperar los 15 minutos.

Para probar esto utilizando una herramienta de fuzzing, como Burp Suite's Intruder, navegue a "Resource Pool". Luego configure el máximo de solicitudes concurrentes a 1 y el retraso entre solicitudes a 2 segundos. Intente la autenticación inválida 200 veces, luego intente usar las credenciales válidas 3 veces directamente después de que la herramienta de fuzzing termine. Espere 2 minutos e intente iniciar sesión. Si el inicio de sesión es exitoso entonces, Cognito puede estar en uso. Se pueden realizar pruebas adicionales para validar el uso de Cognito intentando aumentar el tiempo de bloqueo, pero puede ser más fácil validar esta información con el cliente.

Un CAPTCHA puede dificultar los ataques de fuerza bruta, pero pueden venir con su propio conjunto de debilidades, y no deberían reemplazar un mecanismo de bloqueo. Un mecanismo CAPTCHA puede ser evadido si se implementa incorrectamente. Las fallas de CAPTCHA incluyen:

1. Desafío fácilmente derrotado, como aritmética o conjunto limitado de preguntas.
2. CAPTCHA verifica el código de respuesta HTTP en lugar del éxito de la respuesta.
3. La lógica del lado del servidor de CAPTCHA por defecto a una resolución exitosa.
4. El resultado del desafío CAPTCHA nunca se valida del lado del servidor.
5. El campo de entrada CAPTCHA o parámetro se procesa manualmente, y se valida o escapa incorrectamente.

Para evaluar la efectividad del CAPTCHA:

1. Evalúe los desafíos CAPTCHA e intente automatizar soluciones dependiendo de la dificultad.
2. Intente enviar la solicitud sin resolver CAPTCHA a través del mecanismo(s) normal de UI.
3. Intente enviar la solicitud con falla intencional del desafío CAPTCHA.
4. Intente enviar la solicitud sin resolver CAPTCHA (asumiendo que algunos valores por defecto pueden ser pasados por código del lado del cliente, etc.) mientras usa un proxy de pruebas (solicitud enviada directamente del lado del servidor).
5. Intente fuzz los puntos de entrada de datos CAPTCHA (si están presentes) con payloads comunes de inyección o secuencias de caracteres especiales.
6. Verifique si la solución al CAPTCHA podría ser el texto alt de la(s) imagen(es), nombre(s) de archivo, o un valor en un campo oculto asociado.
7. Intente reenviar respuestas buenas conocidas previamente identificadas.
8. Verifique si borrar cookies causa que el CAPTCHA sea evadido (por ejemplo si el CAPTCHA solo se muestra después de un número de fallas).
9. Si el CAPTCHA es parte de un proceso de múltiples pasos, intente simplemente acceder o completar un paso más allá del CAPTCHA (por ejemplo si CAPTCHA es el primer paso en un proceso de inicio de sesión, intente simplemente enviar el segundo paso [nombre de usuario y contraseña]).
10. Verifique métodos alternativos que podrían no tener CAPTCHA aplicado, como un endpoint de API destinado a facilitar el acceso de aplicaciones móviles.

Repita este proceso a cada funcionalidad posible que podría requerir un mecanismo de bloqueo.

### Mecanismo de Desbloqueo

Para evaluar la resistencia del mecanismo de desbloqueo al desbloqueo no autorizado de cuentas, inicie el mecanismo de desbloqueo y busque debilidades. Los mecanismos de desbloqueo típicos pueden involucrar preguntas secretas o un enlace de desbloqueo enviado por correo electrónico. El enlace de desbloqueo debería ser un enlace único de un solo uso, para evitar que un atacante adivine o repita el enlace y realice ataques de fuerza bruta en lotes.

Tenga en cuenta que un mecanismo de desbloqueo debería usarse solo para desbloquear cuentas. No es lo mismo que un mecanismo de recuperación de contraseña, aunque podría seguir las mismas prácticas de seguridad.

## Remediation

Aplique mecanismos de desbloqueo de cuentas dependiendo del nivel de riesgo. En orden de menor a mayor garantía:

1. Bloqueo y desbloqueo basado en tiempo.
2. Desbloqueo de autoservicio (envía correo de desbloqueo a la dirección de correo registrada).
3. Desbloqueo manual por administrador.
4. Desbloqueo manual por administrador con identificación positiva del usuario.

Factores a considerar al implementar un mecanismo de bloqueo de cuentas:

1. ¿Cuál es el riesgo de adivinanza de contraseñas por fuerza bruta contra la aplicación?
2. ¿Es un CAPTCHA suficiente para mitigar este riesgo?
3. ¿Se está utilizando un mecanismo de bloqueo del lado del cliente (ej. JavaScript)? (Si es así, deshabilite el código del lado del cliente para probar.)
4. Número de intentos fallidos de inicio de sesión antes del bloqueo. Si el umbral de bloqueo es demasiado bajo entonces los usuarios válidos pueden ser bloqueados demasiado a menudo. Si el umbral de bloqueo es demasiado alto entonces más intentos puede hacer un atacante para forzar la cuenta antes de que se bloquee. Dependiendo del propósito de la aplicación, un rango de 5 a 10 intentos fallidos es un umbral típico de bloqueo.
5. ¿Cómo se desbloquearán las cuentas?
     1. Manualmente por un administrador: este es el método de bloqueo más seguro, pero puede causar inconvenientes a los usuarios y tomar el tiempo "valioso" del administrador.
         1. Tenga en cuenta que el administrador también debería tener un método de recuperación en caso de que su cuenta se bloquee.
         2. Este mecanismo de desbloqueo puede llevar a un ataque de denegación de servicio si el objetivo del atacante es bloquear las cuentas de todos los usuarios de la aplicación web.
     2. Después de un período de tiempo: ¿Cuál es la duración del bloqueo? ¿Es esto suficiente para la aplicación siendo protegida? Ej. una duración de bloqueo de 5 a 30 minutos puede ser un buen compromiso entre mitigar ataques de fuerza bruta e inconvenienciar a usuarios válidos.
     3. Vía un mecanismo de autoservicio: Como se declaró antes, este mecanismo de autoservicio debe ser lo suficientemente seguro para evitar que el atacante desbloquee cuentas él mismo.

## Referencias

- Vea el artículo de OWASP sobre ataques de [Fuerza Bruta](https://owasp.org/www-community/attacks/Brute_force_attack).
- [Hoja de Trucos de Contraseña Olvidada](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).