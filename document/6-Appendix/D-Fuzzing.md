# D. Fuzzing {#fuzzing}

## Introducción

El fuzzing es el proceso o la técnica de enviar varias solicitudes a un sitio objetivo durante un intervalo de tiempo determinado. En otras palabras, también es similar al ataque de fuerza bruta. El fuzzing es un proceso que puede llevarse a cabo con herramientas como Wfuzz, ffuf, entre otras. Como evaluador, debes proporcionar a la herramienta la URL objetivo, el parámetro, el endpoint, etc., y algún tipo de entradas. Luego, la herramienta de fuzzing crea solicitudes y las envía al objetivo. Una vez finalizado el fuzzing, es necesario analizar las respuestas, los tiempos, los códigos de estado y otras características para detectar posibles vulnerabilidades.

## ¿Por qué fuzzing?

Probar vulnerabilidades introduciendo entradas manualmente, una por una, puede ser caótico. En la actualidad, cuando las personas tienen menos tiempo y poca paciencia, la idea de introducir entradas manualmente para encontrar errores o vulnerabilidades puede resultar abrumadora. Para evitarlo y ahorrar tiempo, el fuzzing puede ser de gran ayuda. El fuzzing es un proceso automatizado en el que una herramienta de fuzzing se encarga de gran parte del trabajo. Lo único que tiene que hacer un analista es analizar las distintas características una vez finalizado el proceso. Considera un sitio con muchos campos de entrada que se deben probar para detectar XSS. En un enfoque manual, introducimos payloads de XSS en el campo de entrada, uno por uno, lo cual sería muy tedioso. En cambio, para un enfoque automatizado, solo tienes que proporcionar al fuzzer la lista de payloads de XSS; el fuzzer se encarga de todas las solicitudes.

## Herramientas para fuzzing

Hay cientos de herramientas disponibles en el sector para hacer fuzzing. A continuación, se enumeran algunas de las herramientas de fuzzing más populares y mejor valoradas.

### Wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz) funciona sustituyendo por valores de la lista de palabras el lugar donde aparece el marcador de posición `FUZZ`. Para entenderlo mejor, veamos un ejemplo:

```bash
wfuzz -w userIDs.txt https://example.com/view_photo?userId=FUZZ
```

En el comando anterior, `userIDs.txt` es un archivo de lista de palabras que contiene valores de ID numéricos. Aquí, indicamos a wfuzz que haga fuzzing de la solicitud a la URL de ejemplo. La palabra `FUZZ` en la URL actúa como marcador de posición que wfuzz sustituye por valores de la lista de palabras. Todos los valores de ID numéricos del archivo `userIDs.txt` se insertan en lugar de la palabra clave `FUZZ`.

### Ffuf

[Ffuf](https://github.com/ffuf/ffuf) es una herramienta de fuzzing web escrita en Go, muy rápida y de naturaleza recursiva. Funciona de forma similar a Wfuzz, pero, a diferencia de este, es recursiva. Ffuf también funciona sustituyendo el marcador de posición `FUZZ` por valores de la lista de palabras. Por ejemplo:

```bash
ffuf -w userIDs.txt -u https://example.com/view_photo?userId=FUZZ
```

Aquí, `-w` es la opción para la lista de palabras y `-u` es la opción para la URL objetivo. El resto del mecanismo de funcionamiento es igual que en Wfuzz. Sustituye el marcador de posición `FUZZ` por los valores de `userIDs.txt`.

### GoBuster

[GoBuster](https://github.com/OJ/gobuster) es otro fuzzer escrito en Go, que se utiliza principalmente para hacer fuzzing de URI, directorios/rutas, subdominios DNS, buckets de AWS S3 y nombres de vhost, y admite la concurrencia. Por ejemplo:

```bash
gobuster dir -w endpoints.txt -u https://example.com
```

En el comando anterior, `dir` especifica que se hará fuzzing de un directorio, `-u` es la opción para la URL y `-w` es la opción para la lista de palabras, de la que se tomarán los payloads del archivo `endpoints.txt`. El comando ejecuta solicitudes concurrentes al endpoint para encontrar directorios disponibles.

### ZAP

[ZAP](https://www.zaproxy.org) es un escáner de seguridad de aplicaciones web que se puede usar para encontrar vulnerabilidades y debilidades en aplicaciones web. También incluye un [fuzzer](https://www.zaproxy.org/docs/desktop/addons/fuzzer/).

Una de las características clave de ZAP es su capacidad para realizar análisis pasivos y activos. Los análisis pasivos consisten en observar el tráfico entre el usuario y la aplicación web, mientras que los análisis activos consisten en enviar payloads de prueba a la aplicación web para identificar vulnerabilidades.

### Listas de palabras y referencias

En los ejemplos anteriores, hemos visto por qué necesitamos una lista de palabras. Sin embargo, no basta con tener una lista de palabras: esta debe ser adecuada para el escenario de fuzzing. Si no encuentras listas de palabras que se ajusten al escenario necesario, considera generar tu propia lista. A continuación, se proporcionan algunas listas de palabras y referencias populares.

- [Hoja de referencia de cross-site scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [AwesomeXSS](https://github.com/s0md3v/AwesomeXSS)
- [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [Big List of Naughty Strings](https://github.com/minimaxir/big-list-of-naughty-strings)
- [Bo0oM Fuzz List](https://github.com/Bo0oM/fuzz.txt)
- [FuzzDB](https://github.com/fuzzdb-project/fuzzdb)
- [bl4de Dictionaries](https://github.com/bl4de/dictionaries)
- [Payloads para redirecciones abiertas](https://github.com/cujanovic/Open-Redirect-Payloads)
- [Hoja de referencia de recompensas por errores de EdOverflow](https://github.com/EdOverflow/bugbounty-cheatsheet)
- [Daniel Miessler - SecLists](https://github.com/danielmiessler/SecLists)
- [Feed de Twitter de XssPayloads](https://twitter.com/XssPayloads)
- [Lista de XssPayloads](https://github.com/payload-box/xss-payload-list)