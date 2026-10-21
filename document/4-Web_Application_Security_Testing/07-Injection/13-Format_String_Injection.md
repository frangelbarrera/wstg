# Inyección de cadena de formato

|ID          |
|------------|
|WSTG-INJT-13|

## Resumen

Una cadena de formato es una secuencia de caracteres terminada en nulo que también contiene especificadores de conversión que se interpretan o convierten en tiempo de ejecución. Si el código del lado del servidor [concatena la entrada de un usuario con una cadena de formato](https://www.invicti.com/blog/web-security/string-concatenation-format-string-vulnerabilities), un atacante puede añadir especificadores de conversión adicionales para provocar un error en tiempo de ejecución, una divulgación de información o un desbordamiento de búfer.

Los casos más graves de vulnerabilidades de cadenas de formato se producen en lenguajes que no comprueban los argumentos y que también incluyen un especificador `%n` que escribe en memoria. Si un atacante explota estas funciones modificando una cadena de formato, podría provocar [divulgación de información y ejecución de código](https://en.wikipedia.org/wiki/Uncontrolled_format_string):

- C y C++ [printf](https://en.cppreference.com/w/c/io/fprintf) y métodos similares, como fprintf, sprintf y snprintf
- Perl [printf](https://perldoc.perl.org/functions/printf.html) y sprintf

Estas funciones de cadenas de formato no pueden escribir en memoria, pero los atacantes aún pueden provocar una divulgación de información modificando las cadenas de formato para mostrar valores que los desarrolladores no pretendían enviar:

- Python [str.format](https://docs.python.org/3/library/stdtypes.html#str.format) puede explotarse mediante la inyección de cadenas de formato que accedan a [otras variables](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/) en el contexto de formato

Las siguientes funciones de cadenas de formato pueden provocar errores en tiempo de ejecución si el atacante añade especificadores de conversión:

- Java [String.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format%28java.util.Locale%2Cjava.lang.String%2Cjava.lang.Object...%29) y [PrintStream.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/PrintStream.html#format%2528java.util.Locale%252Cjava.lang.String%252Cjava.lang.Object...%2529)
- PHP [printf](https://www.php.net/manual/es/function.printf.php)

El patrón de código que provoca una vulnerabilidad de cadena de formato es una llamada a una función de formato de cadenas que contiene entrada de usuario no saneada. El siguiente ejemplo muestra cómo un `printf` de depuración podría hacer que un programa fuera vulnerable:

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

En este ejemplo concreto, si el atacante asignara a `userName` uno o más especificadores de conversión, se produciría un comportamiento no deseado. El ejemplo en C [imprimiría el contenido de la memoria](https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf) si `userName` contuviera `%p%p%p%p%p`, y podría corromper el contenido de la memoria si la cadena incluyera `%n`. En el ejemplo en Java, un `username` que contuviera cualquier especificador que requiera una entrada (incluidos `%x` o `%s`) haría que el programa se bloqueara con `IllegalFormatException`. Aunque los ejemplos también pueden presentar otros problemas, la vulnerabilidad puede corregirse usando argumentos de printf: `printf("DEBUG Current user: %s", userName)`.

## Objetivos de la prueba

- Evaluar si la inyección de especificadores de conversión de cadenas de formato en campos controlados por el usuario provoca un comportamiento no deseado de la aplicación.

## Cómo probar

Las pruebas incluyen el análisis del código y la inyección de especificadores de conversión como entrada de usuario en la aplicación bajo prueba.

### Análisis estático

Las herramientas de análisis estático pueden detectar vulnerabilidades de cadenas de formato en el código o en archivos binarios. Algunos ejemplos de herramientas son:

- C y C++: [Flawfinder](https://dwheeler.com/flawfinder/)
- Java: regla de FindSecurityBugs [FORMAT_STRING_MANIPULATION](https://find-sec-bugs.github.io/bugs.htm#FORMAT_STRING_MANIPULATION)
- PHP: analizador de formateadores de cadenas de [phpsa](https://github.com/ovr/phpsa/blob/master/docs/05_Analyzers.md#function_string_formater)

### Inspección manual del código

El análisis estático puede pasar por alto casos más sutiles, como las cadenas de formato generadas por código complejo. Para buscar vulnerabilidades manualmente en una base de código, un evaluador puede buscar todas las llamadas de la base de código que aceptan una cadena de formato y rastrear su origen para asegurarse de que la entrada no confiable no pueda modificarla.

### Inyección de especificadores de conversión

Los evaluadores pueden comprobarlo a nivel de pruebas unitarias o de pruebas del sistema completo mediante el envío de especificadores de conversión en cualquier entrada de tipo cadena. Aplique fuzzing al programa con todos los especificadores de conversión de los lenguajes que use el sistema bajo prueba. Consulte la página [Ataque de cadena de formato de OWASP](https://owasp.org/www-community/attacks/Format_string_attack) para ver posibles entradas. Si la prueba falla, el programa se bloqueará o mostrará una salida inesperada. Si la prueba pasa, se debería bloquear el intento de enviar un especificador de conversión o la cadena debería atravesar el sistema sin problemas, como cualquier otra entrada válida.

Los ejemplos de las subsecciones siguientes usan una URL con este formato:

`https://vulnerable_host/userinfo?username=x`

- El valor controlado por el usuario es `x` (para el parámetro `username`).

#### Inyección manual

Los evaluadores pueden realizar una prueba manual con un navegador web u otras herramientas de depuración de API web. Acceda a la aplicación o al sitio web usando una consulta que incluya especificadores de conversión. Tenga en cuenta que la mayoría de los especificadores de conversión necesitan [codificarse](https://tools.ietf.org/html/rfc3986#section-2.1) si se envían en una URL, porque contienen caracteres especiales como `%` y `{`. La prueba puede introducir una cadena de especificadores `%s%s%s%n` accediendo a la siguiente URL:

`https://vulnerable_host/userinfo?username=%25s%25s%25s%25n`

Si el sitio web es vulnerable, el navegador o la herramienta deberían recibir un error, que puede incluir un tiempo de espera o un código de respuesta HTTP 500.

El código Java devuelve el error:

`java.util.MissingFormatArgumentException: Format specifier '%s'`

Según la implementación de C, el proceso podría bloquearse por completo con `Segmentation Fault`.

#### Fuzzing asistido por herramientas

Las herramientas de fuzzing, como [wfuzz](https://github.com/xmendez/wfuzz), pueden automatizar las pruebas de inyección. Para wfuzz, empiece con un archivo de texto (fuzz.txt en este ejemplo) que contenga una entrada por línea:

fuzz.txt:

```text
alice
%s%s%s%n
%p%p%p%p%p
{event.__init__.__globals__[CONFIG][SECRET_KEY]}
```

El archivo `fuzz.txt` contiene lo siguiente:

- Una entrada válida `alice` para verificar que la aplicación puede procesar una entrada normal
- Dos cadenas con especificadores de conversión similares a los de C
- Un especificador de conversión de Python para intentar leer variables globales

Para enviar el archivo de entradas de fuzzing a la aplicación web bajo prueba, use el siguiente comando:

`wfuzz -c -z file,fuzz.txt,urlencode https://vulnerable_host/userinfo?username=FUZZ`

En la llamada anterior, el argumento `urlencode` habilita el escape adecuado de las cadenas y `FUZZ` (en mayúsculas) indica a la herramienta dónde introducir las entradas.

Un ejemplo de salida es el siguiente:

```text
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000002:   500        0 L      5 W      142 Ch      "%25s%25s%25s%25n"
000000003:   500        0 L      5 W      137 Ch      "%25p%25p%25p%25p%25p"
000000004:   200        0 L      1 W      48 Ch       "%7Bevent.__init__.__globals__%5BCONFIG%5D%5BSECRET_KEY%5D%7D"
000000001:   200        0 L      1 W      5 Ch        "alice"
```

El resultado anterior confirma que la aplicación es vulnerable a la inyección de especificadores de conversión similares a los de C, como `%s` y `%p`.