# Descripción general de las pruebas de API

## Introducción a las API web

Una interfaz de programación de aplicaciones web (API, por sus siglas en inglés) facilita la comunicación y el intercambio de datos entre diferentes sistemas de software a través de una red o de internet. Las API web permiten que diferentes aplicaciones interactúen entre sí de manera estandarizada y eficiente, lo que les permite aprovechar las funcionalidades y los datos de las demás.

La adopción de diferentes tecnologías, como la computación en la nube, las arquitecturas de microservicios y las aplicaciones de página única, ha contribuido a la adopción de las API como movimiento arquitectónico.

Como ocurre con la introducción de cualquier concepto nuevo, pueden surgir defectos y vulnerabilidades que requieren pruebas. De lo contrario, las API con una seguridad deficiente pueden proporcionar una vía directa y sin restricciones a datos sensibles.

Este capítulo busca orientar al investigador de seguridad sobre los conceptos necesarios para probar las API. Esta sección, en particular, examina las diferentes tecnologías de API y su historia.

## ¿Qué tecnología de API?

Antes de hacer suposiciones sobre el tipo de API que estamos probando, puede ser útil conocer todo el alcance del espacio del problema que el investigador de seguridad podría encontrar. Entre ellas se incluyen:

1. API de transferencia de estado representacional (REST)
2. API del protocolo simple de acceso a objetos (SOAP)
3. API de GraphQL
4. Llamadas a procedimientos remotos de gRPC (gRPC)
5. API de WebSockets

## API REST (transferencia de estado representacional)

### ¿Qué es REST?

REST es un conjunto de reglas y convenciones para interactuar con recursos web. Los componentes clave de URI, métodos HTTP, encabezados y códigos de estado respaldan los principios de REST.

### Historia
  
Debido a su simplicidad, escalabilidad y compatibilidad con la infraestructura web existente, las API basadas en REST se han convertido en la arquitectura de API más común en internet al momento de escribir este documento. Las API basadas en REST no surgieron de inmediato, sino que recorrieron un largo camino desde la investigación hasta su adopción.

En 1994, Roy Fielding, uno de los principales autores de la especificación HTTP, comenzó a trabajar en REST como parte de su tesis doctoral en la Universidad de California, Irvine. Para el año 2000, publicó su tesis, [Estilos arquitectónicos y diseño de arquitecturas de software basadas en redes](https://ics.uci.edu/~fielding/pubs/dissertation/top.htm), donde introdujo y definió REST como un estilo arquitectónico. REST se diseñó para aprovechar las características existentes de HTTP, haciendo hincapié en la escalabilidad, las interacciones sin estado y una interfaz uniforme.

En la década de 2010, REST se convirtió en el estándar de facto para las API web debido a su simplicidad y compatibilidad con la arquitectura subyacente de la web. El uso generalizado de las API RESTful fue impulsado por el crecimiento de las aplicaciones móviles, la computación en la nube y la arquitectura de microservicios. El desarrollo de herramientas y frameworks como Swagger/OpenAPI, RAML y API Blueprint facilitó el diseño, la documentación y las pruebas de las API REST.

En la década de 2020, los avances modernos hicieron evolucionar REST con tecnologías como GraphQL. Además, la especificación OpenAPI/Swagger se convirtió en un estándar ampliamente adoptado para describir API REST, lo que permitió una mejor integración y automatización.

### Identificadores uniformes de recursos

Las API REST utilizan identificadores uniformes de recursos (URI) para acceder a los recursos. Los URI son un elemento crucial de una arquitectura REST. Un URI es una cadena de caracteres que identifica de manera única un recurso determinado. Los URI se utilizan ampliamente en internet para localizar e interactuar con recursos, como páginas web, archivos y servicios.

Un URI consta de varios componentes, cada uno con un propósito específico. A continuación, se muestra la sintaxis genérica de URI definida en [RFC3986](https://tools.ietf.org/html/rfc3986):

> `URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]`

En REST, el **esquema** suele ser `HTTP` o `HTTPS`, pero, en términos generales, indica el protocolo o método utilizado para acceder al recurso. Otros esquemas comunes incluyen `ftp`, `mailto` y `file`.

La **autoridad** especifica el nombre de dominio o la dirección IP del servidor donde se encuentra el recurso y puede incluir un número de puerto. También puede incluir información de usuario como subcomponente.

La **ruta** especifica la ubicación concreta del recurso en el servidor. Nos interesa la ruta del URI por la relación entre el usuario y los recursos. Por ejemplo, `https://api.example.com/admin/testing/report` puede mostrar un informe de prueba. Existe una relación entre el usuario admin y sus informes.

La ruta de cualquier URI define un modelo de recursos de API REST. Los recursos se separan con una barra diagonal y se organizan según un diseño de arriba hacia abajo.

Por ejemplo:

- `https://api.example.com/admin/testing/report`
- `https://api.example.com/admin/testing/`
- `https://api.example.com/admin/`

La **consulta** proporciona parámetros adicionales para el recurso. Comienza con un `?` y consta de pares clave-valor separados por `&`.

El **fragmento** indica una parte específica del recurso, como una sección dentro de una página web. Comienza con un `#`. Cabe señalar que los identificadores de fragmento solo se procesan en el cliente y no se envían al servidor.

### Métodos HTTP

Las API REST utilizan métodos HTTP estándar para realizar operaciones en los recursos, de acuerdo con los [métodos de solicitud HTTP](https://tools.ietf.org/html/rfc7231#section-4) definidos en [RFC7231](https://tools.ietf.org/html/rfc7231). Estos métodos se corresponden con CRUD, las cuatro funciones básicas del almacenamiento persistente en informática. CRUD significa Crear, Leer, Actualizar y Eliminar (Create, Read, Update, Delete), las cuatro operaciones que se pueden realizar con los datos.

Los métodos de solicitud HTTP son:

| Métodos | Descripción                                      |
|---------|--------------------------------------------------|
| GET     | Obtener la representación del estado del recurso |
| POST    | Crear un recurso nuevo                           |
| PUT     | Actualizar un recurso                            |
| DELETE  | Eliminar un recurso                              |
| HEAD    | Obtener metadatos asociados al estado del recurso |
| OPTIONS | Enumerar los métodos disponibles                 |

#### Encabezados

REST depende de los encabezados para facilitar la comunicación de información adicional en la solicitud o la respuesta. Entre ellos se incluyen:

- `Content-Type`: Indica el tipo de medio del recurso (por ejemplo, `application/json`).
- `Authorization`: Contiene credenciales para la autenticación (por ejemplo, tokens).
- `Accept`: Especifica los tipos de medio aceptables para la respuesta.

#### Códigos de estado

Las API de aplicaciones que se ajustan a los principios REST utilizan el código de estado de la respuesta HTTP para notificar al cliente el resultado de su solicitud.

| Código de respuesta | Mensaje de respuesta   | Descripción                                                                                                      |
|---------------------|------------------------|------------------------------------------------------------------------------------------------------------------|
| 200                 | OK                     | La solicitud del cliente se procesó correctamente                                                               |
| 201                 | Created                | Se creó un recurso nuevo                                                                                        |
| 301                 | Moved Permanently      | Redirección permanente                                                                                          |
| 304                 | Not Modified           | Respuesta relacionada con el almacenamiento en caché que se devuelve cuando el cliente tiene la misma copia del recurso que el servidor |
| 307                 | Temporary Redirect     | Redirección temporal del recurso                                                                                |
| 400                 | Bad Request            | Solicitud mal formada por el cliente                                                                            |
| 401                 | Unauthorized           | El cliente no tiene permiso para realizar solicitudes ni acceder a un recurso determinado                       |
| 403                 | Forbidden              | El cliente tiene prohibido acceder al recurso                                                                   |
| 404                 | Not Found              | El recurso no existe o la solicitud no es correcta                                                             |
| 405                 | Method Not Allowed     | Se utilizó un método no válido o desconocido                                                                    |
| 500                 | Internal Server Error  | El servidor no pudo procesar la solicitud debido a un error interno                                             |

## Referencias

1. [Hoja de referencia de seguridad REST de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
2. [Hoja de referencia para la evaluación de REST de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)
3. [Proyecto de seguridad de API de OWASP](https://owasp.org/www-project-api-security/)
4. [Herramientas de seguridad de API de OWASP](https://owasp.org/www-community/api_security_tools)