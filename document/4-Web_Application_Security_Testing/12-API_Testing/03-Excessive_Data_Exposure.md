# Exposición excesiva de datos

|ID          |
|------------|
|WSTG-APIT-03|

## Resumen

La exposición excesiva de datos ocurre cuando una API devuelve en sus respuestas más información de la que el cliente necesita para mostrar o hacer funcionar la aplicación. Esto suele suceder cuando los desarrolladores implementan endpoints de API serializando objetos backend completos (como modelos de bases de datos o estructuras de datos internas) directamente en las respuestas de la API y confían en que el código del lado del cliente filtre los campos sensibles antes de presentarlos al usuario.

El problema es que cualquier persona que pueda realizar solicitudes puede acceder directamente a las respuestas de la API, independientemente de lo que la interfaz de usuario decida mostrar. Un atacante que inspeccione el tráfico sin procesar de la API puede observar todos los campos devueltos, incluidos aquellos que la interfaz oculta intencionalmente. Esto puede exponer contraseñas, tokens de autenticación, identificadores internos, información de identificación personal (PII), datos financieros, detalles de infraestructura o lógica de negocio interna.

Esta vulnerabilidad se corresponde con [OWASP API Security Top 10 API3:2023 Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/).

## Objetivos de las pruebas

- Identificar respuestas de API que contengan más datos de los que la aplicación cliente muestra o necesita.
- Detectar campos sensibles devueltos en respuestas de API, como hashes de contraseñas, tokens de autenticación, ID internos de objetos, PII o detalles de infraestructura.
- Determinar si la API depende del filtrado del lado del cliente, en lugar de seleccionar los campos del lado del servidor, para controlar la exposición de datos.

## Cómo probar

### Comparar las respuestas de la API con los datos mostrados

La prueba más directa consiste en comparar lo que muestra la interfaz de usuario con lo que devuelve realmente la API.

1. Abra la aplicación en un navegador y vaya a una página que muestre datos de usuario u objetos (por ejemplo, una página de perfil, un resumen de pedidos o un panel).
2. Abra las herramientas de desarrollo del navegador (pestaña Network) o configure un proxy de interceptación, como Burp Suite o ZAP.
3. Identifique las solicitudes de API que cargan el contenido de la página.
4. Compare los campos que se muestran en la interfaz con el conjunto completo de campos de la respuesta de la API.

Por ejemplo, una página de perfil de usuario podría mostrar solo un nombre y un correo electrónico, pero la respuesta subyacente de la API podría devolver:

```json
{
  "id": 12045,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "password_hash": "$2b$12$LJ3m4ys5qGn...",
  "role": "admin",
  "internal_account_id": "ACC-2024-88421",
  "ssn": "123-45-6789",
  "api_key": "REDACTED_EXAMPLE_KEY_12345",
  "created_by": "system_migration_v2",
  "last_login_ip": "10.0.3.47"
}
```

En este caso, la API devuelve hashes de contraseñas, números de la seguridad social, claves de API, identificadores internos y direcciones IP internas que la interfaz nunca muestra.

### Inspeccionar respuestas de distintos endpoints de API

No limite las pruebas a un solo endpoint. Compruebe las respuestas de varios tipos de endpoints:

- **Endpoints de usuario y cuenta:** `GET /api/users/{id}`, `GET /api/me`, `GET /api/account`
- **Endpoints de listas y búsquedas:** `GET /api/users`, `GET /api/orders?status=completed` (los endpoints de listas suelen devolver el mismo objeto completo para cada elemento de la lista, lo que amplifica la exposición)
- **Endpoints de objetos relacionados:** `GET /api/orders/{id}` puede incluir objetos de usuario completos, detalles de pago o direcciones de envío en la respuesta del pedido
- **Respuestas de error:** Las solicitudes fallidas pueden devolver mensajes de error detallados que contienen trazas de pila, rutas de archivos internas, detalles de consultas a bases de datos o valores de configuración
- **Endpoints administrativos o internos:** Compruebe si los endpoints destinados al uso interno son accesibles desde el exterior y devuelven datos con privilegios elevados

### Probar con distintos niveles de privilegio de usuario

El mismo endpoint puede devolver distintas cantidades de datos según el rol del usuario que realiza la solicitud. Compruebe si:

- Un usuario con pocos privilegios recibe los mismos campos que un administrador al solicitar el mismo recurso.
- Una solicitud no autenticada devuelve datos para los que debería requerirse autenticación.
- Solicitar los datos de otro usuario (IDOR) devuelve el mismo objeto completo que solicitar los propios.

Por ejemplo, `GET /api/users/124` podría devolver el objeto completo, incluidos los campos sensibles, independientemente de si quien realiza la solicitud es el usuario 124, otro usuario o alguien no autenticado.

### Analizar objetos anidados e incrustados

Las API modernas suelen incluir objetos relacionados en las respuestas. Una sola llamada a la API puede devolver estructuras profundamente anidadas que exponen datos de varios sistemas backend.

Ejemplo: un endpoint de pedidos que devuelve un objeto de cliente, uno de pago y uno de envío incrustados:

```json
{
  "order_id": "ORD-9923",
  "items": [...],
  "customer": {
    "id": 5001,
    "name": "Bob Smith",
    "email": "bob@example.com",
    "phone": "+1-555-0199",
    "date_of_birth": "1985-03-14",
    "loyalty_tier": "gold"
  },
  "payment": {
    "method": "card",
    "card_last_four": "4242",
    "card_brand": "Visa",
    "card_fingerprint": "fp_abc123xyz",
    "billing_address": {...}
  },
  "internal_notes": "Customer flagged for manual review - suspected fraud"
}
```

Se debe revisar cada objeto anidado para identificar campos que excedan lo que necesita el cliente que los consume.

### Probar la asignación masiva

En los endpoints que aceptan un cuerpo JSON o XML, pruebe si existen vulnerabilidades de asignación masiva añadiendo al cuerpo de la solicitud campos adicionales a los que muestra la interfaz de la aplicación. Los campos como los identificadores de rol, los indicadores de privilegios o los atributos del estado de la cuenta son objetivos de alto valor.

### Comprobar si hay datos sensibles en las respuestas de GraphQL

Las API GraphQL presentan una forma específica de este problema. Aunque GraphQL permite a los clientes seleccionar los campos que desean, el esquema puede exponer campos sensibles que las consultas predeterminadas de la interfaz no solicitan, pero que un atacante puede añadir a una consulta personalizada.

1. Si la introspección está habilitada, recupere el esquema y revise los campos disponibles en cada tipo (consulte [Pruebas de GraphQL](99-GraphQL.md)):

    ```graphql
    {
      __type(name: "User") {
        fields {
          name
          type { name }
        }
      }
    }
    ```

2. Busque campos como `passwordHash`, `apiKey`, `ssn`, `internalId`, `role` o `isAdmin` que se puedan consultar, pero que no use el frontend.
3. Intente consultar estos campos directamente:

    ```graphql
    {
      user(id: "123") {
        name
        email
        passwordHash
        apiKey
        role
      }
    }
    ```

### Examinar los mapas de origen de JavaScript y los bundles del lado del cliente

Los bundles de JavaScript del frontend y los mapas de origen pueden revelar inadvertidamente la estructura de respuesta de la API prevista, incluidos los campos que la interfaz oculta intencionalmente. Si los mapas de origen son accesibles públicamente (por ejemplo, en `main.js.map`), pueden exponer:

- La lista completa de campos que el frontend espera recibir en las respuestas de la API
- Rutas de endpoints internos de la API
- La lógica del flujo de autenticación y autorización
- Claves de API, tokens o valores de configuración codificados de forma fija

Compruebe si hay mapas de origen disponibles:

```http
GET /static/js/main.js.map
GET /assets/app.js.map
```

Si se puede acceder al mapa de origen y contiene `sourcesContent`, es posible reconstruir el código fuente original y analizarlo para identificar las estructuras de respuesta de la API previstas y los campos ocultos.

### Comprobar las respuestas de error y la salida de depuración

Las API pueden filtrar información sensible mediante las respuestas de error, sobre todo cuando se ejecutan en modo de desarrollo o depuración:

- **Trazas de pila** que revelan rutas de archivos internas, versiones de frameworks y la estructura del código
- **Mensajes de error de bases de datos** que exponen nombres de tablas, nombres de columnas o la sintaxis de las consultas
- **Errores de validación detallados** que enumeran todos los campos esperados, incluidos los que no están documentados en la API pública
- **Encabezados de depuración** como `X-Debug-Token`, `X-Powered-By` o encabezados personalizados que contienen información interna

Provoque respuestas de error enviando datos malformados, usando una autenticación no válida o solicitando recursos inexistentes, e inspeccione la respuesta completa, incluidos los encabezados.

## Mitigación

- **Implemente el filtrado de respuestas del lado del servidor.** No devuelva nunca objetos sin procesar de bases de datos ni serializaciones completas de modelos en las respuestas de la API. Defina explícitamente esquemas de respuesta que incluyan solo los campos que necesita el cliente para cada endpoint y contexto.
- **Use objetos de transferencia de datos (DTO) o listas de permitidos para la serialización.** Asigne los objetos internos a objetos de respuesta específicos para cada propósito que contengan solo los campos permitidos. La mayoría de los frameworks web ofrecen mecanismos para ello (por ejemplo, serializadores en Django REST Framework, JsonView en Spring o ActiveModel::Serializers en Rails).
- **Aplique controles de acceso a nivel de campo.** Algunos campos (como las direcciones de correo electrónico o los números de teléfono) pueden ser apropiados para un consumidor, pero no para otro. Implemente lógica para incluir o excluir campos según el rol del usuario que realiza la solicitud y su relación con el recurso solicitado.
- **Evite exponer identificadores internos.** Use identificadores destinados a usuarios externos (como UUID o tokens opacos) en lugar de ID secuenciales internos de la base de datos, que revelan el número de registros y permiten enumerarlos.
- **Restrinja los esquemas de GraphQL.** Elimine por completo los campos sensibles del esquema de GraphQL en lugar de confiar en comprobaciones a nivel de resolver. Si un campo debe existir en el esquema para uso interno, aplique directivas de autorización para impedir el acceso no autorizado.
- **Deshabilite los mapas de origen en producción.** No implemente mapas de origen de JavaScript en entornos de producción. Configure el proceso de compilación para excluir los archivos `.map` de los artefactos de producción.
- **Desinfecte las respuestas de error.** Devuelva mensajes de error genéricos en producción. Registre información detallada de los errores en el servidor, pero no incluya trazas de pila, rutas internas ni detalles de consultas en las respuestas dirigidas al cliente.

## Herramientas

- **Burp Suite:** Use la herramienta Comparer para comparar el contenido que se muestra en la interfaz con las respuestas sin procesar de la API. Las herramientas Repeater e Intruder permiten probar sistemáticamente endpoints con distintos parámetros y contextos de autenticación.
- **ZAP (Zed Attack Proxy):** Intercepte e inspeccione respuestas de API, use reglas de análisis activo para identificar divulgaciones de información y capacidades de fuzzing para provocar respuestas de error detalladas.
- **Postman:** Envíe solicitudes de API con distintos tokens de autenticación y compare los cuerpos de las respuestas para identificar diferencias en la exposición de datos según el rol.
- **Herramientas de desarrollo del navegador:** La pestaña Network ofrece visibilidad inmediata de las respuestas de la API sin necesidad de configurar un proxy. Es útil para comparar rápidamente los datos mostrados por la interfaz con el contenido sin procesar de la respuesta.
- **jq:** Procesador de JSON de línea de comandos para extraer y comparar campos entre respuestas de API: `curl -s https://api.example.com/users/me | jq 'keys'` muestra todos los campos devueltos.

## Referencias

- [OWASP API Security Top 10: API3:2023 Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [OWASP ASVS V5: API and Web Service](https://github.com/OWASP/ASVS/blob/v5.0.0/5.0/en/0x13-V4-API-and-Web-Service.md)
- [Referencias directas inseguras a objetos (IDOR)](../05-Authorization/04-Insecure_Direct_Object_References.md)