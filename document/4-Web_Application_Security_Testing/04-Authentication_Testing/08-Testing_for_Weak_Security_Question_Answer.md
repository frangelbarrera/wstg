# Pruebas de preguntas de seguridad débiles

|ID          |
|------------|
|WSTG-ATHN-08|

## Resumen

A menudo llamadas "preguntas secretas" y respuestas, las preguntas y respuestas de seguridad se utilizan frecuentemente para recuperar contraseñas olvidadas (véase [Pruebas de funcionalidades débiles de cambio o restablecimiento de contraseña](09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md)), o como seguridad adicional además de la contraseña.

Típicamente se generan al crear la cuenta y requieren que el usuario seleccione de preguntas pregeneradas y proporcione una respuesta apropiada. Pueden permitir al usuario generar sus propios pares de pregunta y respuesta. Ambos métodos son propensos a inseguridades. Idealmente, las preguntas de seguridad deberían generar respuestas que solo conozca el usuario, y no sean adivinables o descubribles por nadie más. Esto es más difícil de lo que parece.
Las preguntas y respuestas de seguridad dependen de la secrecía de la respuesta. Las preguntas y respuestas deben elegirse para que las respuestas solo sean conocidas por el titular de la cuenta. Sin embargo, aunque muchas respuestas pueden no ser públicamente conocidas, la mayoría de las preguntas que los sitios implementan promueven respuestas que son pseudo-privadas.

### Preguntas pregeneradas

La mayoría de las preguntas pregeneradas son bastante simplistas en naturaleza y pueden llevar a respuestas inseguras. Por ejemplo:

- Las respuestas pueden ser conocidas por familiares o amigos cercanos del usuario, p. ej. "¿Cuál es el apellido de soltera de tu madre?", "¿Cuál es tu fecha de nacimiento?"
- Las respuestas pueden ser fácilmente adivinables, p. ej. "¿Cuál es tu color favorito?", "¿Cuál es tu equipo de béisbol favorito?"
- Las respuestas pueden ser forzadas por fuerza bruta, p. ej. "¿Cuál es el nombre de pila de tu profesor favorito de secundaria?" - la respuesta probablemente esté en algunas listas descargables fácilmente de nombres populares, y por lo tanto se puede scriptar un ataque de fuerza bruta simple.
- Las respuestas pueden ser públicamente descubribles, p. ej. "¿Cuál es tu película favorita?" - la respuesta puede encontrarse fácilmente en la página de perfil de redes sociales del usuario.

### Preguntas autogeneradas

El problema de permitir a los usuarios generar sus propias preguntas es que les permite generar preguntas muy inseguras, o incluso eludir el propósito de tener una pregunta de seguridad en primer lugar. Aquí hay algunos ejemplos del mundo real que ilustran este punto:

- "¿Cuánto es 1+1?"
- "¿Cuál es tu nombre de usuario?"
- "¡Mi contraseña es S3curIty!"

## Objetivos de la prueba

- Determinar la complejidad y cuán directas son las preguntas.
- Evaluar posibles respuestas de usuario y capacidades de fuerza bruta.

## Cómo probar

### Pruebas de preguntas pregeneradas débiles

Intente obtener una lista de preguntas de seguridad creando una nueva cuenta o siguiendo el proceso "No recuerdo mi contraseña". Intente generar tantas preguntas como sea posible para tener una buena idea del tipo de preguntas de seguridad que se hacen. Si alguna de las preguntas de seguridad cae en las categorías descritas anteriormente, son vulnerables a ser atacadas (adivinadas, forzadas por fuerza bruta, disponibles en redes sociales, etc.).

### Pruebas de preguntas autogeneradas débiles

Intente crear preguntas de seguridad creando una nueva cuenta o configurando las propiedades de recuperación de contraseña de su cuenta existente. Si el sistema permite al usuario generar sus propias preguntas de seguridad, es vulnerable a tener preguntas inseguras creadas. Si el sistema utiliza las preguntas de seguridad autogeneradas durante la funcionalidad de contraseña olvidada y si los nombres de usuario pueden enumerarse (véase [Pruebas de enumeración de cuentas y cuenta de usuario adivinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)), entonces debería ser fácil para el probador enumerar una serie de preguntas autogeneradas. Se debe esperar encontrar varias preguntas autogeneradas débiles utilizando este método.

### Pruebas de respuestas forzables por fuerza bruta

Utilice los métodos descritos en [Pruebas de mecanismo de bloqueo débil](03-Testing_for_Weak_Lock_Out_Mechanism.md) para determinar si un número de respuestas de seguridad incorrectamente proporcionadas activan un mecanismo de bloqueo.

Lo primero a considerar al intentar explotar preguntas de seguridad es el número de preguntas que necesitan ser respondidas. La mayoría de las aplicaciones solo necesitan que el usuario responda una sola pregunta, mientras que algunas aplicaciones críticas pueden requerir que el usuario responda dos o incluso más preguntas.

El siguiente paso es evaluar la fortaleza de las preguntas de seguridad. ¿Podrían las respuestas obtenerse con una simple búsqueda en Google o con un ataque de ingeniería social? Como probador de penetración, aquí hay un tutorial paso a paso de explotación de un esquema de preguntas de seguridad:

- ¿Permite la aplicación al usuario final elegir la pregunta que necesita ser respondida? Si es así, concéntrese en preguntas que tengan:

    - Una respuesta "pública"; por ejemplo, algo que podría encontrarse con una consulta simple de motor de búsqueda.
    - Una respuesta factual como una "primera escuela" u otros hechos que pueden buscarse.
    - Pocas respuestas posibles, como "qué modelo fue tu primer coche". Estas preguntas presentarían al atacante con una lista corta de posibles respuestas, y basándose en estadísticas el atacante podría clasificar respuestas de más a menos probables.

- Determine cuántas conjeturas tiene si es posible.
    - ¿Permite el restablecimiento de contraseña intentos ilimitados?
    - ¿Hay un período de bloqueo después de X respuestas incorrectas? Tenga en cuenta que un sistema de bloqueo puede ser un problema de seguridad en sí mismo, ya que puede ser explotado por un atacante para lanzar una Denegación de Servicio contra usuarios legítimos.
    - Elija la pregunta apropiada basada en el análisis de los puntos anteriores, e investigue para determinar las respuestas más probables.

La clave para explotar exitosamente y eludir un esquema de preguntas de seguridad débil es encontrar una pregunta o conjunto de preguntas que den la posibilidad de encontrar fácilmente las respuestas. Siempre busque preguntas que le den la mayor oportunidad estadística de adivinar la respuesta correcta, si está completamente inseguro de cualquiera de las respuestas. Al final, un esquema de preguntas de seguridad es solo tan fuerte como la pregunta más débil.

## Referencias

- [The Curse of the Secret Question](https://www.schneier.com/essay-081.html)
- [The OWASP Security Questions Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.html)