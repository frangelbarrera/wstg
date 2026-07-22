# Autorización Rota a Nivel de Objeto de API

|ID          |
|------------|
|WSTG-APIT-02|

## Resumen

La Autorización Rota a Nivel de Objeto (BOLA, por sus siglas en inglés) ocurre cuando una API no impone adecuadamente verificaciones de autorización para cada objeto accedido por el cliente. Los atacantes pueden manipular identificadores de objetos en las solicitudes de API (tales como IDs, GUIDs o tokens) para acceder o modificar recursos para los cuales no están autorizados. Esta vulnerabilidad es crítica en las APIs debido a su acceso directo a los objetos subyacentes y la prevalencia de las APIs en las aplicaciones modernas.

La explotación de BOLA puede llevar a acceso no autorizado a datos sensibles, suplantación de usuarios, escalada de privilegios horizontal (acceder a recursos de otros usuarios) y escalada de privilegios vertical (obtener acceso no autorizado a nivel de administrador).

## Objetivos de Prueba

- El objetivo de esta prueba es identificar si la API impone verificaciones adecuadas de **autorización a nivel de objeto**, asegurando que los usuarios solo puedan acceder y manipular objetos que están autorizados a manejar.

## Cómo Probar

### Entender los Endpoints de la API y las Referencias a Objetos

Revisar la documentación de la API (por ejemplo, especificación OpenAPI), el tráfico, o usar un proxy de intercepción (por ejemplo, **Burp Suite**, **ZAP**) para identificar los endpoints que aceptan identificadores de objetos de interés. Estos pueden estar en forma de **IDs**, **UUIDs** u otras referencias.

Ejemplos:

- `GET /api/users/{user_id}`
- `GET /api/orders/{order_id}`
- `POST /graphql`\
        `query: {user(id: "123") }`

Con el conocimiento gained en el paso anterior, revisar y recopilar identificadores de objetos de terceros (por ejemplo, IDs de usuario, IDs de órdenes, etc.) que puedan ser utilizados posteriormente en la manipulación de identificadores de objetos.

Adicionalmente, generar una lista de identificadores de objetos potenciales para fuerza bruta. Por ejemplo, si una API está recuperando una orden de compra de un usuario autenticado, generar varios IDs de órdenes de compra para pruebas.

### Manipular Identificadores de Objetos en las Solicitudes de API

Con el objetivo de determinar si los usuarios pueden acceder o modificar objetos que no poseen alterando identificadores de objetos en la solicitud de API, cambiar el identificador de objeto (por ejemplo, ID de usuario, ID de orden) en la URL o en el cuerpo de la solicitud.

Ejemplo: Modificar una solicitud como `GET /api/users/123/profile` (donde 123 es el ID del usuario actual) a `GET /api/users/124/profile` (donde 124 es el ID de otro usuario).

Dependiendo del contexto de la aplicación, utilizar dos cuentas diferentes para realizar las pruebas. Con una cuenta A, crear recursos que pertenezcan exclusivamente a esa cuenta (por ejemplo, orden de compra) y con una cuenta B, intentar acceder al recurso de la cuenta A (por ejemplo, orden de compra).

### Probar Acceso a Nivel de Objeto con Diferentes Métodos HTTP

Probar varios **métodos HTTP** para vulnerabilidades BOLA:

- **GET**: Intentar acceder a objetos no autorizados manipulando el ID del objeto en la solicitud.
- **POST/PUT/PATCH**: Intentar crear o modificar objetos que pertenezcan a otros usuarios.
- **DELETE**: Intentar eliminar un objeto propiedad de otro usuario.

### Probar BOLA en APIs GraphQL

Para **APIs GraphQL**, enviar una consulta con un ID de objeto modificado en los parámetros de la consulta (ver [Pruebas de GraphQL](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL)):

Ejemplo: `query { user(id: "124") { name, email } }`.

### Probar Acceso Masivo a Objetos

Probar si la API permite **acceso masivo** no autorizado a objetos. Esto podría ocurrir en endpoints que devuelven listas de objetos.

Ejemplo: `GET /api/users` devuelve datos de todos los usuarios en lugar de solo los datos del usuario autenticado.

## Indicadores de BOLA

- **Explotación exitosa**: Si modificar un ID de objeto en la solicitud devuelve datos o permite acciones sobre objetos que pertenecen a otros usuarios, la API es vulnerable a BOLA.
- **Respuestas de error**: Las APIs adecuadamente aseguradas en general devolverían `403 Forbidden` o `401 Unauthorized` para acceso no autorizado a objetos. Una respuesta `200 OK` para el objeto de otro usuario indica BOLA.
- **Respuestas inconsistentes**: Si algunos endpoints imponen autorización y otros no, señala controles de seguridad incompletos o inconsistentes.

## Remediación

- **Verificaciones de Propiedad de Objetos**: Asegurar que las verificaciones de autorización a nivel de objeto se realicen para cada solicitud de API. Siempre verificar que el usuario que hace la solicitud esté autorizado para acceder al objeto solicitado.
- **Control de Acceso Basado en Roles (RBAC)**: Implementar políticas RBAC que definan qué roles pueden acceder o modificar objetos específicos.
- **Principio de Menor Privilegio**: Aplicar el principio de menor privilegio para asegurar que los usuarios solo puedan acceder al conjunto mínimo de objetos que necesitan para su rol.
- **Usar UUIDs o IDs No Secuenciales**: Preferir identificadores de objetos no predecibles y no secuenciales (por ejemplo, **UUIDs** en lugar de enteros simples) para dificultar los ataques de enumeración y fuerza bruta.

## Herramientas

- **ZAP**: Escáneres automatizados o herramientas de proxy manuales pueden ayudar a probar referencias a objetos en solicitudes de API.
- **Burp Suite**: Usar las herramientas **Repeater** o **Intruder** para manipular IDs de objetos y enviar múltiples solicitudes para probar el control de acceso.
- **Postman**: Enviar solicitudes con IDs de objetos alterados y observar las respuestas.
- **Herramientas de Fuzzing**: Usar fuzzers para forzar bruscamente IDs de objetos y verificar accesos no autorizados.

## Referencias

- [OWASP API Security Top 10: BOLA](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP Testing Guide: Pruebas de Referencias Directas a Objetos Inseguras (IDOR)](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [OWASP Testing Guide: Pruebas de GraphQL](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL)
