# Autorización rota a nivel de función de API

|ID          |
|------------|
|WSTG-APIT-04|

## Resumen

La autorización rota a nivel de función (BFLA, por sus siglas en inglés) ocurre cuando una API impone de forma inadecuada restricciones a los usuarios que acceden a determinadas funciones u operaciones. Esta vulnerabilidad permite a los atacantes invocar funciones sensibles que no están autorizados a ejecutar, como funciones administrativas u otras operaciones de alto privilegio.

BFLA suele surgir cuando las API exponen varios endpoints que atienden a diferentes roles de usuario (por ejemplo, usuario frente a administrador), pero no restringen el acceso a estas funciones según el nivel de autorización del usuario.

La explotación de BFLA puede tener consecuencias graves, como la escalada de privilegios, el acceso no autorizado a funciones sensibles (por ejemplo, operaciones administrativas) o la exposición de funcionalidades críticas que solo deberían estar disponibles para roles de usuario específicos.

## Objetivos de las pruebas

- El objetivo de esta prueba es determinar si la API aplica **controles de acceso basados en roles o privilegios** para impedir que los usuarios accedan a funciones o las ejecuten sin autorización. Esto garantiza que se apliquen correctamente los límites de seguridad a nivel de función.

## Cómo probar

### Identificar endpoints a nivel de función

Revisar la documentación de la API (por ejemplo, la especificación OpenAPI) y el tráfico, o usar un proxy de interceptación (por ejemplo, **Burp Suite**, **ZAP**) para identificar distintos endpoints a nivel de función. Estos pueden incluir:

- **Funciones administrativas** (por ejemplo, `/api/admin/deleteUser`, `/api/admin/getAllUsers`)
- **Operaciones basadas en roles** (por ejemplo, `/api/admin/promoteUser`, `/api/user/createOrder`)
- **Funciones críticas** para los usuarios (por ejemplo, `/api/user/withdrawFunds`)

Centrarse en las **diferencias de funcionalidad** entre los distintos roles de usuario (por ejemplo, usuario común, administrador, socio, invitado) y en los endpoints que ofrecen capacidades más sensibles.

### Manipular los controles de acceso basados en roles

Intentar acceder a operaciones sensibles expuestas en endpoints de la API, o ejecutarlas, cuando deberían estar restringidas según los roles de usuario.

Iniciar sesión como un usuario con menos privilegios (por ejemplo, un invitado o un usuario común) y enviar solicitudes a endpoints que realizan acciones sensibles reservadas para roles con más privilegios (por ejemplo, el administrador).

Ejemplo: como **usuario común**, enviar una solicitud al siguiente endpoint administrativo para eliminar un usuario aleatorio:

```http
POST /api/admin/deleteUser
Authorization: Bearer <regular_user_token>

{ "userId": "12345" }
```

### Probar el acceso a nivel de función con diferentes métodos HTTP

Probar varios **métodos HTTP** para detectar vulnerabilidades BFLA. Probar cada recurso identificado con métodos HTTP alternativos. Enviar solicitudes con métodos distintos de los documentados (como `PUT`, `DELETE` o `PATCH`) a endpoints que normalmente solo aceptan `GET` o `POST`. Registrar cualquier diferencia en el comportamiento de la respuesta o en el resultado de la autorización.

- **GET**: Intentar acceder a información disponible solo para usuarios con altos privilegios, como los administradores.
    - Ejemplo: `GET /api/admin/getAllUsers`
- **POST/PUT/PATCH**: Intentar modificar o crear recursos sensibles, como cambiar roles de usuario o datos críticos del sistema.
    - Ejemplo: `POST /api/admin/promoteUser { "userId": "12345", "newRole": "admin" }`
- **DELETE**: Intentar eliminar recursos sensibles, como cuentas de usuario o datos.
    - Ejemplo: `DELETE /api/admin/deleteUser/12345`

### Probar BFLA en API GraphQL

En las **API GraphQL**, probar si un usuario puede invocar funciones restringidas a roles con más privilegios mediante la modificación de consultas GraphQL.

Ejemplo: `mutation { deleteUser(id: "12345") { success } }`.

## Indicadores de BFLA

- **Explotación exitosa**: Un usuario con menos privilegios (por ejemplo, un usuario común o un invitado) puede ejecutar funciones de alto privilegio o realizar acciones reservadas para otros roles (por ejemplo, el administrador).
- **Respuestas de error**: En general, las API protegidas correctamente deberían devolver `403 Forbidden` o `401 Unauthorized` cuando se invocan funciones restringidas, en lugar de una respuesta `200 OK`.
- **Aplicación incoherente**: Algunos endpoints aplican restricciones basadas en roles y otros no, lo que indica que los controles de seguridad son incoherentes.

## Medidas correctivas

Para prevenir las vulnerabilidades BFLA, implementar las siguientes medidas:

- **Aplicar el control de acceso basado en roles (RBAC)**: Asegurarse de que la API compruebe los roles y permisos del usuario a **nivel de función** antes de permitir el acceso a determinadas operaciones. Solo los roles autorizados deberían poder invocar funciones sensibles.
- **Principio de mínimo privilegio**: Aplicar el principio de mínimo privilegio y asegurarse de que los usuarios solo puedan acceder al conjunto mínimo de funciones que necesitan para su rol.
- **Lógica centralizada de control de acceso**: Usar lógica centralizada de control de acceso para garantizar la coherencia en todos los endpoints de la API. Esto evita brechas en las que algunas funciones podrían carecer de controles de acceso adecuados.

## Herramientas

- **ZAP**: Usar escáneres automatizados y herramientas de proxy manuales para inspeccionar solicitudes y respuestas de la API en busca de vulnerabilidades BFLA.
- **Burp Suite**: Usar **Repeater** o **Intruder** para enviar solicitudes como usuarios con menos privilegios y comprobar si se aplican las restricciones a nivel de función.
- **Postman**: Enviar manualmente solicitudes a la API con distintos roles de usuario y observar las respuestas.
- **Herramientas de fuzzing**: Usar herramientas de fuzzing para probar distintos parámetros y métodos de función e identificar posibles debilidades de autorización.

## Referencias

- [OWASP API Security Top 10: BFLA](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)
- [Escalada de privilegios](../05-Authorization/03-Privilege_Escalation.md)
- [GraphQL](99-GraphQL.md)