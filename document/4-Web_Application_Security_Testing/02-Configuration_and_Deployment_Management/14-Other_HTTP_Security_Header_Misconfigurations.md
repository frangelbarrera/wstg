# Otras configuraciones incorrectas de encabezados de seguridad HTTP

| ID         |
|------------|
|WSTG-CONF-14|

## Resumen

Los encabezados de seguridad desempeñan un papel fundamental en la protección de las aplicaciones web frente a una amplia variedad de ataques, incluidos Cross-Site Scripting (XSS), Clickjacking y ataques de inyección de datos. Estos encabezados indican al navegador cómo gestionar los aspectos relacionados con la seguridad de la comunicación de un sitio web, lo que reduce la exposición a vectores de ataque conocidos. Sin embargo, una configuración incorrecta puede generar vulnerabilidades, debilitar las protecciones previstas o hacer que resulten ineficaces. En esta sección se describen configuraciones incorrectas comunes de los encabezados de seguridad, sus riesgos y cómo probarlos correctamente.

## Objetivos de las pruebas

- Identificar encabezados de seguridad configurados incorrectamente.
- Evaluar el impacto de los encabezados de seguridad mal configurados.
- Validar la implementación correcta de los encabezados de seguridad requeridos.

## Encabezados incluidos en esta sección

Según el [proyecto Secure Headers de OWASP](https://owasp.org/projects/secure-headers-project), esta sección cubre los encabezados de seguridad que no se abordan en otras pruebas CONF de esta guía:

- X-Frame-Options (y su sucesiva directiva CSP `frame-ancestors`): protección contra Clickjacking.
- X-Content-Type-Options: evita la detección automática de MIME.
- Referrer-Policy: controla la filtración de información del referente.
- Permissions-Policy: restringe el acceso a las funciones y API del navegador.
- Cross-Origin-Opener-Policy (COOP) y Cross-Origin-Embedder-Policy (COEP): aislamiento de procesos, necesario para el aislamiento entre orígenes.
- Clear-Site-Data: borra datos de navegación (caché, cookies y almacenamiento) al cerrar sesión o durante transiciones sensibles.

Obsoletos; no se recomiendan: `X-XSS-Protection` está obsoleto y deshabilitado o eliminado en los navegadores modernos; además, puede introducir vulnerabilidades XSS en navegadores antiguos. En su lugar, use CSP. Del mismo modo, `Public-Key-Pins` (HPKP) está obsoleto; consulte el punto sobre encabezados heredados u obsoletos más abajo.

## Configuraciones incorrectas comunes de encabezados de seguridad

- **Encabezado de seguridad con un valor vacío:** Los encabezados presentes pero sin valor pueden ser ignorados por los navegadores, lo que los hace ineficaces.
- **Encabezado de seguridad con un valor o nombre no válido (errores tipográficos):** Los nombres incorrectos o los errores ortográficos hacen que los encabezados no se reconozcan ni se apliquen.
- **Encabezados de seguridad demasiado permisivos:** Los encabezados configurados de forma demasiado amplia (por ejemplo, con caracteres comodín `*` o directivas excesivamente permisivas) pueden filtrar información o permitir el acceso a recursos fuera del alcance previsto.
- **Encabezados de seguridad duplicados:** Varias instancias del mismo encabezado con valores en conflicto pueden provocar un comportamiento impredecible del navegador y, potencialmente, deshabilitar por completo las medidas de seguridad.
- **Encabezados heredados u obsoletos:** La inclusión de encabezados obsoletos (por ejemplo, HPKP, `X-XSS-Protection`) o directivas (por ejemplo, `ALLOW-FROM` en X-Frame-Options) que los navegadores modernos ya no admiten puede generar riesgos innecesarios.
- **Ubicación incorrecta de los encabezados de seguridad:** Algunos encabezados solo son eficaces en condiciones específicas. Por ejemplo, encabezados como HSTS deben enviarse a través de HTTPS; si se envían a través de HTTP, son ineficaces.
- **Errores en el manejo de etiquetas META:** Cuando las políticas de seguridad, como Content-Security-Policy (CSP), se aplican tanto mediante encabezados HTTP como mediante etiquetas META (con `http-equiv`), existe el riesgo de que el valor de la etiqueta META sobrescriba la lógica segura definida en el encabezado HTTP o entre en conflicto con ella. Esto puede hacer que una política insegura tenga precedencia de forma inadvertida y debilite la postura de seguridad general.
- **Inyección de encabezados hop-by-hop:** Se produce cuando los intermediarios procesan incorrectamente el encabezado `Connection`, lo que permite a los atacantes enumerar y «eliminar» encabezados de seguridad internos sensibles (como `X-Forwarded-For`) antes de que la solicitud llegue al backend.

## Riesgos de los encabezados de seguridad mal configurados

- **Eficacia reducida:** Los encabezados mal configurados podrían no ofrecer la protección prevista y dejar la aplicación vulnerable a ataques como XSS, Clickjacking o ataques relacionados con CORS.
- **Anulación de medidas de seguridad:** Los encabezados duplicados o las directivas en conflicto pueden hacer que los navegadores ignoren por completo los encabezados de seguridad HTTP y, por tanto, deshabiliten las protecciones previstas.
- **Introducción de nuevos vectores de ataque:** El uso de encabezados heredados u obsoletos puede generar riesgos en lugar de mitigarlos si los navegadores modernos ya no admiten las medidas de seguridad previstas.

## Cómo probar

### Obtener y revisar los encabezados de seguridad HTTP

Para inspeccionar los encabezados de seguridad que usa una aplicación, utilice los siguientes métodos:

- **Proxies de interceptación:** Use herramientas como **Burp Suite** para analizar las respuestas del servidor.
- **Herramientas de línea de comandos:** Ejecute un comando curl para obtener los encabezados de respuesta HTTP: `curl -I https://example.com`
    - En ocasiones, la aplicación web redirige a una página nueva. Para seguir la redirección, utilice el siguiente comando:`curl -L -I https://example.com`
    - Algunos firewalls pueden bloquear el User-Agent predeterminado de curl y algunos errores de TLS/SSL también pueden impedir que devuelva la información correcta. En ese caso, puede probar el siguiente comando:
    `curl -I -L -k --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36" https://example.com`
- **Herramientas de desarrollo del navegador:** Abra las herramientas de desarrollo (F12), vaya a la pestaña **Network**, seleccione una solicitud y consulte la sección **Headers**.

### Comprobar cada encabezado frente a los valores recomendados

Para cada encabezado, confirme que esté presente y que su valor coincida con las recomendaciones actuales (consulte la [tabla de referencia autorizada del proyecto OWASP Secure Headers](https://owasp.org/projects/secure-headers-project)):

```http
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Clear-Site-Data: "cache","cookies","storage"
```

- Marque `X-XSS-Protection` si está presente: está obsoleto; confirme que se use CSP en su lugar para mitigar XSS.
- Marque `X-Frame-Options: ALLOW-FROM ...`: los navegadores modernos no lo admiten; recomiende la directiva CSP `frame-ancestors`.
- Confirme que `Clear-Site-Data` solo se envíe cuando corresponda (por ejemplo, en los endpoints para cerrar sesión), ya que borra datos del estado del lado del cliente.

### Comprobar si hay encabezados de seguridad demasiado permisivos

- **Identificar encabezados riesgosos:** Busque encabezados que puedan permitir un acceso excesivo, como los siguientes:
- **Evaluar las directivas:** Compruebe si se aplican directivas estrictas. Por ejemplo, una configuración demasiado permisiva podría ser la siguiente:

    ```http
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Credentials: true
    X-Permitted-Cross-Domain-Policies: all
    Referrer-Policy: unsafe-url
    ```

    Una configuración segura sería la siguiente:

    ```http
    Access-Control-Allow-Origin: {theallowedoriginurl}
    X-Permitted-Cross-Domain-Policies: none
    Referrer-Policy: no-referrer
    ```

- **Consultar la documentación:** Use recursos como [Mozilla Developer Network: encabezados de seguridad](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers) para revisar las directivas seguras e inseguras.

### Comprobar si hay encabezados duplicados, obsoletos o en desuso

- **Encabezados duplicados:** Asegúrese de que el mismo encabezado no esté definido varias veces con valores en conflicto.
- **Encabezados obsoletos:** Identifique y elimine los encabezados obsoletos (por ejemplo, HPKP) y las directivas desactualizadas (por ejemplo, `ALLOW-FROM` en X-Frame-Options). Consulte fuentes como [Mozilla Developer Network: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options) para conocer los estándares actuales.

### Confirmar la ubicación correcta de los encabezados de seguridad

- **Requisitos específicos del protocolo:** Valide que los encabezados destinados a contextos seguros (por ejemplo, HSTS) se envíen únicamente en las condiciones apropiadas (es decir, a través de HTTPS).
- **Envío condicional:** Algunos encabezados solo son eficaces en determinadas circunstancias. Verifique que se cumplan las condiciones necesarias para que el encabezado funcione según lo previsto.

### Evaluar el manejo de las etiquetas META

- **Comprobaciones de aplicación dual:** Cuando una política de seguridad como CSP se aplica tanto mediante un encabezado HTTP como mediante una etiqueta META con `http-equiv`, confirme que la etiqueta META no sobrescriba inadvertidamente el encabezado HTTP (que, por lo general, se considera más autoritativo).
- **Revisar el comportamiento del navegador:** Pruebe la aplicación en distintos navegadores para detectar posibles diferencias debidas a la presencia de directivas en conflicto. Siempre que sea posible, evite las definiciones dobles para prevenir fallas de seguridad imprevistas.

### Probar la eliminación de encabezados (inyección hop-by-hop)

Los atacantes pueden explotar esta situación incluyendo encabezados de seguridad sensibles en el encabezado `Connection`. El proxy, al seguir el estándar, elimina esos encabezados antes de reenviar la solicitud al backend. Esto puede permitir:

- Evadir listas de control de acceso (ACL) basadas en IP.
- Evadir las comprobaciones de identidad o autenticación realizadas en el perímetro.
- Deshabilitar las funciones de seguridad aplicadas mediante encabezados de intermediarios.

#### Identificación de encabezados internos o sensibles (reconocimiento)

Para realizar esta prueba, primero debe identificar qué encabezados usa la infraestructura interna. Puede identificarlos de las siguientes maneras:

- **Provocar páginas de error:** Envíe solicitudes mal formadas para provocar páginas de error (404, 500), que podrían filtrar encabezados internos en la respuesta.
- **Endpoints de reflexión:** Busque páginas de depuración o páginas de «eco» (por ejemplo, `/phpinfo`, `/debug`, `/env`) que muestren todos los encabezados recibidos por el backend.
- **Adivinar encabezados:** Entre los objetivos comunes se incluyen `X-Forwarded-For`, `X-Real-IP`, `X-Forwarded-Proto` y `X-Authenticated-User`.

#### Ejecución de la inyección

Intente «eliminar» un encabezado objetivo agregándolo como valor del encabezado `Connection`.

##### Escenario A: Evadir restricciones basadas en IP

```http
GET /admin HTTP/1.1
Host: example.com
X-Forwarded-For: 203.0.113.10
Connection: close, X-Forwarded-For
```

##### Escenario B: Eliminar el contexto de autenticación

```http
GET /api/user/profile HTTP/1.1
Host: example.com
X-Authenticated-User: victim_user
Connection: close, X-Authenticated-User
```

#### Analizar la respuesta

- **Vulnerable:** El comportamiento de la aplicación cambia (por ejemplo, se concede el acceso o desaparece una IP reflejada).
- **Segura:** El comportamiento de la aplicación permanece sin cambios o el proxy devuelve un `400 Bad Request`.

## Remediación

- **Configuración correcta de encabezados:** Asegúrese de que los encabezados estén implementados correctamente, con valores adecuados y sin errores tipográficos.
- **Aplicar directivas estrictas:** Configure los encabezados con los valores más seguros que permitan la funcionalidad requerida. Por ejemplo, evite usar `*` en las políticas CORS a menos que sea absolutamente necesario.
- **Eliminar encabezados obsoletos:** Sustituya los encabezados de seguridad heredados por equivalentes modernos y elimine los que ya no sean compatibles.
- **Evitar definiciones en conflicto:** Evite las definiciones duplicadas de encabezados y asegúrese de que las etiquetas META no entren en conflicto con los encabezados HTTP de las políticas de seguridad.
- **Restringir el encabezado Connection:** Configure los proxies para que ignoren los valores proporcionados por el cliente en el encabezado `Connection` que coincidan con encabezados internos sensibles.
- **Confianza cero:** Evite depender exclusivamente de encabezados hop-by-hop para tomar decisiones críticas de seguridad.

## Herramientas

- [Mozilla Observatory](https://developer.mozilla.org/en-US/observatory)
- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- Herramientas de desarrollo del navegador (Chrome, Firefox, Edge)

## Referencias

- [Proyecto Secure Headers de OWASP](https://owasp.org/projects/secure-headers-project)
- [Mozilla Developer Network: encabezados de seguridad](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)
- [RFC 6797 - HTTP Strict Transport Security (HSTS)](https://datatracker.ietf.org/doc/html/rfc6797)
- [Directrices de seguridad web de Google](https://web.dev/security-headers/)
- [HPKP ya no existe](https://scotthelme.co.uk/hpkp-is-no-more/)
- [RFC 9110 - Semántica de HTTP: encabezado Connection](https://datatracker.ietf.org/doc/html/rfc9110#section-7.6.1)
- [Abuso de encabezados de solicitud HTTP hop-by-hop](https://nathandavison.com/blog/abusing-http-hop-by-hop-request-headers)