# Contrabando de solicitudes HTTP

|ID          |
|------------|
|WSTG-INJT-16|

## Resumen

El contrabando de solicitudes HTTP es una clase de vulnerabilidades causada por inconsistencias en la forma en que los componentes frontend y backend analizan las solicitudes HTTP. Cuando intermediarios como proxies inversos, balanceadores de carga o puertas de enlace de API interpretan de manera diferente los límites de las solicitudes que los servidores backend, los atacantes pueden inyectar o «contrabandear» solicitudes ocultas que se procesan fuera de secuencia.

Las infraestructuras modernas amplían significativamente la superficie de ataque al incorporar HTTP/2, degradaciones de protocolo (HTTP/2 → HTTP/1.1) y actualizaciones a conexiones de texto claro (H2C), donde la lógica de normalización y traducción de solicitudes suele apartarse de las expectativas de RFC.

Las vulnerabilidades de contrabando de solicitudes surgen cuando dos o más analizadores HTTP no se ponen de acuerdo sobre dónde comienza o termina una solicitud. Históricamente, esta discrepancia se observaba con mayor frecuencia en interpretaciones contradictorias de los encabezados `Content-Length` (CL) y `Transfer-Encoding` (TE).

En las arquitecturas modernas, surgen vectores adicionales de desincronización debido a:

- Capas de traducción de HTTP/2 a HTTP/1.1
- Mecanismos de actualización a HTTP/2 en texto claro (H2C)
- Desajustes en la normalización de encabezados
- Reintroducción de encabezados prohibidos durante la degradación de protocolo
- Reutilización de conexiones entre límites de protocolo

Estos comportamientos pueden provocar desincronización persistente, envenenamiento de caché, secuestro de credenciales y elusión de controles de acceso.

## Objetivos de las pruebas

- Identificar inconsistencias en los límites de las solicitudes entre los componentes frontend y backend
- Detectar vulnerabilidades clásicas de desincronización CL/TE
- Evaluar la lógica de traducción de protocolo (HTTP/2 → HTTP/1.1)
- Evaluar el manejo de las actualizaciones H2C y la seguridad de las degradaciones
- Confirmar el envenenamiento de la cola de solicitudes del backend

## Cómo realizar las pruebas

### Pruebas de caja negra

#### Desincronización CL.TE

En un escenario CL.TE, el frontend utiliza `Content-Length` para determinar el tamaño de la solicitud, mientras que el backend respeta `Transfer-Encoding`.

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Foo: x
```

Resultado esperado:

- El backend deja de analizar al llegar al fragmento `0`
- La solicitud contrabandeada permanece en el búfer
- Las solicitudes legítimas posteriores se corrompen o devuelven respuestas inesperadas (por ejemplo, 404)

#### Desincronización TE.CL

En un escenario TE.CL, el frontend procesa correctamente la codificación por fragmentos, pero el backend se basa en `Content-Length`.

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

Resultado esperado:

- El backend se detiene antes de tiempo
- La carga útil restante se interpreta como una nueva solicitud
- Puede producirse el acceso no autorizado a un endpoint o el envenenamiento de solicitudes

#### TE.TE (Transfer-Encoding ofuscado)

Si ambos servidores admiten `Transfer-Encoding`, la ofuscación del encabezado puede provocar que uno de los analizadores lo ignore.

Entre las técnicas habituales se incluyen:

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

### Vectores de ataque modernos

#### Desincronización de HTTP/2 a HTTP/1.1

En muchos despliegues, los clientes se comunican con servidores perimetrales mediante HTTP/2, mientras que los servicios backend siguen funcionando con HTTP/1.1. Durante la traducción de protocolo, los intermediarios deben reconstruir las solicitudes HTTP/1.1 a partir de tramas HTTP/2.

Entre los puntos de fallo habituales se incluyen:

- Reconstrucción incorrecta de `Content-Length`
- Reintroducción de encabezados salto a salto
- Varias solicitudes lógicas agrupadas en una sola solicitud backend

> Nota: La degradación de HTTP/2 no es vulnerable por sí sola.  
> La explotación es posible cuando la traducción de protocolo reconstruye una solicitud HTTP/1.1 que incumple las suposiciones de análisis del backend, lo que provoca la desincronización de los límites de las solicitudes.

Enfoque de prueba:

- Enviar varias tramas DATA de HTTP/2 con semánticas de longitud contradictorias
- Observar el comportamiento del backend mediante discrepancias de tiempo o división de respuestas
- Supervisar si se envenena la cola de solicitudes

##### Ejemplo: contrabando mediante degradación de HTTP/2 y reconstrucción de solicitudes

En este escenario, el cliente se comunica con el frontend mediante HTTP/2, mientras que el backend solo admite HTTP/1.1. El intermediario reconstruye una solicitud HTTP/1.1 a partir de varias tramas DATA de HTTP/2.

**HTTP/2 (representación conceptual):**

- Trama DATA 1:

```http
0\r\n\r\n
```

- Trama DATA 2:

```http
GET /admin HTTP/1.1
Host: internal
```

**Solicitud HTTP/1.1 reconstruida (vista del backend):**

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 0

GET /admin HTTP/1.1
Host: internal
```

Si el frontend considera que la solicitud está completa mientras el backend sigue analizando los datos almacenados en el búfer, la segunda solicitud podría procesarse fuera de secuencia, lo que daría lugar a un contrabando de solicitudes.

> Degradaciones implícitas:
> Incluso si no existe un mecanismo explícito `Upgrade: h2c`, muchas CDN y muchos proxies inversos degradan silenciosamente las conexiones HTTP/2 de los clientes a HTTP/1.1 al reenviar solicitudes a los servicios backend.  
> Estas degradaciones implícitas amplían la superficie de ataque de contrabando, especialmente cuando se combinan con la reutilización de conexiones y una normalización insuficiente de las solicitudes.

#### Contrabando H2C (actualización a HTTP/2 en texto claro)

H2C permite actualizar una conexión HTTP/1.1 a HTTP/2 mediante el mecanismo `Upgrade: h2c`.  
A diferencia de las degradaciones de protocolo, el contrabando H2C ocurre durante una transición de protocolo en la misma conexión, cuando los componentes frontend y backend pueden discrepar temporalmente sobre el estado de análisis activo de esa conexión y dejar bytes residuales en el búfer del backend.

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

Factores de riesgo:

- Aceptación parcial de la actualización
- El backend sigue analizando como HTTP/1.1
- La solicitud contrabandeada se procesa después de la actualización

#### Envenenamiento de la cola de solicitudes mediante degradación de protocolo

Algunos proxies degradan las solicitudes HTTP/2 a HTTP/1.1, pero no depuran por completo:

- `Content-Length`
- Encabezados duplicados
- Orden no válido de los seudoencabezados

Los atacantes pueden aprovechar esto para envenenar conexiones backend persistentes y afectar a varios usuarios.

### Indicadores de vulnerabilidad

- Respuestas inconsistentes a solicitudes idénticas
- Respuestas 404 o 400 inesperadas
- Respuestas retrasadas o que no corresponden a la solicitud
- Filtración de respuestas entre usuarios

## Mitigación

- Aplicar un análisis estricto que cumpla con RFC
- Normalizar el manejo de solicitudes en todos los intermediarios
- Deshabilitar H2C cuando no sea necesario
- Evitar las degradaciones de protocolo en conexiones que no sean de confianza
- Finalizar y volver a validar las conexiones backend cuando se produzcan errores de análisis

## Herramientas

- [HTTP Request Smuggler (extensión de Burp Suite)](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
- [Smuggler (Python), de defparam](https://github.com/defparam/smuggler)
- [h2csmuggler, de Bishop Fox](https://github.com/BishopFox/h2csmuggler)

## Referencias

- [James Kettle, «Ataques de desincronización HTTP: el renacer del contrabando de solicitudes» (PortSwigger Research)](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
- [James Kettle, «HTTP/2: la secuela siempre es peor» (PortSwigger Research)](https://portswigger.net/research/http2)
- [Jake Miller, «Contrabando H2C: contrabando de solicitudes mediante HTTP/2 en texto claro» (Bishop Fox)](https://bishopfox.com/blog/h2c-smuggling-request)
- [Amit Klein, Chaim Linhart, Ronen Heled, Steve Orrin: «Contrabando de solicitudes HTTP» (2005)](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
- [RFC 7230, sección 3.3.3: longitud del cuerpo del mensaje](https://datatracker.ietf.org/doc/html/rfc7230#section-3.3.3)
- [RFC 7540: Protocolo de transferencia de hipertexto versión 2 (HTTP/2)](https://datatracker.ietf.org/doc/html/rfc7540)