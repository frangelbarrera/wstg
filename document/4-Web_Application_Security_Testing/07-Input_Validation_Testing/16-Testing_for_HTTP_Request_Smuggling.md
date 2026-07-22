# Pruebas de HTTP Request Smuggling

|ID          |
|------------|
|WSTG-INPV-16|

## Resumen

HTTP Request Smuggling es una clase de vulnerabilidades causada por inconsistencias en cómo se analizan las solicitudes HTTP por componentes frontend y backend. Cuando intermediarios tales como reverse proxies, load balancers, o API gateways interpretan los límites de solicitud de manera diferente a los servidores backend, los atacantes podrían inyectar o "smugglear" solicitudes ocultas que se procesan fuera de secuencia.

Las infraestructuras modernas expanden significativamente la superficie de ataque introduciendo HTTP/2, downgrades de protocolo (HTTP/2 → HTTP/1.1), y upgrades en texto claro (H2C), donde la lógica de normalización y traducción de solicitudes frecuentemente diverge de las expectativas RFC.

Las explotaciones de request smuggling surgen cuando dos o más parsers HTTP no se ponen de acuerdo sobre dónde comienza o termina una solicitud. Históricamente, esta discrepancia se observó más comúnmente en interpretaciones en conflicto de los encabezados `Content-Length` (CL) y `Transfer-Encoding` (TE).

En arquitecturas modernas, vectores adicionales de desincronización emergen de:

- Capas de traducción HTTP/2 a HTTP/1.1
- Mecanismos de upgrade Cleartext HTTP/2 (H2C)
- Desajustes de normalización de encabezados
- Reintroducción de encabezados prohibidos durante downgrade de protocolo
- Reuso de conexión a través de límites de protocolo

Estos comportamientos pueden llevar a desincronización persistente, cache poisoning, hijacking de credenciales, y evasión de control de acceso.

## Objetivos de Prueba

- Identificar inconsistencias de límite de solicitud entre componentes frontend y backend
- Detectar vulnerabilidades clásicas de desincronización CL/TE
- Evaluar lógica de traducción de protocolo (HTTP/2 → HTTP/1.1)
- Evaluar manejo de upgrade H2C y seguridad de downgrade
- Confirmar envenenamiento de cola de solicitudes del backend

## Cómo Probar

### Pruebas de Caja Negra

#### Probar Desincronización CL.TE

En un escenario CL.TE, el frontend usa `Content-Length` para determinar el tamaño de la solicitud, mientras el backend honora `Transfer-Encoding`.

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x
```

Resultado Esperado:

- El backend deja de analizar en el chunk `0`
- La solicitud smuggleada permanece bufferizada
- Las solicitudes legítimas subsiguientes se corrompen o devuelven respuestas inesperadas (por ejemplo, 404)

#### Probar Desincronización TE.CL

En un escenario TE.CL, el frontend procesa chunked encoding correctamente, pero el backend depende de `Content-Length`.

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GET /admin HTTP/1.1
Content-Length: 0

0
```

Resultado Esperado:

- El backend se detiene temprano
- El payload restante se interpreta como una nueva solicitud
- Podría ocurrir acceso no autorizado a endpoint o envenenamiento de solicitud

#### Probar TE.TE (Transfer-Encoding Ofuscado)

Si ambos servidores soportan `Transfer-Encoding`, la ofuscación de encabezado podría causar que un parser lo ignore.

Las técnicas comunes incluyen:

- Manipulación de espacios en blanco
- Duplicación de encabezados
- Separadores no estándar

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 44
Transfer-Encoding:\tchunked
Transfer-Encoding: identity

0

GET /404 HTTP/1.1
Foo: bar
```

### Vectores de Ataque Modernos

#### Desincronización HTTP/2 a HTTP/1.1

En muchos despliegues, los clientes se comunican con servidores edge usando HTTP/2, mientras los servicios backend aún operan sobre HTTP/1.1. Durante la traducción de protocolo, los intermediarios deben reconstruir solicitudes HTTP/1.1 desde frames HTTP/2.

Los puntos de falla comunes incluyen:

- Reconstrucción incorrecta de `Content-Length`
- Reintroducción de encabezados hop-by-hop
- Múltiples solicitudes lógicas colapsadas en una sola solicitud backend

> Nota: El downgrade HTTP/2 no es inherentemente vulnerable por sí mismo.
> La explotación se vuelve posible cuando la traducción de protocolo reconstruye una solicitud HTTP/1.1 que viola las suposiciones de análisis del backend, llevando a desincronización de límite de solicitud.

Enfoque de Prueba:

- Enviar múltiples frames DATA HTTP/2 con semántica de longitud en conflicto
- Observar el comportamiento del backend vía discrepancias de tiempo o response splitting
- Monitorear envenenamiento de cola de solicitudes

##### Ejemplo: Smuggling HTTP/2 Downgrade vía Reconstrucción de Solicitud

En este escenario, el cliente se comunica con el frontend sobre HTTP/2, mientras el backend solo soporta HTTP/1.1. El intermediario reconstruye una solicitud HTTP/1.1 desde múltiples frames DATA HTTP/2.

**HTTP/2 (Representación Conceptual):**

- Frame DATA 1:

```http
0\r\n\r\n
```

- Frame DATA 2:

```http
GET /admin HTTP/1.1
Host: internal
```

**Solicitud HTTP/1.1 Reconstruida (Vista del Backend):**

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 0

GET /admin HTTP/1.1
Host: internal
```

Si el frontend trata la solicitud como completa mientras el backend continúa analizando datos bufferizados, la segunda solicitud podría procesarse fuera de secuencia, resultando en request smuggling.

> Downgrades Implícitos:
> Incluso en ausencia de un mecanismo explícito `Upgrade: h2c`, muchos CDNs y reverse proxies silenciosamente hacen downgrade de conexiones de cliente HTTP/2 a HTTP/1.1 al reenviar solicitudes a servicios backend.
> Estos downgrades implícitos expanden la superficie de ataque de smuggling, especialmente cuando se combinan con reuso de conexión y normalización insuficiente de solicitudes.

#### Smuggling H2C (Upgrade Cleartext HTTP/2)

H2C permite upgrade de una conexión HTTP/1.1 a HTTP/2 usando el mecanismo `Upgrade: h2c`.
A diferencia de los downgrades de protocolo, el smuggling H2C ocurre durante una transición de protocolo in-place, donde los componentes frontend y backend podrían temporalmente no estar de acuerdo sobre el estado activo de análisis de la misma conexión, potencialmente dejando bytes residuales en el buffer del backend.

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c
HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA

0

GET /admin HTTP/1.1
Host: internal
```

Factores de Riesgo:

- Aceptación parcial de upgrade
- El backend continúa analizando como HTTP/1.1
- La solicitud smuggleada se procesa post-upgrade

#### Envenenamiento de Cola de Solicitudes vía Downgrade de Protocolo

Algunos proxies hacen downgrade de solicitudes HTTP/2 a HTTP/1.1 pero fallan en sanitizar completamente:

- `Content-Length`
- Encabezados duplicados
- Ordenamiento inválido de pseudo-encabezados

Los atacantes pueden explotar esto para envenenar conexiones backend persistentes, impactando a múltiples usuarios.

### Indicadores de Vulnerabilidad

- Respuestas inconsistentes a través de solicitudes idénticas
- Respuestas 404 o 400 inesperadas
- Respuestas demoradas o desajustadas
- Fuga de respuesta cross-user

## Remediación

- Imponer análisis estricto que cumpla con RFC
- Normalizar manejo de solicitudes a través de todos los intermediarios
- Deshabilitar H2C donde no se requiera
- Evitar downgrades de protocolo en conexiones no confiables
- Terminar y revalidar conexiones backend ante errores de análisis

## Herramientas

- [HTTP Request Smuggler (Extensión de Burp Suite)](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
- [Smuggler (Python) por defparam](https://github.com/defparam/smuggler)
- [h2csmuggler por Bishop Fox](https://github.com/BishopFox/h2csmuggler)

## Referencias

- [James Kettle, "HTTP Desync Attacks: Request Smuggling Reborn" (PortSwigger Research)](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
- [James Kettle, "HTTP/2: The Sequel is Always Worse" (PortSwigger Research)](https://portswigger.net/research/http2)
- [Jake Miller, "h2c Smuggling: Request Smuggling Via HTTP/2 Cleartext" (Bishop Fox)](https://bishopfox.com/blog/h2c-smuggling-request)
- [Amit Klein, Chaim Linhart, Ronen Heled, Steve Orrin: "HTTP Request Smuggling" (2005)](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
- [RFC 7230, Section 3.3.3: Message Body Length](https://datatracker.ietf.org/doc/html/rfc7230#section-3.3.3)
- [RFC 7540: Hypertext Transfer Protocol Version 2 (HTTP/2)](https://datatracker.ietf.org/doc/html/rfc7540)
