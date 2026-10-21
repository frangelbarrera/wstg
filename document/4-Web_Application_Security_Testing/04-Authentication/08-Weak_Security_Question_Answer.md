# Preguntas de seguridad débiles

|ID          |
|------------|
|WSTG-ATHN-08|

## Resumen

A menudo llamadas preguntas y respuestas «secretas», las preguntas y respuestas de seguridad se utilizan frecuentemente para recuperar contraseñas olvidadas (véase [funcionalidades débiles de cambio o restablecimiento de contraseña](09-Weak_Password_Change_or_Reset_Functionalities.md)), o como una capa adicional de seguridad además de la contraseña.

Normalmente se generan al crear la cuenta y requieren que el usuario seleccione una pregunta de entre varias preguntas pregeneradas y proporcione una respuesta adecuada. También pueden permitir que el usuario genere sus propios pares de preguntas y respuestas. Ambos métodos son propensos a presentar problemas de seguridad. Idealmente, las respuestas a las preguntas de seguridad deberían ser conocidas únicamente por el usuario y no deberían poder adivinarse ni descubrirse por nadie más. Esto es más difícil de lo que parece.
Las preguntas y respuestas de seguridad dependen de que la respuesta sea secreta. Las preguntas y respuestas deberían elegirse de modo que solo el titular de la cuenta conozca las respuestas. Sin embargo, aunque muchas respuestas quizá no sean de conocimiento público, la mayoría de las preguntas que implementan los sitios propician respuestas seudoprivadas.

### Preguntas pregeneradas

La mayoría de las preguntas pregeneradas son bastante simplistas y pueden dar lugar a respuestas inseguras. Por ejemplo:

- Las respuestas pueden ser conocidas por familiares o amigos cercanos del usuario; por ejemplo: «¿Cuál es el apellido de soltera de tu madre?» o «¿Cuál es tu fecha de nacimiento?».
- Las respuestas pueden ser fáciles de adivinar; por ejemplo: «¿Cuál es tu color favorito?» o «¿Cuál es tu equipo de béisbol favorito?».
- Las respuestas pueden ser vulnerables a ataques de fuerza bruta; por ejemplo: «¿Cuál es el nombre de pila de tu profesor favorito de secundaria?». Es probable que la respuesta aparezca en alguna lista de nombres de pila populares que se pueda descargar fácilmente, por lo que se puede automatizar un ataque de fuerza bruta simple.
- Las respuestas pueden descubrirse públicamente; por ejemplo: «¿Cuál es tu película favorita?». La respuesta podría encontrarse fácilmente en el perfil del usuario en las redes sociales.

### Preguntas autogeneradas

El problema de permitir que los usuarios generen sus propias preguntas es que pueden crear preguntas muy inseguras o incluso eludir por completo el propósito de tener una pregunta de seguridad. Estos son algunos ejemplos del mundo real que ilustran este punto:

- «¿Cuánto es 1+1?»
- «¿Cuál es tu nombre de usuario?»
- «¡Mi contraseña es S3curIty!»

## Objetivos de la prueba

- Determinar la complejidad de las preguntas y hasta qué punto son directas.
- Evaluar las posibles respuestas de los usuarios y las posibilidades de aplicar fuerza bruta.

## Cómo probar

### Preguntas pregeneradas débiles

Intente obtener una lista de preguntas de seguridad creando una cuenta nueva o siguiendo el proceso «No recuerdo mi contraseña». Intente generar tantas preguntas como sea posible para hacerse una buena idea del tipo de preguntas de seguridad que se hacen. Si alguna de las preguntas de seguridad pertenece a las categorías descritas anteriormente, es vulnerable a ataques (adivinación, fuerza bruta, información disponible en redes sociales, etc.).

### Preguntas autogeneradas débiles

Intente crear preguntas de seguridad al crear una cuenta nueva o configurar las propiedades de recuperación de contraseña de su cuenta existente. Si el sistema permite que los usuarios generen sus propias preguntas de seguridad, es vulnerable a que se creen preguntas inseguras. Si el sistema utiliza las preguntas de seguridad autogeneradas en el proceso de recuperación de contraseñas olvidadas y se pueden enumerar los nombres de usuario (véase [enumeración de cuentas y cuentas de usuario adivinables](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md)), debería ser fácil para el evaluador enumerar varias preguntas autogeneradas. Con este método, cabe esperar encontrar varias preguntas autogeneradas débiles.

### Respuestas vulnerables a la fuerza bruta

Utilice los métodos descritos en [mecanismo débil de bloqueo de cuentas](03-Weak_Lock_Out_Mechanism.md) para determinar si un número de respuestas de seguridad incorrectas activa un mecanismo de bloqueo.

Lo primero que se debe tener en cuenta al intentar explotar preguntas de seguridad es cuántas preguntas hay que responder. La mayoría de las aplicaciones solo requieren que el usuario responda una pregunta, mientras que algunas aplicaciones críticas pueden exigir que responda dos o más.

El siguiente paso consiste en evaluar la fortaleza de las preguntas de seguridad. ¿Se podrían obtener las respuestas con una simple búsqueda en Google o mediante un ataque de ingeniería social? Como evaluador de penetración, puede seguir estos pasos para explotar un esquema de preguntas de seguridad:

- ¿Permite la aplicación que el usuario final elija la pregunta que debe responder? Si es así, céntrese en las preguntas que tengan:

    - Una respuesta «pública»; por ejemplo, algo que se pueda encontrar con una simple consulta en un motor de búsqueda.
    - Una respuesta factual, como «la primera escuela», u otros datos que se puedan buscar.
    - Pocas respuestas posibles, como «¿qué modelo era tu primer coche?». Estas preguntas ofrecerían al atacante una lista corta de posibles respuestas y, basándose en estadísticas, podría ordenarlas de la más a la menos probable.

- Determine, si es posible, cuántos intentos tiene.
    - ¿Permite el restablecimiento de contraseña intentos ilimitados?
    - ¿Hay un período de bloqueo después de X respuestas incorrectas? Tenga en cuenta que un sistema de bloqueo puede constituir por sí mismo un problema de seguridad, ya que un atacante podría explotarlo para lanzar un ataque de denegación de servicio contra usuarios legítimos.
    - Elija la pregunta adecuada según el análisis de los puntos anteriores e investigue para determinar las respuestas más probables.

La clave para explotar y eludir con éxito un esquema débil de preguntas de seguridad es encontrar una pregunta o un conjunto de preguntas cuyas respuestas puedan encontrarse fácilmente. Si no está seguro de ninguna de las respuestas, busque siempre preguntas que ofrezcan la mayor probabilidad estadística de adivinar la respuesta correcta. En definitiva, un esquema de preguntas de seguridad solo es tan sólido como su pregunta más débil.

## Referencias

- [La maldición de la pregunta secreta](https://www.schneier.com/essay-081.html)
- [Hoja de referencia de OWASP sobre preguntas de seguridad](https://cheatsheetseries.owasp.org/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.html)