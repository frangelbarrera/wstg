# Pruebas de asignación masiva

|ID          |
|------------|
|WSTG-INPV-20|

## Resumen

Las aplicaciones web modernas se basan muy a menudo en marcos de trabajo. Muchos de estos marcos de trabajo permiten el enlace automático de la entrada del usuario (en forma de parámetros de solicitud HTTP) a objetos internos. Esto se denomina a menudo autobinding.
Esta característica puede ser explotada a veces para acceder a campos que nunca se pretendieron modificar desde el exterior, lo que lleva a escalada de privilegios, manipulación de datos, evasión de mecanismos de seguridad y más.
En este caso, existe una vulnerabilidad de asignación masiva.

Ejemplos de propiedades sensibles:

- **Propiedades relacionadas con permisos**: solo deben ser configuradas por usuarios privilegiados (ej. `is_admin`, `role`, `approved`).
- **Propiedades dependientes del proceso**: solo deben ser configuradas internamente, después de que se complete un proceso (ej. `balance`, `status`, `email_verified`)
- **Propiedades internas**: solo deben ser configuradas internamente por la aplicación (ej. `created_at`, `updated_at`)

## Objetivos de la prueba

- Identificar solicitudes que modifican objetos
- Evaluar si es posible modificar campos que nunca se pretendieron modificar desde el exterior

## Cómo probar

Lo siguiente es un ejemplo clásico que puede ayudar a ilustrar el problema.

Suponga una aplicación web Java con un objeto `User` similar al siguiente:

```java
public class User {
   private String username;
   private String password;
   private String email;
   private boolean isAdmin;

   //Getters & Setters
}
```

Para crear un nuevo `User`, la aplicación web implementa la siguiente vista:

```html
<form action="/createUser" method="POST">
     <input name="username" type="text">
     <input name="password" type="text">
     <input name="email" text="text">
     <input type="submit" value="Create">
</form>
```

El controlador que maneja la solicitud de creación (Spring proporciona el enlace automático con el modelo `User`):

```java
@RequestMapping(value = "/createUser", method = RequestMethod.POST)
public String createUser(User user) {
   userService.add(user);
   return "successPage";
}
```

Cuando se envía el formulario, el navegador genera la siguiente solicitud:

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test
```

Sin embargo, debido al autobinding, un atacante puede agregar el parámetro `isAdmin` a la solicitud, que el controlador enlazará automáticamente al modelo.

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test&isAdmin=true
```

El usuario se crea entonces con la propiedad `isAdmin` configurada en `true`, dándole derechos administrativos en la aplicación.

### Pruebas de caja negra

#### Detectar manejadores

Para determinar qué parte de la aplicación es vulnerable a la asignación masiva, enumere todas las partes de la aplicación que aceptan contenido del usuario y que pueden potencialmente mapearse con un modelo. Esto incluye todas las solicitudes HTTP (muy probablemente GET, POST y PUT) que parecen permitir operaciones de creación o actualización en el backend.
Uno de los indicadores más simples para asignaciones masivas potenciales es la presencia de sintaxis de corchetes para nombres de parámetros de entrada, como por ejemplo:

```html
<input name="user[name]" type="text">
```

Cuando se encuentren tales patrones, intente agregar una entrada relacionada con un atributo no existente (ej. `user[nonexistingattribute]`) y analice la respuesta/comportamiento.
Si la aplicación no implementa ningún control (ej. lista de campos permitidos), es probable que responda con un error (ej. 500) debido a que la aplicación no encuentra el atributo asociado al objeto. Más interesante aún, esos errores a veces facilitan el descubrimiento de nombres de atributos y tipos de datos de valores necesarios para explotar el problema, sin acceso al código fuente.

#### Identificar campos sensibles

Dado que en las pruebas de caja negra el probador no tiene visibilidad del código fuente, es necesario encontrar otras formas de recopilar información sobre los atributos asociados a los objetos.
Analice las respuestas recibidas del backend, prestando especial atención a:

- Código fuente de la página HTML
- Código JavaScript personalizado
- Respuestas de API

Por ejemplo, muy a menudo, es posible explotar manejadores que devuelven detalles sobre un objeto para recopilar pistas sobre los campos asociados.
Suponga, por ejemplo, un manejador que devuelve el perfil del usuario (ej. `GET /profile`), esto puede incluir atributos adicionales relacionados con el usuario (en este ejemplo, el atributo `isAdmin` parece particularmente interesante).

```json
{"_id":12345,"username":"bob","age":38,"email":"bob@domain.test","isAdmin":false}
```

Luego intente explotar manejadores que permitan la modificación o creación de usuarios, agregando el atributo `isAdmin` configurado en `true`.

Otro enfoque es usar listas de palabras para intentar enumerar todos los atributos potenciales. La enumeración puede automatizarse (ej. vía wfuzz, Burp Intruder, ZAP fuzzer, etc.). La herramienta sqlmap incluye una lista de palabras [common-columns.txt](https://github.com/sqlmapproject/sqlmap/blob/master/data/txt/common-columns.txt) que puede ser útil para identificar atributos sensibles potenciales.
Un pequeño ejemplo de nombres de atributos comunes interesantes son los siguientes:

- `is_admin`
- `is_administrator`
- `isAdmin`
- `isAdministrator`
- `admin`
- `administrator`
- `role`

Cuando hay múltiples roles disponibles, intente comparar solicitudes realizadas por diferentes niveles de usuario (preste especial atención a roles privilegiados). Por ejemplo, si se incluyen parámetros extra en solicitudes realizadas por un usuario administrativo, intente esos como un usuario de bajo privilegio/anónimo.

#### Verificar impacto

El impacto de una asignación masiva puede variar dependiendo del contexto, por lo tanto, para cada entrada de prueba intentada en la fase anterior, analice el resultado y determine si representa una vulnerabilidad que tiene un impacto realista en la seguridad de la aplicación web.
Por ejemplo, la modificación del `id` de un objeto puede llevar a una denegación de servicio de la aplicación o escalada de privilegios. Otro ejemplo está relacionado con la posibilidad de modificar el rol/estado del usuario (ej. `role` o `isAdmin`) llevando a escalada de privilegios vertical.

### Pruebas de caja gris

Cuando el análisis se realiza con un enfoque de pruebas de caja gris, es posible seguir la misma metodología para verificar el problema. Sin embargo, el mayor conocimiento de la aplicación permite identificar más fácilmente marcos de trabajo y manejadores sujetos a vulnerabilidad de asignación masiva.
En particular, cuando el código fuente está disponible, es posible buscar los vectores de entrada más fácilmente y con precisión. Durante una revisión del código fuente, use herramientas simples (como el comando grep) para buscar uno o más patrones comunes dentro del código de la aplicación.
El acceso al esquema de la base de datos o al código fuente permite también identificar fácilmente campos sensibles.

#### Java

Spring MVC permite enlazar automáticamente la entrada del usuario en objetos. Identifique los controladores que manejan solicitudes que cambian el estado (ej. encuentre las ocurrencias de `@RequestMapping`) luego verifique si hay controles en su lugar (tanto en el controlador como en los modelos involucrados). Limitaciones en la explotación de la asignación masiva pueden estar, por ejemplo, en forma de:

- lista de campos enlazables vía método `setAllowedFields` de la clase `DataBinder` (ej. `binder.setAllowedFields(["username","password","email"])`)
- lista de campos no enlazables vía método `setDisallowedFields` de la clase `DataBinder` (ej. `binder.setDisallowedFields(["isAdmin"])`)

También es aconsejable prestar atención al uso de la anotación `@ModelAttribute` que permite especificar un nombre/clave diferente.

#### PHP

Laravel Eloquent ORM proporciona un método `create` que permite la asignación automática de atributos. Sin embargo, las últimas versiones de Eloquent ORM proporcionan protección predeterminada contra vulnerabilidades de asignación masiva requiriendo especificar explícitamente atributos permitidos que pueden asignarse automáticamente, a través del array `$fillable`, o atributos que deben protegerse (no enlazables), a través del array `$guarded`. Por lo tanto, al analizar los modelos (clases que extienden la clase `Model`) es posible identificar qué atributos están permitidos o denegados y por lo tanto señalar vulnerabilidades potenciales.

#### .NET

El enlace de modelos en ASP.NET enlaza automáticamente las entradas del usuario a propiedades de objetos. Esto también funciona con tipos complejos y convertirá automáticamente los datos de entrada a las propiedades si los nombres de las propiedades coinciden con la entrada.
Identifique los controladores luego verifique si hay controles en su lugar (tanto dentro del controlador como en los modelos involucrados). Limitaciones en la explotación de la asignación masiva pueden estar, por ejemplo, en forma de:

- campos declarados como `ReadOnly`
- lista de campos enlazables vía atributo `Bind` (ej. `[Bind(Include = "FirstName, LastName")] Student std`), vía `includeProperties` (ej. `includeProperties: new[] { "FirstName, LastName" }`) o a través de `TryUpdateModel`
- lista de campos no enlazables vía atributo `Bind` (ej. `[Bind(Exclude = "Status")] Student std`) o vía `excludeProperties` (ej. `excludeProperties: new[] { "Status" }`)

## Remediation

Use características integradas, proporcionadas por marcos de trabajo, para definir campos enlazables y no enlazables. Un enfoque basado en campos permitidos (enlazables), en el que solo las propiedades que deben actualizarse por el usuario se definen explícitamente, es preferible.
Un enfoque arquitectónico para prevenir el problema es usar el patrón *Data Transfer Object* (DTO) para evitar el enlace directo. El DTO debe incluir solo los campos que se pretenden editar por el usuario.

## Referencias

- [OWASP: API Security](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa6-mass-assignment.md)
- [OWASP: Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes](https://cwe.mitre.org/data/definitions/915.html)