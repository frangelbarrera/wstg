# Pruebas de referencias directas a objetos inseguras

|ID          |
|------------|
|WSTG-ATHZ-04|

## Summary

Las referencias directas a objetos inseguras (IDOR) ocurren cuando una aplicación proporciona acceso directo a objetos basado en la entrada proporcionada por el usuario. Como resultado de esta vulnerabilidad, los atacantes pueden eludir la autorización y acceder directamente a recursos en el sistema, por ejemplo, registros de base de datos o archivos.

Las referencias directas a objetos inseguras permiten a los atacantes eludir la autorización y acceder directamente a recursos modificando el valor de un parámetro utilizado para apuntar directamente a un objeto. Tales recursos pueden ser entradas de base de datos pertenecientes a otros usuarios, archivos en el sistema, y más. Esto se debe a que la aplicación toma la entrada proporcionada por el usuario y la utiliza para recuperar un objeto sin realizar comprobaciones de autorización suficientes.

## Test Objectives

- Identificar puntos donde pueden ocurrir referencias a objetos.
- Evaluar las medidas de control de acceso y si son vulnerables a IDOR.

## How to Test

Para probar esta vulnerabilidad, el evaluador primero necesita mapear todas las ubicaciones en la aplicación donde la entrada del usuario se utiliza para referenciar objetos directamente. Por ejemplo, ubicaciones donde la entrada del usuario se utiliza para acceder a una fila de base de datos, un archivo, páginas de aplicación y más. A continuación, el evaluador debe modificar el valor del parámetro utilizado para referenciar objetos y evaluar si es posible recuperar objetos pertenecientes a otros usuarios o eludir la autorización de otra manera.

La mejor manera de probar referencias directas a objetos sería tener al menos dos (a menudo más) usuarios para cubrir diferentes objetos propiedad y funciones. Por ejemplo, dos usuarios cada uno con acceso a diferentes objetos (como información de compra, mensajes privados, etc.), y (si es relevante) usuarios con diferentes privilegios (por ejemplo, usuarios administradores) para ver si hay referencias directas a la funcionalidad de la aplicación. Al tener múltiples usuarios, el evaluador ahorra tiempo valioso de prueba al adivinar diferentes nombres de objetos, ya que puede intentar acceder a objetos que pertenecen al otro usuario.

A continuación se muestran varios escenarios típicos para esta vulnerabilidad y los métodos para probar cada uno:

### El valor de un parámetro se utiliza directamente para recuperar un registro de base de datos

Solicitud de ejemplo:

```text
https://foo.bar/somepage?invoice=12345
```

En este caso, el valor del parámetro *invoice* se utiliza como índice en una tabla de facturas en la base de datos. La aplicación toma el valor de este parámetro y lo utiliza en una consulta a la base de datos. La aplicación luego devuelve la información de la factura al usuario.

Dado que el valor de *invoice* va directamente a la consulta, modificando el valor del parámetro es posible recuperar cualquier objeto de factura, independientemente del usuario al que pertenece la factura. Para probar este caso, el evaluador debe obtener el identificador de una factura perteneciente a un usuario de prueba diferente (asegurándose de que no se supone que vea esta información según la lógica de negocio de la aplicación), y luego verificar si es posible acceder a objetos sin autorización.

### El valor de un parámetro se utiliza directamente para realizar una operación en el sistema

Solicitud de ejemplo:

```text
https://foo.bar/changepassword?user=someuser
```

En este caso, el valor del parámetro `user` se utiliza para indicar a la aplicación para qué usuario debe cambiar la contraseña. En muchos casos, este paso será parte de un asistente o una operación de múltiples pasos. En el primer paso, la aplicación obtendrá una solicitud indicando para qué contraseña de usuario se va a cambiar, y en el siguiente paso el usuario proporcionará una nueva contraseña (sin pedir la actual).

El parámetro `user` se utiliza para referenciar directamente el objeto del usuario para el cual se realizará la operación de cambio de contraseña. Para probar este caso, el evaluador debe intentar proporcionar un nombre de usuario de prueba diferente al que está actualmente conectado, y verificar si es posible modificar la contraseña de otro usuario.

### El valor de un parámetro se utiliza directamente para recuperar un recurso del sistema de archivos

Solicitud de ejemplo:

```text
https://foo.bar/showImage?img=img00011
```

En este caso, el valor del parámetro `file` se utiliza para indicar a la aplicación qué archivo el usuario intenta recuperar. Al proporcionar el nombre o identificador de un archivo diferente (por ejemplo, file=image00012.jpg), el atacante podrá recuperar objetos pertenecientes a otros usuarios.

Para probar este caso, el evaluador debe obtener una referencia que el usuario no se supone que pueda acceder y intentar acceder a ella utilizándola como el valor del parámetro `file`. Nota: Esta vulnerabilidad a menudo se explota en conjunto con una vulnerabilidad de traversal de directorio/ruta (ver [Pruebas de traversal de directorio/inclusión de archivos](01-Testing_Directory_Traversal_File_Include-es.md))

### El valor de un parámetro se utiliza directamente para acceder a la funcionalidad de la aplicación

Solicitud de ejemplo:

```text
https://foo.bar/accessPage?menuitem=12
```

En este caso, el valor del parámetro `menuitem` se utiliza para indicar a la aplicación a qué elemento de menú (y por lo tanto qué funcionalidad de aplicación) el usuario está intentando acceder. Suponga que el usuario se supone que está restringido y por lo tanto tiene enlaces disponibles solo para acceder a elementos de menú 1, 2 y 3. Modificando el valor del parámetro `menuitem` es posible eludir la autorización y acceder a funcionalidad adicional de la aplicación. Para probar este caso, el evaluador identifica una ubicación donde la funcionalidad de la aplicación se determina por referencia a un elemento de menú, mapea los valores de elementos de menú a los que el usuario de prueba dado puede acceder, y luego intenta otros elementos de menú.

En los ejemplos anteriores, la modificación de un solo parámetro es suficiente. Sin embargo, a veces la referencia al objeto puede estar dividida entre más de un parámetro, y las pruebas deben ajustarse en consecuencia.

## References

[Top 10 2013-A4-Referencias directas a objetos inseguras](https://owasp.org/www-project-top-ten/2017/Release_Notes)