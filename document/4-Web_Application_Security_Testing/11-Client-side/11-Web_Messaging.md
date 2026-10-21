# Mensajería web

|ID          |
|------------|
|WSTG-CLNT-11|

## Resumen

La mensajería web (también conocida como [mensajería entre documentos](https://html.spec.whatwg.org/multipage/web-messaging.html#web-messaging)) permite que las aplicaciones que se ejecutan en diferentes dominios se comuniquen de manera segura. Antes de la introducción de la mensajería web, la comunicación entre diferentes orígenes (entre iframes, pestañas y ventanas) estaba restringida por la política del mismo origen y el navegador la hacía cumplir. Los desarrolladores usaban varios trucos para realizar estas tareas, y la mayoría eran principalmente inseguros.

Esta restricción del navegador existe para evitar que un sitio web malicioso lea datos confidenciales de otros iframes, pestañas, etc.; sin embargo, hay algunos casos legítimos en los que dos sitios web de confianza necesitan intercambiar datos entre sí. Para satisfacer esta necesidad, la mensajería entre documentos se introdujo en el borrador de la especificación [WHATWG HTML5](https://html.spec.whatwg.org/multipage/) y se implementó en todos los principales navegadores. Permite comunicaciones seguras entre varios orígenes a través de iframes, pestañas y ventanas.

La API de mensajería introdujo el [método `postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage), que permite enviar mensajes de texto sin formato entre distintos orígenes. Consta de dos parámetros: el mensaje y el dominio.

El uso de `*` como dominio plantea algunos problemas de seguridad que se analizan a continuación. Para recibir mensajes, el sitio web receptor debe añadir un nuevo manejador de eventos, que tiene los siguientes atributos:

- Datos: el contenido del mensaje entrante;
- Origen del documento remitente; y
- Fuente: la ventana de origen.

A continuación se muestra un ejemplo del uso de la API de mensajería. Para enviar un mensaje:

```js
iframe1.contentWindow.postMessage("Hello world","https://www.example.com");
```

Para recibir un mensaje:

```js
window.addEventListener("message", handler, true);
function handler(event) {
    if(event.origin === 'chat.example.com') {
        /* process message (event.data) */
    } else {
        /* ignore messages from untrusted domains */
    }
}
```

### Seguridad del origen

El origen está compuesto por un esquema, un nombre de host y un puerto. Identifica de forma única el dominio que envía o recibe el mensaje, y no incluye la ruta ni el fragmento de la URL. Por ejemplo, `https://example.com` se considerará diferente de `http://example.com` porque el esquema del primero es `https`, mientras que el del segundo es `http`. Esto también se aplica a los servidores web que se ejecutan en el mismo dominio, pero en puertos diferentes.

## Objetivos de las pruebas

- Evaluar la seguridad del origen del mensaje.
- Validar que se usen métodos seguros y se validen los datos de entrada.

## Cómo probar

### Examinar la seguridad del origen

Los evaluadores deben comprobar si el código de la aplicación filtra y procesa mensajes de dominios de confianza. En el dominio remitente, también deben asegurarse de que el dominio receptor se especifique explícitamente y de que no se use `*` como segundo argumento de `postMessage()`. Esta práctica podría plantear problemas de seguridad y, en caso de una redirección o de que el origen cambie por otros medios, podría provocar que el sitio web envíe datos a hosts desconocidos y, por lo tanto, filtre datos confidenciales a servidores maliciosos.

Si el sitio web no incorpora controles de seguridad para restringir los dominios u orígenes que pueden enviarle mensajes, es probable que introduzca un riesgo de seguridad. Los evaluadores deben examinar el código en busca de escuchas de eventos de mensajes y obtener la función de devolución de llamada del método `addEventListener` para analizarla más a fondo. Los dominios siempre deben verificarse antes de manipular los datos.

### Examinar la validación de entrada

Aunque, en teoría, el sitio web solo acepta mensajes de dominios de confianza, los datos deben seguir tratándose como datos externos no confiables y procesarse con los controles de seguridad adecuados. Los evaluadores deben analizar el código y buscar métodos inseguros, en particular aquellos en los que se evalúen datos mediante `eval()` o se inserten en el DOM mediante la propiedad `innerHTML`, lo que puede crear vulnerabilidades XSS basadas en DOM.

### Análisis estático del código

Se debe analizar el código JavaScript para determinar cómo se implementa la mensajería web. En particular, a los evaluadores les debe interesar cómo restringe el sitio web los mensajes de dominios no confiables y cómo se manejan los datos, incluso cuando provienen de dominios de confianza.

En este ejemplo, se necesita acceso a cada subdominio (www, chat, forums, ...) del dominio owasp.org. El código intenta aceptar cualquier dominio que contenga `.owasp.org`:

```js
window.addEventListener("message", callback, true);

function callback(e) {
    if(e.origin.indexOf(".owasp.org")!=-1) {
        /* process message (e.data) */
    }
}
```

La intención es permitir subdominios como:

- `www.owasp.org`
- `chat.owasp.org`
- `forums.owasp.org`

Desafortunadamente, esto introduce vulnerabilidades. Un atacante puede eludir fácilmente el filtro, ya que un dominio como `www.owasp.org.attacker.com` coincidirá.

A continuación se muestra un ejemplo de código que no verifica el origen. Esto es muy inseguro, ya que aceptará datos de entrada de cualquier dominio:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        /* process message (e.data) */
}
```

A continuación se muestra un ejemplo con vulnerabilidades de validación de entrada que pueden provocar un ataque XSS:

```js
window.addEventListener("message", callback, true);

function callback(e) {
        if(e.origin === "trusted.domain.com") {
            element.innerHTML= e.data;
        }
}
```

Un enfoque más seguro sería usar la propiedad `innerText` en lugar de `innerHTML`.

Para consultar más recursos de OWASP sobre mensajería web, véase la [hoja de referencia de seguridad HTML5 de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)