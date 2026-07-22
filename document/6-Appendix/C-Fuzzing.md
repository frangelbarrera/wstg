# Fuzzing

## Introducción

El fuzzing es el proceso o técnica de enviar un número de solicitudes a un sitio objetivo en un cierto intervalo de tiempo. En otras palabras, también es similar al bruteforcing. El fuzzing es un proceso que puede lograrse usando herramientas como Wfuzz, ffuf, y así sucesivamente. Como probador necesitarías proporcionar a la herramienta la URL objetivo, parámetro, endpoint, etc, y algún tipo de entradas. Luego la herramienta de fuzzing crea solicitudes y las envía al objetivo. Después de que el fuzzing haya terminado, las respuestas, tiempos, códigos de estado, y otras características necesitan ser analizadas para vulnerabilidades potenciales.

## ¿Por qué fuzzing?

Probar vulnerabilidades alimentando entrada una por una manualmente puede ser caótico. En la era presente donde la gente tiene menos tiempo y bajos niveles de paciencia, la idea de alimentar entrada manualmente para encontrar bugs/vulnerabilidades puede ser abrumadora. Para reducir esta percepción y ahorrar tiempo; el fuzzing puede ser un gran punto a favor. El fuzzing es un proceso automatizado donde mucho del trabajo duro es manejado por una herramienta de fuzzing. Todo lo que un analista tiene que hacer es analizar las varias características después de que el proceso esté hecho. Considera un sitio donde hay muchos campos de entrada para probar XSS. En un enfoque manual, todo lo que hacemos es alimentar el campo de entrada con payloads XSS uno por uno, lo cual sería demasiado caótico. En contraste, para un enfoque automatizado, todo lo que necesitas es proporcionar la lista de payloads XSS al fuzzer y todas las solicitudes son manejadas por el fuzzer.

## Herramientas para Usar para Fuzzing

Hay cientos de herramientas disponibles en la industria para hacer fuzzing. Pero algunas de las mejores calificadas, herramientas de fuzzing populares están listadas abajo.

### Wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz) funciona reemplazando el valor de la lista de palabras al lugar donde hay placeholder `FUZZ`. Para entender esto más claramente consideremos un ejemplo:

```bash
wfuzz -w userIDs.txt https://example.com/view_photo?userId=FUZZ
```

En el comando arriba, `userIds.txt` es un archivo de lista de palabras conteniendo valores de ID numéricos. Aquí, estamos diciéndole a wfuzz que fuzze la solicitud a la URL de ejemplo. Nota que la palabra `FUZZ` en la URL, actuará como placeholder para wfuzz para reemplazar con valores de la lista de palabras. Todos los valores de ID numéricos del archivo `userIDs.txt` serán insertados reemplazando la palabra clave `FUZZ`.

### Ffuf

[Ffuf](https://github.com/ffuf/ffuf) es una herramienta de fuzzing web escrita en el lenguaje Go que es muy rápida y recursiva en naturaleza. Funciona similar a Wfuzz pero en contraste es recursiva. Ffuf también funciona reemplazando el placeholder `FUZZ` con valores de lista de palabras. Por ejemplo:

```bash
ffuf -w userIDs.txt -u https://example.com/view_photo?userId=FUZZ
```

Aquí el `-w` es el flag para lista de palabras y `-u` es el flag para la URL objetivo. El resto del mecanismo de trabajo es el mismo que Wfuzz. Reemplaza el placeholder `FUZZ` con valores de `userIDs.txt`.

### GoBuster

[GoBuster](https://github.com/OJ/gobuster) es otro fuzzer escrito en el lenguaje Go que se usa más para fuzzing URIs, directorios/rutas, subdominios DNS, buckets AWS S3, nombres vhost, y soporta concurrencia. Por ejemplo:

```bash
gobuster dir -w endpoints.txt -u https://example.com
```

En el comando arriba `dir` especifica que estamos fuzzing un directorio, `-u` es el flag para URL, y `-w` es el flag para lista de palabras donde `endpoints.txt` es el archivo de lista de palabras de donde se tomarán los payloads. El comando ejecuta solicitudes concurrentes al endpoint para encontrar directorios disponibles.

### ZAP

[ZAP](https://www.zaproxy.org) es un escáner de seguridad de aplicaciones web que puede usarse para encontrar vulnerabilidades y debilidades en aplicaciones web. También incluye un [Fuzzer](https://www.zaproxy.org/docs/desktop/addons/fuzzer/).

Una de las características clave de ZAP es su capacidad para realizar tanto escáneres pasivos como activos. Los escáneres pasivos involucran observar el tráfico entre el usuario y la aplicación web, mientras que los escáneres activos involucran enviar payloads de prueba a la aplicación web para identificar vulnerabilidades.

### Listas de Palabras y Referencias

En los ejemplos arriba hemos visto por qué necesitamos una lista de palabras. Solo listas de palabras no son suficientes, la lista de palabras debe ser grande para tu escenario de fuzzing. Si no encuentras listas de palabras que coincidan con el escenario necesario entonces considera generar tu propia lista de palabras. Algunas listas de palabras populares y referencias se proporcionan abajo.

- [Hoja de trucos de cross-site scripting (XSS)](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [AwesomeXSS](https://github.com/s0md3v/AwesomeXSS)
- [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [Big List of Naughty Strings](https://github.com/minimaxir/big-list-of-naughty-strings)
- [Bo0oM Fuzz List](https://github.com/Bo0oM/fuzz.txt)
- [FuzzDB](https://github.com/fuzzdb-project/fuzzdb)
- [bl4de Dictionaries](https://github.com/bl4de/dictionaries)
- [Open Redirect Payloads](https://github.com/cujanovic/Open-Redirect-Payloads)
- [EdOverflow Bug Bounty Cheat Sheet](https://github.com/EdOverflow/bugbounty-cheatsheet)
- [Daniel Miessler - SecLists](https://github.com/danielmiessler/SecLists)
- [XssPayloads Twitter Feed](https://twitter.com/XssPayloads)
- [XssPayloads List](https://github.com/payloadbox/xss-payload-list)