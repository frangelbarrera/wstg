# Probar Otras Mala Configuraciones de Encabezados de Seguridad HTTP

| ID         |
|------------|
|WSTG-CONF-14|

## Resumen

Los encabezados de seguridad juegan un rol vital en proteger las aplicaciones web de una amplia gama de ataques, incluyendo Cross-Site Scripting (XSS), Clickjacking, y ataques de inyección de datos. Estos encabezados instruyen al navegador sobre cómo manejar aspectos relacionados con la seguridad de la comunicación de un sitio web, reduciendo la exposición a vectores de ataque conocidos. Sin embargo, las malas configuraciones pueden llevar a vulnerabilidades, debilitando las protecciones de seguridad previstas o haciéndolas inefectivas. Esta sección describe malas configuraciones comunes de encabezados de seguridad, sus riesgos, y cómo probarlas apropiadamente.

## Objetivos de Prueba

- Identificar encabezados de seguridad mal configurados.
- Evaluar el impacto de encabezados de seguridad mal configurados.
- Validar la implementación correcta de los encabezados de seguridad requeridos.

## Malas Configuraciones Comunes de Encabezados de Seguridad

- **Encabezado de Seguridad con Valor Vacío:** Los encabezados que están presentes pero carecen de valor podrían ser ignorados por los navegadores, haciéndolos inefectivos.
- **Encabezado de Seguridad con Valor o Nombre Inválido (Typos):** Nombres de encabezado incorrectos o errores ortográficos resultan en encabezados no reconocidos o no impuestos.
- **Encabezados de Seguridad Demasiado Permisivos:** Encabezados configurados demasiado ampliamente (por ejemplo, usando caracteres comodín `*` o directivas demasiado permisivas) pueden filtrar información o permitir acceso a recursos más allá del alcance previsto.
- **Encabezados de Seguridad Duplicados:** Múltiples ocurrencias del mismo encabezado con valores en conflicto pueden llevar a comportamiento impredecible del navegador, potencialmente deshabilitando las medidas de seguridad enteramente.
- **Encabezados Heredados o Obsoletos:** La inclusión de encabezados obsoletos (por ejemplo, HPKP) o directivas (por ejemplo, `ALLOW-FROM` en X-Frame-Options) que ya no son soportadas por navegadores modernos podría crear riesgos innecesarios.
- **Colocación Inválida de Encabezados de Seguridad:** Algunos encabezados solo son efectivos bajo condiciones específicas. Por ejemplo, encabezados como HSTS deben entregarse sobre HTTPS; si se envían sobre HTTP, se vuelven inefectivos.
- **Errores de Manejo de Etiquetas META:** En casos donde políticas de seguridad tales como Content-Security-Policy (CSP) se imponen vía tanto encabezados HTTP como etiquetas META (usando `http-equiv`), hay un riesgo de que el valor de la etiqueta META pueda sobrescribir o entrar en conflicto con la lógica segura definida en el encabezado HTTP. Esto puede llevar a un escenario donde una política insegura inadvertidamente toma precedencia, debilitando la postura de seguridad general.
- **Inyección de Encabezados Hop-by-Hop:** Ocurre cuando intermediarios procesan incorrectamente el encabezado `Connection`, permitiendo a los atacantes listar y "stripear" encabezados sensibles de seguridad internos (como `X-Forwarded-For`) antes de que la solicitud alcance el backend.

## Riesgos de Encabezados de Seguridad Mal Configurados

- **Efectividad Reducida:** Los encabezados mal configurados podrían no proporcionar la protección prevista, dejando la aplicación vulnerable a ataques tales como XSS, Clickjacking, o explotaciones relacionadas con CORS.
- **Ruptura de Medidas de Seguridad:** Encabezados duplicados o directivas en conflicto pueden resultar en navegadores ignorando los encabezados de seguridad HTTP enteramente, por lo tanto deshabilitando las protecciones previstas.
- **Introducción de Nuevos Vectores de Ataque:** El uso de encabezados heredados u obsoletos podría introducir riesgos en lugar de mitigarlos si los navegadores modernos ya no soportan las medidas de seguridad previstas.

## Cómo Probar

### Obtener y Revisar Encabezados de Seguridad HTTP

Para inspeccionar los encabezados de seguridad usados por una aplicación, emplear los siguientes métodos:

- **Proxies de Intercepción:** Usar herramientas tales como **Burp Suite** para analizar respuestas del servidor.
- **Herramientas de Línea de Comandos:** Ejecutar un comando curl para recuperar encabezados de respuesta HTTP: `curl -I https://example.com`
    - A veces la aplicación web redirigirá a una nueva página, para seguir la redirección usar el siguiente comando:`curl -L -I https://example.com`
    - Algunos Firewalls podrían bloquear el User-Agent por defecto de curl y algunos errores TLS/SSL también prevendrán que devuelva la información correcta, en este caso se podría intentar usar el siguiente comando:
    `curl -I -L -k --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36" https://example.com`
- **Herramientas de Desarrollador del Navegador:** Abrir herramientas de desarrollador (F12), navegar a la pestaña **Network**, seleccionar una solicitud, y ver la sección **Headers**.

### Verificar Encabezados de Seguridad Demasiado Permisivos

- **Identificar Encabezados Riesgosos:** Buscar encabezados que podrían permitir acceso excesivo, tales como:
- **Evaluar Directivas:** Verificar si se imponen directivas estrictas. Por ejemplo, una configuración demasiado permisiva podría aparecer como:

    ```http
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Credentials: true
    X-Permitted-Cross-Domain-Policies: all
    Referrer-Policy: unsafe-url
    ```

    Una configuración segura se vería como:

    ```http
    Access-Control-Allow-Origin: {theallowedoriginurl}
    X-Permitted-Cross-Domain-Policies: none
    Referrer-Policy: no-referrer
    ```

- **Referencia Cruzada de Documentación:** Usar recursos tales como [Mozilla Developer Network: Security Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) para revisar directivas seguras e inseguras.

### Verificar Encabezados Duplicados, Obsoletos

- **Encabezados Duplicados:** Asegurar que el mismo encabezado no esté definido múltiples veces con valores en conflicto.
- **Encabezados Obsoletos:** Identificar y remover encabezados obsoletos (por ejemplo, HPKP) y directivas desactualizadas (por ejemplo, `ALLOW-FROM` en X-Frame-Options). Referirse a fuentes como [Mozilla Developer Network: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) para estándares actuales.

### Confirmar Colocación Apropiada de Encabezados de Seguridad

- **Requisitos Específicos de Protocolo:** Validar que los encabezados destinados a contextos seguros (por ejemplo, HSTS) se entreguen solo bajo condiciones apropiadas (i.e., sobre HTTPS).
- **Entrega Condicional:** Algunos encabezados podrían solo ser efectivos bajo circunstancias específicas. Verificar que estas condiciones se cumplan para que el encabezado funcione como se pretende.

### Evaluar Manejo de Etiquetas META

- **Verificaciones de Imposición Dual:** Cuando una política de seguridad como CSP se aplica a través de tanto un encabezado HTTP como una etiqueta META usando `http-equiv`, confirmar que el encabezado HTTP (que generalmente se considera más autoritativo) no sea inadvertidamente sobrescrito por la etiqueta META.
- **Revisar Comportamiento del Navegador:** Probar la aplicación en varios navegadores para ver si ocurren diferencias debido a la presencia de directivas en conflicto. Donde sea posible, evitar usar definiciones duales para prevenir lapsos de seguridad no previstos.

### Probar Stripping de Encabezados (Inyección Hop-by-Hop)

Los atacantes pueden explotar esto listando encabezados sensibles de seguridad dentro del encabezado `Connection`. El proxy, siguiendo el estándar, stripea estos encabezados antes de reenviar la solicitud al backend. Esto puede llevar a:

- Evadir Listas de Control de Acceso (ACLs) basadas en IP.
- Evadir verificaciones de Identidad/Autenticación realizadas en el edge.
- Deshabilitar características de seguridad impuestas por encabezados intermediarios.

#### Identificación de Encabezados Internos/Sensibles (Reconocimiento)

Para realizar esta prueba, primero necesitas identificar qué encabezados usa la infraestructura interna. Puedes identificarlos por:

- **Disparar Páginas de Error:** Enviar solicitudes mal formadas para disparar páginas de error (404, 500), las cuales podrían filtrar encabezados internos en la respuesta.
- **Endpoints de Reflexión:** Buscar páginas de depuración o "Echo" (por ejemplo, `/phpinfo`, `/debug`, `/env`) que muestren todos los encabezados recibidos por el backend.
- **Adivinación de Encabezados:** Objetivos comunes incluyen `X-Forwarded-For`, `X-Real-IP`, `X-Forwarded-Proto`, y `X-Authenticated-User`.

#### Ejecución de la Inyección

Intentar "stripear" un encabezado objetivo añadiéndolo como valor al encabezado `Connection`.

##### Escenario A: Evadiendo Restricciones Basadas en IP

```http
GET /admin HTTP/1.1
Host: example.com
X-Forwarded-For: 203.0.113.10
Connection: close, X-Forwarded-For
```

##### Escenario B: Stripeando Contexto de Autenticación

```http
GET /api/user/profile HTTP/1.1
Host: example.com
X-Authenticated-User: victim_user
Connection: close, X-Authenticated-User
```

#### Analizando la Respuesta

- **Vulnerable:** El comportamiento de la aplicación cambia (por ejemplo, se otorga acceso, o una IP reflejada desaparece).
- **Seguro:** El comportamiento de la aplicación permanece sin cambios, o el proxy devuelve un `400 Bad Request`.

## Remediación

- **Configuración Correcta de Encabezados:** Asegurar que los encabezados se implementen correctamente con valores apropiados y sin typos.
- **Imponer Directivas Estrictas:** Configurar encabezados con las configuraciones más seguras que aún permitan la funcionalidad requerida. Por ejemplo, evitar usar `*` en políticas CORS a menos que sea absolutamente necesario.
- **Remover Encabezados Obsoletos:** Reemplazar encabezados de seguridad heredados con equivalentes modernos y remover cualquiera que ya no sea soportado.
- **Evitar Definiciones en Conflicto:** Prevenir definiciones de encabezado duplicadas y asegurar que las etiquetas META no entren en conflicto con encabezados HTTP para políticas de seguridad.
- **Restringir Encabezado Connection:** Configurar proxies para ignorar valores proporcionados por el cliente en el encabezado `Connection` que coincidan con encabezados internos sensibles.
- **Zero Trust:** Evitar depender únicamente de encabezados hop-by-hop para decisiones críticas de seguridad.

## Herramientas

- [Mozilla Observatory](https://observatory.mozilla.org/)
- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- Herramientas de Desarrollador del Navegador (Chrome, Firefox, Edge)

## Referencias

- [Proyecto de Encabezados Seguros de OWASP](https://owasp.org/www-project-secure-headers/)
- [Mozilla Developer Network: Security Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [RFC 6797 - HTTP Strict Transport Security (HSTS)](https://datatracker.ietf.org/doc/html/rfc6797)
- [Google Web Security Guidelines](https://web.dev/security-headers/)
- [HPKP is No More](https://scotthelme.co.uk/hpkp-is-no-more/)
- [RFC 9110 - HTTP Semantics: Connection Header](https://datatracker.ietf.org/doc/html/rfc9110#section-7.6.1)
- [Abusing HTTP Hop-by-Hop Request Headers](https://nathandavison.com/blog/abusing-http-hop-by-hop-request-headers)
