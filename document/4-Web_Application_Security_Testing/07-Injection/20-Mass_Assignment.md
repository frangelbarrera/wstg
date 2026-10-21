# Asignación masiva

|ID          |
|------------|
|WSTG-INJT-20|

## Resumen

Las aplicaciones web modernas suelen basarse en marcos de trabajo. Muchos de estos marcos permiten enlazar automáticamente la entrada del usuario (en forma de parámetros de solicitudes HTTP) con objetos internos. A esto se le suele llamar enlace automático.
En ocasiones, esta característica puede explotarse para acceder a campos que nunca se pretendió que pudieran modificarse desde el exterior, lo que puede dar lugar a una escalada de privilegios, manipulación de datos, evasión de mecanismos de seguridad y otros problemas.
En este caso, existe una vulnerabilidad de asignación masiva.

Ejemplos de propiedades sensibles:

- **Propiedades relacionadas con permisos**: solo deben configurarlas los usuarios privilegiados (p. ej., `is_admin`, `role`, `approved`).
- **Propiedades dependientes del proceso**: solo deben configurarse internamente, una vez completado un proceso (p. ej., `balance`, `status`, `email_verified`).
- **Propiedades internas**: solo debe configurarlas internamente la aplicación (p. ej., `created_at`, `updated_at`).

## Objetivos de la prueba

- Identificar solicitudes que modifican objetos
- Evaluar si es posible modificar campos que nunca se pretendió que pudieran modificarse desde el exterior

## Cómo probar

El siguiente es un ejemplo clásico que puede ayudar a ilustrar el problema.

Suponga que existe una aplicación web Java con un objeto `User` similar al siguiente:

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

El controlador que gestiona la solicitud de creación (Spring proporciona el enlace automático con el modelo `User`):

```java
@RequestMapping(value = "/createUser", method = RequestMethod.POST)
public String createUser(User user) {
   userService.add(user);
   return "successPage";
}
```

Al enviar el formulario, el navegador genera la siguiente solicitud:

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test
```

Sin embargo, debido al enlace automático, un atacante puede agregar el parámetro `isAdmin` a la solicitud, y el controlador lo enlazará automáticamente con el modelo.

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test&isAdmin=true
```

El usuario se crea entonces con la propiedad `isAdmin` establecida en `true`, lo que le otorga derechos administrativos en la aplicación.

### Pruebas de caja negra

#### Detectar manejadores

Para determinar qué partes de la aplicación son vulnerables a la asignación masiva, enumere todas las partes que aceptan contenido del usuario y que potencialmente pueden asignarse a un modelo. Esto incluye todas las solicitudes HTTP (probablemente GET, POST y PUT) que parecen permitir operaciones de creación o actualización en el back end.
Uno de los indicadores más sencillos de una posible asignación masiva es la presencia de sintaxis de corchetes en los nombres de los parámetros de entrada, por ejemplo:

```html
<input name="user[name]" type="text">
```

Cuando encuentre estos patrones, intente agregar una entrada relacionada con un atributo inexistente (p. ej., `user[nonexistingattribute]`) y analice la respuesta o el comportamiento.
Si la aplicación no implementa ningún control (p. ej., una lista de campos permitidos), es probable que responda con un error (p. ej., 500) porque no encuentra el atributo asociado al objeto. Más interesante aún, estos errores a veces facilitan el descubrimiento de nombres de atributos y tipos de datos de valores necesarios para explotar el problema, sin acceso al código fuente.

#### Identificar campos sensibles

Dado que en las pruebas de caja negra el evaluador no tiene visibilidad del código fuente, es necesario encontrar otras formas de recopilar información sobre los atributos asociados a los objetos.
Analice las respuestas recibidas del back end y preste especial atención a:

- Código fuente de la página HTML
- Código JavaScript personalizado
- Respuestas de API

Por ejemplo, a menudo es posible explotar manejadores que devuelven detalles de un objeto para obtener indicios sobre los campos asociados.
Suponga, por ejemplo, que hay un manejador que devuelve el perfil del usuario (p. ej., `GET /profile`); este puede incluir atributos adicionales relacionados con el usuario (en este ejemplo, el atributo `isAdmin` parece particularmente interesante).

```json
{"_id":12345,"username":"bob","age":38,"email":"bob@domain.test","isAdmin":false}
```

A continuación, intente explotar manejadores que permitan modificar o crear usuarios y agregue el atributo `isAdmin` establecido en `true`.

Otro enfoque consiste en usar listas de palabras para intentar enumerar todos los atributos posibles. La enumeración puede automatizarse (p. ej., mediante wfuzz, Burp Intruder, ZAP fuzzer, etc.). La herramienta sqlmap incluye una lista de palabras [common-columns.txt](https://github.com/sqlmapproject/sqlmap/blob/master/data/txt/common-columns.txt) que puede ser útil para identificar posibles atributos sensibles.
Algunos ejemplos de nombres de atributos comunes e interesantes son:

- `is_admin`
- `is_administrator`
- `isAdmin`
- `isAdministrator`
- `admin`
- `administrator`
- `role`

Cuando haya varios roles disponibles, intente comparar las solicitudes realizadas por usuarios con distintos niveles de privilegios (preste especial atención a los roles privilegiados). Por ejemplo, si las solicitudes de un usuario administrador incluyen parámetros adicionales, intente usar esos parámetros como usuario con pocos privilegios o anónimo.

#### Verificar el impacto

El impacto de una asignación masiva puede variar según el contexto; por lo tanto, para cada entrada de prueba utilizada en la fase anterior, analice el resultado y determine si representa una vulnerabilidad con un impacto realista en la seguridad de la aplicación web.
Por ejemplo, modificar el `id` de un objeto puede provocar una denegación de servicio de la aplicación o una escalada de privilegios. Otro ejemplo es la posibilidad de modificar el rol o el estado del usuario (p. ej., `role` o `isAdmin`), lo que puede dar lugar a una escalada vertical de privilegios.

### Pruebas de caja gris

Cuando el análisis se realiza con un enfoque de pruebas de caja gris, es posible seguir la misma metodología para verificar el problema. Sin embargo, el mayor conocimiento de la aplicación permite identificar con más facilidad los marcos de trabajo y los manejadores vulnerables a la asignación masiva.
En particular, si se dispone del código fuente, es posible buscar los vectores de entrada con más facilidad y precisión. Durante una revisión del código fuente, utilice herramientas sencillas (como el comando grep) para buscar uno o varios patrones comunes en el código de la aplicación.
El acceso al esquema de la base de datos o al código fuente también permite identificar fácilmente los campos sensibles.

#### Java

Spring MVC permite enlazar automáticamente la entrada del usuario con los objetos. Identifique los controladores que gestionan solicitudes que cambian el estado (p. ej., busque las apariciones de `@RequestMapping`) y verifique si existen controles, tanto en el controlador como en los modelos involucrados. Algunas formas de limitar la explotación de la asignación masiva son:

- lista de campos enlazables mediante el método `setAllowedFields` de la clase `DataBinder` (p. ej., `binder.setAllowedFields(["username","password","email"])`)
- lista de campos no enlazables mediante el método `setDisallowedFields` de la clase `DataBinder` (p. ej., `binder.setDisallowedFields(["isAdmin"])`)

También se recomienda prestar atención al uso de la anotación `@ModelAttribute`, que permite especificar un nombre o una clave diferente.

#### PHP

Laravel Eloquent ORM proporciona un método `create` que permite la asignación automática de atributos. Sin embargo, las versiones más recientes de Eloquent ORM ofrecen protección predeterminada contra las vulnerabilidades de asignación masiva y requieren especificar explícitamente los atributos permitidos que pueden asignarse automáticamente mediante el arreglo `$fillable`, o los atributos que deben protegerse (no enlazables) mediante el arreglo `$guarded`. Por lo tanto, al analizar los modelos (clases que extienden la clase `Model`), es posible identificar qué atributos están permitidos o denegados y, por consiguiente, señalar posibles vulnerabilidades.

#### .NET

El enlace de modelos en ASP.NET enlaza automáticamente las entradas del usuario con las propiedades de los objetos. También funciona con tipos complejos y convierte automáticamente los datos de entrada para asignarlos a las propiedades cuando sus nombres coinciden con los de la entrada.
Identifique los controladores y verifique si existen controles, tanto dentro del controlador como en los modelos involucrados. Algunas formas de limitar la explotación de la asignación masiva son:

- campos declarados como `ReadOnly`
- lista de campos enlazables mediante el atributo `Bind` (p. ej., `[Bind(Include = "FirstName, LastName")] Student std`), mediante `includeProperties` (p. ej., `includeProperties: new[] { "FirstName, LastName" }`) o mediante `TryUpdateModel`
- lista de campos no enlazables mediante el atributo `Bind` (p. ej., `[Bind(Exclude = "Status")] Student std`) o mediante `excludeProperties` (p. ej., `excludeProperties: new[] { "Status" }`)

## Remediación

Utilice las características integradas que proporcionan los marcos de trabajo para definir los campos enlazables y no enlazables. Es preferible usar un enfoque basado en campos permitidos (enlazables), en el que se definan explícitamente solo las propiedades que el usuario debe poder actualizar.
Un enfoque arquitectónico para prevenir este problema consiste en usar el patrón *Data Transfer Object* (DTO) para evitar el enlace directo. El DTO debe incluir únicamente los campos que el usuario debe poder editar.

## Referencias

- [OWASP: API Security](https://github.com/OWASP/API-Security/blob/master/editions/2019/en/0xa6-mass-assignment.md)
- [OWASP: Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes](https://cwe.mitre.org/data/definitions/915.html)