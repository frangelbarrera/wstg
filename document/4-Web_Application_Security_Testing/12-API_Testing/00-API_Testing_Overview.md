# Descripción General de Pruebas de API

## Introducción a las APIs Web

Una Interfaz de Programación de Aplicaciones Web (API, por sus siglas en inglés) facilita la comunicación y el intercambio de datos entre diferentes sistemas de software a través de una red o de internet. Las APIs web permiten que diferentes aplicaciones interactúen entre sí de manera estandarizada y eficiente, lo que les permite aprovechar las funcionalidades y los datos de cada una.

La adopción de diferentes tecnologías como la computación en la nube, las arquitecturas de microservicios y las aplicaciones de página única (SPA) han contribuido a la adopción de las APIs como un movimiento arquitectónico.

Como ocurre con la introducción de cualquier concepto nuevo, pueden existir defectos y vulnerabilidades que requieren pruebas. De lo contrario, las APIs mal aseguradas pueden proporcionar una vía directa y sin restricciones hacia datos sensibles.

Este capítulo intenta guiar al investigador de seguridad en los conceptos necesarios para probar las APIs. Esta sección en particular investiga las diferentes tecnologías de API y su historia.

## ¿Qué Tecnología de API?

Antes de hacer suposiciones sobre el tipo de API que estamos probando, puede ser útil ser consciente del alcance completo del espacio del problema que el investigador de seguridad puede encontrar. Esto incluye:

1. APIs REST (Transferencia de Estado Representacional)
2. APIs SOAP (Protocolo Simple de Acceso a Objetos)
3. APIs GraphQL
4. gRPC (Llamadas a Procedimientos Remotos de gRPC)
5. APIs WebSockets

## APIs REST (Transferencia de Estado Representacional)

### ¿Qué es REST?

REST es un conjunto de reglas y convenciones para interactuar con recursos web. Los componentes clave de URI, Métodos HTTP, Encabezados y Códigos de Estado respaldan los principios de REST.

### Historia

Debido a su simplicidad, escalabilidad y compatibilidad con la infraestructura web existente, las APIs basadas en REST se han convertido en la arquitectura de API más común en internet al momento de escribir esto. Las APIs basadas en REST no se manifestaron de inmediato, sino que tienen un largo camino desde la investigación hasta la adopción.

En 1994, Roy Fielding, uno de los principales autores de la especificación HTTP, comenzó su trabajo en REST como parte de su tesis doctoral en la Universidad de California, Irvine. Para el año 2000, publicó su tesis, [Estilos Arquitectónicos y el Diseño de Arquitecturas de Software Basadas en Red](https://ics.uci.edu/~fielding/pubs/dissertation/top.htm), donde introdujo y definió REST como un estilo arquitectónico. REST fue diseñado para aprovechar las características existentes de HTTP, enfatizando la escalabilidad, las interacciones sin estado y una interfaz uniforme.

En la década de 2010, REST se convirtió en el estándar de facto para las APIs web debido a su simplicidad y compatibilidad con la arquitectura subyacente de la web. El uso generalizado de las APIs RESTful fue impulsado por el crecimiento de las aplicaciones móviles, la computación en la nube y la arquitectura de microservicios. El desarrollo de herramientas y frameworks como Swagger/OpenAPI, RAML y API Blueprint facilitó el diseño, la documentación y las pruebas de las APIs REST.

Para la década de 2020, los desarrollos modernos evolucionaron REST con tecnologías como GraphQL. Además, la especificación OpenAPI/Swagger se convirtió en un estándar ampliamente adoptado para describir APIs REST, permitiendo una mejor integración y automatización.

### Identificadores de Recursos Uniformes (URI)

Las APIs REST utilizan Identificadores de Recursos Uniformes (URIs) para acceder a los recursos. Las URIs son un elemento crucial de una arquitectura REST. Una URI es una cadena de caracteres que identifica de manera única un recurso en particular. Las URIs se utilizan extensamente en internet para localizar e interactuar con recursos, como páginas web, archivos y servicios.

Una URI consta de varios componentes, cada uno con un propósito específico. La sintaxis genérica de URI definida en [RFC3986](https://tools.ietf.org/html/rfc3986) es la siguiente:

> `URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]`

Para REST, el **esquema** (scheme) suele ser `HTTP` o `HTTPS`, pero de forma genérica indica el protocolo o método utilizado para acceder al recurso. Otros esquemas comunes incluyen `ftp`, `mailto` y `file`.

La **autoridad** (authority) especifica el nombre de dominio o la dirección IP del servidor donde reside el recurso, y puede incluir un número de puerto. También puede incluir información de usuario como subcomponente.

La **ruta** (path) especifica la ubicación específica del recurso en el servidor. Estamos interesados en la ruta de la URI debido a la relación entre el usuario y los recursos. Por ejemplo, `https://api.example.com/admin/testing/report` puede mostrar un informe de prueba. Existe una relación entre el usuario admin y sus informes.

La ruta de cualquier URI definirá un modelo de recursos de la API REST. Los recursos se separan con una barra diagonal y se basan en un diseño de arriba hacia abajo.

Por ejemplo:

- `https://api.example.com/admin/testing/report`
- `https://api.example.com/admin/testing/`
- `https://api.example.com/admin/`

La **consulta** (query) proporciona parámetros adicionales para el recurso. Comienza con un `?` y consta de pares clave-valor separados por `&`.

El **fragmento** indica una parte específica del recurso, como una sección dentro de una página web. Comienza con un `#`. Cabe señalar que los identificadores de fragmento solo se procesan del lado del cliente y no se envían al servidor.

### Métodos HTTP

Las APIs REST utilizan métodos HTTP estándar para realizar operaciones sobre los recursos siguiendo los [Métodos de Solicitud HTTP](https://tools.ietf.org/html/rfc7231#section-4) definidos en [RFC7231](https://tools.ietf.org/html/rfc7231). Estos métodos se mapean a CRUD, las cuatro funciones básicas de almacenamiento persistente en informática. CRUD significa Crear, Leer, Actualizar y Eliminar (Create, Read, Update, Delete), que son las cuatro operaciones que se pueden realizar sobre los datos.

Los métodos de solicitud HTTP son:

| Métodos | Descripción                                       |
|---------|---------------------------------------------------|
| GET     | Obtener la representación del estado del recurso  |
| POST    | Crear un nuevo recurso                            |
| PUT     | Actualizar un recurso                             |
| DELETE  | Eliminar un recurso                               |
| HEAD    | Obtener metadatos asociados con el estado del recurso |
| OPTIONS | Listar los métodos disponibles                    |

#### Encabezados

REST depende de los encabezados para soportar la comunicación de información adicional dentro de la solicitud o respuesta. Estos incluyen:

- `Content-Type`: Indica el tipo de medio del recurso (por ejemplo, `application/json`).
- `Authorization`: Contiene credenciales para autenticación (por ejemplo, tokens).
- `Accept`: Especifica los tipos de medios que son aceptables para la respuesta.

#### Códigos de Estado

Las APIs de aplicaciones que se ajustan a los principios REST utilizan el código de estado de respuesta de un mensaje de respuesta HTTP para notificar al cliente sobre el resultado de su solicitud.

| Código de Respuesta | Mensaje de Respuesta   | Descripción                                                                                           |
|---------------------|------------------------|-------------------------------------------------------------------------------------------------------|
| 200                 | OK                     | Éxito al procesar la solicitud del cliente                                                            |
| 201                 | Created                | Nuevo recurso creado                                                                                  |
| 301                 | Moved Permanently      | Redirección permanente                                                                                |
| 304                 | Not Modified           | Respuesta relacionada con el caché que se devuelve cuando el cliente tiene la misma copia del recurso que el servidor |
| 307                 | Temporary Redirect     | Redirección temporal del recurso                                                                      |
| 400                 | Bad Request            | Solicitud mal formada por el cliente                                                                  |
| 401                 | Unauthorized           | El cliente no tiene permitido hacer solicitudes o acceder a un recurso en particular                  |
| 403                 | Forbidden              | El cliente tiene prohibido acceder al recurso                                                         |
| 404                 | Not Found              | El recurso no existe o es incorrecto según la solicitud                                               |
| 405                 | Method Not Allowed     | Método inválido o método desconocido utilizado                                                        |
| 500                 | Internal Server Error  | El servidor no pudo procesar la solicitud debido a un error interno                                   |

## Referencias

1. [Hoja de Referencia de Seguridad REST de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
2. [Hoja de Referencia de Evaluación REST de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)
3. [Proyecto de Seguridad de API de OWASP](https://owasp.org/www-project-api-security/)
4. [Herramientas de Seguridad de API de OWASP](https://owasp.org/www-community/api_security_tools)
