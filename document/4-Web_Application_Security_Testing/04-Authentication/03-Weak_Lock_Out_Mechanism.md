# Mecanismo de bloqueo débil

|ID          |
|------------|
|WSTG-ATHN-03|

## Resumen

Los mecanismos de bloqueo de cuentas se utilizan para mitigar los ataques de fuerza bruta. Algunos de los ataques que pueden evitarse mediante el uso de mecanismos de bloqueo son:

- Adivinación de contraseñas o ataques de fuerza bruta.
- Adivinación de respuestas a preguntas de seguridad.
- Adivinación de códigos OTP para la autenticación de dos factores (2FA/MFA).

Los mecanismos de bloqueo de cuentas requieren un equilibrio entre proteger las cuentas del acceso no autorizado y proteger a los usuarios para que no se les deniegue el acceso autorizado. Por lo general, las cuentas se bloquean después de 3 a 5 intentos fallidos y solo pueden desbloquearse tras un período predeterminado, mediante un mecanismo de desbloqueo de autoservicio o con la intervención de un administrador.

Aunque es fácil llevar a cabo ataques de fuerza bruta, el resultado de un ataque exitoso es peligroso, ya que el atacante tendrá acceso completo a la cuenta del usuario y, con ella, a todas las funciones y servicios a los que este tiene acceso.

## Objetivos de la prueba

- Evaluar la capacidad del mecanismo de bloqueo de cuentas para mitigar la adivinación de contraseñas por fuerza bruta.
- Evaluar la resistencia del mecanismo de desbloqueo frente al desbloqueo no autorizado de cuentas.

## Cómo probar

### Relleno de credenciales y fuerza bruta distribuida

El relleno de credenciales utiliza pares de nombres de usuario y contraseñas filtrados previamente en filtraciones de datos para obtener acceso no autorizado a las cuentas. La fuerza bruta distribuida y la pulverización de contraseñas envían numerosos intentos de autenticación desde varias direcciones IP. Ambos métodos pueden eludir los mecanismos de bloqueo que solo cuentan los errores por dirección IP.

Muchas aplicaciones bloquean las cuentas basándose únicamente en los intentos de inicio de sesión repetidos desde una sola dirección IP. Los atacantes evitan ese control al distribuir las solicitudes entre redes de bots o al rotar los proxies.

Verifique si la aplicación utiliza protecciones basadas en cuentas, y no únicamente restricciones basadas en IP, al gestionar estos ataques.

1. Obtenga un conjunto controlado de cuentas de prueba y credenciales cuyo uso esté autorizado. Para el relleno de credenciales, utilice pares conocidos de nombres de usuario y contraseñas. Para la fuerza bruta distribuida o la pulverización de contraseñas, utilice una lista de contraseñas contra uno o más nombres de usuario de prueba.
2. Envíe solicitudes de autenticación con esas credenciales.
3. Distribuya los intentos entre varias direcciones IP mediante proxies o puntos de acceso VPN.
4. Observe si la aplicación aplica protecciones basadas en cuentas o solo restricciones basadas en IP.
5. Supervise las respuestas para detectar bloqueos de cuentas, desafíos CAPTCHA o limitación de tasa.

Ejemplo de solicitud de autenticación:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=user1@example.com&password=Password123
```

Ejemplo del uso de ffuf para probar contraseñas contra una sola cuenta de prueba (fuerza bruta distribuida o pulverización de contraseñas cuando se combina con la rotación de IP):

```bash
ffuf -w passwords.txt -X POST -d "username=test&password=FUZZ" -u https://example.com/login
```

Para el relleno de credenciales, use Burp Intruder, ZAP Fuzzer, Hydra o un script personalizado para enviar pares conocidos de nombres de usuario y contraseñas, rotando las direcciones IP de origen.

### Mecanismo de bloqueo

Para probar la solidez de los mecanismos de bloqueo, necesitará acceso a una cuenta que esté dispuesto a bloquear o cuyo bloqueo pueda permitirse. Si solo tiene una cuenta con la que puede iniciar sesión en la aplicación web, realice esta prueba al final de su plan de pruebas para evitar perder tiempo de prueba al quedar bloqueado.

Para evaluar la capacidad del mecanismo de bloqueo de cuentas para mitigar la adivinación de contraseñas por fuerza bruta, intente iniciar sesión con una contraseña incorrecta varias veces y, a continuación, utilice la contraseña correcta para verificar si la cuenta quedó bloqueada. Un ejemplo de prueba puede ser el siguiente:

1. Intente iniciar sesión con una contraseña incorrecta 3 veces.
2. Inicie sesión correctamente con la contraseña correcta, lo que demuestra que el mecanismo de bloqueo no se activa después de 3 intentos de autenticación incorrectos.
3. Intente iniciar sesión con una contraseña incorrecta 4 veces.
4. Inicie sesión correctamente con la contraseña correcta, lo que demuestra que el mecanismo de bloqueo no se activa después de 4 intentos de autenticación incorrectos.
5. Intente iniciar sesión con una contraseña incorrecta 5 veces.
6. Intente iniciar sesión con la contraseña correcta. La aplicación devuelve "Su cuenta está bloqueada.", lo que confirma que la cuenta queda bloqueada después de 5 intentos de autenticación incorrectos.
7. Intente iniciar sesión con la contraseña correcta 5 minutos después. La aplicación devuelve "Su cuenta está bloqueada.", lo que demuestra que el mecanismo de bloqueo no desbloquea automáticamente la cuenta después de 5 minutos.
8. Intente iniciar sesión con la contraseña correcta 10 minutos después. La aplicación devuelve "Su cuenta está bloqueada.", lo que demuestra que el mecanismo de bloqueo no desbloquea automáticamente la cuenta después de 10 minutos.
9. Inicie sesión correctamente con la contraseña correcta 15 minutos después, lo que demuestra que el mecanismo de bloqueo desbloquea automáticamente la cuenta después de un período de entre 10 y 15 minutos.

#### Mecanismos de bloqueo particulares

Hay implementaciones menos comunes de mecanismos de bloqueo que siguen siendo aceptables. Una de ellas es [Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/authentication.html#authentication-flow-lockout-behavior) de AWS. Utiliza un algoritmo de escalado simple que mitiga los ataques de fuerza bruta sin permitir ataques de denegación de servicio a largo plazo contra el usuario afectado. Después de 5 intentos fallidos de inicio de sesión con una contraseña, el usuario queda bloqueado durante un segundo. La duración del bloqueo se duplica con cada intento fallido, hasta un máximo de 15 minutos. Los intentos de inicio de sesión realizados durante el período de bloqueo generan excepciones denominadas `Password attempts exceeded`, que en la mayoría de las implementaciones no se devuelven al usuario que inició el inicio de sesión. Si no se muestran al usuario, un probador podría pensar que no se está utilizando ningún mecanismo de bloqueo, ya que 200 intentos rápidos de inicio de sesión generarían muchas de estas excepciones y muy pocos intentos legítimos de inicio de sesión fallidos.
  
La fórmula del mecanismo es: `2^(n-5)`, donde `n` es el número de intentos fallidos de inicio de sesión. El resultado es la cantidad de segundos durante los que el usuario queda bloqueado. Para restablecer a cero el contador de bloqueos, el usuario debe iniciar sesión correctamente o esperar 15 minutos.

Para probar esto con una herramienta de fuzzing, como Intruder de Burp Suite, vaya a "Resource Pool". A continuación, establezca en 1 el número máximo de solicitudes simultáneas y en 2 segundos el retraso entre solicitudes. Intente realizar una autenticación no válida 200 veces y, justo después de que la herramienta de fuzzing termine, intente utilizar las credenciales válidas 3 veces. Espere 2 minutos e intente iniciar sesión. Si entonces puede iniciar sesión correctamente, es posible que se esté utilizando Cognito. Se pueden realizar pruebas adicionales para validar el uso de Cognito intentando aumentar el tiempo de bloqueo, aunque quizá sea más fácil validar esta información con el cliente.

Un CAPTCHA puede dificultar los ataques de fuerza bruta, pero puede tener sus propias debilidades y no debería reemplazar un mecanismo de bloqueo. Un mecanismo CAPTCHA puede eludirse si se implementa incorrectamente. Entre las fallas de CAPTCHA se incluyen:

1. Desafío fácil de superar, como uno basado en operaciones aritméticas o en un conjunto limitado de preguntas.
2. El CAPTCHA comprueba el código de respuesta HTTP en lugar de comprobar si la respuesta fue correcta.
3. La lógica del CAPTCHA en el servidor acepta por defecto una resolución correcta.
4. El resultado del desafío CAPTCHA nunca se valida en el servidor.
5. El campo de entrada o el parámetro del CAPTCHA se procesa manualmente y se valida o escapa de forma incorrecta.
6. Una solución correcta del CAPTCHA puede reutilizarse varias veces.

Para evaluar la eficacia del CAPTCHA:

1. Evalúe los desafíos CAPTCHA e intente automatizar las soluciones en función de su dificultad.
2. Intente enviar la solicitud sin resolver el CAPTCHA mediante los mecanismos normales de la interfaz de usuario.
3. Intente enviar la solicitud provocando deliberadamente un fallo en el desafío CAPTCHA.
4. Intente enviar la solicitud sin resolver el CAPTCHA (suponiendo que el código del lado del cliente pueda enviar algunos valores predeterminados, etc.) mediante un proxy de pruebas (con la solicitud enviada directamente al servidor).
5. Intente aplicar fuzzing a los puntos de entrada de datos del CAPTCHA (si existen) con cargas útiles de inyección comunes o secuencias de caracteres especiales.
6. Compruebe si la solución del CAPTCHA podría estar en el texto alternativo de las imágenes, en sus nombres de archivo o en un valor de un campo oculto asociado.
7. Intente volver a enviar respuestas correctas conocidas que se hayan identificado previamente.
8. Compruebe si borrar las cookies hace que se eluda el CAPTCHA (por ejemplo, si el CAPTCHA solo aparece después de varios fallos).
9. Si el CAPTCHA forma parte de un proceso de varios pasos, intente acceder directamente a un paso posterior o completarlo (por ejemplo, si el CAPTCHA es el primer paso de un proceso de inicio de sesión, intente enviar directamente el segundo paso [nombre de usuario y contraseña]).
10. Compruebe si hay métodos alternativos en los que quizá no se aplique el CAPTCHA, como un endpoint de API destinado a facilitar el acceso desde una aplicación móvil.

Repita este proceso para todas las funcionalidades posibles que puedan requerir un mecanismo de bloqueo.

### Mecanismo de desbloqueo

Para evaluar la resistencia del mecanismo de desbloqueo frente al desbloqueo no autorizado de cuentas, inicie el mecanismo de desbloqueo y busque debilidades. Los mecanismos de desbloqueo habituales pueden incluir preguntas secretas o un enlace de desbloqueo enviado por correo electrónico. El enlace de desbloqueo debería ser único y de un solo uso, para impedir que un atacante pueda adivinarlo o reproducirlo y realizar ataques de fuerza bruta por lotes.

Tenga en cuenta que un mecanismo de desbloqueo solo debería utilizarse para desbloquear cuentas. No es lo mismo que un mecanismo de recuperación de contraseña, aunque podría seguir las mismas prácticas de seguridad.

## Remediación

Aplique mecanismos de desbloqueo de cuentas según el nivel de riesgo. En orden de menor a mayor garantía:

1. Bloqueo y desbloqueo basados en el tiempo.
2. Desbloqueo de autoservicio (envía un correo de desbloqueo a la dirección de correo electrónico registrada).
3. Desbloqueo manual por parte de un administrador.
4. Desbloqueo manual por parte de un administrador con identificación positiva del usuario.

Factores que deben tenerse en cuenta al implementar un mecanismo de bloqueo de cuentas:

1. ¿Cuál es el riesgo de adivinar contraseñas por fuerza bruta contra la aplicación?
2. ¿Es suficiente un CAPTCHA para mitigar este riesgo?
3. ¿Se está utilizando un mecanismo de bloqueo del lado del cliente (por ejemplo, JavaScript)? (Si es así, desactive el código del lado del cliente para probarlo).
4. Número de intentos fallidos de inicio de sesión antes del bloqueo. Si el umbral de bloqueo es demasiado bajo, los usuarios legítimos podrían quedar bloqueados con demasiada frecuencia. Si el umbral es demasiado alto, un atacante podrá realizar más intentos para forzar la cuenta antes de que quede bloqueada. Según el propósito de la aplicación, un umbral habitual de bloqueo es de entre 5 y 10 intentos fallidos.
5. ¿Cómo se desbloquearán las cuentas?
    1. Manualmente por parte de un administrador: este es el método de bloqueo más seguro, pero puede causar inconvenientes a los usuarios y consumir tiempo "valioso" del administrador.
        1. El administrador también debería disponer de un método de recuperación en caso de que su cuenta quede bloqueada.
        2. Este mecanismo de desbloqueo puede dar lugar a un ataque de denegación de servicio si el objetivo del atacante es bloquear las cuentas de todos los usuarios de la aplicación web.
    2. Después de un período de tiempo: ¿Cuál es la duración del bloqueo? ¿Es suficiente para la aplicación que se protege? Por ejemplo, un bloqueo de entre 5 y 30 minutos puede ser un buen equilibrio entre mitigar los ataques de fuerza bruta y causar inconvenientes a los usuarios legítimos.
    3. Mediante un mecanismo de autoservicio: Como se indicó antes, este mecanismo de autoservicio debe ser lo suficientemente seguro para impedir que el atacante desbloquee cuentas por su cuenta.

## Herramientas

- Burp Suite Intruder
- Zed Attack Proxy (ZAP) Fuzzer
- Hydra
- ffuf
- Scripts personalizados que utilicen rotación de proxies (por ejemplo, Python + requests + listas de proxies)

## Referencias

- Consulte el artículo de OWASP sobre los ataques de [fuerza bruta](https://owasp.org/www-community/attacks/Brute_force_attack).
- [Hoja de referencia sobre contraseñas olvidadas](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).