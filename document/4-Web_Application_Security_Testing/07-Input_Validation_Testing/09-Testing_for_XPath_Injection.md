# Pruebas de inyección XPath

|ID          |
|------------|
|WSTG-INPV-09|

## Resumen

XPath es un lenguaje diseñado y desarrollado principalmente para abordar partes de un documento XML. En las pruebas de inyección XPath, se evalúa si es posible inyectar sintaxis XPath en una solicitud interpretada por la aplicación, permitiendo a un atacante ejecutar consultas XPath controladas por el usuario. Cuando se explota con éxito, esta vulnerabilidad puede permitir a un atacante eludir mecanismos de autenticación o acceder a información sin la autorización adecuada.

Las aplicaciones web utilizan ampliamente bases de datos para almacenar y acceder a los datos que necesitan para sus operaciones. Históricamente, las bases de datos relacionales han sido, con mucho, la tecnología más común para el almacenamiento de datos, pero en los últimos años estamos presenciando una creciente popularidad de las bases de datos que organizan los datos utilizando el lenguaje XML. Al igual que las bases de datos relacionales se acceden mediante el lenguaje SQL, las bases de datos XML utilizan XPath como su lenguaje de consulta estándar.

Dado que, desde un punto de vista conceptual, XPath es muy similar a SQL en su propósito y aplicaciones, un resultado interesante es que los ataques de inyección XPath siguen la misma lógica que los ataques de [inyección SQL](https://owasp.org/www-community/attacks/SQL_Injection). En algunos aspectos, XPath es incluso más potente que el SQL estándar, ya que toda su potencia ya está presente en sus especificaciones, mientras que un gran número de las técnicas que se pueden utilizar en un ataque de inyección SQL dependen de las características del dialecto SQL utilizado por la base de datos objetivo. Esto significa que los ataques de inyección XPath pueden ser mucho más adaptables y ubicuos. Otra ventaja de un ataque de inyección XPath es que, a diferencia de SQL, no se aplican ACL, ya que nuestra consulta puede acceder a cualquier parte del documento XML.

## Objetivos de la prueba

- Identificar puntos de inyección XPath.

## Cómo probar

El [patrón de ataque XPath fue publicado por primera vez por Amit Klein](https://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf) y es muy similar a la inyección SQL habitual. Para obtener una primera comprensión del problema, imaginemos una página de inicio de sesión que gestiona la autenticación en una aplicación en la que el usuario debe ingresar su nombre de usuario y contraseña. Supongamos que nuestra base de datos está representada por el siguiente archivo XML:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <account>admin</account>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <account>guest</account>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <account>guest</account>
    </user>
</users>
```

Una consulta XPath que devuelve la cuenta cuyo nombre de usuario es `gandalf` y la contraseña es `!c3` sería la siguiente:

`string(//user[username/text()='gandalf' and password/text()='!c3']/account/text())`

Si la aplicación no filtra correctamente la entrada del usuario, el evaluador podrá inyectar código XPath e interferir con el resultado de la consulta. Por ejemplo, el evaluador podría ingresar los siguientes valores:

```text
Username: ' or '1' = '1
Password: ' or '1' = '1
```

¿Parece familiar, verdad? Utilizando estos parámetros, la consulta se convierte en:

`string(//user[username/text()='' or '1' = '1' and password/text()='' or '1' = '1']/account/text())`

Al igual que en un ataque común de inyección SQL, hemos creado una consulta que siempre se evalúa como verdadera, lo que significa que la aplicación autenticará al usuario incluso si no se ha proporcionado un nombre de usuario o una contraseña. Y al igual que en un ataque común de inyección SQL, con la inyección XPath, el primer paso es insertar una comilla simple (`'`) en el campo a probar, introduciendo un error de sintaxis en la consulta, y verificar si la aplicación devuelve un mensaje de error.

Si no hay conocimiento sobre los detalles internos de los datos XML y si la aplicación no proporciona mensajes de error útiles que nos ayuden a reconstruir su lógica interna, es posible realizar un ataque de [inyección XPath ciega](https://owasp.org/www-community/attacks/Blind_XPath_Injection), cuyo objetivo es reconstruir toda la estructura de datos. La técnica es similar a la inyección SQL basada en inferencia, ya que el enfoque es inyectar código que crea una consulta que devuelve un bit de información. La [inyección XPath ciega](https://owasp.org/www-community/attacks/Blind_XPath_Injection) se explica con más detalle por Amit Klein en el documento referenciado.

## Referencias

### Documentos técnicos

- [Amit Klein: "Blind XPath Injection"](https://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf)
- [Especificaciones XPath 1.0](https://www.w3.org/TR/1999/REC-xpath-19991116/)