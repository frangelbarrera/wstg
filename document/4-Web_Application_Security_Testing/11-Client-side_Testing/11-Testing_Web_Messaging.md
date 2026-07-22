# Pruebas de Web Messaging

|ID          |
|------------|
|WSTG-CLNT-11|

## Resumen

Web Messaging (también conocido como [Cross Document Messaging](https://html.spec.whatwg.org/multipage/web-messaging.html#web-messaging)) permite a aplicaciones ejecutándose en diferentes dominios comunicarse de manera segura. Antes de la introducción de web messaging, la comunicación de diferentes orígenes (entre iframes, pestañas y ventanas) estaba restringida por la política del mismo origen y impuesta por el navegador. Los desarrolladores usaron múltiples hacks para lograr estas tareas, y la mayoría eran principalmente inseguros.

Esta restricción dentro del navegador está en su lugar para prevenir que un sitio web malicioso lea datos confidenciales de otros iframes, pestañas, etc.; sin embargo, hay algunos casos legítimos donde dos sitios web confiables necesitan intercambiar datos entre sí. Para satisfacer esta necesidad, Cross Document Messaging fue introducido en la especificación draft de [WHATWG HTML5](https://html.spec.whatwg.org/multipage/) y fue implementado en todos los navegadores principales. Habilita comunicaciones seguras entre múltiples orígenes a través de iframes, pestañas y ventanas.

La API de messaging introdujo el [método `postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage), con el cual mensajes de texto plano pueden ser enviados cross-origin. Consta de dos parámetros: mensaje y dominio.

Hay algunas preocupaciones de seguridad cuando se usa `*` como dominio que discutimos a continuación. Para recibir mensajes, el sitio web receptor necesita añadir un nuevo manejador de eventos, el cual tiene los siguientes atributos:

- Data, el contenido del mensaje entrante;
- Origin del documento remitente; y
- Source, la ventana de origen.

Aquí hay un ejemplo de la API de messaging en uso. Para enviar un mensaje:

```js
iframe1.contentWindow.postMessage("Hello world","https://www.example.com");
```

Para recibir un mensaje:

```js
window.addEventListener("message", handler, true);
function handler(event) {
    if(event.origin === 'chat.example.com') {
        /* procesar mensaje (event.data) */
    } else {
        /* ignorar mensajes de dominios no confiables */
    }
}
```

### Seguridad de Origin

El origen está compuesto por un esquema, hostname y puerto. Identifica de manera única el dominio que envía o recibe el mensaje, y no incluye la ruta o la parte del fragmento de la URL. Por ejemplo, `https://example.com` será considerado diferente de `http://example.com` porque el esquema del primero es `https`, mientras que el último es `http`. Esto también aplica a servidores web ejecutándose en el mismo dominio pero en diferentes puertos.

## Objetivos de Prueba

- Evaluar la seguridad del origen del mensaje.
- Validar que se estén usando métodos seguros y validando su entrada.

## Cómo Probar

### Examinar la Seguridad del Origin

Los testers deberían verificar si el código de la aplicación está filtrando y procesando mensajes de dominios confiables. Dentro del dominio emisor, también asegurar que el dominio receptor esté explícitamente establecido, y que `*` no sea usado como segundo argumento de `postMessage()`. Esta práctica podría introducir preocupaciones de seguridad y podría llevar a, en caso de una redirección o si el origen cambia por otros medios, el sitio web enviando datos a hosts desconocidos, y por lo tanto, filtrando datos confidenciales a servidores maliciosos.

Si el sitio web falla en añadir controles de seguridad para restringir los dominios u orígenes que tienen permitido enviar mensajes a un sitio web, es probable que introduzca un riesgo de seguridad. Los testers deberían examinar el código en busca de listeners de eventos de mensaje y obtener la función callback del método `addEventListener` para análisis posterior. Los dominios siempre deben ser verificados antes de la manipulación de datos.

### Examinar Validación de Entrada

Aunque el sitio web teóricamente está aceptando mensajes solo de dominios confiables, los datos todavía deben ser tratados como datos externos no confiables, y procesados con los controles de seguridad apropiados. Los testers deberían analizar el código y buscar métodos inseguros, en particular donde los datos estén siendo evaluados vía `eval()` o insertados en el DOM vía la propiedad `innerHTML`, lo cual puede crear vulnerabilidades XSS basadas en DOM.

### Análisis de Código Estático

El código JavaScript debería ser analizado para determinar cómo se implementa el web messaging. En particular, los testers deberían estar interesados en cómo el sitio web está restringiendo mensajes de dominios no confiables, y cómo se manejan los datos incluso para dominios confiables.

En este ejemplo, se necesita acceso para cada subdominio (www, chat, forums, ...) dentro del dominio owasp.org. El código está intentando aceptar cualquier dominio con `.owasp.org`:

```js
window.addEventListener("message", callback, true);

function callback(e) {
    if(e.origin.indexOf(".owasp.org")!=-1) {
        /* procesar mensaje (e.data) */
    }
}
```

La intención es permitir subdominios tales como:

- `www.owasp.org`
- `chat.owasp.org`
- `forums.owasp.org`

Desafortunadamente, esto introduce vulnerabilidades. Un atacante puede evadir fácilmente el filtro ya que un dominio como `www.owasp.org.attacker.com` coincidirá.

Aquí hay un ejemplo de código que carece de una verificación de origen. Esto es muy inseguro, ya que aceptará entrada de cualquier dominio:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        /* procesar mensaje (e.data) */
}
```

Aquí hay un ejemplo con vulnerabilidades de validación de entrada que pueden llevar a un ataque XSS:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        if(e.origin === "trusted.domain.com") {
            element.innerHTML= e.data;
        }
}
```

Un enfoque más seguro sería usar la propiedad `innerText` en lugar de `innerHTML`.

Para más recursos de OWASP respecto a web messaging, ver [Hoja de Referencia de Seguridad HTML5 de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
