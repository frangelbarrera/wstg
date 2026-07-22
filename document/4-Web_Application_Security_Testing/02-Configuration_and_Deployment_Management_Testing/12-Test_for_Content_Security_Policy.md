# Pruebas de Content Security Policy

|ID          |
|------------|
|WSTG-CONF-12|

## Resumen

Content Security Policy (CSP) es una política declarativa de lista de permitidos impuesta a través del encabezado de respuesta `Content-Security-Policy` o elemento `<meta>` equivalente. Permite a los desarrolladores restringir las fuentes desde las cuales se cargan recursos tales como JavaScript, CSS, imágenes, archivos, etc. CSP es una técnica efectiva de defensa en profundidad para mitigar el riesgo de vulnerabilidades tales como Cross Site Scripting (XSS) y Clickjacking.

Content Security Policy soporta directivas que permiten control granular al flujo de políticas. (Ver [Referencias](#referencias) para más detalles.)

## Objetivos de Prueba

- Revisar el encabezado Content-Security-Policy o elemento meta para identificar malas configuraciones.

## Cómo Probar

Probar las debilidades de Content Security Policy (CSP) requiere más que verificar la presencia del encabezado. El tester debería evaluar si la política reduce significativamente la superficie de ataque y se impone propiamente.

### Identificar y Confirmar la Imposición de CSP

- Inspeccionar respuestas HTTP para la presencia del encabezado `Content-Security-Policy`.
- Verificar `Content-Security-Policy-Report-Only`. Si solo está presente Report-Only, la política no se impone.
- Verificar si CSP se entrega vía encabezado HTTP o etiqueta `<meta>` (se prefieren encabezados HTTP).
- Considerar que CSP entregada vía etiquetas `<meta>` no soporta ciertas directivas tales como `frame-ancestors`, `report-uri`, `report-to`, o `sandbox`.
- Confirmar que la política se aplica consistentemente a través de endpoints sensibles.

### Revisar Directivas de Alto Riesgo

Inspeccionar la política en busca de directivas inseguras o demasiado permisivas:

- `unsafe-inline` permite scripts o estilos inline y debilita significativamente las protecciones XSS.
- `unsafe-eval` permite evaluación dinámica de código (`eval()`), aumentando el riesgo de evasión.
- `unsafe-hashes` podría permitir ejecución inline si los hashes son predecibles o impropiamente delimitados.
- Fuentes comodín (`*`) podrían permitir cargar recursos desde cualquier origen.
    - Comodines parciales tales como `https://*` o `*.cdn.com` deberían evaluarse cuidadosamente.
    - Determinar si los dominios en lista blanca hospedan endpoints JSONP o contenido controlado por el usuario.
- Ausencia o mal uso de `frame-ancestors` podría exponer la aplicación a clickjacking.
- Directivas `object-src`, `base-uri` o `default-src` restrictivas faltantes podrían debilitar la efectividad de la política.
- Revisar el uso de `require-trusted-types-for` y `trusted-types`. En aplicaciones de alto riesgo, la ausencia de Trusted Types podría dejar sinks de inyección basados en DOM expuestos. Si se definen políticas `trusted-types`, asegurar que no sean demasiado permisivas.
- Verificar directivas duplicadas o definiciones de política en conflicto que podrían resultar en comportamiento de imposición no previsto.

### Validar Uso de Nonce y strict-dynamic

Si la política usa nonces:

- Confirmar que los nonces son criptográficamente aleatorios.
- Verificar que los nonces se regeneran por respuesta y no se reutilizan.
- Asegurar que los patrones de script inline heredados no se confíen inadvertidamente.

Si se usa `strict-dynamic`:

- Entender que la confianza propaga desde scripts basados en nonce o hash.
- Confirmar que ninguna cadena de confianza insegura permite carga de scripts controlados por el atacante.

### Evaluar Mecanismos de Reporte CSP

Si `report-uri` o `report-to` está configurado:

- Verificar que los endpoints de reporte sean alcanzables y funcionales.
- Determinar si información sensible se expone en reportes.
- Confirmar que el reporte no cree nuevos vectores de inyección o denegación de servicio.

### Intentar Técnicas de Evasión Controladas

Donde sea apropiado y autorizado, intentar validar la imposición probando payloads controlados:

- Intentos de inyección de script inline.
- Payloads basados en data URL.
- Manipulación de callback JSONP desde dominios en lista blanca.
- Encadenamiento de gadgets basados en DOM usando fuentes de script confiables.

La ejecución exitosa de JavaScript inyectado indica mala configuración CSP o imposición inefectiva.

### Evaluar la Fuerza de la Política

Las aplicaciones críticas para el negocio deberían aspirar a implementar una política estricta. Un CSP robusto típicamente:

- Evita `unsafe-inline` y `unsafe-eval`.
- Usa controles de script basados en nonce o hash.
- Restringe embebido de objetos (`object-src 'none'`).
- Restringe manipulación de etiqueta base (`base-uri 'none'`).
- Restringe enmarcado usando `frame-ancestors`.

### Patrones Comunes de Evasión CSP

Incluso cuando CSP está presente, malas configuraciones o debilidades de diseño podrían permitir evasión. Los testers deberían considerar los siguientes patrones comunes:

#### JSONP y Endpoints de Terceros Confiables

Si un CSP lista dominios de terceros (por ejemplo, CDNs), determinar si esos dominios exponen endpoints JSONP o contenido controlado por el usuario. Los atacantes podrían aprovechar la inyección de callback para ejecutar JavaScript arbitrario mientras aún cumplen con la política.

#### Políticas de Fuente Comodín y Amplias

Las políticas que dependen de comodines (`*`, `https://*`, `*.example.com`) expanden significativamente el límite de confianza. Subdomain takeovers o servicios de terceros comprometidos podrían habilitar evasión dentro del alcance permitido.

#### Reuso de Nonce o Nonces Predecibles

Si los nonces se reutilizan a través de solicitudes o se generan predeciblemente, los atacantes podrían reutilizarlos o adivinarlos para ejecutar scripts inyectados. Cada respuesta debería generar un nonce fresco y criptográficamente fuerte.

#### Sobre-dependencia en strict-dynamic

Mientras que `strict-dynamic` mejora las políticas basadas en nonce, la confianza propaga desde scripts confiables. Si un script confiable carga recursos controlados por el atacante, la protección podría debilitarse.

#### Directivas de Defensa en Profundidad Faltantes

La ausencia de directivas tales como:

- `object-src 'none'`
- `base-uri 'none'`
- `frame-ancestors`

podría dejar la aplicación expuesta a inyección de objetos, manipulación de etiqueta base, o clickjacking, incluso si la ejecución de script está parcialmente restringida.

#### CSP en Modo Report-Only

Las aplicaciones a veces despliegan CSP en modo `Report-Only` indefinidamente. En tales casos, las violaciones se registran pero no se bloquean, proporcionando ninguna protección real.

## Remediación

Las organizaciones deberían implementar una Content Security Policy fuerte y bien delimitada que reduzca significativamente la superficie de ataque en lugar de simplemente satisfacer requisitos de cumplimiento.

### Recomendaciones Generales

- Evitar el uso de `unsafe-inline` y `unsafe-eval`.
- Preferir controles de script basados en nonce o hash.
- Usar una directiva `default-src` restrictiva.
- Definir explícitamente:
    - `object-src 'none'`
    - `base-uri 'none'`
    - `frame-ancestors`
- Minimizar el número de dominios de terceros en lista blanca.
- Revisar regularmente las políticas CSP al introducir nuevas dependencias frontend.

### Mejores Prácticas de Despliegue

- Desplegar `Content-Security-Policy-Report-Only` temporalmente para pruebas, luego imponer `Content-Security-Policy` una vez validado.
- Monitorear reportes de violación para ruptura legítima e intentos de ataque potenciales.
- Preferir `report-to` (CSP Level 3) mientras se mantiene `report-uri` para compatibilidad hacia atrás donde se requiera.
- Asegurar que los nonces sean criptográficamente aleatorios y regenerados por respuesta.
- Revisar políticas durante refactorización importante del frontend o cambios de framework.

### Guía de Política Estricta

Una política estricta proporciona protección contra ataques XSS almacenados, reflejados y ciertos basados en DOM y debería ser la meta para aplicaciones de alto riesgo o críticas para el negocio.

Basado en enfoques basados en nonce, políticas de ejemplo incluyen:

Política Estricta Moderada:

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none';
base-uri 'none';
```

Política Estricta Bloqueada:

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none';
base-uri 'none';
```

> Nota: `'nonce-r4nd0m'` se muestra como placeholder de ejemplo. En la práctica, los nonces deben ser valores criptográficamente fuertes, codificados en base64, que se generan de manera única para cada respuesta HTTP.

Los equipos deberían adaptar políticas estrictas cuidadosamente, asegurando compatibilidad con la arquitectura de aplicación mientras mantienen objetivos de seguridad.

## Herramientas

- [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP Auditor - Extensión de Burp Suite](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator Chrome](https://chrome.google.com/webstore/detail/content-security-policy-c/ahlnecfloencbkpfnpljbojmjkfgnmdc) / [Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)
- [CSP Validator](https://cspvalidator.netlify.app/)
- [OWASP ZAP](https://www.zaproxy.org/) – Incluye análisis automatizado y pasivo para malas configuraciones CSP.
- [CSPBypass](https://cspbypass.com/) – Herramienta diseñada para ayudar a testers de seguridad a analizar e intentar técnicas de evasión contra implementaciones restrictivas de CSP.

## Referencias

- [Hoja de Referencia de Content Security Policy de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Mozilla Developer Network: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP Level 3 W3C](https://www.w3.org/TR/CSP3/)
- [CSP with Google](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [CSP A Successful Mess Between Hardening And Mitigation](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
- [The unsafe-hashes Source List Keyword](https://content-security-policy.com/unsafe-hashes/)
