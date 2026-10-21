# Contaminación de prototipos

|ID          |
|------------|
|WSTG-INJT-22|

## Resumen

JavaScript es un lenguaje basado en prototipos. Casi todos los objetos heredan de `Object.prototype`, y cualquier propiedad que no se encuentra directamente en un objeto se busca a través de la cadena de prototipos. La contaminación de prototipos ocurre cuando una aplicación usa una entrada controlada por el atacante para establecer las *claves* de un objeto durante una combinación recursiva, una clonación o una asignación basada en rutas, lo que permite al atacante llegar a `Object.prototype` mediante una clave especial como `__proto__`, `constructor` o `prototype`. Como ese prototipo es compartido por todos los objetos del entorno de ejecución, la propiedad inyectada se vuelve visible silenciosamente en partes no relacionadas de la aplicación.

> Nota: Esto no es lo mismo que [la contaminación de parámetros HTTP](04-HTTP_Parameter_Pollution.md); a pesar de que los nombres son similares, las dos vulnerabilidades no están relacionadas.

La contaminación por sí sola rara vez causa daño directamente. Su impacto depende de un *gadget*: código existente que posteriormente lee una propiedad que el atacante logró introducir y la usa de forma sensible. La misma causa raíz aparece en dos contextos:

- **Lado del servidor (Node.js)**: según el gadget disponible, el impacto puede ir desde la denegación de servicio y la evasión de la lógica de seguridad hasta la ejecución remota de código. Un ejemplo conocido es la ejecución remota de código en Kibana, [CVE-2019-7609](https://nvd.nist.gov/vuln/detail/CVE-2019-7609).
- **Lado del cliente (navegador)**: combinada con un gadget adecuado, suele provocar [Cross-Site Scripting](01-Reflected_Cross_Site_Scripting.md) basado en DOM y puede usarse para evadir defensas del lado del cliente.

## Objetivos de las pruebas

- Identificar funciones y bibliotecas que combinan, clonan o asignan recursivamente propiedades controladas por el usuario.
- Determinar si la entrada del usuario puede llegar a `Object.prototype` y modificarlo.
- Identificar gadgets que conviertan la contaminación de prototipos en un impacto concreto.

## Cómo probar

El siguiente ejemplo ilustra la causa raíz. Una combinación recursiva ingenua copia todas las claves de un objeto controlado por el atacante en un objeto de destino:

```javascript
function merge(target, source) {
    for (const key in source) {
        if (typeof source[key] === "object" && typeof target[key] === "object") {
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}
```

Si el origen se analiza a partir de una entrada del usuario, como `{"__proto__": {"polluted": "yes"}}`, la asignación recorre `__proto__` y escribe en `Object.prototype`. Después, todos los objetos del entorno de ejecución heredan la propiedad introducida:

```javascript
merge({}, JSON.parse('{"__proto__": {"polluted": "yes"}}'));
({}).polluted;   // "yes"  -> Object.prototype fue contaminado
```

### Pruebas de caja negra

#### Identificar las fuentes

La contaminación de prototipos puede alcanzarse mediante cualquier entrada cuyas claves terminen convirtiéndose en nombres de propiedades de objetos. Revisar la aplicación en busca de:

- La cadena de consulta de la URL y el fragmento de la URL (hash), con notación de corchetes o de puntos; por ejemplo, `?__proto__[key]=value` o `?__proto__.key=value`.
- Cuerpos de solicitudes JSON que se deserializan y luego se combinan o clonan (los endpoints de configuración, perfil o ajustes son candidatos habituales).
- Otras entradas estructuradas que se analizan como objetos anidados, como datos de formularios o cookies.

#### Contaminación de prototipos del lado del cliente

Enviar una sonda que intente establecer una propiedad con un nombre único en el prototipo mediante una fuente candidata. Las dos codificaciones siguientes expresan la misma intención:

```text
https://example.com/#__proto__[testpolluted]=reflected
https://example.com/#constructor[prototype][testpolluted]=reflected
```

Luego, confirmar en la consola de desarrollo del navegador si la propiedad se filtró al prototipo global, leyéndola desde un objeto vacío recién creado:

```javascript
({}).testpolluted;
// "reflected" -> Object.prototype está contaminado a través de esta fuente
// undefined   -> no está contaminado a través de esta fuente
```

Si se devuelve el valor, la fuente es explotable y el siguiente paso es encontrar un gadget que convierta la propiedad contaminada en DOM XSS (por ejemplo, una propiedad que una biblioteca lea al generar marcado o configurar el comportamiento de un script). Las herramientas del navegador que buscan tanto fuentes como gadgets, como DOM Invader (véase Herramientas), agilizan considerablemente esta fase.

#### Contaminación de prototipos del lado del servidor

En este caso, el evaluador no puede leer el prototipo desde una consola, así que la detección es indirecta: contaminar una propiedad y observar un cambio de comportamiento visible externamente. Enviar un cuerpo JSON que incluya la clave especial dentro de un objeto que, por lo demás, sea normal, a un endpoint que combine o clone los datos de la solicitud:

```http
POST /api/update HTTP/1.1
Host: example.com
Content-Type: application/json

{"name":"test","__proto__":{"json spaces":10}}
```

Un indicador fiable y no destructivo en aplicaciones Express es contaminar la propiedad `json spaces`: si una respuesta JSON posterior de la aplicación aparece con formato (sangría) cuando antes no lo hacía, el servidor leyó la configuración de sangría del prototipo contaminado, lo que confirma la vulnerabilidad. Otros indicadores de comportamiento incluyen:

- La aparición en respuestas JSON posteriores de una propiedad que el cliente nunca envió.
- Un cambio en el estado HTTP, los encabezados o la negociación de contenido después de contaminar una propiedad que el framework lee internamente.
- Un error distintivo o un cambio en el análisis de entradas que antes se aceptaban.

#### Evaluar el impacto

Como el impacto depende del gadget, analizar cada caso confirmado de contaminación para determinar sus consecuencias realistas. En el lado del cliente, lo más habitual es que provoque DOM XSS o la evasión de un control de seguridad. En el lado del servidor, los gadgets han permitido históricamente escalar a denegación de servicio, evasión de la autenticación o autorización y ejecución remota de código. Tratar cualquier caso confirmado de contaminación de prototipos como potencialmente de gravedad alta hasta que el análisis de gadgets descarte su impacto.

### Pruebas de caja gris

Cuando se dispone del código fuente, buscar directamente los patrones vulnerables en lugar de sondear a ciegas.

Buscar rutinas recursivas de combinación, clonación, extensión o asignación profunda, ya sean implementadas manualmente o proporcionadas por una biblioteca de utilidades. Las funciones que recorren una ruta anidada de claves y asignan valores a un objeto son los principales puntos de entrada. Una búsqueda típica con grep apuntaría a funciones auxiliares de combinación y asignación, utilidades de clonación profunda y acceso a propiedades mediante rutas controladas por el usuario.

Para cada posible punto de entrada, confirmar si las claves se validan antes de asignarlas. El código seguro rechaza u omite `__proto__`, `constructor` y `prototype`, o utiliza objetos sin prototipo:

```javascript
// Vulnerable: asigna a ciegas en una ruta de clave anidada
target[key] = source[key];

// Más seguro: rechazar las claves peligrosas
if (key === "__proto__" || key === "constructor" || key === "prototype") {
    continue;
}
```

Confirmar también las versiones de las dependencias que tengan avisos conocidos sobre contaminación de prototipos, ya que muchas utilidades populares se han corregido con el tiempo. Cuando se dispone del código fuente, es sencillo rastrear la entrada del usuario desde el analizador de solicitudes hasta estos puntos de entrada y establecer si son alcanzables, así como identificar gadgets en la aplicación o sus dependencias.

## Remediación

- Sanear las claves de las propiedades antes de asignarlas; rechazar explícitamente `__proto__`, `constructor` y `prototype`.
- Usar objetos sin prototipo para datos similares a mapas, por ejemplo, `Object.create(null)`, o usar la estructura de datos `Map` en lugar de objetos simples.
- Congelar el prototipo base con `Object.freeze(Object.prototype)` cuando el comportamiento de la aplicación lo permita.
- Validar los datos estructurados entrantes con un esquema estricto (por ejemplo, JSON Schema) para descartar las claves inesperadas.
- Preferir utilidades de combinación y clonación bien mantenidas, y mantener todas las dependencias actualizadas a versiones que mitiguen la contaminación de prototipos.
- En Node.js, la opción de entorno de ejecución `--disable-proto=delete` elimina el accesor `__proto__` como medida de defensa en profundidad.

## Herramientas

- [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) - detección automatizada de fuentes y gadgets para la contaminación de prototipos del lado del cliente
- [Burp Suite](https://portswigger.net/burp) - interceptación y creación de cargas útiles JSON para pruebas del lado del servidor
- [ppmap](https://github.com/kleiton0x00/ppmap) - escáner de contaminación de prototipos del lado del cliente
- [ppfuzz](https://github.com/dwisiswant0/ppfuzz) - herramienta de fuzzing para la contaminación de prototipos

## Referencias

- [PortSwigger: Prototype Pollution](https://portswigger.net/web-security/prototype-pollution)
- [PortSwigger: Widespread Prototype Pollution Gadgets](https://portswigger.net/research/widespread-prototype-pollution-gadgets)
- [Olivier Arteau: Prototype Pollution Attacks in NodeJS Applications (NorthSec 2018)](https://github.com/HoLyVieR/prototype-pollution-nsec18)
- [BlackFan: Client-Side Prototype Pollution](https://github.com/BlackFan/client-side-prototype-pollution)
- [Michał Bentkowski: Prototype Pollution in Kibana (CVE-2019-7609)](https://slides.com/securitymb/prototype-pollution-in-kibana)
- [CWE-1321: Improperly Controlled Modification of Object Prototype Attributes ('Prototype Pollution')](https://cwe.mitre.org/data/definitions/1321.html)