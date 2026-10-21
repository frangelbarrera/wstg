# Reconocimiento de API

|ID          |
|------------|
|WSTG-APIT-01|

## Resumen

El reconocimiento es un paso importante en cualquier evaluación de pentesting. Esto incluye el pentesting de APIs. El reconocimiento mejora significativamente la eficacia del proceso de pruebas al recopilar información sobre la API y desarrollar una comprensión del objetivo. Esta fase no solo aumenta la probabilidad de descubrir problemas críticos de seguridad, sino que también garantiza una evaluación exhaustiva de la postura de seguridad de las APIs.

Esta guía tiene una sección sobre [Recopilación de información](../01-Information_Gathering/README.md) que puede aplicarse al auditar APIs. Sin embargo, hay algunas diferencias. Como investigadores de seguridad, a menudo nos enfocamos en áreas específicas y buscar en esta guía las secciones aplicables puede llevar mucho tiempo. Para que el investigador tenga un único lugar en el que enfocarse en las APIs, esta sección se centra en los elementos aplicables a las APIs y proporciona referencias a contenido complementario en otras partes de la guía.

### Tipos de API

Las APIs pueden ser públicas o privadas.

#### APIs públicas

Las APIs públicas suelen tener sus detalles publicados en un documento Swagger/OpenAPI. Obtener acceso a este documento es importante para comprender la superficie de ataque. Igual de importante es encontrar versiones anteriores del documento que puedan mostrar código obsoleto, pero todavía funcional, que podría tener vulnerabilidades de seguridad.

Hay que tener en cuenta que este documento, por buenas que sean las intenciones, puede no ser preciso y también puede no revelar la API completa.

Las APIs públicas también pueden estar documentadas en bibliotecas compartidas o directorios de APIs.

#### APIs privadas

La visibilidad de las APIs privadas depende de quién sea el consumidor previsto. Una API puede ser privada, pero estar accesible solo para clientes suscritos (también conocidos como `partners`) o solo para clientes internos, como otros departamentos de la misma empresa. También es importante encontrar APIs privadas mediante técnicas de reconocimiento. Estas APIs se pueden descubrir mediante varias técnicas que se analizan a continuación.

## Objetivos de prueba

- Encontrar todos los endpoints de API compatibles con el código del servidor backend, documentados o no.
- Encontrar todos los parámetros de cada endpoint compatible con el servidor backend, documentados o no.
- Descubrir datos interesantes relacionados con las APIs en el HTML y JavaScript enviados a los clientes.

## Cómo probar

### Encontrar la documentación

Tanto en el caso de las APIs públicas como en el de las privadas, la documentación será útil en función de su calidad y precisión. La documentación de las APIs públicas suele compartirse con todo el mundo, mientras que la documentación de las APIs privadas solo se comparte con el cliente previsto. Sin embargo, en ambos casos, encontrar documentación, ya sea filtrada accidentalmente o de otra manera, será útil para la investigación.

Independientemente de la visibilidad de la API, buscar su documentación puede permitir encontrar documentación antigua, aún no publicada o filtrada accidentalmente. Esta documentación será muy útil para comprender la superficie de ataque que expone la API.

En las evaluaciones con tiempo limitado, solicite al cliente los artefactos de API legibles por máquina lo antes posible para que las herramientas de prueba puedan importarlos o procesarlos directamente. Solicite definiciones de OpenAPI, especificaciones antiguas de Swagger 2.0 y colecciones de solicitudes actualizadas (por ejemplo, [Postman](https://www.postman.com/) o [Bruno](https://www.usebruno.com/)). Si se incluyen en el alcance APIs que no son REST (por ejemplo, GraphQL), solicite esquemas o esquemas exportados si la introspección está deshabilitada. También son útiles las definiciones para reproducir solicitudes (por ejemplo, [Hurl](https://hurl.dev/)). Si la API es antigua, en algunos entornos pueden existir archivos WSDL o WADL como artefactos heredados. Esto resulta especialmente útil para APIs privadas, internas o recién implementadas para las que quizá no exista documentación pública.

### Directorios de APIs

Entre las fuentes alternativas de documentación de APIs pueden encontrarse los directorios de APIs, como:

- GitHub en general
- [Repositorio de APIs públicas de GitHub](https://github.com/public-apis/public-apis)
- [APIs.guru](https://apis.guru)
- [RapidAPI](https://rapidapi.com/)
- [PublicAPIs](https://publicapis.dev/) y [PublicAPIs](https://publicapis.io/)
- [Postman API Network](https://www.postman.com/explore)

### Buscar en lugares conocidos

Si la documentación no es evidente, se puede buscar activamente en el objetivo, basándose en algunos nombres o rutas obvios. Entre ellos se incluyen:

- /api-docs
- /doc
- /swagger
- /swagger.json
- /openapi.json
- /.well-known/schema-discovery

### Robots.txt

`robots.txt` es un archivo de texto que los propietarios de sitios crean para indicar a los rastreadores web (como los bots de los motores de búsqueda) cómo rastrear e indexar su sitio. Forma parte del Protocolo de exclusión de robots (REP), que regula la forma en que los bots interactúan con los sitios.

Este archivo puede proporcionar pistas adicionales sobre la estructura de rutas o los endpoints de API.

La sección [Recopilación de información](../01-Information_Gathering/README.md) hace referencia a robots.txt en varios casos, incluidos WSTG-INFO-01, WSTG-INFO-03, WSTG-INFO-05 y WSTG-INFO-08.

### GitDorking

Si la aplicación utiliza GitHub, GitLab u otros repositorios públicos basados en Git, también podemos buscar pistas o contenido sensible (también conocido como `GitDorking`). Esta información puede incluir contraseñas, claves de API, archivos de configuración y otros datos confidenciales que los desarrolladores pueden incluir accidental o involuntariamente en sus repositorios. Las organizaciones pueden compartir accidentalmente código sensible, de ejemplo o de prueba que proporcione pistas sobre los detalles de implementación. Las cuentas personales de GitHub de los empleados del objetivo también pueden divulgar accidentalmente información que proporcione pistas.

### Navegar y rastrear la aplicación

Aunque se disponga de la documentación de la API, es buena idea navegar por la aplicación. La documentación puede estar desactualizada, ser inexacta o estar incompleta.

Navegar por la aplicación con un proxy de interceptación, como ZAP o Burp Suite, registra endpoints para inspeccionarlos posteriormente. Además, la funcionalidad de rastreo integrada en estos proxies puede ayudar a generar una lista exhaustiva de endpoints. En las URLs rastreadas, busque enlaces con esquemas de nombres de URL de API evidentes. Entre ellos se incluyen:

- `https://example.com/api/v1` (o v2, etc.)  
- `https://example.com/graphql`

O subdominios que las aplicaciones puedan consumir o de los que puedan depender:

- `https://api.example.com/api/v1`

Es importante que el pentester intente utilizar la mayor cantidad posible de funciones de la aplicación. Esto no solo permite generar una lista exhaustiva de endpoints, sino también evitar problemas con la carga diferida (lazy loading) y la división de código (code splitting). Además, la evaluación de pentesting debe incluir cuentas de muestra con distintos niveles de privilegios para que el navegador y el rastreador puedan acceder a los endpoints y descubrir la mayor cantidad posible de funciones.

Una vez completado este proceso, la información de endpoints obtenida al navegar y rastrear la aplicación puede ayudar al pentester a crear documentación y colecciones de API con herramientas como [Postman](https://www.postman.com/) y [Bruno](https://www.usebruno.com/). Las herramientas para reproducir solicitudes, como [Hurl](https://hurl.dev/), pueden usar esa información para automatizar flujos de prueba repetibles. Algunos clientes, como Postman y Bruno, también pueden usar scripts previos y posteriores a las solicitudes, que pueden ayudar a reproducir flujos realistas al probar APIs con estado.

### Analizar solicitudes interceptadas

Al auditar APIs REST, utilice un proxy de interceptación para recopilar solicitudes HTTP completas. Los servicios REST utilizan más que solo parámetros de URL, por lo que es fundamental capturar el cuerpo completo de la solicitud y los encabezados.

Analice las solicitudes recopiladas para identificar parámetros no estándar u ocultos:

- Identifique los encabezados HTTP que influyen en el comportamiento de la aplicación o en la selección de recursos.
- Determine si un segmento de URL presenta un patrón repetitivo en varias URLs. Los patrones que contienen fechas, números o valores de identificador pueden indicar un parámetro integrado en la URL. Por ejemplo, en la URL `https://api.example.com/user/2026-10-03/profile`, el segmento de fecha puede representar un valor de parámetro.
- Identifique valores de parámetros estructurados con formato JSON, XML u otras estructuras personalizadas.
- Examine el elemento final de una URL. Si no tiene extensión de archivo, puede ser un parámetro.
- Busque segmentos de URL muy variables. Si un único segmento cambia con frecuencia en cientos de solicitudes, es más probable que represente un valor de parámetro que un componente estático de la ruta.
- Verifique los segmentos de URL variables sospechosos asignándoles un valor claramente no válido. Esto se aplica tanto a los segmentos intermedios que parecen dinámicos como a los segmentos terminales sin extensión de archivo. Un `404` generado en el nivel de enrutamiento antes del procesamiento de la aplicación puede indicar un segmento de ruta estático. Las respuestas `404` generadas por la aplicación no bastan por sí solas para establecer esa distinción. Si la solicitud llega a la lógica de la aplicación y devuelve un error de recurso no encontrado o de validación, es posible que el segmento esté enrutado por la aplicación y parametrizado. Considere esto una heurística y correlacione el origen de la respuesta con el comportamiento de enrutamiento observado.

### Google Dorking

El uso de técnicas de reconocimiento pasivo, como Google Dorking con directivas como `site` e `inurl`, permite adaptar una búsqueda a palabras clave de API comunes que el indexador de Google podría haber encontrado. Consulte [Realizar reconocimiento mediante motores de búsqueda para detectar fugas de información](../01-Information_Gathering/01-Conduct_Search_Engine_Reconnaissance_for_Information_Leakage.md) para obtener más información.

A continuación se muestran algunos ejemplos específicos de API:

`site:"mytargetsite.com" inurl:"/api"`

`inurl:apikey filetype:env`

Otras palabras clave pueden incluir `"v1"`, `"api"`, `"graphql"`.

Podemos ampliar el Google Dorking para incluir subdominios del objetivo.

Las listas de palabras son útiles para obtener una lista exhaustiva de palabras comunes utilizadas en las APIs.

### Mirar atrás, mucho más atrás

En general, las APIs cambian con el tiempo. Sin embargo, las versiones obsoletas o antiguas pueden seguir operativas, ya sea intencionadamente o por una configuración incorrecta. También deben probarse, ya que es muy probable que contengan vulnerabilidades que las versiones más recientes hayan corregido. Además, los cambios en las APIs revelan funciones nuevas que pueden ser menos robustas y, por tanto, buenas candidatas para las pruebas.

Para descubrir versiones antiguas, podemos usar la `Wayback machine` para encontrar endpoints antiguos. Una herramienta útil llamada [WayBackUrls](https://github.com/tomnomnom/waybackurls), de TomNomNom, obtiene todas las URLs que la Wayback Machine conoce para un dominio.

- [WayBackUrls](https://github.com/tomnomnom/waybackurls). Obtiene todas las URLs que la Wayback Machine conoce para un dominio.
- [waymore](https://github.com/xnl-h4ck3r/waymore). Encuentra mucho más contenido en Wayback Machine, Common Crawl, Alien Vault OTX, URLScan y VirusTotal.
- [gau](https://github.com/lc/gau). Obtiene URLs conocidas de AlienVault's Open Threat Exchange, la Wayback Machine y Common Crawl.

### La aplicación del lado del cliente

El HTML y JavaScript que el servidor envía al cliente son una excelente fuente de información sobre APIs y otros datos. A veces, la aplicación cliente filtra información sensible, incluidas APIs y secretos. La sección [Revisar el contenido de la página web para detectar fugas de información](../01-Information_Gathering/05-Review_Web_Page_Content_for_Information_Leakage.md) contiene información general sobre cómo revisar el contenido web para detectar fugas. Aquí ampliaremos el tema para centrarnos en revisar el contenido JavaScript en busca de secretos relacionados con las APIs.

Hay diversas herramientas que pueden ayudar a extraer información sensible del JavaScript transmitido al navegador. Estas herramientas suelen basarse en uno de dos enfoques: expresiones regulares o árboles de sintaxis abstracta (AST). También hay herramientas de propósito general que ayudan a organizar o gestionar archivos JS para su análisis con herramientas basadas en AST y expresiones regulares.

Las expresiones regulares ofrecen un método más sencillo para buscar patrones conocidos en contenido JS o HTML. Sin embargo, este enfoque puede pasar por alto contenido que no esté identificado explícitamente en la expresión regular. Dada la estructura de algunos archivos JS, este enfoque puede pasar por alto muchas cosas. Por otro lado, los AST son estructuras en forma de árbol que representan la sintaxis del código fuente. Cada nodo del árbol corresponde a una parte del código. En JavaScript, un AST descompone el código en componentes básicos, lo que permite a las herramientas y los compiladores comprenderlo y modificarlo fácilmente.

#### Herramientas generales

1. [Uproot](https://github.com/0xDexter0us/uproot-JS). Un plugin de BurpSuite que guarda en disco cualquier archivo JS que encuentre. Esto ayuda a extraer los archivos para analizarlos con herramientas de línea de comandos.
2. [OpenAPI Support](https://www.zaproxy.org/docs/desktop/addons/openapi-support/). Este complemento de ZAP permite rastrear e importar definiciones de OpenAPI (Swagger), versiones 1.2, 2.0 y 3.0.
3. [OpenAPI Parser](https://github.com/aress31/openapi-parser). Un plugin de BurpSuite que analiza documentos OpenAPI y los incorpora a Burp Suite para automatizar evaluaciones de seguridad de APIs basadas en OpenAPI.

#### Herramientas de expresiones regulares

1. [JSParser](https://github.com/nahamsec/JSParser). Un script de Python 2.7 que utiliza Tornado y JSBeautifier para analizar URLs relativas de archivos JavaScript.
2. [JSMiner](https://github.com/PortSwigger/js-miner). Un plugin de BurpSuite que intenta encontrar elementos interesantes en archivos estáticos, principalmente archivos JavaScript y JSON. Esta herramienta analiza "pasivamente" mientras se rastrea la aplicación.
3. [JSpector](https://github.com/hisxo/JSpector). Un plugin de BurpSuite que rastrea pasivamente archivos JavaScript y crea automáticamente incidencias con las URLs, los endpoints y los métodos peligrosos que encuentra en los archivos JS.
4. [Link Finder](https://github.com/GerbenJavado/LinkFinder). Un script de Python que encuentra endpoints en archivos JavaScript.

#### Herramientas AST

1. [JSLuice](https://github.com/BishopFox/jsluice). Una herramienta de línea de comandos que extrae URLs, rutas, secretos y otros datos interesantes del código fuente JavaScript.

### Otras herramientas de reconocimiento

1. [Attack Surface Detector](https://github.com/secdec/attack-surface-detector-burp). Un plugin de BurpSuite que utiliza análisis estático de código para identificar endpoints de aplicaciones web mediante el análisis de rutas y la identificación de parámetros.
2. [Param Miner](https://github.com/portswigger/param-miner). Un plugin de BurpSuite que identifica parámetros ocultos y no enlazados.
3. [xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder). Una herramienta de Python para descubrir endpoints, posibles parámetros y una lista de palabras específica para un objetivo determinado.
4. [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension). Extensión de Burp para encontrar posibles endpoints y parámetros, y generar una lista de palabras personalizada para el objetivo.

### Código fuente del lado del servidor

Si tiene acceso al código fuente del lado del servidor, extraiga los endpoints de este. [OWASP Noir](https://owasp.org/projects/noir) analiza el código de enrutamiento de muchos frameworks web y enumera todos los endpoints, con sus métodos y parámetros, incluidas las APIs internas u obsoletas que ningún cliente actual utiliza. Consulte [Identificar los puntos de entrada de la aplicación](../01-Information_Gathering/06-Identify_Application_Entry_Points.md#owasp-noir) para ver una guía paso a paso de la herramienta.

Para probar APIs, exporte los resultados como OpenAPI. Este comando escribe un documento OpenAPI 3.0 para cada endpoint del código fuente de [OWASP RailsGoat](https://github.com/OWASP/railsgoat):

```bash
noir scan railsgoat -f oas3 -o railsgoat-openapi.json
```

El archivo generado comienza así:

```json
{
  "openapi": "3.0.3",
  "info": {
    "title": "Generated by Noir",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "http://localhost"
    }
  ],
  "paths": {
    "/api/v1/mobile/1": {
      "get": {
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object"
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "class",
            "in": "query",
            "required": false,
            "schema": {
              "type": "string"
            }
          }
        ],
        "x-noir-unmapped-path-params": [
          "id"
        ]
      }
    },
```

Importe el documento con el complemento [OpenAPI Support](https://www.zaproxy.org/docs/desktop/addons/openapi-support/) de ZAP o la extensión OpenAPI Parser de Burp Suite (consulte [Herramientas generales](#general-tools)). El proxy añade todos los endpoints a su árbol de sitios, para que pueda ejecutar las pruebas de esta sección con ellos. Noir reemplaza cada parámetro de ruta por un valor de muestra, de modo que una ruta con un marcador de posición `:id` aparece como `/api/v1/mobile/1`, y conserva los nombres originales en `x-noir-unmapped-path-params`. Utilícelos al probar [API Broken Object Level Authorization](02-API_Broken_Object_Level_Authorization.md).

### Escaneo de puertos

Los servidores de API suelen implementarse en puertos no estándar o junto con otros servicios expuestos (paneles de administración internos, escuchas de gRPC o WebSocket, endpoints de depuración o métricas) que no se detectarían con una revisión limitada a la documentación. [naabu](https://github.com/projectdiscovery/naabu) es un escáner de puertos rápido que puede enumerar rápidamente los puertos abiertos de los hosts identificados y enviar directamente sus resultados a Nmap para detectar servicios solo en los puertos que haya encontrado abiertos:

```bash
naabu -p - -rate 2000 -c 50 -retries 2 -warm-up-time 1 -silent -host api.example.com -nmap-cli 'nmap -sV -oX scan.xml'
```

Ejecutar primero este análisis y después el análisis más lento de detección completa de servicios de Nmap solo en los puertos que naabu informa como abiertos reduce considerablemente el tiempo total del escaneo, en comparación con ejecutar directamente un escaneo de Nmap de todo el rango. Consulte [Mejorar los escaneos de puertos en servidores de API](https://danaepp.com/improving-port-scans-against-api-servers) para obtener más recomendaciones de ajuste específicas para objetivos de API, incluido por qué los escaneos CONNECT pueden superar a los escaneos SYN con alta concurrencia y cómo evitar pasar por alto servicios debido a límites de velocidad o reglas de firewall.

Consulte también [Identificación de la superficie de ataque](../01-Information_Gathering/04-Attack_Surface_Identification.md) para obtener más información sobre la detección de puertos no estándar.

### Fuzzing activo

El fuzzing activo consiste en utilizar herramientas con listas de palabras y filtrar los resultados de las solicitudes para descubrir endpoints mediante fuerza bruta.

Después de descubrir endpoints y parámetros, utilice los valores de solicitud recopilados para optimizar el fuzzing de valores:

- Compare los valores observados para identificar los rangos y formatos que probablemente sean válidos o no válidos.
- Empiece con valores no válidos marginales (por ejemplo, `0` en campos que siempre contienen enteros positivos).
- Busque secuencias para poder probar valores fuera del rango asignado actualmente al usuario autenticado.

#### Kiterunner

[KiteRunner](https://github.com/assetnote/kiterunner) es una herramienta que realiza descubrimiento de contenido tradicional y fuerza bruta de rutas/endpoints en aplicaciones y APIs modernas.

```console
kr [scan|brute] <input> [flags]
```

Para analizar un objetivo en busca de APIs mediante una lista de palabras, podemos ejecutar:

```console
kr scan https://example.com/api -w /usr/share/wordlists/apis/routes-large.kite --fail-status-codes 404,403
```

#### FFUF/DirBuster/GoBuster

FFUF, DirBuster y GoBuster están diseñadas para descubrir rutas y archivos ocultos en servidores web mediante técnicas de fuerza bruta. Las tres utilizan listas de palabras personalizables para generar solicitudes al servidor web objetivo e intentar identificar directorios y archivos válidos. Las tres admiten procesamiento multihilo o altamente eficiente para acelerar el proceso de fuerza bruta.

Algunos archivos de listas de palabras comunes para APIs son: [SecLists](https://github.com/danielmiessler/SecLists), en la sección Discovery/Web-Content/API; [GraphQL Wordlist](https://github.com/Escape-Technologies/graphql-wordlist); y [Assetnote](https://wordlists.assetnote.io/).

Ejemplo de GoBuster:

`gobuster dir -u <target url> -w <wordlist file>`

#### Fuzzing autenticado

Al realizar fuzzing en APIs autenticadas, reproduzca la misma autenticación y gestión de sesión que espera el objetivo. Incluya tokens personalizados, encabezados de autorización no estándar, metadatos de solicitud firmados o cookies, según sea necesario; de lo contrario, las herramientas pueden producir falsos negativos al no llegar nunca a la lógica de la aplicación. Si el mecanismo utiliza campos dinámicos (por ejemplo, marcas de tiempo, nonces o firmas vinculadas al cuerpo), vuelva a calcular esos valores para cada solicitud modificada, normalmente mediante scripts previos a la solicitud o interceptores de proxy/solicitudes.

## Referencias

### Recursos de OWASP

- [Hoja de referencia para la evaluación de REST](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)

### Libros

- Corey J. Ball - "Hacking APIs : breaking web application programming interfaces", No Starch, 2022 - ISBN-13: 978-1-7185-0244-4
- Confidence Staveley - "API Security for White Hat Hackers, Packt, 2024 - ISBN 978-1-80056-080-2