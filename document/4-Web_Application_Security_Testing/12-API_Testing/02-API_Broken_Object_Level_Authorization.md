# Autorización rota a nivel de objeto en API

|ID          |
|------------|
|WSTG-APIT-02|

## Resumen

La autorización rota a nivel de objeto (BOLA, por sus siglas en inglés) ocurre cuando una API no aplica correctamente las comprobaciones de autorización para cada objeto al que accede el cliente. Los atacantes pueden manipular los identificadores de objetos en las solicitudes de API (como IDs, GUIDs o tokens) para acceder a recursos o modificarlos sin autorización. Esta vulnerabilidad es crítica en las API debido a su acceso directo a los objetos subyacentes y a la prevalencia de las API en las aplicaciones modernas.

La explotación de BOLA puede permitir el acceso no autorizado a datos confidenciales, la suplantación de usuarios, la escalada horizontal de privilegios (acceso a los recursos de otros usuarios) y la escalada vertical de privilegios (obtención de acceso no autorizado con nivel de administrador).

## Objetivos de la prueba

- El objetivo de esta prueba es identificar si la API aplica correctamente las comprobaciones de **autorización a nivel de objeto**, para garantizar que los usuarios solo puedan acceder a los objetos con los que están autorizados a interactuar y manipularlos.

## Cómo probar

### Comprender los endpoints de la API y las referencias a objetos

Revisar la documentación de la API (por ejemplo, la especificación OpenAPI), el tráfico o usar un proxy de interceptación (por ejemplo, **Burp Suite**, **ZAP**) para identificar los endpoints que aceptan identificadores de objetos de interés. Estos pueden ser **IDs**, **UUIDs** u otras referencias.

Ejemplos:

- `GET /api/users/{user_id}`
- `GET /api/orders/{order_id}`
- `POST /graphql`\
        `query: {user(id: "123") }`

Con la información obtenida en el paso anterior, revisar y recopilar identificadores de objetos de terceros (por ejemplo, IDs de usuarios, IDs de órdenes, etc.) que puedan utilizarse posteriormente para manipular identificadores de objetos.

Además, generar una lista de posibles identificadores de objetos para probar por fuerza bruta. Por ejemplo, si una API recupera una orden de compra de un usuario autenticado, generar varios IDs de órdenes de compra para probarlos.

### Manipular identificadores de objetos en las solicitudes de API

Para determinar si los usuarios pueden acceder a objetos que no les pertenecen o modificarlos alterando los identificadores de objetos en una solicitud de API, cambiar el identificador del objeto (por ejemplo, el ID de usuario o el ID de orden) en la URL o en el cuerpo de la solicitud.

Ejemplo: Modificar una solicitud como `GET /api/users/123/profile` (donde 123 es el ID del usuario actual) para que sea `GET /api/users/124/profile` (donde 124 es el ID de otro usuario).

Según el contexto de la aplicación, utilizar dos cuentas diferentes para realizar las pruebas. Con una cuenta A, crear recursos que pertenezcan exclusivamente a esa cuenta (por ejemplo, una orden de compra) y, con una cuenta B, intentar acceder al recurso de la cuenta A (por ejemplo, la orden de compra).

### Probar el acceso a nivel de objeto con diferentes métodos HTTP

Probar varios **métodos HTTP** para detectar vulnerabilidades BOLA:

- **GET**: Intentar acceder a objetos no autorizados manipulando el ID del objeto en la solicitud.
- **POST/PUT/PATCH**: Intentar crear o modificar objetos que pertenezcan a otros usuarios.
- **DELETE**: Intentar eliminar un objeto que pertenezca a otro usuario.

### Probar BOLA en API GraphQL

Para las **API GraphQL**, enviar una consulta con un ID de objeto modificado en los parámetros de la consulta (ver [GraphQL](99-GraphQL.md)):

Ejemplo: `query { user(id: "124") { name, email } }`.

### Probar el acceso masivo a objetos

Probar si la API permite el **acceso masivo** no autorizado a objetos. Esto podría ocurrir en endpoints que devuelven listas de objetos.

Ejemplo: `GET /api/users` devuelve datos de todos los usuarios en lugar de solo los datos del usuario autenticado.

## Indicadores de BOLA

- **Explotación exitosa**: Si modificar un ID de objeto en la solicitud devuelve datos o permite realizar acciones sobre objetos que pertenecen a otros usuarios, la API es vulnerable a BOLA.
- **Respuestas de error**: En general, las API protegidas correctamente devolverían `403 Forbidden` o `401 Unauthorized` cuando se intenta acceder sin autorización a un objeto. Una respuesta `200 OK` para el objeto de otro usuario indica BOLA.
- **Respuestas inconsistentes**: Si algunos endpoints aplican la autorización y otros no, esto indica que los controles de seguridad son incompletos o inconsistentes.

## Remediación

- **Comprobaciones de propiedad de los objetos**: Asegurarse de que se realicen comprobaciones de autorización a nivel de objeto en cada solicitud de API. Verificar siempre que el usuario que realiza la solicitud esté autorizado a acceder al objeto solicitado.
- **Control de acceso basado en roles (RBAC)**: Implementar políticas RBAC que definan qué roles pueden acceder a objetos específicos o modificarlos.
- **Principio de mínimo privilegio**: Aplicar el principio de mínimo privilegio para garantizar que los usuarios solo puedan acceder al conjunto mínimo de objetos que necesitan para su rol.
- **Usar UUIDs o IDs no secuenciales**: Preferir identificadores de objetos impredecibles y no secuenciales (por ejemplo, **UUIDs** en lugar de enteros simples) para dificultar los ataques de enumeración y de fuerza bruta.

## Herramientas

- **ZAP**: Los escáneres automatizados o las herramientas de proxy manuales pueden ayudar a probar las referencias a objetos en las solicitudes de API.
- **Burp Suite**: Usar las herramientas **Repeater** o **Intruder** para manipular IDs de objetos y enviar varias solicitudes con el fin de probar el control de acceso.
- **Postman**: Enviar solicitudes con IDs de objetos modificados y observar las respuestas.
- **Herramientas de fuzzing**: Usar herramientas de fuzzing para probar IDs de objetos por fuerza bruta y comprobar si se obtiene acceso no autorizado.

## Referencias

- [OWASP API Security Top 10: BOLA](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [Referencias directas inseguras a objetos (IDOR)](../05-Authorization/04-Insecure_Direct_Object_References.md)
- [GraphQL](99-GraphQL.md)