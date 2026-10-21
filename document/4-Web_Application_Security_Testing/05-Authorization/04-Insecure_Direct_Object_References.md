# Referencias directas a objetos inseguras

|ID          |
|------------|
|WSTG-ATHZ-04|

## Resumen

Las referencias directas a objetos inseguras (IDOR) ocurren cuando una aplicación proporciona acceso directo a objetos en función de datos proporcionados por el usuario. Como resultado de esta vulnerabilidad, los atacantes pueden eludir la autorización y acceder directamente a recursos del sistema, como registros de bases de datos o archivos.

Las referencias directas a objetos inseguras permiten a los atacantes eludir la autorización y acceder directamente a recursos modificando el valor de un parámetro que apunta directamente a un objeto. Estos recursos pueden ser registros de bases de datos pertenecientes a otros usuarios, archivos del sistema y más. Esto se debe a que la aplicación toma datos proporcionados por el usuario y los utiliza para recuperar un objeto sin realizar suficientes comprobaciones de autorización.

## Objetivos de las pruebas

- Identificar puntos en los que puede haber referencias a objetos.
- Evaluar las medidas de control de acceso y determinar si son vulnerables a IDOR.

## Cómo probar

Para probar esta vulnerabilidad, el evaluador primero debe identificar todas las ubicaciones de la aplicación donde se utilizan datos del usuario para referenciar objetos directamente. Por ejemplo, ubicaciones donde se utilizan datos del usuario para acceder a una fila de la base de datos, un archivo, páginas de la aplicación y más. A continuación, el evaluador debe modificar el valor del parámetro utilizado para referenciar objetos y comprobar si es posible recuperar objetos pertenecientes a otros usuarios o eludir la autorización de otra manera.

La mejor manera de probar las referencias directas a objetos es contar con al menos dos usuarios (a menudo, más) para abarcar distintos objetos y funciones de su propiedad. Por ejemplo, dos usuarios, cada uno con acceso a objetos diferentes (como información de compras, mensajes privados, etc.) y, si corresponde, usuarios con distintos privilegios (por ejemplo, usuarios administradores), para comprobar si hay referencias directas a funciones de la aplicación. Al contar con varios usuarios, el evaluador ahorra tiempo valioso de prueba, ya que no necesita adivinar distintos nombres de objetos y puede intentar acceder a objetos que pertenecen al otro usuario.

A continuación se presentan varios escenarios típicos de esta vulnerabilidad y los métodos para probar cada uno:

### El valor de un parámetro se utiliza directamente para recuperar un registro de la base de datos

Solicitud de ejemplo:

```text
https://foo.bar/somepage?invoice=12345
```

En este caso, el valor del parámetro *invoice* se utiliza como índice en una tabla de facturas de la base de datos. La aplicación toma el valor de este parámetro y lo utiliza en una consulta a la base de datos. Luego, la aplicación devuelve la información de la factura al usuario.

Dado que el valor de *invoice* se incorpora directamente a la consulta, es posible recuperar cualquier objeto de factura modificando el valor del parámetro, independientemente del usuario al que pertenezca la factura. Para probar este caso, el evaluador debe obtener el identificador de una factura perteneciente a otro usuario de prueba (asegurándose de que, según la lógica de negocio de la aplicación, no debería poder ver esta información) y, a continuación, comprobar si es posible acceder a objetos sin autorización.

### El valor de un parámetro se utiliza directamente para realizar una operación en el sistema

Solicitud de ejemplo:

```text
https://foo.bar/changepassword?user=someuser
```

En este caso, el valor del parámetro `user` indica a la aplicación para qué usuario debe cambiar la contraseña. En muchos casos, este paso formará parte de un asistente o de una operación de varios pasos. En el primer paso, la aplicación recibirá una solicitud que indica la contraseña de qué usuario se va a cambiar y, en el siguiente paso, el usuario proporcionará una contraseña nueva (sin que se le solicite la actual).

El parámetro `user` se utiliza para referenciar directamente el objeto del usuario para el que se realizará la operación de cambio de contraseña. Para probar este caso, el evaluador debe intentar proporcionar un nombre de usuario de prueba distinto del que tiene la sesión iniciada y comprobar si es posible modificar la contraseña de otro usuario.

### El valor de un parámetro se utiliza directamente para recuperar un recurso del sistema de archivos

Solicitud de ejemplo:

```text
https://foo.bar/showImage?img=img00011
```

En este caso, el valor del parámetro `file` indica a la aplicación qué archivo pretende recuperar el usuario. Al proporcionar el nombre o el identificador de otro archivo (por ejemplo, file=image00012.jpg), el atacante podrá recuperar objetos pertenecientes a otros usuarios.

Para probar este caso, el evaluador debe obtener una referencia a la que el usuario no debería poder acceder e intentar acceder a ella utilizándola como valor del parámetro `file`. Nota: Esta vulnerabilidad suele explotarse junto con una vulnerabilidad de recorrido de directorios/rutas (véase [Recorrido de directorios](01-Directory_Traversal_File_Include.md)).

### El valor de un parámetro se utiliza directamente para acceder a funciones de la aplicación

Solicitud de ejemplo:

```text
https://foo.bar/accessPage?menuitem=12
```

En este caso, el valor del parámetro `menuitem` indica a la aplicación a qué elemento del menú (y, por lo tanto, a qué función de la aplicación) intenta acceder el usuario. Supongamos que el usuario debe tener acceso restringido y, por lo tanto, solo tiene enlaces disponibles para acceder a los elementos del menú 1, 2 y 3. Al modificar el valor del parámetro `menuitem`, es posible eludir la autorización y acceder a otras funciones de la aplicación. Para probar este caso, el evaluador identifica una ubicación donde las funciones de la aplicación se determinan mediante una referencia a un elemento del menú, registra los valores de los elementos del menú a los que puede acceder el usuario de prueba y, luego, prueba otros elementos del menú.

En los ejemplos anteriores, basta con modificar un solo parámetro. Sin embargo, a veces la referencia al objeto puede estar dividida entre varios parámetros, por lo que las pruebas deben adaptarse en consecuencia.

## Referencias

[A01:2025 Control de acceso roto](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)