# Pruebas de Prototype Pollution

|ID          |
|------------|
|WSTG-INPV-22|

## Resumen

JavaScript es un lenguaje basado en prototipos. Casi todo objeto hereda de `Object.prototype`, y cualquier propiedad que no se encuentre directamente en un objeto se busca a través de la cadena de prototipo. La prototype pollution (contaminación de prototipo) ocurre cuando una aplicación usa entrada controlada por el atacante para establecer las *claves* de un objeto durante una mezcla recursiva, clonación, o asignación basada en ruta, permitiendo al atacante alcanzar `Object.prototype` a través de una clave especial tal como `__proto__`, `constructor` o `prototype`. Debido a que ese prototipo es compartido por cada objeto en el runtime, la propiedad inyectada silenciosamente se vuelve visible a partes no relacionadas de la aplicación.

> Nota: Esto no es lo mismo que [HTTP Parameter Pollution](04-Testing_for_HTTP_Parameter_Pollution.md); a pesar del nombre similar, las dos vulnerabilidades no están relacionadas.

La contaminación por sí sola raramente causa daño directamente. Su impacto depende de un *gadget*: código existente que posteriormente lee una propiedad que el atacante logró plantar y luego la usa de manera sensible. La misma causa raíz aparece en dos contextos:

- **Lado del servidor (Node.js)**: dependiendo del gadget disponible, el impacto varía desde denegación de servicio y evasión de lógica de seguridad hasta ejecución remota de código. Un ejemplo bien conocido es el RCE de Kibana, [CVE-2019-7609](https://nvd.nist.gov/vuln/detail/CVE-2019-7609).
- **Lado del cliente (navegador)**: combinado con un gadget adecuado comúnmente lleva a [Cross-Site Scripting](01-Testing_for_Reflected_Cross_Site_Scripting.md) basado en DOM y puede usarse para evadir defensas del lado del cliente.

## Objetivos de Prueba

- Identificar funciones y bibliotecas que recursivamente mezclan, clonan, o asignan propiedades controladas por el usuario.
- Determinar si la entrada del usuario puede alcanzar y modificar `Object.prototype`.
- Identificar gadgets que conviertan la prototype pollution en un impacto concreto.

## Cómo Probar

El siguiente ejemplo ilustra la causa raíz. Una mezcla recursiva ingenua copia cada clave de un objeto controlado por el atacante a un objetivo:

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

Si la fuente se analiza desde entrada del usuario tal como `{"__proto__": {"polluted": "yes"}}`, la asignación camina hacia `__proto__` y escribe en `Object.prototype`. Posteriormente cada objeto en el runtime hereda la propiedad plantada:

```javascript
merge({}, JSON.parse('{"__proto__": {"polluted": "yes"}}'));
({}).polluted;   // "yes"  -> Object.prototype fue contaminado
```

### Pruebas de Caja Negra

#### Identificar las Fuentes

La prototype pollution es alcanzable a través de cualquier entrada cuyas claves terminen como nombres de propiedades de objeto. Revisar la aplicación en busca de:

- Cadena de consulta URL y el fragmento de URL (hash), usando notación de corchetes o punto, por ejemplo `?__proto__[key]=value` o `?__proto__.key=value`.
- Cuerpos de solicitud JSON que se deserializan y luego se mezclan o clonan (configuración, perfil, o endpoints de configuración son candidatos comunes).
- Otras entradas estructuradas analizadas en objetos anidados, tales como datos de formulario o cookies.

#### Probar Prototype Pollution del Lado del Cliente

Enviar una sonda que intente establecer una propiedad con nombre único en el prototipo a través de una fuente candidata. Las dos codificaciones a continuación expresan la misma intención:

```text
https://example.com/#__proto__[testpolluted]=reflected
https://example.com/#constructor[prototype][testpolluted]=reflected
```

Luego confirmar en la consola de desarrollador del navegador si la propiedad se filtró al prototipo global leyéndola desde un objeto vacío nuevo:

```javascript
({}).testpolluted;
// "reflected" -> Object.prototype está contaminado vía esta fuente
// undefined   -> no contaminado a través de esta fuente
```

Si el valor se devuelve, la fuente es explotable y el siguiente paso es encontrar un gadget que convierta la propiedad contaminada en DOM XSS (por ejemplo, una propiedad que una biblioteca lee al construir markup o configurar comportamiento de script). Herramientas del navegador que escanean tanto fuentes como gadgets, tales como DOM Invader (ver Herramientas), aceleran significativamente esta fase.

#### Probar Prototype Pollution del Lado del Servidor

El tester no puede leer el prototipo desde una consola aquí, así que la detección es indirecta: contaminar una propiedad y observar un cambio externamente visible en el comportamiento. Enviar un cuerpo JSON que anide la clave especial dentro de un objeto de otra manera normal, apuntando a un endpoint que mezcla o clona datos de solicitud:

```http
POST /api/update HTTP/1.1
Host: example.com
Content-Type: application/json

{"name":"test","__proto__":{"json spaces":10}}
```

Un indicador confiable y no destructivo en aplicaciones Express es contaminar la propiedad `json spaces`: si una respuesta JSON posterior de la aplicación vuelve pretty-printed (indentada) cuando previamente no lo estaba, el servidor leyó la configuración de indentación del prototipo contaminado, confirmando la vulnerabilidad. Otros indicadores de comportamiento incluyen:

- Una propiedad que el cliente nunca envió apareciendo en respuestas JSON subsiguientes.
- Un cambio en estado HTTP, encabezados, o negociación de contenido después de contaminar una propiedad que el framework lee internamente.
- Un error distintivo o cambio de análisis para entradas que previamente se aceptaban.

#### Evaluar el Impacto

Como el impacto depende del gadget, analizar cada contaminación confirmada para consecuencias realistas. En el lado del cliente esto es más a menudo DOM XSS o una evasión de un control de seguridad. En el lado del servidor, los gadgets han escalado históricamente a denegación de servicio, evasión de autenticación o autorización, y ejecución remota de código. Tratar cualquier prototype pollution confirmada como potencialmente de alta severidad hasta que el análisis de gadget descarte impacto.

### Pruebas de Caja Gris

Cuando el código fuente está disponible, buscar los patrones vulnerables directamente en lugar de sondear a ciegas.

Buscar rutinas recursivas de mezcla, clonación, extend, o asignación profunda, ya sean escritas a mano o proporcionadas por una biblioteca de utilidades. Las funciones que caminan una ruta anidada de claves y asignan en un objeto son los sinks primarios. Un grep típico apuntaría a helpers de mezcla y asignación, utilidades de clonación profunda, y acceso a propiedad por ruta controlada por el usuario.

Para cada sink candidato, confirmar si las claves se validan antes de la asignación. Código seguro rechaza o salta `__proto__`, `constructor` y `prototype`, o usa objetos sin prototipo:

```javascript
// Vulnerable: asigna ciegamente en una ruta de clave anidada
target[key] = source[key];

// Más seguro: rehusar claves peligrosas
if (key === "__proto__" || key === "constructor" || key === "prototype") {
    continue;
}
```

También confirmar las versiones de cualquier dependencia con avisos conocidos de prototype pollution, ya que muchas utilidades populares se han parcheado con el tiempo. Donde el código fuente está disponible es directo rastrear la entrada del usuario desde el parser de solicitud a estos sinks y establecer alcanzabilidad, e identificar gadgets en la aplicación o sus dependencias.

## Remediación

- Saneamiento de claves de propiedad antes de la asignación, rechazando explícitamente `__proto__`, `constructor` y `prototype`.
- Usar objetos sin prototipo para datos tipo mapa, por ejemplo `Object.create(null)`, o usar la estructura de datos `Map` en lugar de objetos planos.
- Congelar el prototipo base con `Object.freeze(Object.prototype)` donde el comportamiento de la aplicación lo permita.
- Validar datos estructurados entrantes contra un esquema estricto (por ejemplo, JSON Schema) para que las claves inesperadas se descarten.
- Preferir utilidades de mezcla y clonación bien mantenidas, y mantener todas las dependencias actualizadas a versiones que mitigan la prototype pollution.
- En Node.js, el flag de runtime `--disable-proto=delete` remueve el accessor `__proto__` como medida de defensa en profundidad.

## Herramientas

- [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) - descubrimiento automatizado de fuente y gadget para prototype pollution del lado del cliente
- [Burp Suite](https://portswigger.net/burp) - interceptación y creación de payloads JSON para pruebas del lado del servidor
- [ppmap](https://github.com/kleiton0x00/ppmap) - escáner para prototype pollution del lado del cliente
- [ppfuzz](https://github.com/dwisiswant0/ppfuzz) - fuzzer de prototype pollution

## Referencias

- [PortSwigger: Prototype Pollution](https://portswigger.net/web-security/prototype-pollution)
- [PortSwigger: Widespread Prototype Pollution Gadgets](https://portswigger.net/research/widespread-prototype-pollution-gadgets)
- [Olivier Arteau: Prototype Pollution Attacks in NodeJS Applications (NorthSec 2018)](https://github.com/HoLyVieR/prototype-pollution-nsec18)
- [BlackFan: Client-Side Prototype Pollution](https://github.com/BlackFan/client-side-prototype-pollution)
- [Michał Bentkowski: Prototype Pollution in Kibana (CVE-2019-7609)](https://slides.com/securitymb/prototype-pollution-in-kibana)
- [CWE-1321: Improperly Controlled Modification of Object Prototype Attributes ('Prototype Pollution')](https://cwe.mitre.org/data/definitions/1321.html)
