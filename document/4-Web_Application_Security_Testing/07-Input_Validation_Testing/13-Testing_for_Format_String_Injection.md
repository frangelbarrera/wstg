# Pruebas de inyección de cadena de formato

|ID          |
|------------|
|WSTG-INPV-13|

## Resumen

Una cadena de formato es una secuencia de caracteres terminada en nulo que también contiene especificadores de conversión interpretados o convertidos en tiempo de ejecución. Si el código del lado del servidor [concatena la entrada de un usuario con una cadena de formato](https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/), un atacante puede añadir especificadores de conversión adicionales para causar un error en tiempo de ejecución, divulgación de información o desbordamiento de búfer.

Los peores casos para vulnerabilidades de cadenas de formato ocurren en lenguajes que no verifican argumentos e incluyen un especificador `%n` que escribe en memoria. Estas funciones, si son explotadas por un atacante modificando una cadena de formato, podrían causar [divulgación de información y ejecución de código](https://www.veracode.com/security/format-string):

- C y C++ [printf](https://en.cppreference.com/w/c/io/fprintf) y métodos similares fprintf, sprintf, snprintf
- Perl [printf](https://perldoc.perl.org/functions/printf.html) y sprintf

Estas funciones de cadena de formato no pueden escribir en memoria, pero los atacantes aún pueden causar divulgación de información cambiando las cadenas de formato para mostrar valores que los desarrolladores no pretendían enviar:

- Python 2.6 y 2.7 [str.format](https://docs.python.org/2/library/string.html) y Python 3 unicode [str.format](https://docs.python.org/3/library/stdtypes.html#str.format) pueden ser modificados inyectando cadenas que pueden apuntar a [otras variables](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/) en memoria

Las siguientes funciones de cadena de formato pueden causar errores en tiempo de ejecución si el atacante añade especificadores de conversión:

- Java [String.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format%28java.util.Locale%2Cjava.lang.String%2Cjava.lang.Object...%29) y [PrintStream.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/PrintStream.html#format%2528java.util.Locale%252Cjava.lang.String%252Cjava.lang.Object...%2529)
- PHP [printf](https://www.php.net/manual/es/function.printf.php)

El patrón de código que causa una vulnerabilidad de cadena de formato es una llamada a una función de formato de cadena que contiene entrada de usuario no saneada. El siguiente ejemplo muestra cómo un `printf` de depuración podría hacer vulnerable a un programa:

El ejemplo en C:

```c
char *userName = /* input from user controlled field */;

printf("DEBUG Current user: ");
// Vulnerable debugging code
printf(userName);
```

El ejemplo en Java:

```java
final String userName = /* input from user controlled field */;

System.out.printf("DEBUG Current user: ");
// Vulnerable code:
System.out.printf(userName);
```

En este ejemplo particular, si el atacante establece su `userName` para tener uno o más especificadores de conversión, habría un comportamiento no deseado. El ejemplo en C [imprimiría contenidos de memoria](https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf) si `userName` contenía `%p%p%p%p%p`, y puede corromper contenidos de memoria si hay un `%n` en la cadena. En el ejemplo en Java, un `username` conteniendo cualquier especificador que necesite una entrada (incluyendo `%x` o `%s`) haría que el programa se bloquee con `IllegalFormatException`. Aunque los ejemplos están sujetos a otros problemas, la vulnerabilidad puede ser corregida con argumentos printf de `printf("DEBUG Current user: %s", userName)`.

## Objetivos de la prueba

- Evaluar si inyectar especificadores de conversión de cadena de formato en campos controlados por el usuario causa un comportamiento no deseado de la aplicación.

## Cómo probar

Las pruebas incluyen análisis del código e inyección de especificadores de conversión como entrada de usuario a la aplicación bajo prueba.

### Análisis estático

Las herramientas de análisis estático pueden encontrar vulnerabilidades de cadena de formato en el código o en binarios. Ejemplos de herramientas incluyen:

- C y C++: [Flawfinder](https://dwheeler.com/flawfinder/)
- Java: Regla FindSecurityBugs [FORMAT_STRING_MANIPULATION](https://find-sec-bugs.github.io/bugs.htm#FORMAT_STRING_MANIPULATION)
- PHP: Analizador de formateador de cadena en [phpsa](https://github.com/ovr/phpsa/blob/master/docs/05_Analyzers.md#function_string_formater)

### Inspección manual del código

El análisis estático puede perder casos más sutiles incluyendo cadenas de formato generadas por código complejo. Para buscar vulnerabilidades manualmente en una base de código, un probador puede buscar todas las llamadas en la base de código que aceptan una cadena de formato y rastrear hacia atrás para asegurar que la entrada no confiable no puede cambiar la cadena de formato.

### Inyección de especificador de conversión

Los probadores pueden verificar a nivel de prueba unitaria o de sistema completo enviando especificadores de conversión en cualquier entrada de cadena. [Fuzzea](https://owasp.org/www-community/Fuzzing) el programa usando todos los especificadores de conversión para todos los lenguajes que usa el sistema bajo prueba. Vea la página [Ataque de cadena de formato OWASP](https://owasp.org/www-community/attacks/Format_string_attack) para posibles entradas a usar. Si la prueba falla, el programa se bloqueará o mostrará una salida inesperada. Si la prueba pasa, el intento de enviar un especificador de conversión debería ser bloqueado, o la cadena debería pasar por el sistema sin problemas como con cualquier otra entrada válida.

Los ejemplos en las siguientes subsecciones tienen una URL de esta forma:

`https://vulnerable_host/userinfo?username=x`

- El valor controlado por el usuario es `x` (para el parámetro `username`).

#### Inyección manual

Los probadores pueden realizar una prueba manual usando un navegador web u otras herramientas de depuración de API web. Navegue a la aplicación web o sitio de tal manera que la consulta tenga especificadores de conversión. Note que la mayoría de los especificadores de conversión necesitan [codificación](https://tools.ietf.org/html/rfc3986#section-2.1) si se envían dentro de una URL porque contienen caracteres especiales incluyendo `%` y `{`. La prueba puede introducir una cadena de especificadores `%s%s%s%n` navegando con la siguiente URL:

`https://vulnerable_host/userinfo?username=%25s%25s%25s%25n`

Si el sitio web es vulnerable, el navegador o herramienta debería recibir un error, que puede incluir un tiempo de espera o un código de retorno HTTP 500.

El código Java devuelve el error

`java.util.MissingFormatArgumentException: Format specifier '%s'`

Dependiendo de la implementación en C, el proceso puede bloquearse completamente con `Segmentation Fault`.

#### Fuzzing asistido por herramienta

Herramientas de fuzzing incluyendo [wfuzz](https://github.com/xmendez/wfuzz) pueden automatizar pruebas de inyección. Para wfuzz, comience con un archivo de texto (fuzz.txt en este ejemplo) con una entrada por línea:

fuzz.txt:

```text
alice
%s%s%s%n
%p%p%p%p%p
{event.__init__.__globals__[CONFIG][SECRET_KEY]}
```

El archivo `fuzz.txt` contiene lo siguiente:

- Una entrada válida `alice` para verificar que la aplicación puede procesar una entrada normal
- Dos cadenas con especificadores de conversión tipo C
- Un especificador de conversión Python para intentar leer variables globales

Para enviar el archivo de entrada de fuzzing a la aplicación web bajo prueba, use el siguiente comando:

`wfuzz -c -z file,fuzz.txt,urlencode https://vulnerable_host/userinfo?username=FUZZ`

En la llamada anterior, el argumento `urlencode` habilita el escape apropiado para las cadenas y `FUZZ` (con letras mayúsculas) le dice a la herramienta dónde introducir las entradas.

Un ejemplo de salida es el siguiente

```text
ID           Response   Lines    Word     Chars       Payload
=======================================================================

000000002:   500        0 L      5 W      142 Ch      "%25s%25s%25s%25n"
000000003:   500        0 L      5 W      137 Ch      "%25p%25p%25p%25p%25p"
000000004:   200        0 L      1 W      48 Ch       "%7Bevent.__init__.__globals__%5BCONFIG%5D%5BSECRET_KEY%5D%7D"
000000001:   200        0 L      1 W      5 Ch        "alice"
```

El resultado anterior valida la debilidad de la aplicación a la inyección de especificadores de conversión tipo C `%s` y `%p`.