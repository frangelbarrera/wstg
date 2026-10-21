# Validación de datos de lógica de negocio

|ID          |
|------------|
|WSTG-BUSL-01|

## Resumen

La aplicación debe garantizar que solo se puedan introducir datos lógicamente válidos tanto en el frontend como directamente en el lado del servidor de una aplicación o sistema. Verificar los datos únicamente en el cliente/frontend puede dejar las aplicaciones vulnerables a inyecciones en el servidor a través de proxies o durante las transferencias a otros sistemas. Esto difiere de realizar simplemente un Análisis de Valores Límite (BVA, por sus siglas en inglés), porque es más difícil y, en la mayoría de los casos, no se puede verificar simplemente en el punto de entrada, sino que suele requerir comprobar algún otro sistema.

Por ejemplo: Una aplicación puede pedirte tu número de Seguro Social. En el BVA, la aplicación debería comprobar los formatos y la semántica (si el valor tiene 9 dígitos, no es negativo y no está compuesto únicamente por ceros) de los datos introducidos, pero también hay consideraciones lógicas. Los números de Seguro Social están agrupados y categorizados. ¿Está esta persona en un registro de defunciones? ¿Es de una región determinada del país?

Las vulnerabilidades relacionadas con la validación de datos de negocio son únicas porque son específicas de cada aplicación y difieren de las vulnerabilidades relacionadas con la falsificación de solicitudes: se centran más en los datos lógicos que en simplemente romper el flujo de trabajo de la lógica de negocio.

El frontend y el backend de la aplicación deben verificar y validar que los datos que tienen, utilizan y transmiten sean lógicamente válidos. Incluso si el usuario proporciona datos válidos a una aplicación, la lógica de negocio puede hacer que esta se comporte de manera diferente según los datos o las circunstancias.

### Ejemplo 1

Supongamos que administras un sitio de comercio electrónico de varios niveles que permite a los usuarios pedir alfombras. El usuario selecciona la alfombra, introduce el tamaño, realiza el pago y la aplicación frontend ha verificado que toda la información introducida sea correcta y válida en cuanto a los datos de contacto y al tamaño, la marca y el color de la alfombra. Sin embargo, la lógica de negocio en segundo plano tiene dos rutas: si la alfombra está en existencias, se envía directamente desde tu almacén; pero si no está en existencias en tu almacén, se realiza una llamada al sistema de un socio y, si la tiene en existencias, este envía el pedido desde su almacén y recibe el reembolso correspondiente.

¿Qué ocurre si un atacante puede continuar una transacción válida de un producto en existencias y enviarla a tu socio como si el producto estuviera agotado? ¿Qué ocurre si un atacante puede interponerse y enviar mensajes al almacén del socio para pedir alfombras sin pagar?

### Ejemplo 2

Muchos sistemas de tarjetas de crédito ahora descargan los saldos de las cuentas durante la noche para que los clientes puedan pagar más rápidamente los importes inferiores a cierto valor. Lo contrario también es cierto. Si pago mi tarjeta de crédito por la mañana, puede que no pueda utilizar el crédito disponible por la tarde. Otro ejemplo sería que, si uso mi tarjeta de crédito en varios lugares en muy poco tiempo, quizá pueda superar mi límite si los sistemas basan sus decisiones en los datos de la noche anterior.

### Ejemplo 3

**[Denegación distribuida de dólares (DDo$)](https://www.zdnet.com/article/pirate-bay-defendant-urges-ddo-attack-on-lawyer/):**
Esta fue una campaña propuesta por el fundador del sitio "The Pirate Bay" contra el bufete de abogados que inició acciones judiciales contra "The Pirate Bay". El objetivo era aprovechar los errores en el diseño de las funciones de negocio y en el proceso de validación de transferencias de crédito.

Este ataque se llevó a cabo enviando cantidades muy pequeñas de dinero, 1 SEK ($0.13 USD), al bufete de abogados.
La cuenta bancaria a la que se dirigían los pagos solo tenía 1000 transferencias gratuitas; después de ese límite, cada transferencia tenía un recargo para el titular de la cuenta (2 SEK). Después de las primeras mil transacciones por internet, cada donación de 1 SEK al bufete de abogados acabaría costándole 1 SEK en lugar de generarle un ingreso.

### Ejemplo 4

Una página de perfil permite a los usuarios elegir un avatar. El frontend siempre envía la imagen como una URI `data:` codificada en base64, por lo que los desarrolladores nunca validan el contenido del parámetro `avatar`. Un atacante intercepta la solicitud y sustituye el valor por `http://attacker.example/pixel.png`. La aplicación lo acepta y, posteriormente, un proceso del servidor o el navegador de un administrador obtiene esa URL. La solicitud llega al atacante, quien descubre que la entrada nunca se validó y podría usarla para acceder a hosts internos o rastrear a otros usuarios.

### Ejemplo 5

Un formulario de reseñas de productos ofrece una valoración de 1 a 5 estrellas mediante un conjunto de botones de opción. Si el backend solo comprueba que el valor sea un número, se almacenarán valores como `rating=-100` o `rating=99999` y se distorsionará la puntuación media; además, un valor como `rating=NaN` podría provocar errores en las páginas que la muestran.

## Objetivos de prueba

- Identificar puntos de inyección de datos.
- Validar que todas las comprobaciones se realicen en el backend y no puedan eludirse.
- Intentar alterar el formato de los datos esperados y analizar cómo los gestiona la aplicación.
- Determinar si los parámetros que el frontend restringe (tipo, rango, conjunto de valores permitidos o relación con otros campos) también se restringen en el servidor.

## Cómo probar

### Método de prueba genérico

- Revisar la documentación del proyecto y realizar pruebas exploratorias para buscar puntos de entrada de datos o puntos de transferencia entre sistemas o programas.
- Una vez encontrados, intentar introducir datos lógicamente inválidos en la aplicación o el sistema.

Método de prueba específico:

- Realizar pruebas funcionales válidas de la GUI del frontend para garantizar que solo se acepten los valores "válidos".
- Usar un proxy de interceptación para observar las solicitudes HTTP POST/GET y buscar dónde se transmiten variables como el costo y la cantidad. En particular, buscar "transferencias" entre aplicaciones o sistemas que puedan ser puntos de inyección o manipulación.
- Una vez encontradas las variables, empezar a probar el campo con datos lógicamente "inválidos", como números de Seguro Social o identificadores únicos que no existen o que no se ajustan a la lógica de negocio. Estas pruebas verifican que el servidor funcione correctamente y no acepte datos lógicamente inválidos.

### Casos de prueba basados en parámetros

Para cada parámetro observado en los pasos anteriores, repetir la solicitud con un valor modificado y comparar la respuesta, los datos almacenados y cualquier comportamiento posterior (correos electrónicos, procesos en segundo plano, vistas de otros usuarios) con los de la solicitud válida. El objetivo es encontrar valores que el frontend nunca enviaría, pero que el servidor acepta. Algunas variaciones útiles son:

- **Rango y signo:** valores inferiores al mínimo o superiores al máximo que ofrece la interfaz, cero, números negativos y números muy grandes (por ejemplo, una valoración de `-1` o `1000`, una cantidad de `0` o una transferencia de `-50`).
- **Conjunto permitido:** valores que no están en la lista que ofrece la interfaz, como un rol, país, plan, estado o moneda que no aparece en el menú desplegable, o un valor válido diferente que pertenece a otro usuario o inquilino.
- **Tipo y formato:** un tipo de dato distinto del esperado (una cadena en lugar de un número, una matriz o un objeto en lugar de una cadena, `null`, un valor vacío o `true` en lugar de `1`), o el mismo campo con otro formato (una URL cuando la aplicación espera una URI `data:` o un nombre de archivo, una fecha con otro formato o dígitos Unicode).
- **Longitud y codificación:** valores vacíos, valores que superan la longitud permitida por el frontend y valores que contienen caracteres que la interfaz filtra.
- **Relaciones entre campos:** valores que son válidos por separado, pero incompatibles entre sí; por ejemplo, una fecha de finalización anterior a una fecha de inicio, un descuento superior al precio o un país de envío que no coincide con el país de pago.
- **Parámetros ausentes y adicionales:** eliminar parámetros que parecen obligatorios y añadir parámetros que la interfaz nunca envía (consulta [Asignación masiva](../07-Injection/20-Mass_Assignment.md)); también, repetir un parámetro con distintos valores (consulta [Contaminación de parámetros HTTP](../07-Injection/04-HTTP_Parameter_Pollution.md)).
- **Valores de transferencia:** valores que luego se transmiten a otro sistema, como una dirección de correo electrónico, una URL o una ruta de archivo (consulta [Falsificación de solicitudes del lado del servidor](../07-Injection/19-Server-Side_Request_Forgery.md)). Utiliza un valor único que controles para poder detectar cuándo y dónde se usa, incluso si se utiliza en una solicitud diferida o fuera de banda.

Registra cuáles de las variaciones se aceptaron, porque el impacto depende de cómo utiliza la aplicación el valor posteriormente, no solo de si lo rechazó.

## Casos de prueba relacionados

- Todos los casos de prueba de [Pruebas de inyección](../07-Injection/README.md).
- [Contaminación de parámetros HTTP](../07-Injection/04-HTTP_Parameter_Pollution.md).
- [Asignación masiva](../07-Injection/20-Mass_Assignment.md).
- [Falsificación de solicitudes del lado del servidor](../07-Injection/19-Server-Side_Request_Forgery.md).
- [Funcionalidad de pago](10-Payment_Functionality.md).
- [Enumeración de cuentas y cuentas de usuario adivinables](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md).
- [Elusión del esquema de gestión de sesiones](../06-Session_Management/01-Session_Management_Schema.md).
- [Variables de sesión expuestas](../06-Session_Management/04-Exposed_Session_Variables.md).

## Remediación

La aplicación o el sistema debe garantizar que solo se acepten datos "lógicamente válidos" en todos los puntos de entrada y transferencia de la aplicación o el sistema, y que no se confíe simplemente en los datos una vez que hayan entrado en el sistema.

- Validar cada parámetro en el servidor, incluso cuando el frontend lo restrinja. Comprobar el tipo, el rango, la longitud, el formato y la pertenencia al conjunto de valores permitidos (preferiblemente mediante una lista de permitidos).
- Validar las relaciones entre campos y entre los valores y el estado del usuario, la cuenta o el pedido, en lugar de validar cada valor por separado.
- Rechazar los parámetros inesperados o desconocidos en lugar de utilizarlos o ignorarlos silenciosamente.
- Tratar los datos recibidos de otros sistemas con la misma cautela que los datos recibidos de los usuarios.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [OWASP Top 10 Proactive Controls (2024) C3: Validar entradas y gestionar excepciones](https://top10proactive.owasp.org/archive/2024/the-top-10/c3-validate-input-and-handle-exceptions/)
- [OWASP Cheat Sheet Series - Hoja de referencia sobre validación de entradas](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)