# Pruebas de Exposición Excesiva de Datos

|ID          |
|------------|
|WSTG-APIT-03|

## Resumen

La exposición excesiva de datos ocurre cuando una API devuelve más información en sus respuestas de la que el cliente necesita para mostrar o funcionar. Esto comúnmente ocurre cuando los desarrolladores implementan endpoints de API serializando objetos backend completos (tales como modelos de base de datos o estructuras de datos internas) directamente en las respuestas de la API, confiando en el código del lado del cliente para filtrar los campos sensibles antes de presentarlos al usuario.

El problema es que las respuestas de la API son directamente accesibles para cualquiera que pueda hacer solicitudes, independientemente de lo que la interfaz de usuario elija mostrar. Un atacante que inspeccione el tráfico crudo de la API puede observar todos los campos devueltos, incluyendo aquellos que la UI oculta intencionalmente. Esto puede exponer contraseñas, tokens de autenticación, identificadores internos, información de identificación personal (PII), datos financieros, detalles de infraestructura o lógica de negocio interna.

Esta vulnerabilidad se mapea a [OWASP API Security Top 10 API3:2023 Autorización Rota a Nivel de Propiedad de Objeto](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/).

## Objetivos de Prueba

- Identificar respuestas de API que contengan más datos de los que la aplicación cliente muestra o requiere.
- Detectar campos sensibles devueltos en respuestas de API tales como hashes de contraseñas, tokens de autenticación, IDs internos de objetos, PII o detalles de infraestructura.
- Determinar si la API depende del filtrado del lado del cliente en lugar de la selección de campos del lado del servidor para controlar la exposición de datos.

## Cómo Probar

### Comparar Respuestas de API Contra Datos Mostrados

La prueba más directa es comparar lo que la interfaz de usuario muestra con lo que la API realmente devuelve.

1. Abrir la aplicación en un navegador y navegar a una página que muestre datos de usuario u objeto (por ejemplo, una página de perfil, un resumen de orden o un panel de control).
2. Abrir las herramientas de desarrollador del navegador (pestaña Network) o configurar un proxy de intercepción como Burp Suite o ZAP.
3. Identificar las solicitudes de API que poblan el contenido de la página.
4. Comparar los campos mostrados en la UI contra el conjunto completo de campos en la respuesta de la API.

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

En este caso, la API devuelve hashes de contraseñas, números de seguro social, claves de API, identificadores internos y direcciones IP internas que la UI nunca muestra.

### Inspeccionar Respuestas a Través de Diferentes Endpoints de API

No limitar las pruebas a un solo endpoint. Verificar respuestas de múltiples tipos de endpoints:

- **Endpoints de usuario y cuenta:** `GET /api/users/{id}`, `GET /api/me`, `GET /api/account`
- **Endpoints de lista y búsqueda:** `GET /api/users`, `GET /api/orders?status=completed` (los endpoints de lista frecuentemente devuelven el mismo objeto completo por cada elemento en la lista, amplificando la exposición)
- **Endpoints de objetos relacionados:** `GET /api/orders/{id}` puede embeber objetos completos de usuario, detalles de pago o direcciones de envío dentro de la respuesta de la orden
- **Respuestas de error:** Las solicitudes fallidas pueden devolver mensajes de error verbosos que contienen trazas de pila, rutas de archivos internas, detalles de consultas de base de datos o valores de configuración
- **Endpoints administrativos o internos:** Verificar si los endpoints destinados a uso interno son accesibles externamente y devuelven datos elevados

### Probar con Diferentes Niveles de Privilegio de Usuario

El mismo endpoint puede devolver diferentes cantidades de datos dependiendo del rol del usuario solicitante. Probar si:

- Un usuario de bajo privilegio recibe los mismos campos que un administrador al solicitar el mismo recurso.
- Una solicitud no autenticada devuelve datos que deberían requerir autenticación.
- Solicitar los datos de otro usuario (IDOR) devuelve el mismo objeto rico que solicitar los propios.

Por ejemplo, `GET /api/users/124` podría devolver el objeto completo incluyendo campos sensibles independientemente de si el solicitante es el usuario 124, otro usuario o no está autenticado.

### Analizar Objetos Anidados y Embebidos

Las APIs modernas frecuentemente embeben objetos relacionados dentro de las respuestas. Una sola llamada de API puede devolver estructuras profundamente anidadas que exponen datos de múltiples sistemas backend.

Ejemplo: Un endpoint de orden devolviendo un objeto de cliente embebido, un objeto de pago y un objeto de envío:

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
  "internal_notes": "Cliente marcado para revisión manual - fraude sospechado"
}
```

Cada objeto anidado debería revisarse para identificar campos que excedan lo que el cliente consumidor necesita.

### Probar Asignación Masiva (Mass Assignment)

Para endpoints que aceptan un cuerpo JSON o XML, probar vulnerabilidades de asignación masiva añadiendo campos extra al cuerpo de la solicitud más allá de los mostrados en la interfaz de la aplicación. Campos tales como identificadores de rol, indicadores de privilegio o atributos de estado de cuenta son objetivos de alto valor.

### Verificar Datos Sensibles en Respuestas GraphQL

Las APIs GraphQL presentan una forma específica de este problema. Mientras GraphQL permite a los clientes seleccionar qué campos desean, el esquema puede exponer campos sensibles que las consultas UI predeterminadas no solicitan pero que un atacante puede añadir a una consulta personalizada.

1. Si la introspección está habilitada, recuperar el esquema y revisar los campos disponibles en cada tipo (ver [Pruebas de GraphQL](99-Testing_GraphQL.md)):

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

2. Buscar campos tales como `passwordHash`, `apiKey`, `ssn`, `internalId`, `role` o `isAdmin` que son consultables pero no usados por el frontend.
3. Intentar consultar estos campos directamente:

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

### Examinar Source Maps de JavaScript y Bundles del Lado del Cliente

Los bundles de JavaScript del frontend y los source maps pueden revelar inadvertidamente la estructura esperada de respuesta de la API, incluyendo campos que la UI oculta intencionalmente. Si los source maps son públicamente accesibles (por ejemplo, en `main.js.map`), pueden exponer:

- La lista completa de campos que el frontend espera de las respuestas de la API
- Rutas de endpoints de API internos
- Lógica de flujo de autenticación y autorización
- Claves de API, tokens o valores de configuración hardcodeados

Probar la disponibilidad de source maps:

```http
GET /static/js/main.js.map
GET /assets/app.js.map
```

Si el source map es accesible y contiene `sourcesContent`, el código fuente original puede ser reconstruido y analizado para identificar estructuras de respuesta de API esperadas y campos ocultos.

### Verificar Respuestas de Error y Salida de Depuración

Las APIs pueden filtrar información sensible a través de respuestas de error, particularmente cuando se ejecutan en modo desarrollo o depuración:

- **Trazas de pila** revelando rutas de archivos internas, versiones de framework y estructura de código
- **Mensajes de error de base de datos** exponiendo nombres de tablas, nombres de columnas o sintaxis de consulta
- **Errores de validación verbosos** listando todos los campos esperados, incluyendo aquellos no documentados en la API pública
- **Encabezados de depuración** tales como `X-Debug-Token`, `X-Powered-By` o encabezados personalizados que contienen información interna

Disparar respuestas de error enviando entrada mal formada, autenticación inválida o solicitudes de recursos inexistentes, e inspeccionar la respuesta completa incluyendo encabezados.

## Remediación

- **Implementar filtrado de respuesta del lado del servidor.** Nunca devolver objetos crudos de base de datos o serializaciones completas de modelos en respuestas de API. Definir explícitamente esquemas de respuesta que incluyan solo los campos que el cliente necesita para cada endpoint y contexto.
- **Usar Objetos de Transferencia de Datos (DTOs) o listas blancas de serialización.** Mapear objetos internos a objetos de respuesta específicos por propósito que contengan solo los campos permitidos. La mayoría de los frameworks web proporcionan mecanismos para esto (por ejemplo, serializadores en Django REST Framework, JsonView en Spring o ActiveModel::Serializers en Rails).
- **Aplicar control de acceso a nivel de campo.** Algunos campos (tales como direcciones de correo electrónico o números de teléfono) pueden ser apropiados para un consumidor pero no para otro. Implementar lógica para incluir o excluir campos basada en el rol del usuario solicitante y su relación con el recurso solicitado.
- **Evitar exponer identificadores internos.** Usar identificadores de cara externa (tales como UUIDs u tokens opacos) en lugar de IDs internos secuenciales de base de datos, los cuales revelan conteos de registros y habilitan enumeración.
- **Restringir esquemas GraphQL.** Eliminar campos sensibles del esquema GraphQL enteramente en lugar de confiar en verificaciones a nivel de resolver. Si un campo debe existir en el esquema para uso interno, aplicar directivas de autorización para prevenir acceso no autorizado.
- **Deshabilitar source maps en producción.** No desplegar source maps de JavaScript en entornos de producción. Configurar el pipeline de build para excluir archivos `.map` de los artefactos de producción.
- **Saneamiento de respuestas de error.** Devolver mensajes de error genéricos en producción. Registrar información detallada de errores del lado del servidor pero no incluir trazas de pila, rutas internas o detalles de consulta en respuestas de cara al cliente.

## Herramientas

- **Burp Suite:** Usar la herramienta Comparer para hacer diff del contenido mostrado en la UI contra las respuestas crudas de la API. Las herramientas Repeater e Intruder pueden ser usadas para probar sistemáticamente endpoints con diferentes parámetros y contextos de autenticación.
- **ZAP (Zed Attack Proxy):** Intercepta e inspecciona respuestas de API, usa reglas de escaneo activo para identificar divulgación de información y capacidades de fuzzing para disparar respuestas de error verbosas.
- **Postman:** Enviar solicitudes de API con diferentes tokens de autenticación y comparar cuerpos de respuesta para identificar diferencias de exposición de datos dependientes del rol.
- **Herramientas de Desarrollador del Navegador:** La pestaña Network proporciona visibilidad inmediata de las respuestas de API sin requerir configuración de proxy. Útil para comparación rápida de datos renderizados por UI vs contenido crudo de respuesta.
- **jq:** Procesador JSON de línea de comandos para extraer y comparar campos a través de respuestas de API: `curl -s https://api.example.com/users/me | jq 'keys'` lista todos los campos devueltos.

## Referencias

- [OWASP API Security Top 10: API3:2023 Autorización Rota a Nivel de Propiedad de Objeto](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [OWASP ASVS V5: API y Servicio Web](https://github.com/OWASP/ASVS/blob/v5.0.0/5.0/en/0x13-V4-API-and-Web-Service.md)
- [Pruebas de Referencias Directas a Objetos Inseguras (IDOR)](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)
