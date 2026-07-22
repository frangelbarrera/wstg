# Autorización Rota a Nivel de Función de API

|ID          |
|------------|
|WSTG-APIT-04|

## Resumen

La Autorización Rota a Nivel de Función (BFLA, por sus siglas en inglés) ocurre cuando una API impone inadecuadamente restricciones sobre los usuarios que acceden a ciertas funciones u operaciones. Esta vulnerabilidad permite a los atacantes invocar funciones sensibles para las cuales no están autorizados a ejecutar, tales como funciones administrativas u otras operaciones de alto privilegio.

BFLA comúnmente surge cuando las APIs exponen múltiples endpoints que sirven a diferentes roles de usuario (por ejemplo, usuario vs. administrador) pero fallan en restringir el acceso a estas funciones basándose en el nivel de autorización del usuario.

La explotación de BFLA puede llevar a consecuencias serias tales como escalada de privilegios, acceso no autorizado a funciones sensibles (por ejemplo, operaciones administrativas) o exposición de funcionalidades críticas que deberían solo ser accesibles a roles de usuario específicos.

## Objetivos de Prueba

- El objetivo de esta prueba es determinar si la API impone **control de acceso basado en rol o privilegio** para restringir a los usuarios de acceder o ejecutar funciones para las cuales no están autorizados a usar. Esto asegura que los límites de seguridad a nivel de función sean propiamente impuestos.

## Cómo Probar

### Identificar Endpoints a Nivel de Función

Revisar la documentación de la API (por ejemplo, especificación OpenAPI), el tráfico, o usar un proxy de intercepción (por ejemplo, **Burp Suite**, **ZAP**) para identificar diferentes endpoints a nivel de función. Estos podrían incluir:

- **Funciones administrativas** (por ejemplo, `/api/admin/deleteUser`, `/api/admin/getAllUsers`)
- **Operaciones basadas en rol** (por ejemplo, `/api/admin/promoteUser`, `/api/user/createOrder`)
- **Funciones críticas** para usuarios (por ejemplo, `/api/user/withdrawFunds`)

Enfocarse en las **diferencias de funcionalidad** entre diferentes roles de usuario (por ejemplo, usuario regular, administrador, socio, invitado) y endpoints que ofrecen capacidades más sensibles.

### Manipular Controles de Acceso Basados en Rol

Intentar acceder o realizar operaciones sensibles expuestas en endpoints de API que deberían estar restringidas basándose en roles de usuario.

Iniciar sesión como un usuario de menor privilegio (por ejemplo, invitado o usuario regular) y enviar solicitudes a endpoints que realicen acciones sensibles reservadas para roles de mayor privilegio (por ejemplo, administrador).

Ejemplo: como **usuario regular**, enviar una solicitud al siguiente endpoint administrativo para eliminar un usuario aleatorio:

```http
POST /api/admin/deleteUser
Authorization: Bearer <regular_user_token>

{ "userId": "12345" }
```

### Probar Acceso a Nivel de Función con Diferentes Métodos HTTP

Probar varios **métodos HTTP** para vulnerabilidades BFLA. Probar cada recurso identificado con métodos HTTP alternativos. Emitir solicitudes usando métodos distintos a los documentados (tales como `PUT`, `DELETE` o `PATCH`) contra endpoints que normalmente aceptan solo `GET` o `POST`. Registrar cualquier diferencia en el comportamiento de respuesta o resultado de autorización.

- **GET**: Intentar acceder a información disponible solo para usuarios de alto privilegio, tales como administradores.
    - Ejemplo: `GET /api/admin/getAllUsers`
- **POST/PUT/PATCH**: Intentar modificar o crear recursos sensibles, tales como cambiar roles de usuario o datos críticos del sistema.
    - Ejemplo: `POST /api/admin/promoteUser { "userId": "12345", "newRole": "admin" }`
- **DELETE**: Intentar eliminar recursos sensibles, tales como remover cuentas de usuario o datos.
    - Ejemplo: `DELETE /api/admin/deleteUser/12345`

### Probar BFLA en APIs GraphQL

En **APIs GraphQL**, probar si un usuario puede invocar funciones restringidas a roles de mayor privilegio modificando consultas GraphQL.

Ejemplo: `mutation { deleteUser(id: "12345") { success } }`.

## Indicadores de BFLA

- **Explotación exitosa**: Si un usuario de menor privilegio (por ejemplo, usuario regular o invitado) puede ejecutar funciones de alto privilegio o realizar acciones reservadas para otros roles (por ejemplo, administrador).
- **Respuestas de error**: Las APIs adecuadamente aseguradas en general devolverían `403 Forbidden` o `401 Unauthorized` cuando se invocan funciones restringidas en lugar de una respuesta `200 OK`.
- **Imposición inconsistente**: Algunos endpoints imponen restricciones basadas en rol mientras que otros no, lo cual indica controles de seguridad inconsistentes.

## Remediaciones

Para prevenir vulnerabilidades BFLA, implementar las siguientes mitigaciones:

- **Imponer Control de Acceso Basado en Roles (RBAC)**: Asegurar que la API verifique los roles y permisos de usuario a **nivel de función** antes de permitir acceso a ciertas operaciones. Solo los roles autorizados deberían poder invocar funciones sensibles.
- **Principio de Menor Privilegio**: Aplicar el principio de menor privilegio asegurando que los usuarios solo puedan acceder al conjunto mínimo de funciones que necesitan para su rol.
- **Lógica de Control de Acceso Centralizada**: Usar lógica de control de acceso centralizada para asegurar consistencia a través de todos los endpoints de API. Esto evita brechas donde algunas funciones pueden carecer de verificaciones de acceso apropiadas.

## Herramientas

- **ZAP**: Usar escáneres automatizados y herramientas de proxy manuales para inspeccionar solicitudes y respuestas de API en busca de vulnerabilidades BFLA.
- **Burp Suite**: Usar **Repeater** o **Intruder** para enviar solicitudes como usuarios de menor privilegio para probar si las restricciones a nivel de función son impuestas.
- **Postman**: Enviar manualmente solicitudes de API como diferentes roles de usuario y observar las respuestas.
- **Herramientas de Fuzzing**: Usar fuzzers para probar diferentes parámetros y métodos de función para identificar debilidades potenciales de autorización.

## Referencias

- [OWASP API Security Top 10: BFLA](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)
- [OWASP Testing Guide: Pruebas de Escalada de Privilegios](../05-Authorization_Testing/03-Testing_for_Privilege_Escalation.md)
- [OWASP Testing Guide: Pruebas de GraphQL](99-Testing_GraphQL.md)
