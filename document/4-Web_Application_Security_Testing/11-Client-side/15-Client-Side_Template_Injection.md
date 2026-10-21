# Inyección de Plantillas del Lado del Cliente

|ID          |
|------------|
|WSTG-CLNT-15|

## Resumen

La Inyección de Plantillas del Lado del Cliente (CSTI, por sus siglas en inglés), también conocida como Inyección de Plantillas Basada en DOM, surge cuando aplicaciones que usan frameworks del lado del cliente (como Angular, Vue.js o Alpine.js) insertan dinámicamente entradas del usuario en el DOM de la página web. Si esta entrada se inserta en una expresión de plantilla o es interpretada por el motor de plantillas del framework, un atacante puede inyectar directivas maliciosas.

A diferencia de la [Inyección de Plantillas del Lado del Servidor (SSTI)](../07-Injection/18-Server-side_Template_Injection.md), donde la plantilla se renderiza en el servidor, la CSTI ocurre enteramente dentro del navegador del usuario. Cuando el framework analiza el DOM en busca de contenido dinámico, puede ejecutar las expresiones de plantilla inyectadas. Esto suele provocar Cross-Site Scripting (XSS), pero el método de inyección y explotación difiere del XSS estándar porque la carga útil debe seguir la sintaxis específica del motor de plantillas (por ejemplo, {% raw %}`{{ 7*7 }}`{% endraw %}).

Esta vulnerabilidad es particularmente común en las aplicaciones de página única (SPA), donde los desarrolladores podrían confiar en la representación del lado del cliente sin una separación estricta de contextos.

## Objetivos de la prueba

- Identificar el framework del lado del cliente y su versión utilizados por la aplicación.
- Detectar puntos de inyección donde la entrada del usuario se refleja en el DOM y es procesada por el motor de plantillas.
- Evaluar si la inyección permite la ejecución arbitraria de JavaScript (XSS) mediante la sintaxis de la plantilla.

## Cómo probar

### Pruebas de caja negra

#### Identificación del framework

El primer paso es identificar si se usa un framework del lado del cliente. Los evaluadores deben buscar atributos específicos en el código fuente HTML o encabezados de respuesta HTTP específicos.

- **Angular:** Buscar atributos como `ng-app`, `ng-model` o `ng-bind`.
- **Vue.js:** Buscar atributos que comiencen con `v-`, como `v-if`, `v-for`, o la presencia del objeto global de Vue en la consola.
- **Alpine.js:** Buscar `x-data`, `x-html`.

#### Detección de inyección

Para detectar CSTI, los evaluadores deben inyectar caracteres que tengan importancia sintáctica para el motor de plantillas. La sintaxis de interpolación más común en muchos frameworks son las llaves dobles {% raw %}`{{ }}`{% endraw %}.

Una operación aritmética simple es la sonda estándar. Si la aplicación evalúa la operación matemática, es vulnerable.

**Sonda genérica:**

{% raw %}

```txt
Inyectar la cadena: {{ 7*7 }}
```

{% endraw %}

- Si la aplicación muestra `49`, hay CSTI.
- Si la aplicación muestra {% raw %}`{{ 7*7 }}`{% endraw %}, es probable que no sea vulnerable o que se aplique un escape contextual estricto.

#### Angular

Angular actúa sobre el DOM. Si un atacante puede inyectar en el DOM una cadena que contenga expresiones de Angular antes de que Angular se inicialice o compile el contenido, la expresión se ejecutará.

**Cargas útiles para la detección:**

{% raw %}

- `{{ 7*7 }}`
- `{{ constructor.constructor('alert(1)')() }}`

{% endraw %}

En versiones antiguas de Angular (1.x), el motor de plantillas funciona en un sandbox. Para explotar la vulnerabilidad, es necesario salir de este sandbox. La complejidad de la carga útil depende en gran medida de la versión específica.

**Carga útil de ejemplo (elusión del sandbox de Angular 1.5.x):**

{% raw %}

```javascript
{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}
```

{% endraw %}

#### Vue.js

Vue.js también es vulnerable si los desarrolladores usan la directiva `v-html` con entradas del usuario o montan una instancia de Vue en un elemento DOM que ya contiene HTML controlado por el usuario.

**Cargas útiles para la detección:**

{% raw %}

- `{{ 7*7 }}`

{% endraw %}

**Carga útil de ejemplo (Vue.js 2.x):**

{% raw %}

```javascript
{{_v.constructor('alert(1)')()}}
```

{% endraw %}

#### Alpine.js

Alpine.js depende en gran medida de los atributos DOM. Si un atacante puede controlar el nombre de un atributo o inyectar contenido en una directiva, puede ejecutar código.

**Carga útil de ejemplo:**

```html
<div x-data="" x-html="'<img src=x onerror=alert(1)>'"></div>
```

### Pruebas de caja gris

#### Revisión de código

En un escenario de caja gris, los evaluadores verifican cómo se maneja la entrada del usuario en el código del frontend. La clave es identificar dónde se usan «sinks» que interpretan HTML o sintaxis de plantillas con fuentes «tainted» (entradas del usuario).

**Sinks de Angular:**
Buscar usos de `$compile` o `ng-bind-html`.
Si se usa `ng-bind-html`, comprobar si `$sce` (Strict Contextual Escaping) está configurado correctamente o si se usa `$sce.trustAsHtml()` con datos no confiables.

```javascript
// Ejemplo vulnerable en Angular
$scope.htmlSnippet = $sce.trustAsHtml(userInput);
```

**Sinks de Vue.js:**
Buscar la directiva `v-html`. Usar `v-html` con contenido proporcionado por el usuario es una de las principales causas de CSTI/XSS en Vue.

```html
<div v-html="userProvidedComment"></div>
```

**Sinks de React:**
Aunque React suele ser más seguro frente a la inyección de plantillas porque no analiza el DOM en busca de plantillas (JSX se compila), el uso inadecuado de `dangerouslySetInnerHTML` permite XSS basado en DOM, que en React es el equivalente del perfil de riesgo descrito aquí.

{% raw %}

```javascript
// Ejemplo vulnerable en React
<div dangerouslySetInnerHTML={{__html: userContent}} />
```

{% endraw %}

#### Revisión de la configuración

Comprobar específicamente los encabezados de Content Security Policy (CSP). Una CSP sólida puede mitigar el impacto de la CSTI al restringir desde dónde se pueden cargar scripts o impedir la ejecución de scripts en línea (incluidos los generados por motores de plantillas).

Buscar `unsafe-eval` en la CSP. Muchos motores de plantillas (especialmente los antiguos) requieren `unsafe-eval` para compilar plantillas dinámicamente. Si `unsafe-eval` está presente, los ataques CSTI son mucho más fáciles de explotar.

## Remediación

- **Evitar representar entradas del usuario como HTML:** Siempre que sea posible, usar directivas seguras que traten las entradas únicamente como texto.
    - Angular: Usar `ng-bind` en lugar de `ng-bind-html`.
    - Vue: Usar `v-text` o llaves {% raw %}`{{ }}`{% endraw %} (que escapan automáticamente el HTML) en lugar de `v-html`.
    - React: Evitar `dangerouslySetInnerHTML`.
- **Saneamiento:** Si es necesario representar HTML, usar una biblioteca de saneamiento específica (como DOMPurify) para eliminar las etiquetas y los atributos peligrosos antes de pasar los datos al framework.
- **Política de seguridad de contenido (CSP):** Implementar una CSP estricta que deshabilite `unsafe-eval` y restrinja las fuentes de scripts.
- **Usar compilación fuera de línea:** Para frameworks como Vue o React, preferir pasos de compilación (webpack, vite) que compilen las plantillas de antemano, en lugar de usar el compilador en tiempo de ejecución que analiza el contenido del DOM.

### Herramientas

- [DOMPurify](https://github.com/cure53/DOMPurify): Un sanitizador XSS exclusivo para DOM, ultrarrápido y muy permisivo, para HTML, MathML y SVG.

## Referencias

### Artículos y documentos técnicos

- [PortSwigger: Inyección de plantillas del lado del cliente](https://portswigger.net/research/server-side-template-injection)
- [Gareth Heyes: Elusiones del sandbox de Angular](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)
- [Guía de seguridad de Vue.js](https://vuejs.org/guide/best-practices/security.html)
- [Guía de seguridad de Angular](https://angular.dev/best-practices/security)
- [{% raw %}{{alert(’CSTI’)}}{% endraw %}: Detección a gran escala de la inyección de plantillas del lado del cliente](https://ieeexplore.ieee.org/document/11352459)