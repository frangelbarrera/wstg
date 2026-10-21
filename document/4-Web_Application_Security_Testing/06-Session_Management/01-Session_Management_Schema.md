# Esquema de gestión de sesiones

|ID          |
|------------|
|WSTG-SESS-01|

## Resumen

Uno de los componentes fundamentales de cualquier aplicación basada en web es el mecanismo mediante el cual controla y mantiene el estado de un usuario que interactúa con ella. Para evitar la autenticación continua en cada página de un sitio o servicio, las aplicaciones web implementan diversos mecanismos para almacenar y validar credenciales durante un período de tiempo predeterminado. Estos mecanismos se conocen como gestión de sesiones.

En esta prueba, el evaluador quiere comprobar que las cookies y otros tokens de sesión se crean de manera segura e impredecible. Un atacante capaz de predecir y falsificar una cookie débil puede secuestrar fácilmente las sesiones de usuarios legítimos.

Las cookies se utilizan para implementar la gestión de sesiones y se describen en detalle en RFC 2965. En resumen, cuando un usuario accede a una aplicación que necesita llevar un registro de sus acciones e identidad en varias solicitudes, el servidor genera una o varias cookies y las envía al cliente. A continuación, el cliente devuelve la cookie al servidor en todas las conexiones posteriores hasta que la cookie expire o se destruya. Los datos almacenados en la cookie pueden proporcionar al servidor una amplia variedad de información sobre quién es el usuario, qué acciones ha realizado hasta el momento, cuáles son sus preferencias, etc., y así proporcionar estado a un protocolo sin estado como HTTP.

Un ejemplo típico es un carrito de compras en línea. Durante la sesión de un usuario, la aplicación debe llevar un registro de su identidad, su perfil, los productos que ha elegido comprar, la cantidad, los precios individuales, los descuentos, etc. Las cookies son una forma eficaz de almacenar y transmitir esta información (otros métodos son los parámetros de URL y los campos ocultos).

Debido a la importancia de los datos que almacenan, las cookies son fundamentales para la seguridad general de la aplicación. La posibilidad de manipular cookies puede provocar el secuestro de sesiones de usuarios legítimos, la obtención de privilegios más elevados en una sesión activa y, en general, la influencia no autorizada en las operaciones de la aplicación.

En esta prueba, el evaluador debe comprobar si las cookies emitidas a los clientes pueden resistir una amplia variedad de ataques dirigidos a interferir con las sesiones de usuarios legítimos y con la propia aplicación. El objetivo general es poder falsificar una cookie que la aplicación considere válida y que proporcione algún tipo de acceso no autorizado (secuestro de sesión, escalada de privilegios, ...).

Por lo general, los principales pasos del patrón de ataque son los siguientes:

- **recopilación de cookies**: recopilación de un número suficiente de muestras de cookies;
- **ingeniería inversa de cookies**: análisis del algoritmo de generación de cookies;
- **manipulación de cookies**: falsificación de una cookie válida para realizar el ataque. Este último paso puede requerir un gran número de intentos, según cómo se cree la cookie (ataque de fuerza bruta a cookies).

Otro patrón de ataque consiste en desbordar una cookie. En sentido estricto, este ataque es de otra naturaleza, ya que en este caso los evaluadores no intentan recrear una cookie perfectamente válida. En su lugar, el objetivo es desbordar un área de memoria e interferir así con el comportamiento correcto de la aplicación y, posiblemente, inyectar (y ejecutar de forma remota) código malicioso.

## Objetivos de la prueba

- Recopilar tokens de sesión del mismo usuario y, cuando sea posible, de usuarios diferentes.
- Analizar y asegurarse de que haya suficiente aleatoriedad para impedir ataques de falsificación de sesiones.
- Modificar las cookies que no estén firmadas y contengan información manipulable.

## Cómo probar

### Pruebas de caja negra y ejemplos

Todas las interacciones entre el cliente y la aplicación deben probarse, como mínimo, con respecto a los siguientes criterios:

- ¿Todas las directivas `Set-Cookie` tienen la marca `Secure`?
- ¿Se realizan operaciones con cookies a través de un transporte no cifrado?
- ¿Se puede forzar el envío de la cookie a través de un transporte no cifrado?
- En ese caso, ¿cómo mantiene la aplicación la seguridad?
- ¿Hay cookies persistentes?
- ¿Qué tiempos de `Expires` se utilizan en las cookies persistentes y son razonables?
- ¿Están configuradas como transitorias las cookies que deberían serlo?
- ¿Qué configuraciones de `Cache-Control` de HTTP/1.1 se utilizan para proteger las cookies?
- ¿Qué configuraciones de `Cache-Control` de HTTP/1.0 se utilizan para proteger las cookies?

#### Recopilación de cookies

El primer paso para manipular una cookie es comprender cómo la aplicación las crea y gestiona. Para esta tarea, los evaluadores deben intentar responder las siguientes preguntas:

- ¿Cuántas cookies utiliza la aplicación?

  Navegue por la aplicación. Anote cuándo se crean las cookies. Haga una lista de las cookies recibidas, la página que las establece (mediante la directiva set-cookie), el dominio para el que son válidas, su valor y sus características.

- ¿Qué partes de la aplicación generan o modifican la cookie?

  Al navegar por la aplicación, averigüe qué cookies permanecen constantes y cuáles se modifican. ¿Qué eventos modifican la cookie?

- ¿Qué partes de la aplicación requieren esta cookie para poder accederse y utilizarse?

  Averigüe qué partes de la aplicación necesitan una cookie. Acceda a una página y vuelva a intentarlo sin la cookie o con un valor modificado. Intente determinar dónde se utiliza cada cookie.

Una hoja de cálculo que relacione cada cookie con las partes correspondientes de la aplicación y la información relacionada puede ser un resultado valioso de esta fase.

#### Análisis de sesión

Los propios tokens de sesión (Cookie, SessionID o campo oculto) deben examinarse para asegurar su calidad desde una perspectiva de seguridad. Deben probarse con criterios como la aleatoriedad, la unicidad, la resistencia al análisis estadístico y criptográfico, y la fuga de información.

- Estructura del token y fuga de información

La primera etapa consiste en examinar la estructura y el contenido de un ID de sesión proporcionado por la aplicación. Un error común es incluir datos específicos en el token en lugar de emitir un valor genérico y hacer referencia a los datos reales en el lado del servidor.

Si el ID de sesión está en texto claro, la estructura y los datos pertinentes pueden ser evidentes de inmediato, por ejemplo: `192.168.100.1:owaspuser:password:15:58`.

Si parte del token o el token completo parece estar codificado o resumido, debe compararse con varias técnicas para comprobar si hay una ofuscación evidente. Por ejemplo, la cadena `192.168.100.1:owaspuser:password:15:58` se representa en hexadecimal, base64 y como un hash MD5:

- Hex: `3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
- Base64: `MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
- MD5: `01c2fc4f0a817afd8366689bd29dd40a`

Una vez identificado el tipo de ofuscación, es posible que se puedan decodificar los datos originales. Sin embargo, en la mayoría de los casos esto es poco probable. Aun así, puede ser útil deducir la codificación utilizada a partir del formato del mensaje. Además, si se pueden deducir tanto el formato como la técnica de ofuscación, podrían diseñarse ataques automatizados de fuerza bruta.

Los tokens híbridos pueden incluir información como la dirección IP o el ID de usuario junto con una parte codificada, como `owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`.

Tras analizar un token de sesión, debe examinarse una muestra representativa. Un análisis sencillo de los tokens debería revelar de inmediato cualquier patrón evidente. Por ejemplo, un token de 32 bits puede incluir 16 bits de datos estáticos y 16 bits de datos variables. Esto puede indicar que los primeros 16 bits representan un atributo fijo del usuario, por ejemplo, el nombre de usuario o la dirección IP. Si el segundo bloque de 16 bits se incrementa a un ritmo regular, puede indicar que la generación del token incluye un elemento secuencial o incluso basado en el tiempo. Véanse los ejemplos.

Si se identifican elementos estáticos en los tokens, deben recopilarse más muestras y variar un elemento de entrada potencial a la vez. Por ejemplo, los intentos de inicio de sesión con otra cuenta de usuario o desde otra dirección IP pueden producir una variación en la parte del token de sesión que antes era estática.

Durante las pruebas de la estructura de un único ID de sesión y de varios ID de sesión, deben abordarse las siguientes cuestiones:

- ¿Qué partes del ID de sesión son estáticas?
- ¿Qué información confidencial en texto claro se almacena en el ID de sesión? Por ejemplo, nombres de usuario/UID y direcciones IP.
- ¿Qué información confidencial que sea fácil de decodificar se almacena?
- ¿Qué información puede deducirse de la estructura del ID de sesión?
- ¿Qué partes del ID de sesión son estáticas cuando las condiciones de inicio de sesión son las mismas?
- ¿Qué patrones evidentes aparecen en el ID de sesión en su conjunto o en partes individuales?

#### Predictibilidad y aleatoriedad del ID de sesión

Debe analizarse cualquier área variable del ID de sesión para determinar si existen patrones reconocibles o predecibles. Estos análisis pueden realizarse manualmente y con herramientas estadísticas o criptoanalíticas específicas o disponibles en el mercado, para deducir patrones en el contenido del ID de sesión. Las comprobaciones manuales deben incluir comparaciones de ID de sesión emitidos bajo las mismas condiciones de inicio de sesión, por ejemplo, con el mismo nombre de usuario, contraseña y dirección IP.

El tiempo es un factor importante que también debe controlarse. Deben establecerse muchas conexiones simultáneas para recopilar muestras dentro de la misma ventana temporal y mantener constante esa variable. Incluso una cuantización de 50ms o menos puede ser demasiado gruesa, y una muestra obtenida de esta manera puede revelar componentes basados en el tiempo que, de otro modo, pasarían inadvertidos.

Los elementos variables deben analizarse a lo largo del tiempo para determinar si son incrementales. Si lo son, deben investigarse los patrones relacionados con el tiempo absoluto o transcurrido. Muchos sistemas utilizan el tiempo como semilla para sus elementos pseudoaleatorios. Si los patrones parecen aleatorios, debe considerarse la posibilidad de que sean hashes unidireccionales del tiempo u otras variaciones del entorno. Por lo general, el resultado de un hash criptográfico es un número decimal o hexadecimal, por lo que debería poder identificarse.

Al analizar secuencias, patrones o ciclos de ID de sesión, deben tenerse en cuenta los elementos estáticos y las dependencias del cliente como posibles factores que contribuyen a la estructura y el funcionamiento de la aplicación.

- ¿Se puede demostrar que los ID de sesión son aleatorios? ¿Se pueden reproducir los valores resultantes?
- ¿Las mismas condiciones de entrada producen el mismo ID en una ejecución posterior?
- ¿Se puede demostrar que los ID de sesión resisten el análisis estadístico o la criptoanálisis?
- ¿Qué elementos de los ID de sesión están vinculados al tiempo?
- ¿Qué partes de los ID de sesión son predecibles?
- ¿Se puede deducir el siguiente ID con pleno conocimiento del algoritmo de generación y de los ID anteriores?

#### Ingeniería inversa de cookies

Ahora que el evaluador ha enumerado las cookies y tiene una idea general de su uso, es momento de examinar con más detalle las que parecen interesantes. ¿Qué cookies le interesan al evaluador? Para proporcionar un método seguro de gestión de sesiones, una cookie debe combinar varias características, cada una de las cuales está destinada a protegerla de una clase distinta de ataques.

Estas características se resumen a continuación:

1. Impredecibilidad: una cookie debe contener cierta cantidad de datos difíciles de adivinar. Cuanto más difícil sea falsificar una cookie válida, más difícil será acceder a la sesión de un usuario legítimo. Si un atacante puede adivinar la cookie utilizada en una sesión activa de un usuario legítimo, podrá suplantar por completo a ese usuario (secuestro de sesión). Para que una cookie sea impredecible, pueden utilizarse valores aleatorios o criptografía.
2. Resistencia a la manipulación: una cookie debe resistir los intentos maliciosos de modificación. Si el evaluador recibe una cookie como `IsAdmin=No`, es trivial modificarla para obtener derechos administrativos, a menos que la aplicación realice una comprobación adicional (por ejemplo, añadiendo a la cookie un hash cifrado de su valor).
3. Expiración: una cookie crítica solo debe ser válida durante un período de tiempo apropiado y, posteriormente, debe eliminarse del disco o de la memoria para evitar el riesgo de que se vuelva a reproducir. Esto no se aplica a las cookies que almacenan datos no críticos que deben recordarse entre sesiones (por ejemplo, la apariencia del sitio).
4. Marca `Secure`: una cookie cuyo valor sea crítico para la integridad de la sesión debe tener esta marca habilitada para permitir su transmisión únicamente a través de un canal cifrado y disuadir la interceptación.

El enfoque consiste en recopilar un número suficiente de instancias de una cookie y empezar a buscar patrones en sus valores. El significado exacto de «suficiente» puede variar desde un puñado de muestras, si el método de generación de cookies es muy fácil de vulnerar, hasta varios miles, si el evaluador necesita realizar algún análisis matemático (por ejemplo, chi-cuadrados o atractores. Consulte más adelante para obtener más información).

Es importante prestar especial atención al flujo de trabajo de la aplicación, ya que el estado de una sesión puede tener un gran impacto en las cookies recopiladas. Una cookie recopilada antes de la autenticación puede ser muy diferente de una obtenida después de la autenticación.

Otro aspecto que debe tenerse en cuenta es el tiempo. Registre siempre la hora exacta en que se obtuvo una cookie, si existe la posibilidad de que el tiempo influya en su valor (el servidor podría utilizar una marca de tiempo como parte del valor de la cookie). La hora registrada puede ser la hora local, la marca de tiempo del servidor incluida en la respuesta HTTP o ambas.

Al analizar los valores recopilados, el evaluador debe intentar determinar todas las variables que podrían haber influido en el valor de la cookie y variarlas de una en una. Enviar al servidor versiones modificadas de la misma cookie puede ser muy útil para comprender cómo la aplicación la lee y la procesa.

Algunos ejemplos de comprobaciones que deben realizarse en esta etapa son:

- ¿Qué conjunto de caracteres se utiliza en la cookie? ¿La cookie tiene un valor numérico, alfanumérico o hexadecimal? ¿Qué sucede si el evaluador introduce en una cookie caracteres que no pertenecen al conjunto esperado?
- ¿La cookie está compuesta por distintas subpartes que contienen diferentes datos? ¿Cómo se separan las distintas partes? ¿Qué delimitadores se utilizan? Algunas partes de la cookie pueden tener una mayor variabilidad, otras pueden ser constantes y otras pueden adoptar solo un conjunto limitado de valores. Descomponer la cookie en sus componentes básicos es el primer paso y el más fundamental.

A continuación se muestra un ejemplo de una cookie estructurada fácil de identificar:

```text
ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q
```

Este ejemplo muestra 5 campos diferentes que contienen distintos tipos de datos:

- ID - hexadecimal
- CR - entero pequeño
- TM y LM - entero grande. (Y, curiosamente, tienen el mismo valor. Vale la pena ver qué ocurre al modificar uno de ellos).
- S - alfanumérico

Incluso cuando no se utilizan delimitadores, disponer de suficientes muestras puede ayudar a comprender la estructura.

#### Ataques de fuerza bruta

Los ataques de fuerza bruta conducen inevitablemente a preguntas sobre la predictibilidad y la aleatoriedad. La variabilidad de los ID de sesión debe considerarse junto con la duración de las sesiones de la aplicación y los tiempos de espera. Si la variación de los ID de sesión es relativamente pequeña y su validez es prolongada, la probabilidad de que un ataque de fuerza bruta tenga éxito es mucho mayor.

Un ID de sesión largo (o, mejor dicho, uno con una gran variabilidad) y un período de validez más corto dificultarían mucho más que un ataque de fuerza bruta tuviera éxito.

- ¿Cuánto tiempo tomaría un ataque de fuerza bruta que probara todos los ID de sesión posibles?
- ¿El espacio de ID de sesión es lo suficientemente grande como para impedir la fuerza bruta? Por ejemplo, ¿la longitud de la clave es suficiente en relación con su período de validez?
- ¿Los retrasos entre intentos de conexión con distintos ID de sesión mitigan el riesgo de este ataque?

### Pruebas de caja gris y ejemplo

Si el evaluador tiene acceso a la implementación del esquema de gestión de sesiones, puede comprobar lo siguiente:

- Token de sesión aleatorio

  El ID de sesión o la cookie emitidos al cliente no deben ser fáciles de predecir (no se deben utilizar algoritmos lineales basados en variables predecibles, como la dirección IP del cliente). Se recomienda utilizar algoritmos criptográficos con una longitud de clave de 256 bits (como AES).

- Longitud del token

  El ID de sesión debe tener al menos 50 caracteres.

- Tiempo de espera de sesión

  El token de sesión debe tener un tiempo de espera definido (que depende de la criticidad de los datos gestionados por la aplicación).

- Configuración de cookies:
    - no persistente: solo en la memoria RAM
    - segura (establecida solo en el canal HTTPS): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; secure`
    - [HTTPOnly](https://owasp.org/www-community/HttpOnly) (no legible por un script): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; HttpOnly`

Más información aquí: [atributos de las cookies](02-Cookies_Attributes.md)

## Herramientas

- [Zed Attack Proxy Project (ZAP)](https://www.zaproxy.org) - incluye un mecanismo de análisis de tokens de sesión.
- [Burp Sequencer](https://portswigger.net/burp/documentation/desktop/tools/sequencer)
- [YEHG's JHijack](https://github.com/yehgdotnet/JHijack)
- [Gori](https://github.com/hahwul/gori) - incluye un Sequencer para analizar la aleatoriedad de los tokens de sesión.

## Referencias

### Documentos técnicos

- [RFC 2965 "HTTP State Management Mechanism"](https://tools.ietf.org/html/rfc2965)
- [RFC 1750 "Randomness Recommendations for Security"](https://www.ietf.org/rfc/rfc1750.txt)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis" (2001)](https://lcamtuf.coredump.cx/oldtcp/tcpseq.html)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis - One Year Later" (2002)](https://lcamtuf.coredump.cx/newtcp/)
- [Coeficiente de correlación](https://mathworld.wolfram.com/CorrelationCoefficient.html)
- [ENT](https://fourmilab.ch/random/)
- [DMA 2005-0614a - Global Hauri ViRobot Server cookie overflow](https://seclists.org/lists/fulldisclosure/2005/Jun/0188.html)