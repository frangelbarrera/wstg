# Reconocimiento de API

|ID          |
|------------|
|WSTG-APIT-01|

## Resumen

El reconocimiento es un paso importante en cualquier engagement de pentesting. Esto incluye el pentesting de APIs. El reconocimiento mejora significativamente la efectividad del proceso de prueba al recopilar información sobre la API y desarrollar una comprensión del objetivo. Esta fase no solo aumenta la probabilidad de descubrir problemas críticos de seguridad, sino que también garantiza una evaluación exhaustiva de la postura de seguridad de las APIs.

Esta guía tiene una sección sobre [Recopilación de Información](../01-Information_Gathering/README.md) que puede aplicar al auditar APIs. Sin embargo, hay algunas diferencias. Como investigadores de seguridad, a menudo nos enfocamos en áreas específicas y buscar en esta guía las secciones que aplican puede llevar mucho tiempo. Para asegurar que el investigador tenga un solo lugar para enfocarse en las APIs, esta sección concentra los elementos que aplican a las APIs y proporciona referencias a contenido de apoyo en otras partes de la guía.

### Tipos de API

Las APIs pueden ser públicas o privadas.

#### APIs Públicas

Las APIs públicas típicamente tienen sus detalles publicados en un documento Swagger/OpenAPI. Obtener acceso a este documento es importante para entender la superficie de ataque. Igualmente importante es encontrar versiones anteriores de este documento que puedan mostrar código en desuso pero aún funcional que podría tener vulnerabilidades de seguridad.

Hay que tener en cuenta que este documento, sin importar cuán buenas sean las intenciones, puede no ser preciso y también puede no revelar la API completa.

Las APIs públicas también pueden estar documentadas en bibliotecas compartidas o directorios de APIs.

#### APIs Privadas

La visibilidad de las APIs privadas depende de quién sea el consumidor previsto. Una API puede ser privada, pero solo accesible para clientes suscritos (también conocidos como `socios`) o solo accesible para clientes internos, como otros departamentos dentro de la misma empresa. Encontrar APIs privadas utilizando técnicas de reconocimiento también es importante. Estas APIs pueden ser descubiertas utilizando varias técnicas que discutiremos a continuación.

## Objetivos de Prueba

- Encontrar todos los endpoints de API soportados por el código del servidor backend, documentados o no documentados.
- Encontrar todos los parámetros para cada endpoint soportado por el servidor backend, documentados o no documentados.
- Descubrir datos interesantes relacionados con las APIs en el HTML y JavaScript enviados a los clientes.

## Cómo Probar

### Encontrar la Documentación

Tanto en el caso público como en el privado, la documentación de la API será útil según su nivel de calidad y precisión. La documentación de APIs públicas típicamente se comparte con todo el mundo, mientras que la documentación de APIs privadas solo se comparte con el cliente previsto. Sin embargo, en ambos casos, encontrar documentación, ya sea filtrada accidentalmente o de otra manera, será útil en la investigación.

Independientemente de la visibilidad de la API, buscar documentación de la API puede encontrar documentación más antigua, no publicada aún, o filtrada accidentalmente. Esta documentación será muy útil para entender cuál es la superficie de ataque que expone la API.

### Directorios de APIs

Fuentes alternativas de documentación de APIs pueden incluir Directorios de APIs, tales como:

- GitHub en general
- [Repositorio de APIs Públicas en GitHub](https://github.com/public-apis/public-apis)
- [APIs.guru](https://apis.guru)
- [RapidAPI](https://rapidapi.com/)
- [PublicAPIs](https://publicapis.dev/) y [PublicAPIs](https://publicapis.io/)
- [Postman API Network](https://www.postman.com/explore)

### Buscar en Lugares Conocidos

Si la documentación no es fácilmente aparente, entonces se puede buscar activamente en el objetivo documentación basada en algunos nombres o rutas obvias. Estos incluyen:

- /api-docs
- /doc
- /swagger
- /swagger.json
- /openapi.json
- /.well-known/schema-discovery

### Robots.txt

`robots.txt` es un archivo de texto que los propietarios de sitios crean para instruir a los rastreadores web (como los bots de los motores de búsqueda) sobre cómo rastrear e indexar su sitio. Es parte del Protocolo de Exclusión de Robots (REP, por sus siglas en inglés), que regula cómo los bots interactúan con los sitios.

Este archivo puede proporcionar pistas adicionales sobre la estructura de rutas o los endpoints de la API.

La sección de [Recopilación de Información](../01-Information_Gathering/README.md) hace referencia a robots.txt en varios casos incluyendo WSTG-INFO-01, WSTG-INFO-03, WSTG-INFO-05 y WSTG-INFO-08.

### GitDorking

Si la aplicación utiliza GitHub, GitLab u otros repositorios públicos basados en Git, entonces también podemos buscar cualquier pista o contenido sensible (también conocido como `GitDorking`). Esta información puede incluir contraseñas, claves de API, archivos de configuración y otros datos confidenciales que los desarrolladores pueden comprometer accidental o involuntariamente en sus repositorios. Las organizaciones pueden compartir accidentalmente código sensible, código de ejemplo o de prueba que puede proporcionar pistas sobre los detalles de implementación. Las cuentas personales de GitHub de los empleados del objetivo también pueden liberar accidentalmente información que puede proporcionar pistas.

### Navegar y Rastrear la Aplicación

Incluso si se tiene la documentación de la API, navegar por la aplicación es una buena idea. La documentación puede estar desactualizada, ser inexacta o estar incompleta.

Navegar por la aplicación con un proxy de intercepción como ZAP o Burp Suite registra los endpoints para su inspección posterior. Además, utilizando su funcionalidad de rastreo integrada, los proxies de intercepción pueden ayudar a generar una lista completa de endpoints. A partir de las URLs rastreadas, buscar enlaces con esquemas de nombres obvios de URLs de API. Estos incluyen:

- `https://example.com/api/v1` (o v2, etc.)
- `https://example.com/graphql`

O subdominios que las aplicaciones puedan consumir o de los cuales dependan:

- `https://api.example.com/api/v1`

Es importante que el pentester intente ejercer tanta funcionalidad de la aplicación como sea posible. Esto no solo es para generar una lista completa de endpoints, sino también para evitar problemas con la carga diferida (lazy loading) y la división de código (code splitting). Además, el engagement de pentest debe incluir cuentas de muestra en diferentes niveles de privilegio para que el navegador y el rastreo puedan acceder y exponer endpoints para la mayor cantidad de funcionalidad posible.

Una vez completado, la información de endpoints obtenida de la navegación y el rastreo de la aplicación puede ayudar al pentester a componer documentación de la API del objetivo utilizando otras herramientas como Postman.

### Analizar Solicitudes Interceptadas

Al auditar APIs REST, se utiliza un proxy de intercepción para recopilar las solicitudes HTTP completas. Los servicios REST utilizan más que solo parámetros URL, por lo que capturar el cuerpo completo de la solicitud y los encabezados es crítico.

Analizar las solicitudes recopiladas para identificar parámetros no estándar u ocultos:

- Identificar encabezados HTTP que influyan en el comportamiento de la aplicación o la selección de recursos.
- Determinar si un segmento de URL tiene un patrón repetitivo en múltiples URLs. Los patrones que contienen fechas, números o valores de identificador pueden indicar un parámetro incrustado en la URL. Por ejemplo, en la URL `https://api.example.com/user/2026-10-03/profile`, el segmento de fecha puede representar un valor de parámetro.
- Identificar valores de parámetros estructurados formateados en JSON, XML u otras estructuras personalizadas.
- Examinar el elemento final de una URL. Si carece de extensión de archivo, puede ser un parámetro.
- Buscar segmentos de URL muy variables. Si un solo segmento cambia frecuentemente a lo largo de cientos de solicitudes, es más probable que represente un valor de parámetro que un componente de ruta estático.

### Google Dorking

Utilizar técnicas de reconocimiento pasivo como Google Dorking con directivas como `site` e `inurl` nos permite adaptar una búsqueda para palabras clave comunes de API que el indexador de Google pueda haber encontrado. Revisar [Realizar Reconocimiento mediante Motores de Búsqueda para Fugas de Información](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md) para información adicional.

Aquí hay algunos ejemplos específicos de API:

`site:"mytargetsite.com" inurl:"/api"`

`inurl:apikey filetype:env`

Otras palabras clave pueden incluir `"v1"`, `"api"`, `"graphql"`.

Podemos extender el Google Dorking para incluir subdominios del objetivo.

Las listas de palabras (wordlists) son útiles aquí para tener una lista completa de palabras comunes utilizadas en APIs.

### Mirar Atrás, Mucho Atrás

En general, las APIs cambian con el tiempo. Pero las versiones en desuso o más antiguas pueden seguir operativas, ya sea a propósito o por mala configuración. Estas también deberían probarse, ya que hay una buena posibilidad de que contengan vulnerabilidades que las versiones más nuevas han corregido. Además, los cambios en las APIs muestran características más nuevas que pueden ser menos robustas y, por lo tanto, un buen candidato para pruebas.

Para descubrir versiones más antiguas, podemos usar la `Wayback machine` para ayudar a encontrar endpoints antiguos. Una herramienta útil conocida como [WayBackUrls](https://github.com/tomnomnom/waybackurls) de TomNomNom obtiene todas las URLs que la Wayback Machine conoce para un dominio.

- [WayBackUrls](https://github.com/tomnomnom/waybackurls). Obtiene todas las URLs que la Wayback Machine conoce para un dominio.
- [waymore](https://github.com/xnl-h4ck3r/waymore). Encuentra mucho más de la Wayback Machine, Common Crawl, Alien Vault OTX, URLScan y VirusTotal.
- [gau](https://github.com/lc/gau). Obtiene URLs conocidas de AlienVault's Open Threat Exchange, la Wayback Machine y Common Crawl.

### La Aplicación del Lado del Cliente

Una excelente fuente de información de API y otra información es el HTML y JavaScript que el servidor envía al cliente. A veces, la aplicación cliente filtra información sensible incluyendo APIs y secretos. La sección [Revisar el Contenido de la Página Web para Fugas de Información](../01-Information_Gathering/05-Review_Web_Page_Content_for_Information_Leakage.md) tiene información general para revisar el contenido web en busca de fugas. Aquí expandiremos para enfocarnos en la revisión del contenido JavaScript en busca de secretos relacionados con la API.

Hay una variedad de herramientas que podemos usar para ayudarnos a extraer información sensible del JavaScript transmitido al navegador. Estas herramientas típicamente se basan en uno de dos enfoques: Expresiones Regulares o Árboles de Sintaxis Abstracta (AST, por sus siglas en inglés). Luego hay herramientas generalizadas que nos ayudan a organizar o gestionar archivos JS para su investigación por herramientas AST y de Expresiones Regulares.

Las expresiones regulares son más directas al buscar contenido JS o HTML para patrones conocidos. Sin embargo, este enfoque puede pasar por alto contenido no identificado explícitamente en la Expresión Regular. Dada la estructura de algunos JS, este enfoque puede pasar por alto mucho contenido. Los AST, por otro lado, son estructuras tipo árbol que representan la sintaxis del código fuente. Cada nodo en el árbol corresponde a una parte del código. Para JavaScript, un AST descompone el código en componentes básicos, permitiendo a las herramientas y compiladores entender y modificar el código fácilmente.

#### Herramientas Generales

1. [Uproot](https://github.com/0xDexter0us/uproot-JS). Un plugin de BurpSuite que guarda cualquier archivo JS encontrado en el disco. Esto ayuda a extraer los archivos para cualquier análisis por herramientas de línea de comandos.
2. [OpenAPI Support](https://www.zaproxy.org/docs/desktop/addons/openapi-support/). Este add-on de ZAP permite rastrear e importar definiciones OpenAPI (Swagger), versiones 1.2, 2.0 y 3.0.
3. [OpenAPI Parser](https://github.com/aress31/openapi-parser). Un plugin de BurpSuite que analiza documentos OpenAPI en Burp Suite para automatizar evaluaciones de seguridad de APIs basadas en OpenAPI.

#### Herramientas de Expresiones Regulares

1. [JSParser](https://github.com/nahamsec/JSParser). Un script de Python 2.7 que utiliza Tornado y JSBeautifier para analizar URLs relativas de archivos JavaScript.
2. [JSMiner](https://github.com/PortSwigger/js-miner). Un plugin de BurpSuite que intenta encontrar cosas interesantes dentro de archivos estáticos; principalmente archivos JavaScript y JSON. Esta herramienta escanea "pasivamente" mientras se rastrea la aplicación.
3. [JSpector](https://github.com/hisxo/JSpector). Un plugin de BurpSuite que rastrea pasivamente archivos JavaScript y crea automáticamente issues con URLs, endpoints y métodos peligrosos encontrados en los archivos JS.
4. [Link Finder](https://github.com/GerbenJavado/LinkFinder). Un script de Python que encuentra endpoints en archivos JavaScript.

#### Herramientas AST

1. [JSLuice](https://github.com/BishopFox/jsluice). Una herramienta de línea de comandos que extrae URLs, rutas, secretos y otros datos interesantes del código fuente JavaScript.

### Otras Herramientas de Reconocimiento

1. [Attack Surface Detector](https://github.com/secdec/attack-surface-detector-burp). Un plugin de BurpSuite que utiliza análisis de código estático para identificar endpoints de aplicaciones web analizando rutas e identificando parámetros.
2. [Param Miner](https://github.com/portswigger/param-miner). Un plugin de BurpSuite que identifica parámetros ocultos no enlazados.
3. [xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder). Una herramienta de Python utilizada para descubrir endpoints, parámetros potenciales y una wordlist específica del objetivo para un objetivo dado.
4. [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension). Extensión de Burp para encontrar endpoints potenciales, parámetros y generar una wordlist personalizada del objetivo.

### Fuzzing Activo

El Fuzzing Activo implica el uso de herramientas con wordlists y el filtrado de resultados de solicitudes para forzar bruscamente (bruteforce) el descubrimiento de endpoints.

#### Kiterunner

[KiteRunner](https://github.com/assetnote/kiterunner) es una herramienta que realiza descubrimiento de contenido tradicional y fuerza bruta de rutas/endpoints en aplicaciones y APIs modernas.

```console
kr [scan|brute] <input> [flags]
```

Para escanear un objetivo en busca de APIs usando una wordlist podemos:

```console
kr scan https://example.com/api -w /usr/share/wordlists/apis/routes-large.kite --fail-status-codes 404,403
```

#### FFUF/DirBuster/GoBuster

Las tres herramientas FFUF, DirBuster y GoBuster están diseñadas para descubrir rutas y archivos ocultos en servidores web mediante técnicas de fuerza bruta. Las tres utilizan wordlists personalizables para generar solicitudes al servidor web objetivo, intentando identificar directorios y archivos válidos. Las tres soportan procesamiento multi-hilo o altamente eficiente para acelerar el proceso de fuerza bruta.

Algunos archivos comunes de wordlists para APIs incluyen: [SecLists](https://github.com/danielmiessler/SecLists) en la sección Discovery/Web-Content/API, [GraphQL Wordlist](https://github.com/Escape-Technologies/graphql-wordlist) y [Assetnote](https://wordlists.assetnote.io/).

Ejemplo de GoBuster:

`gobuster dir -u <target url> -w <wordlist file>`

## Referencias

### Recursos de OWASP

- [Hoja de Referencia de Evaluación REST](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)

### Libros

- Corey J. Ball - "Hacking APIs : breaking web application programming interfaces", No Starch, 2022 - ISBN-13: 978-1-7185-0244-4
- Confidence Staveley - "API Security for White Hat Hackers, Packt, 2024 - ISBN 978-1-80056-080-2
