# Pruebas del Esquema de Gestión de Sesiones

|ID          |
|------------|
|WSTG-SESS-01|

## Resumen

Uno de los componentes principales de cualquier aplicación basada en web es el mecanismo mediante el cual controla y mantiene el estado para un usuario que interactúa con ella. Para evitar la autenticación continua en cada página de un sitio o servicio, las aplicaciones web implementan varios mecanismos para almacenar y validar credenciales durante un período de tiempo predeterminado. Estos mecanismos se conocen como Gestión de Sesiones.

En esta prueba, el evaluador quiere verificar que las cookies y otros tokens de sesión se crean de manera segura e impredecible. Un atacante que sea capaz de predecir y falsificar una cookie débil puede secuestrar fácilmente las sesiones de usuarios legítimos.

Las cookies se utilizan para implementar la gestión de sesiones y se describen en detalle en RFC 2965. En resumen, cuando un usuario accede a una aplicación que necesita rastrear las acciones e identidad de ese usuario a través de múltiples solicitudes, una cookie (o cookies) es generada por el servidor y enviada al cliente. El cliente enviará entonces la cookie de vuelta al servidor en todas las conexiones siguientes hasta que la cookie expire o sea destruida. Los datos almacenados en la cookie pueden proporcionar al servidor un amplio espectro de información sobre quién es el usuario, qué acciones ha realizado hasta ahora, cuáles son sus preferencias, etc., proporcionando así un estado a un protocolo sin estado como HTTP.

Un ejemplo típico es proporcionado por un carrito de compras en línea. A lo largo de la sesión de un usuario, la aplicación debe mantener el rastro de su identidad, su perfil, los productos que ha elegido comprar, la cantidad, los precios individuales, los descuentos, etc. Las cookies son una forma eficiente de almacenar y pasar esta información de ida y vuelta (otros métodos son parámetros de URL y campos ocultos).

Debido a la importancia de los datos que almacenan, las cookies son por lo tanto vitales en la seguridad general de la aplicación. Ser capaz de manipular cookies puede resultar en el secuestro de sesiones de usuarios legítimos, obtener privilegios más altos en una sesión activa, y en general influir en las operaciones de la aplicación de manera no autorizada.

En esta prueba, el evaluador debe verificar si las cookies emitidas a los clientes pueden resistir una amplia gama de ataques destinados a interferir con las sesiones de usuarios legítimos y con la aplicación en sí. El objetivo general es ser capaz de falsificar una cookie que será considerada válida por la aplicación y que proporcionará algún tipo de acceso no autorizado (secuestro de sesión, escalada de privilegios, ...).

Por lo general, los principales pasos del patrón de ataque son los siguientes:

- **recolección de cookies**: recolección de un número suficiente de muestras de cookies;
- **ingeniería inversa de cookies**: análisis del algoritmo de generación de cookies;
- **manipulación de cookies**: falsificación de una cookie válida para realizar el ataque. Este último paso podría requerir un gran número de intentos, dependiendo de cómo se crea la cookie (ataque de fuerza bruta a cookies).

Otro patrón de ataque consiste en desbordar una cookie. Estrictamente hablando, este ataque tiene una naturaleza diferente, ya que aquí los evaluadores no están tratando de recrear una cookie perfectamente válida. En su lugar, el objetivo es desbordar un área de memoria, interfiriendo así con el comportamiento correcto de la aplicación y posiblemente inyectando (y ejecutando remotamente) código malicioso.

## Objetivos de la Prueba

- Recopilar tokens de sesión, para el mismo usuario y para diferentes usuarios cuando sea posible.
- Analizar y asegurar que existe suficiente aleatoriedad para detener ataques de falsificación de sesiones.
- Modificar cookies que no están firmadas y contienen información que puede ser manipulada.

## Cómo Probar

### Pruebas de Caja Negra y Ejemplos

Toda interacción entre el cliente y la aplicación debe probarse al menos contra los siguientes criterios:

- ¿Todas las directivas `Set-Cookie` están etiquetadas como `Secure`?
- ¿Alguna operación de Cookie tiene lugar sobre transporte no encriptado?
- ¿Puede forzarse la Cookie sobre transporte no encriptado?
- Si es así, ¿cómo mantiene la aplicación la seguridad?
- ¿Alguna Cookie es persistente?
- ¿Qué tiempos de `Expires` se usan en cookies persistentes, y son razonables?
- ¿Las cookies que se esperan que sean transitorias están configuradas como tales?
- ¿Qué configuraciones de `Cache-Control` de HTTP/1.1 se usan para proteger Cookies?
- ¿Qué configuraciones de `Cache-Control` de HTTP/1.0 se usan para proteger Cookies?

#### Recolección de Cookies

El primer paso requerido para manipular la cookie es entender cómo la aplicación crea y gestiona cookies. Para esta tarea, los evaluadores tienen que tratar de responder las siguientes preguntas:

- ¿Cuántas cookies usa la aplicación?

  Navega la aplicación. Nota cuándo se crean cookies. Haz una lista de cookies recibidas, la página que las establece (con la directiva set-cookie), el dominio para el cual son válidas, su valor, y sus características.

- ¿Qué partes de la aplicación generan o modifican la cookie?

  Navegando la aplicación, encuentra qué cookies permanecen constantes y cuáles se modifican. ¿Qué eventos modifican la cookie?

- ¿Qué partes de la aplicación requieren esta cookie para ser accedidas y utilizadas?

  Averigua qué partes de la aplicación necesitan una cookie. Accede a una página, luego intenta de nuevo sin la cookie, o con un valor modificado de ella. Trata de mapear qué cookies se usan dónde.

Una hoja de cálculo mapeando cada cookie a las partes correspondientes de la aplicación y la información relacionada puede ser una salida valiosa de esta fase.

#### Análisis de Sesión

Los tokens de sesión (Cookie, SessionID o Campo Oculto) en sí mismos deben examinarse para asegurar su calidad desde una perspectiva de seguridad. Deben probarse contra criterios como su aleatoriedad, unicidad, resistencia a análisis estadísticos y criptográficos y fuga de información.

- Estructura del Token y Fuga de Información

La primera etapa es examinar la estructura y contenido de un Session ID proporcionado por la aplicación. Un error común es incluir datos específicos en el Token en lugar de emitir un valor genérico y referenciar datos reales del lado del servidor.

Si el Session ID está en texto claro, la estructura y datos pertinentes pueden ser inmediatamente obvios como `192.168.100.1:owaspuser:password:15:58`.

Si parte o todo el token parece estar codificado o hasheado, debe compararse con varias técnicas para verificar ofuscación obvia. Por ejemplo, la cadena `192.168.100.1:owaspuser:password:15:58` se representa en hex, base64, y como un hash MD5:

- Hex: `3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
- Base64: `MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
- MD5: `01c2fc4f0a817afd8366689bd29dd40a`

Habiendo identificado el tipo de ofuscación, puede ser posible decodificar de vuelta a los datos originales. En la mayoría de los casos, sin embargo, esto es improbable. Aun así, puede ser útil enumerar la codificación en su lugar desde el formato del mensaje. Además, si tanto el formato como la técnica de ofuscación pueden deducirse, podrían idearse ataques de fuerza bruta automatizados.

Tokens híbridos pueden incluir información como dirección IP o ID de Usuario junto con una porción codificada, como `owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`.

Habiendo analizado un solo token de sesión, la muestra representativa debe examinarse. Un análisis simple de los tokens debería revelar inmediatamente cualquier patrón obvio. Por ejemplo, un token de 32 bits puede incluir 16 bits de datos estáticos y 16 bits de datos variables. Esto puede indicar que los primeros 16 bits representan un atributo fijo del usuario – ej. el nombre de usuario o dirección IP. Si el segundo trozo de 16 bits está incrementando a una tasa regular, puede indicar un elemento secuencial o incluso basado en tiempo en la generación de tokens. Ver ejemplos.

Si se identifican elementos estáticos en los Tokens, deben recopilarse más muestras, variando un elemento de entrada potencial a la vez. Por ejemplo, intentos de inicio de sesión a través de una cuenta de usuario diferente o desde una dirección IP diferente pueden producir una varianza en la porción previamente estática del token de sesión.

Las siguientes áreas deben abordarse durante las pruebas de estructura de Session ID único y múltiple:

- ¿Qué partes del Session ID son estáticas?
- ¿Qué información confidencial en texto claro se almacena en el Session ID? Ej. nombres de usuario/UID, direcciones IP
- ¿Qué información confidencial fácilmente decodificada se almacena?
- ¿Qué información puede deducirse de la estructura del Session ID?
- ¿Qué porciones del Session ID son estáticas para las mismas condiciones de inicio de sesión?
- ¿Qué patrones obvios están presentes en el Session ID en su conjunto, o porciones individuales?

#### Predictibilidad y Aleatoriedad del Session ID

El análisis de las áreas variables (si las hay) del Session ID debe emprenderse para establecer la existencia de cualquier patrón reconocible o predecible. Estos análisis pueden realizarse manualmente y con herramientas estadísticas o criptoanalíticas específicas o OTS para deducir cualquier patrón en el contenido del Session ID. Las verificaciones manuales deben incluir comparaciones de Session IDs emitidos para las mismas condiciones de inicio de sesión – ej., el mismo nombre de usuario, contraseña, y dirección IP.

El tiempo es un factor importante que también debe controlarse. Deben hacerse altos números de conexiones simultáneas para recopilar muestras en la misma ventana de tiempo y mantener esa variable constante. Incluso una cuantización de 50ms o menos puede ser demasiado gruesa y una muestra tomada de esta manera puede revelar componentes basados en tiempo que de otro modo se perderían.

Los elementos variables deben analizarse a lo largo del tiempo para determinar si son incrementales en naturaleza. Donde son incrementales, deben investigarse patrones relacionados con tiempo absoluto o transcurrido. Muchos sistemas usan tiempo como semilla para sus elementos pseudo-aleatorios. Donde los patrones parecen aleatorios, deben considerarse hashes unidireccionales de tiempo u otras variaciones ambientales como una posibilidad. Típicamente, el resultado de un hash criptográfico es un número decimal o hexadecimal, por lo que debería ser identificable.

En el análisis de secuencias de Session ID, patrones o ciclos, elementos estáticos y dependencias del cliente deben considerarse todos como posibles elementos contribuyentes a la estructura y función de la aplicación.

- ¿Los Session IDs son provablemente aleatorios en naturaleza? ¿Pueden reproducirse los valores resultantes?
- ¿Las mismas condiciones de entrada producen el mismo ID en una ejecución posterior?
- ¿Los Session IDs son provablemente resistentes a análisis estadísticos o criptográficos?
- ¿Qué elementos de los Session IDs están vinculados al tiempo?
- ¿Qué porciones de los Session IDs son predecibles?
- ¿Puede deducirse el siguiente ID, dado el conocimiento completo del algoritmo de generación y IDs anteriores?

#### Ingeniería Inversa de Cookies

Ahora que el evaluador ha enumerado las cookies y tiene una idea general de su uso, es hora de tener una mirada más profunda a las cookies que parecen interesantes. ¿En qué cookies está interesado el evaluador? Una cookie, para proporcionar un método seguro de gestión de sesiones, debe combinar varias características, cada una de las cuales está destinada a proteger la cookie de una clase diferente de ataques.

Estas características se resumen a continuación:

1. Impredecibilidad: una cookie debe contener cierta cantidad de datos difíciles de adivinar. Cuanto más difícil sea falsificar una cookie válida, más difícil es irrumpir en la sesión de un usuario legítimo. Si un atacante puede adivinar la cookie usada en una sesión activa de un usuario legítimo, podrán impersonar completamente a ese usuario (secuestro de sesión). Para hacer una cookie impredecible, pueden usarse valores aleatorios o criptografía.
2. Resistencia a manipulación: una cookie debe resistir intentos maliciosos de modificación. Si el evaluador recibe una cookie como `IsAdmin=No`, es trivial modificarla para obtener derechos administrativos, a menos que la aplicación realice una doble verificación (por instancia, añadiendo a la cookie un hash encriptado de su valor)
3. Expiración: una cookie crítica debe ser válida solo por un período de tiempo apropiado y debe eliminarse del disco o memoria después para evitar el riesgo de ser reproducida. Esto no se aplica a cookies que almacenan datos no críticos que necesitan recordarse a través de sesiones (ej., apariencia del sitio).
4. Bandera `Secure`: una cookie cuyo valor es crítico para la integridad de la sesión debería tener esta bandera habilitada para permitir su transmisión solo en un canal encriptado para disuadir escuchas.

El enfoque aquí es recopilar un número suficiente de instancias de una cookie y comenzar a buscar patrones en sus valores. El significado exacto de "suficiente" puede variar desde un puñado de muestras, si el método de generación de cookies es muy fácil de romper, a varios miles, si el evaluador necesita proceder con algún análisis matemático (ej., chi-cuadrados, atractores. Ver más tarde para más información).

Es importante prestar particular atención al flujo de trabajo de la aplicación, ya que el estado de una sesión puede tener un impacto pesado en las cookies recopiladas. Una cookie recopilada antes de ser autenticado puede ser muy diferente de una cookie obtenida después de la autenticación.

Otro aspecto a tener en cuenta es el tiempo. Siempre registra el tiempo exacto cuando se ha obtenido una cookie, cuando hay la posibilidad de que el tiempo juegue un rol en el valor de la cookie (el servidor podría usar una marca de tiempo como parte del valor de la cookie). El tiempo registrado podría ser el tiempo local o la marca de tiempo del servidor incluida en la respuesta HTTP (o ambos).

Al analizar los valores recopilados, el evaluador debería tratar de figurar todas las variables que podrían haber influido en el valor de la cookie y tratar de variarlas una a la vez. Pasar al servidor versiones modificadas de la misma cookie puede ser muy útil para entender cómo la aplicación lee y procesa la cookie.

Ejemplos de verificaciones a realizar en esta etapa incluyen:

- ¿Qué conjunto de caracteres se usa en la cookie? ¿La cookie tiene un valor numérico? ¿alfanumérico? ¿hexadecimal? ¿Qué sucede si el evaluador inserta en una cookie caracteres que no pertenecen al conjunto esperado?
- ¿La cookie está compuesta de diferentes sub-partes llevando diferentes piezas de información? ¿Cómo se separan las diferentes partes? ¿Con qué delimitadores? Algunas partes de la cookie podrían tener una varianza más alta, otras podrían ser constantes, otras podrían asumir solo un conjunto limitado de valores. Desglosar la cookie a sus componentes base es el primer y fundamental paso.

Un ejemplo de una cookie estructurada fácil de detectar es la siguiente:

```text
ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q
```

Este ejemplo muestra 5 campos diferentes, llevando diferentes tipos de datos:

- ID – hexadecimal
- CR – entero pequeño
- TM y LM – entero grande. (Y curiosamente tienen el mismo valor. Vale la pena ver qué sucede modificando uno de ellos)
- S – alfanumérico

Incluso cuando no se usan delimitadores, tener suficientes muestras puede ayudar a entender la estructura.

#### Ataques de Fuerza Bruta

Los ataques de fuerza bruta inevitablemente llevan a preguntas relacionadas con la predictibilidad y aleatoriedad. La varianza dentro de los Session IDs debe considerarse junto con la duración de la sesión de la aplicación y tiempos de espera. Si la variación dentro de los Session IDs es relativamente pequeña, y la validez del Session ID es larga, la probabilidad de un ataque de fuerza bruta exitoso es mucho más alta.

Un Session ID largo (o más bien uno con una gran cantidad de varianza) y un período de validez más corto lo harían mucho más difícil de tener éxito en un ataque de fuerza bruta.

- ¿Cuánto tomaría un ataque de fuerza bruta en todos los Session IDs posibles?
- ¿El espacio de Session ID es lo suficientemente grande para prevenir fuerza bruta? Por ejemplo, ¿la longitud de la clave es suficiente cuando se compara con la vida útil válida?
- ¿Los retrasos entre intentos de conexión con diferentes Session IDs mitigan el riesgo de este ataque?

### Pruebas de Caja Gris y Ejemplo

Si el evaluador tiene acceso a la implementación del esquema de gestión de sesiones, pueden verificar lo siguiente:

- Token de Sesión Aleatorio

  El Session ID o Cookie emitido al cliente no debería ser fácilmente predecible (no uses algoritmos lineales basados en variables predecibles como la dirección IP del cliente). Se anima el uso de algoritmos criptográficos con longitud de clave de 256 bits (como AES).

- Longitud del Token

  Session ID tendrá al menos 50 caracteres de longitud.

- Tiempo de Espera de Sesión

  El token de sesión debería tener un tiempo de espera definido (depende de la criticidad de los datos de aplicación gestionados)

- Configuración de Cookie:
    - no persistente: solo memoria RAM
    - segura (establecida solo en canal HTTPS): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; secure`
    - [HTTPOnly](https://owasp.org/www-community/HttpOnly) (no legible por un script): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; HttpOnly`

Más información aquí: [Pruebas de atributos de cookies](02-Testing_for_Cookies_Attributes.md)

## Herramientas

- [Zed Attack Proxy Project (ZAP)](https://www.zaproxy.org) - cuenta con un mecanismo de análisis de tokens de sesión.
- [Burp Sequencer](https://portswigger.net/burp/documentation/desktop/tools/sequencer)
- [YEHG's JHijack](https://github.com/yehgdotnet/JHijack)

## Referencias

### Documentos Técnicos

- [RFC 2965 "HTTP State Management Mechanism"](https://tools.ietf.org/html/rfc2965)
- [RFC 1750 "Randomness Recommendations for Security"](https://www.ietf.org/rfc/rfc1750.txt)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis" (2001)](https://lcamtuf.coredump.cx/oldtcp/tcpseq.html)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis - One Year Later" (2002)](https://lcamtuf.coredump.cx/newtcp/)
- [Correlation Coefficient](https://mathworld.wolfram.com/CorrelationCoefficient.html)
- [ENT](https://fourmilab.ch/random/)
- [DMA 2005-0614a - Global Hauri ViRobot Server cookie overflow](https://seclists.org/lists/fulldisclosure/2005/Jun/0188.html)
- [OWASP Code Review Guide](https://wiki.owasp.org/index.php/Category:OWASP_Code_Review_Project)