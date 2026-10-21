# Política de seguridad de contenido

|ID          |
|------------|
|WSTG-CONF-12|

## Resumen

Content Security Policy (CSP) es una política declarativa de lista de permitidos que se aplica mediante el encabezado de respuesta `Content-Security-Policy` o un elemento `<meta>` equivalente. Permite a los desarrolladores restringir las fuentes desde las que se cargan recursos como JavaScript, CSS, imágenes, archivos, etc. CSP es una técnica eficaz de defensa en profundidad para mitigar el riesgo de vulnerabilidades como Cross Site Scripting (XSS) y Clickjacking.

Content Security Policy admite directivas que permiten un control granular del flujo de las políticas. (Consulta [Referencias](#referencias) para obtener más detalles.)

## Objetivos de prueba

- Revisar el encabezado Content-Security-Policy o el elemento meta para identificar errores de configuración.

## Cómo probar

Identificar las debilidades de Content Security Policy (CSP) requiere más que verificar la presencia del encabezado. El tester debería evaluar si la política reduce significativamente la superficie de ataque y se aplica correctamente.

### Identificar y confirmar la aplicación de CSP

- Inspeccionar las respuestas HTTP para comprobar la presencia del encabezado `Content-Security-Policy`.
- Comprobar si está presente `Content-Security-Policy-Report-Only`. Si solo está presente Report-Only, la política no se aplica.
- Verificar si CSP se entrega mediante un encabezado HTTP o una etiqueta `<meta>` (se prefieren los encabezados HTTP).
- Tener en cuenta que CSP entregada mediante etiquetas `<meta>` no admite ciertas directivas como `frame-ancestors`, `report-uri`, `report-to` o `sandbox`.
- Confirmar que la política se aplique de manera uniforme en los endpoints sensibles.

### Revisar directivas de alto riesgo

Inspeccionar la política en busca de directivas inseguras o demasiado permisivas:

- `unsafe-inline` permite scripts o estilos inline y debilita significativamente las protecciones contra XSS.
- `unsafe-eval` permite la evaluación dinámica de código (`eval()`), lo que aumenta el riesgo de evasión.
- `unsafe-hashes` podría permitir la ejecución inline si los hashes son predecibles o su alcance está mal definido.
- Las fuentes comodín (`*`) podrían permitir cargar recursos desde cualquier origen.
    - Los comodines parciales, como `https://*` o `*.cdn.com`, deben evaluarse cuidadosamente.
    - Determinar si los dominios incluidos en la lista de permitidos alojan endpoints JSONP o contenido controlado por usuarios.
- La ausencia de `frame-ancestors` o su uso incorrecto podría exponer la aplicación a clickjacking.
- La ausencia de las directivas `object-src`, `base-uri` o `default-src` restrictivas podría debilitar la eficacia de la política.
- Revisar el uso de `require-trusted-types-for` y `trusted-types`. En aplicaciones de alto riesgo, la ausencia de Trusted Types podría dejar expuestos los puntos de inyección basados en DOM. Si se definen políticas `trusted-types`, comprobar que no sean demasiado permisivas.
- Comprobar si hay directivas duplicadas o definiciones de política en conflicto que puedan dar lugar a una aplicación no prevista.

### Validar el uso de nonces, hashes y strict-dynamic

Si la política usa nonces:

- Confirmar que los nonces sean criptográficamente aleatorios.
- Verificar que los nonces se regeneren en cada respuesta y no se reutilicen.
- Asegurar que no se confíe inadvertidamente en patrones heredados de scripts inline.

Si la política usa fuentes basadas en hashes (p. ej., `'sha256-...'`, `'sha384-...'`, `'sha512-...'`) en lugar de nonces o junto con ellos:

- Confirmar que el hash coincida exactamente con el contenido del script o estilo inline, incluidos los espacios en blanco: cualquier cambio en el contenido inline invalida el hash (esto supone un costo de mantenimiento, no una vía de evasión, pero conviene señalarlo si los equipos lo han sorteado ampliando la política en otros puntos).
- Comprobar si se usa `unsafe-hashes` para permitir controladores de eventos inline con hash (p. ej., `onclick="..."`): esto extiende la confianza a los puntos de inyección basados en atributos y debe limitarse estrictamente.
- Determinar si existe un «gadget de hash» que un atacante pueda encontrar: un script inline existente cuyo hash esté en la lista de permitidos y que pueda reutilizarse o cuyo comportamiento pueda manipularse (p. ej., mediante un punto de inyección en el DOM) para ejecutar lógica controlada por el atacante sin necesitar un nuevo hash.

Si se usa `strict-dynamic`:

- Entender que la confianza se propaga desde scripts basados en nonce o hash.
- Confirmar que no haya ninguna cadena de confianza insegura que permita cargar scripts controlados por el atacante.
- Tener en cuenta que los navegadores que ignoran `strict-dynamic` (navegadores heredados) recurren a los hosts o esquemas indicados en `script-src`: comprobar que esta lista de respaldo no sea demasiado permisiva.

### Revisar la interacción con Trusted Types

[Trusted Types](https://w3c.github.io/trusted-types/dist/spec/) es una API del navegador, aplicada mediante CSP (`require-trusted-types-for 'script'` y `trusted-types <policy-names>`), que mitiga el XSS basado en DOM al exigir que los puntos de inyección peligrosos del DOM (`innerHTML`, `document.write`, `Function()`, `eval()`, etc.) reciban un objeto `TrustedHTML`/`TrustedScript`/`TrustedScriptURL` en lugar de una cadena sin procesar.

- Confirmar si está presente `require-trusted-types-for 'script'`. Sin esta directiva, Trusted Types no se aplica, aunque se declaren nombres de políticas `trusted-types`.
- Enumerar las políticas con nombre permitidas por `trusted-types` (p. ej., `trusted-types policyA policyB;`). Revisar la implementación de `createHTML`/`createScript`/`createScriptURL` de cada política (normalmente en el JavaScript de la aplicación) para detectar deficiencias de saneamiento: una política permisiva o que simplemente deje pasar los datos anula la protección.
- Comprobar si se usa `trusted-types *` o una política `default` demasiado permisiva, ya que cualquiera de ellas puede permitir el paso de cadenas arbitrarias.
- Como Trusted Types solo cubre los puntos de inyección de XSS en el DOM a los que se puede acceder desde un script, cotejar esto con las pruebas de [Cross-Site Scripting basado en DOM](../11-Client-side/01-DOM-based_Cross_Site_Scripting.md): CSP/Trusted Types es un control de mitigación en ese caso, no un sustituto de corregir el punto de inyección subyacente.
- Trusted Types no cubre la inyección basada en marcado (p. ej., la inyección HTML mediante plantillas renderizadas en el servidor); consultar [Inyección HTML](../11-Client-side/03-HTML_Injection.md) para esa clase de problema.

### Evaluar los mecanismos de informes de CSP

Si se configura `report-uri` o `report-to`:

- Verificar que los endpoints de informes sean accesibles y funcionen.
- Determinar si se expone información sensible en los informes (p. ej., URLs completas con cadenas de consulta o tokens en `blocked-uri`/`document-uri`).
- Confirmar que los informes no creen nuevos vectores de inyección o de denegación de servicio (p. ej., un endpoint interno que analice los informes de forma insegura o la falta de limitación de frecuencia que permita inundar el sistema de informes).
- Para `report-to` (CSP Level 3, Reporting API), confirmar que el grupo de endpoints al que hace referencia esté declarado mediante un encabezado `Reporting-Endpoints` (o el encabezado heredado `Report-To`), por ejemplo:

    ```http
    Reporting-Endpoints: csp-endpoint="https://example.com/csp-reports"
    Content-Security-Policy: default-src 'self'; report-to csp-endpoint
    ```

    Si el nombre del grupo `report-to` en CSP no coincide con un endpoint declarado, las infracciones se descartan silenciosamente: este es un error de configuración común al migrar desde `report-uri`.
- Cuando estén presentes tanto `report-uri` como `report-to` por compatibilidad con versiones anteriores, confirmar que apunten a destinos equivalentes y que los testers o la supervisión no dependan de uno solo.

### Intentar técnicas de evasión controladas

Cuando sea apropiado y se cuente con autorización, intentar validar la aplicación de la política mediante pruebas con payloads controlados:

- Intentos de inyección de scripts inline.
- Payloads basados en URL de datos.
- Manipulación de callbacks JSONP desde dominios incluidos en la lista de permitidos.
- Encadenamiento de gadgets basados en DOM mediante fuentes de scripts confiables.

La ejecución satisfactoria de JavaScript inyectado indica un error de configuración de CSP o una aplicación ineficaz de la política.

Herramientas como [CSPBypass](https://cspbypass.com/) ([código fuente](https://github.com/renniepak/CSPBypass)) automatizan esta comprobación: se pega una política candidata y la herramienta informa de técnicas de evasión conocidas que se aplican a esa política (p. ej., hosts JSONP/AngularJS/CDN incluidos en la lista de permitidos y esquemas permisivos). Considera sus resultados como un punto de partida, no como prueba de explotabilidad: antes de informar de un hallazgo, confirma que la evasión señalada realmente permita ejecutar JavaScript controlado por el atacante en el contexto de la aplicación objetivo.

Estos intentos de evasión se solapan con las [Pruebas del lado del cliente](../11-Client-side/README.md) y deben validarse junto con ellas: una evasión de CSP normalmente solo tiene impacto si alcanza un punto de inyección real cubierto allí (XSS basado en DOM, inyección HTML o ejecución de JavaScript).

### Evaluar la solidez de la política

Las aplicaciones críticas para el negocio deberían aspirar a implementar una política estricta. Una CSP robusta normalmente:

- Evita `unsafe-inline` y `unsafe-eval`.
- Usa controles de scripts basados en nonce o hash.
- Restringe la inserción de objetos (`object-src 'none'`).
- Restringe la manipulación de la etiqueta base (`base-uri 'none'`).
- Restringe el enmarcado mediante `frame-ancestors`.

### Patrones comunes de evasión de CSP

Incluso cuando CSP está presente, los errores de configuración o las debilidades de diseño podrían permitir evadirla. Los testers deberían considerar los siguientes patrones comunes:

#### JSONP y endpoints de terceros confiables

Si una CSP incluye dominios de terceros en la lista de permitidos (p. ej., CDN), determinar si esos dominios exponen endpoints JSONP o contenido controlado por usuarios. Los atacantes podrían aprovechar la inyección de callbacks para ejecutar JavaScript arbitrario sin dejar de cumplir la política.

#### Políticas de fuentes comodín y amplias

Las políticas que dependen de comodines (`*`, `https://*`, `*.example.com`) amplían significativamente el límite de confianza. La toma de control de subdominios o la vulneración de servicios de terceros podrían permitir evadir la política dentro del alcance permitido.

#### Reutilización de nonces o nonces predecibles

Si los nonces se reutilizan en varias solicitudes o se generan de forma predecible, los atacantes podrían reutilizarlos o adivinarlos para ejecutar scripts inyectados. Cada respuesta debería generar un nonce nuevo y criptográficamente sólido.

#### Dependencia excesiva de strict-dynamic

Aunque `strict-dynamic` mejora las políticas basadas en nonce, la confianza se propaga desde los scripts confiables. Si un script confiable carga recursos controlados por el atacante, la protección podría debilitarse.

#### Falta de directivas de defensa en profundidad

La ausencia de directivas como:

- `object-src 'none'`
- `base-uri 'none'`
- `frame-ancestors`

podría dejar la aplicación expuesta a la inyección de objetos, la manipulación de la etiqueta base o el clickjacking, incluso si la ejecución de scripts está parcialmente restringida.

#### CSP en modo Report-Only

A veces, las aplicaciones despliegan CSP en modo `Report-Only` de forma indefinida. En esos casos, las infracciones se registran, pero no se bloquean, por lo que no se proporciona ninguna protección real.

## Remediación

Las organizaciones deberían implementar una Content Security Policy sólida y de alcance bien definido que reduzca significativamente la superficie de ataque, en lugar de limitarse a cumplir los requisitos de conformidad.

### Recomendaciones generales

- Evitar el uso de `unsafe-inline` y `unsafe-eval`.
- Preferir controles de scripts basados en nonce o hash.
- Usar una directiva `default-src` restrictiva.
- Definir explícitamente:
    - `object-src 'none'`
    - `base-uri 'none'`
    - `frame-ancestors`
- Reducir al mínimo el número de dominios de terceros incluidos en la lista de permitidos.
- Revisar periódicamente las políticas CSP al incorporar nuevas dependencias del frontend.

### Prácticas recomendadas para el despliegue

- Desplegar temporalmente `Content-Security-Policy-Report-Only` para realizar pruebas y, una vez validada la política, aplicar `Content-Security-Policy`.
- Supervisar los informes de infracciones para detectar interrupciones legítimas y posibles intentos de ataque.
- Preferir `report-to` (CSP Level 3) y mantener `report-uri` para la compatibilidad con versiones anteriores cuando sea necesario.
- Asegurar que los nonces sean criptográficamente aleatorios y se regeneren en cada respuesta.
- Revisar las políticas durante las refactorizaciones importantes del frontend o los cambios de framework.

### Guía para una política estricta

Una política estricta protege contra ataques XSS almacenados, reflejados y algunos basados en DOM, y debería ser el objetivo de las aplicaciones de alto riesgo o críticas para el negocio.

A partir de enfoques basados en nonce, se incluyen las siguientes políticas de ejemplo:

Política estricta moderada:

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none';
base-uri 'none';
```

Política estricta reforzada:

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none';
base-uri 'none';
```

> Nota: `'nonce-r4nd0m'` se muestra como un marcador de posición de ejemplo. En la práctica, los nonces deben ser valores criptográficamente sólidos, codificados en base64 y generados de forma única para cada respuesta HTTP.

Los equipos deberían adaptar cuidadosamente las políticas estrictas para asegurar la compatibilidad con la arquitectura de la aplicación y, al mismo tiempo, mantener los objetivos de seguridad.

## Herramientas

- [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP Auditor - Extensión de Burp Suite](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator para Chrome](https://chromewebstore.google.com/detail/ahlnecfloencbkpfnpljbojmjkfgnmdc) / [Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)
- [CSP Validator](https://cspvalidator.netlify.app/)
- [ZAP](https://www.zaproxy.org/) - Incluye análisis automatizado y pasivo de errores de configuración de CSP.
- [CSPBypass](https://cspbypass.com/) - Herramienta diseñada para ayudar a los testers de seguridad a analizar e intentar técnicas de evasión contra implementaciones restrictivas de CSP.

## Referencias

- [Hoja de referencia de Content Security Policy de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Mozilla Developer Network: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
- [CSP Level 3 W3C](https://www.w3.org/TR/CSP3/)
- [CSP with Google](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [CSP A Successful Mess Between Hardening And Mitigation](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
- [The unsafe-hashes Source List Keyword](https://content-security-policy.com/unsafe-hashes/)
- [Trusted Types W3C](https://w3c.github.io/trusted-types/dist/spec/)
- [Reporting API W3C](https://www.w3.org/TR/reporting-1/)