# Pruebas de Inyección de Plantillas del Lado del Cliente

|ID          |
|------------|
|WSTG-CLNT-15|

## Resumen

La Inyección de Plantillas del Lado del Cliente (CSTI, por sus siglas en inglés), también conocida como Inyección de Plantillas Basada en DOM, surge cuando aplicaciones que usan frameworks del lado del cliente (tales como Angular, Vue.js o Alpine.js) embeben dinámicamente entrada del usuario en el DOM de la página web. Si esta entrada se embebe en una expresión de plantilla o es interpretada por el motor de plantillas del framework, un atacante puede inyectar directivas maliciosas.

A diferencia de la [Inyección de Plantillas del Lado del Servidor (SSTI)](../07-Input_Validation_Testing/18-Testing_for_Server-side_Template_Injection.md), donde la plantilla se renderiza en el servidor, CSTI ocurre enteramente dentro del navegador del usuario. Cuando el framework escanea el DOM en busca de contenido dinámico, puede ejecutar las expresiones de plantilla inyectadas. Esto a menudo lleva a Cross-Site Scripting (XSS), pero el método de inyección y explotación difiere del XSS estándar porque el payload debe seguir la sintaxis específica del motor de plantillas (por ejemplo, {% raw %}`{{ 7*7 }}`{% endraw %}).

Esta vulnerabilidad es particularmente común en Aplicaciones de Página Única (SPAs) donde los desarrolladores podrían confiar en el renderizado del lado del cliente sin separación estricta de contextos.

## Objetivos de Prueba

- Identificar el framework del lado del cliente y su versión usada por la aplicación.
- Detectar puntos de inyección donde la entrada del usuario se refleja en el DOM y es procesada por el motor de plantillas.
- Evaluar si la inyección permite ejecución arbitraria de JavaScript (XSS) vía la sintaxis de plantilla.

## Cómo Probar

### Pruebas de Caja Negra

#### Identificación del Framework

El primer paso es identificar si un framework del lado del cliente está en uso. Los testers deberían buscar atributos específicos en el código fuente HTML o encabezados de respuesta HTTP específicos.

- **Angular:** Buscar atributos como `ng-app`, `ng-model` o `ng-bind`.
- **Vue.js:** Buscar atributos que comiencen con `v-`, tales como `v-if`, `v-for`, o la presencia del objeto global Vue en la consola.
- **Alpine.js:** Buscar `x-data`, `x-html`.

#### Detección de Inyección

Para detectar CSTI, los testers deberían inyectar caracteres que sean sintácticamente significativos para el motor de plantillas. La sintaxis más común para interpolación en muchos frameworks es la doble llave {% raw %}`{{ }}`{% endraw %}.

Una operación aritmética simple es la sonda estándar. Si la aplicación evalúa la matemática, es vulnerable.

**Sonda Genérica:**

{% raw %}

```txt
Inyectar la cadena: {{ 7*7 }}
```

{% endraw %}

- Si la aplicación renderiza `49`, CSTI está presente.
- Si la aplicación renderiza {% raw %}`{{ 7*7 }}`{% endraw %}, es probable que no sea vulnerable o que el escapado contextual estricto esté en su lugar.

#### Angular

Angular actúa sobre el DOM. Si un atacante puede inyectar una cadena que contenga expresiones Angular en el DOM antes de que Angular haga bootstrap o lo compile, la expresión será ejecutada.

**Payloads para Detección:**

{% raw %}

- `{{ 7*7 }}`
- `{{ constructor.constructor('alert(1)')() }}`

{% endraw %}

En versiones antiguas de Angular (1.x), el motor de plantillas funciona en un sandbox. La explotación requiere salir de este sandbox. La complejidad del payload depende fuertemente de la versión específica.

**Payload de Ejemplo (Angular 1.5.x sandbox bypass):**

{% raw %}

```javascript
{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}
```

{% endraw %}

#### Vue.js

Vue.js también es susceptible si los desarrolladores usan la directiva `v-html` con entrada del usuario o si montan una instancia Vue en un elemento DOM que ya contiene HTML controlado por el usuario.

**Payloads para Detección:**

{% raw %}

- `{{ 7*7 }}`

{% endraw %}

**Payload de Ejemplo (Vue.js 2.x):**

{% raw %}

```javascript
{{_v.constructor('alert(1)')()}}
```

{% endraw %}

#### Alpine.js

Alpine.js depende en gran medida de atributos DOM. Si un atacante puede controlar un nombre de atributo o inyectar en una directiva, puede ejecutar código.

**Payload de Ejemplo:**

```html
<div x-data="" x-html="'<img src=x onerror=alert(1)>'"></div>
```

### Pruebas de Caja Gris

#### Revisión de Código

En un escenario de caja gris, los testers verifican cómo se maneja la entrada del usuario en el código frontend. La clave es identificar dónde se usan "sinks" que interpretan HTML o Sintaxis de Plantilla con fuentes "tainted" (entrada del usuario).

**Sinks de Angular:**
Buscar usos de `$compile` o `ng-bind-html`.
Si se usa `ng-bind-html`, verificar si `$sce` (Strict Contextual Escaping) está configurado apropiadamente o si `$sce.trustAsHtml()` se usa en datos no confiables.

```javascript
// Ejemplo vulnerable en Angular
$scope.htmlSnippet = $sce.trustAsHtml(userInput);
```

**Sinks de Vue.js:**
Buscar la directiva `v-html`. Usar `v-html` en contenido proporcionado por el usuario es una causa principal de CSTI/XSS en Vue.

```html
<div v-html="userProvidedComment"></div>
```

**Sinks de React:**
Mientras que React es generalmente más seguro respecto a inyección de plantillas porque no escanea el DOM en busca de plantillas (JSX se compila), el uso inadecuado de `dangerouslySetInnerHTML` permite XSS basado en DOM, que es el equivalente en React del perfil de riesgo discutido aquí.

{% raw %}

```javascript
// Ejemplo vulnerable en React
<div dangerouslySetInnerHTML={{__html: userContent}} />
```

{% endraw %}

#### Revisión de Configuración

Verificar específicamente los encabezados Content Security Policy (CSP). Un CSP fuerte puede mitigar el impacto de CSTI restringiendo de dónde se pueden cargar scripts o previniendo la ejecución de scripts inline (incluyendo aquellos generados por motores de plantillas).

Buscar `unsafe-eval` en el CSP. Muchos motores de plantillas (especialmente los antiguos) requieren `unsafe-eval` para compilar plantillas dinámicamente. Si `unsafe-eval` está presente, los ataques CSTI son significativamente más fáciles de explotar.

## Remediación

- **Evitar Renderizar Entrada del Usuario como HTML:** Siempre que sea posible, usar directivas seguras que traten la entrada solo como texto.
    - Angular: Usar `ng-bind` en lugar de `ng-bind-html`.
    - Vue: Usar `v-text` o llaves {% raw %}`{{ }}`{% endraw %} (que auto-escapan HTML) en lugar de `v-html`.
    - React: Evitar `dangerouslySetInnerHTML`.
- **Saneamiento:** Si se requiere renderizado HTML, usar una biblioteca de saneamiento dedicada (como DOMPurify) para eliminar etiquetas y atributos peligrosos antes de pasar los datos al framework.
- **Política de Seguridad de Contenido (CSP):** Implementar un CSP estricto que deshabilite `unsafe-eval` y restrinja las fuentes de scripts.
- **Usar Compilación Offline:** Para frameworks como Vue o React, preferir usar pasos de build (webpack, vite) que compilan plantillas con anticipación en lugar de usar el compilador de runtime que analiza contenido DOM.

### Herramientas

- [DOMPurify](https://github.com/cure53/DOMPurify): Un sanitizador XSS solo para DOM, súper rápido, uber-tolerante para HTML, MathML y SVG.

## Referencias

### Whitepapers y Artículos

- [PortSwigger: Client-Side Template Injection](https://portswigger.net/research/server-side-template-injection)
- [Gareth Heyes: Angular Sandbox Bypasses](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)
- [Vue.js Security Guide](https://vuejs.org/guide/best-practices/security.html)
- [Angular Security Guide](https://angular.io/guide/security)
- [{% raw %}{{alert('CSTI')}}{% endraw %}: Large-Scale Detection of Client-Side Template Injection](https://ieeexplore.ieee.org/document/11352459)
