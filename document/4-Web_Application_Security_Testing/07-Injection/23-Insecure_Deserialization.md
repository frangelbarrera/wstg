# Deserialización insegura

|ID          |
|------------|
|WSTG-INJT-23|

## Resumen

La serialización de datos es el proceso de convertir un objeto en un formato que se pueda almacenar (por ejemplo, en un archivo o una base de datos) o transmitir (por ejemplo, a través de una red) y reconstruir posteriormente. La deserialización es el proceso inverso: tomar datos estructurados en algún formato y reconstruirlos como un objeto.

La deserialización insegura se produce cuando una aplicación deserializa datos no confiables sin verificar suficientemente que los datos resultantes sean válidos. Los atacantes pueden aprovechar esta situación para manipular objetos serializados e influir en el comportamiento de la aplicación o desencadenar acciones no previstas durante el proceso de deserialización.

El impacto más crítico de la deserialización insegura es la ejecución remota de código (RCE). Sin embargo, también puede provocar denegación de servicio (DoS), omisión de autenticación, problemas de control de acceso o abuso de la lógica de la aplicación.

El JSON que se combina con objetos en entornos de ejecución de JavaScript también puede permitir la [contaminación de prototipos](22-Prototype_Pollution.md). Trátela como un caso de prueba relacionado, no como un formato de deserialización independiente.

## Objetivos de las pruebas

- Identificar los puntos de entrada en los que la aplicación acepta objetos serializados de fuentes no confiables (por ejemplo, encabezados HTTP, parámetros o cookies).
- Determinar el formato de serialización que utiliza la aplicación.
- Evaluar si la entrada serializada se valida o restringe antes de la deserialización.
- Evaluar si la manipulación de datos serializados conduce a un comportamiento inseguro o a un impacto en la seguridad.

## Cómo realizar las pruebas

### Pruebas de caja negra

#### Identificación de datos serializados

Identifique dónde procesa la aplicación los datos serializados. Inspeccione las cookies, los campos ocultos, los cuerpos de las API, los encabezados y los archivos cargados para buscar patrones reconocibles, codificaciones o características estructurales.

##### Serialización de Java

Los objetos serializados de Java suelen comenzar con los bytes hexadecimales `AC ED 00 05`. Cuando se codifican en base64, con frecuencia aparecen como `rO0`.

```http
Cookie: rememberMe=rO0ABXNyABpY...
```

##### Serialización de PHP

La serialización de PHP es legible para las personas y utiliza caracteres específicos para representar tipos de datos (por ejemplo, `O` para objeto, `a` para arreglo, `s` para cadena).

```http
Cookie: user_data=O:4:"User":2:{s:8:"username";s:5:"admin";s:8:"isAdmin";b:1;}
```

##### Pickle de Python

Pickle de Python es un formato binario que puede ser difícil de reconocer visualmente. Las cargas útiles del protocolo 0 son más legibles y suelen terminar con un punto (`.`). Los protocolos superiores se parecen a bloques binarios opacos o codificados en base64 en cookies, tokens o parámetros de tareas/colas.

Después de identificar una posible carga, envíe una prueba controlada fuera de banda (OOB) o una prueba inocua que provoque un error, generada para pickle (por ejemplo, una carga útil que realice una llamada DNS o HTTP a un servidor que usted controle). Confirme si la aplicación deserializa el valor observando la llamada, un error distintivo o el cierre inesperado de un proceso. Nunca empiece con una carga útil destructiva de RCE.

##### Node.js (node-serialize)

En las aplicaciones Node.js que utilizan la biblioteca `node-serialize`, los datos serializados suelen representarse como una cadena similar a JSON. Un indicador clave de riesgo es el marcador de una expresión de función invocada inmediatamente (IIFE), que comienza con `_$$ND_FUNC$$_`.

```javascript
{"rce":"_$$ND_FUNC$$_function (){require('child_process').exec('id')}()"}
```

Si observa este marcador en el tráfico de la aplicación, sustituya el cuerpo de la función por una llamada OOB inocua y vuelva a enviar la solicitud para confirmar la ejecución.

##### Serialización de .NET

Las aplicaciones heredadas que todavía utilizan `BinaryFormatter` o `NetDataContractSerializer` suelen codificar la salida en base64. Un patrón común de datos serializados de .NET comienza con los bytes hexadecimales `00 01 00 00 00 FF FF FF FF` o la cadena base64 `AAEAAAD/////`. Trate `BinaryFormatter` como una tecnología exclusivamente heredada; está obsoleta y se ha eliminado de las versiones modernas de .NET.

```http
Cookie: SessionData=AAEAAAD/////AQAAAAAAAAAMAgAAAF...
```

##### Deserialización basada en JSON con metadatos de tipos

Algunas bibliotecas JSON admiten tipado polimórfico. Si se aceptan metadatos de tipos controlados por un atacante, la deserialización puede crear instancias de clases inesperadas.

Jackson con el tipado predeterminado suele incluir un identificador de tipo como `@class`:

```json
{"@class":"com.example.Exploit","cmd":"id"}
```

Fastjson suele utilizar `@type`:

```json
{"@type":"com.example.Exploit","cmd":"id"}
```

Considere los especificadores de tipo `@class`, `@type` o similares controlados por un atacante como hallazgos de alta prioridad y, a continuación, confirme el problema con un gadget OOB seguro para esa biblioteca.

#### Manipulación de objetos serializados

Una vez identificado un punto de entrada, modifique los datos serializados y observe cómo responde la aplicación.

Entre las técnicas de manipulación habituales se incluyen:

- Cambiar atributos o indicadores de objetos.
- Inyectar campos adicionales.
- Modificar tipos de datos.
- Volver a enviar cargas útiles serializadas modificadas.

Ejemplo (PHP):

Un objeto serializado que representa el estado de un usuario:

```php
O:4:"User":2:{s:8:"username";s:3:"bob";s:8:"isAdmin";b:0;}
```

Si la aplicación utiliza este objeto para tomar decisiones de autorización, cambie `b:0;` por `b:1;` y vuelva a enviar la solicitud. La escalada de privilegios confirma la confianza insegura en el objeto deserializado. Al editar las longitudes de cadenas o arreglos, mantenga coherentes los prefijos de longitud de la serialización de PHP; de lo contrario, el analizador rechazará la carga útil.

#### Efectos secundarios peligrosos

Compruebe si la deserialización desencadena comportamientos no previstos, como:

- Ejecución de comandos del sistema.
- Invocación de funcionalidades internas de la aplicación.
- Consumo excesivo de recursos (memoria, CPU).
- Cierres inesperados de la aplicación o cambios de estado no previstos.

Busque indicios mediante:

- Cambios de comportamiento en las respuestas.
- Mensajes de error o trazas de pila.
- Anomalías de temporización o degradación del servicio.
- Llamadas DNS o HTTP OOB procedentes de cargas de prueba seguras.

#### Abuso de cadenas de gadgets

En muchos entornos, las vulnerabilidades de deserialización requieren una cadena de gadgets: una secuencia de fragmentos de código existentes (gadgets) en la aplicación o sus dependencias que, durante la reconstrucción, producen un resultado malicioso.

Realice las pruebas con esta secuencia:

1. Identifique las bibliotecas de terceros utilizadas (por ejemplo, Commons Collections, Spring o Jackson) a partir de encabezados, errores, archivos de dependencias o herramientas de identificación.
2. Genere una carga útil de confirmación no destructiva con una herramienta especializada.
3. Envíe la carga útil al punto de recepción de datos serializados y observe si se produce una llamada OOB. La RCE a menudo no genera un cuerpo de respuesta directo, por lo que las llamadas DNS o HTTP son la señal fiable.

Ejemplo (Java, con el gadget `URLDNS` de ysoserial únicamente para confirmar):

```bash
java -jar ysoserial.jar URLDNS http://<collaborator-host>/
```

Codifique la salida sin procesar en base64 si la aplicación lo requiere, colóquela en el parámetro o la cookie candidato y vuelva a enviar la solicitud. Una búsqueda DNS de `<collaborator-host>` confirma que la aplicación deserializó el objeto. Utilice gadgets de RCE solo después de la confirmación y únicamente en entornos de prueba autorizados.

Para PHP, utilice PHPGGC para generar una carga útil de prueba equivalente OOB o de escritura de archivos para una cadena de gadgets conocida presente en la aplicación.

### Pruebas de caja blanca

#### Revisión de código y funciones receptoras

Cuando se disponga del código fuente, no se limite a encontrar puntos de recepción. Rastree la entrada no confiable hasta cada punto de recepción y verifique qué controles existen.

Entre los puntos de recepción habituales se incluyen:

- Java: `java.io.ObjectInputStream.readObject()`, `readUnshared()`
- PHP: `unserialize()`
- Python: `pickle.loads()`, `marshal.loads()`, `shelve.open()`
- Node.js: `node-serialize.unserialize()`
- .NET: `BinaryFormatter.Deserialize()`, `NetDataContractSerializer.Deserialize()`

Para cada punto de recepción:

1. Confirme si los parámetros de solicitud, las cookies, los encabezados, los archivos o los mensajes de cola llegan hasta él.
2. Compruebe si existen listas de permitidos de clases, controles de integridad (por ejemplo, HMAC) o filtros de plataforma, como `ObjectInputFilter` de Java.
3. Anote los controles ausentes, las listas de permitidos demasiado amplias o los mecanismos de firma que el cliente pueda eludir.

`ValidatingObjectInputStream` de Apache Commons IO es una forma de implementar una lista de permitidos para Java; prefiera documentar el filtro que utiliza realmente el código, incluida la configuración de `ObjectInputFilter` del JDK.

## Corrección

Evite deserializar datos no confiables siempre que sea posible.

Si se requiere deserializar:

- Restrinja los formatos de serialización y los tipos de objetos aceptados mediante una lista de permitidos estricta.
- Implemente controles de integridad para detectar manipulaciones.
- Aplique validaciones antes de que se produzca la deserialización.
- Prefiera formatos que contengan solo datos, como JSON o XML, analizados sin metadatos de tipos controlados por un atacante ni tipado polimórfico.
- Para Java, aplique un `ObjectInputFilter` o un mecanismo de lista de permitidos, como `ValidatingObjectInputStream` de Commons IO, antes de crear instancias de clases.
- Si es necesario enviar objetos serializados al cliente, fírmelo(s) (por ejemplo, con HMAC) para que el servidor pueda rechazar las cargas útiles modificadas.
- No utilice el `BinaryFormatter` heredado de .NET para datos no confiables.

Consulte la [hoja de referencia de OWASP sobre deserialización](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html) para obtener orientación más detallada sobre la corrección.

## Herramientas

- [ysoserial](https://github.com/frohoff/ysoserial) - generar cargas útiles de deserialización de Java
- [ysoserial.net](https://github.com/pwntester/ysoserial.net) - generar cargas útiles de deserialización de .NET
- [PHPGGC](https://github.com/ambionics/phpggc) - cadenas de gadgets y generador de cargas útiles para PHP
- [Freddy, buscador de errores de deserialización](https://portswigger.net/bappstore/ae1cce0c6d6c47528b4af35faebc3ab3) - extensión de Burp para detectar deserialización de Java y .NET
- [Java Deserialization Scanner](https://portswigger.net/bappstore/228336544ebe4e68824b5146dbbd93ae) - extensión de Burp para analizar y explotar la deserialización de Java
- [GadgetProbe](https://github.com/BishopFox/GadgetProbe) - identificar bibliotecas disponibles en un classpath remoto de Java

## Referencias

- [CWE-502: Deserialización de datos no confiables](https://cwe.mitre.org/data/definitions/502.html)
- [Hoja de referencia de OWASP sobre deserialización](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
- [OWASP Top 10 2021 A08: Fallas de integridad del software y los datos](https://owasp.org/Top10/2021/A08_2021-Software_and_Data_Integrity_Failures)
- [HackTricks: Deserialización](https://book.hacktricks.wiki/en/pentesting-web/deserialization/index.html)