# Probar la Validación de Datos de Lógica de Negocio

|ID          |
|------------|
|WSTG-BUSL-01|

## Resumen

La aplicación debe asegurar que solo datos lógicamente válidos puedan ser ingresados en el frontend así como directamente al lado del servidor de una aplicación o sistema. Verificar datos solo en el cliente/frontend podría dejar aplicaciones vulnerables a inyecciones de servidor a través de proxies o en handoffs con otros sistemas. Esto es diferente de simplemente realizar Análisis de Valor Límite (BVA, por sus siglas en inglés) en que es más difícil y en la mayoría de los casos no puede ser simplemente verificado en el punto de entrada, sino que usualmente requiere verificar algún otro sistema.

Por ejemplo: Una aplicación podría pedir el número de Seguro Social. En BVA, la aplicación debería verificar formatos y semántica (¿el valor tiene 9 dígitos, no es negativo, y no son todos 0's) para los datos ingresados, pero también hay consideraciones lógicas. Los SSNs están agrupados y categorizados. ¿Está esta persona en un archivo de muerte? ¿Son de una cierta parte del país?

Las vulnerabilidades relacionadas con la validación de datos de negocio son únicas en que son específicas de la aplicación y diferentes de las vulnerabilidades relacionadas con falsificar solicitudes en que están más preocupadas por datos lógicos en oposición a simplemente romper el flujo de trabajo de la lógica de negocio.

El frontend y el backend de la aplicación deberían estar verificando y validando que los datos que tiene, está usando, y está pasando sean lógicamente válidos. Incluso si el usuario proporciona datos válidos a una aplicación, la lógica de negocio podría hacer que la aplicación se comporte de manera diferente dependiendo de los datos o circunstancias.

### Ejemplo 1

Suponer que gestionas un sitio de e-commerce multinivel que permite a los usuarios ordenar alfombra. El usuario selecciona su alfombra, ingresa el tamaño, hace el pago, y la aplicación frontend ha verificado que toda la información ingresada es correcta y válida para información de contacto, tamaño, marca y color de la alfombra. Pero, la lógica de negocio en el fondo tiene dos caminos: si la alfombra está en stock se envía directamente desde tu almacén, pero si está agotada en tu almacén se hace una llamada al sistema de un partner y si ellos la tienen en stock enviarán la orden desde su almacén y serán reembolsados por ellos.

¿Qué pasa si un atacante es capaz de continuar una transacción válida en stock y enviarla como agotada a tu partner? ¿Qué pasa si un atacante es capaz de entrar en el medio y enviar mensajes al almacén del partner ordenando alfombra sin pago?

### Ejemplo 2

Muchos sistemas de tarjetas de crédito ahora están descargando saldos de cuenta nocturnamente para que los clientes puedan hacer checkout más rápidamente por montos bajo cierto valor. Lo inverso también es cierto. Si pago mi tarjeta de crédito en la mañana podría no ser capaz de usar el crédito disponible en la tarde. Otro ejemplo podría ser si uso mi tarjeta de crédito en múltiples ubicaciones muy rápidamente podría ser posible exceder mi límite si los sistemas están basando decisiones en los datos de anoche.

### Ejemplo 3

**[Distributed Denial of Dollar (DDo$)](https://news.hitb.org/content/pirate-bay-proposes-distributed-denial-dollars-attack-ddo):**
Esta fue una campaña propuesta por el fundador del sitio "The Pirate Bay" contra la firma de abogados que llevó las acusaciones contra "The Pirate Bay". El objetivo era aprovecharse de errores en el diseño de características de negocio y en el proceso de validación de transferencia de crédito.

Este ataque se realizó enviando cantidades muy pequeñas de dinero de 1 SEK ($0.13 USD) a la firma de abogados.
La cuenta bancaria a la que se dirigían los pagos tenía solo 1000 transferencias gratuitas, después de las cuales cualquier transferencia tiene un recargo para el titular de la cuenta (2 SEK). Después de las primeras mil transacciones por internet cada donación de 1 SEK a la firma de abogados realmente terminará costándole 1 SEK en su lugar.

## Objetivos de Prueba

- Identificar puntos de inyección de datos.
- Validar que todas las verificaciones están ocurriendo en el backend y no pueden ser evadidas.
- Intentar romper el formato de los datos esperados y analizar cómo la aplicación los maneja.

## Cómo Probar

### Método de Prueba Genérico

- Revisar la documentación del proyecto y usar pruebas exploratorias buscando puntos de entrada de datos o puntos de handoff entre sistemas o software.
- Una vez encontrados, intentar insertar datos lógicamente inválidos en la aplicación/sistema.

Método de Prueba Específico:

- Realizar pruebas funcionales válidas GUI en la aplicación para asegurar que solo los valores "válidos" sean aceptados.
- Usando un proxy de intercepción observar el HTTP POST/GET buscando lugares donde variables tales como costo y cantidad se pasan. Específicamente, buscar "hand-offs" entre aplicaciones/sistemas que puedan ser posibles puntos de inyección o manipulación.
- Una vez encontradas las variables, empezar a interrogar el campo con datos lógicamente "inválidos", tales como números de seguro social o identificadores únicos que no existen o que no se ajustan a la lógica de negocio. Esta prueba verifica que el servidor funcione apropiadamente y no acepte datos lógicamente inválidos.

## Casos de Prueba Relacionados

- Todos los casos de prueba de [Validación de Entrada](../07-Input_Validation_Testing/README.md).
- [Pruebas de Enumeración de Cuentas y Cuenta de Usuario Adivinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md).
- [Pruebas de Evasión del Esquema de Gestión de Sesión](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md).
- [Pruebas de Variables de Sesión Expuestas](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md).

## Remediación

La aplicación/sistema debe asegurar que solo se acepten datos "lógicamente válidos" en todos los puntos de entrada y handoff de la aplicación o sistema y que los datos no sean simplemente confiables una vez que han entrado al sistema.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [OWASP Proactive Controls (C5) - Validate All Inputs](https://owasp.org/www-project-proactive-controls/v3/en/c5-validate-inputs)
- [OWASP Cheat Sheet Series - Input_Validation_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
