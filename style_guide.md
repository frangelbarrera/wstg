# Guía de Estilo

La Guía de Pruebas de Seguridad Web (WSTG) es un documento bien conocido confiado por profesionales de seguridad y organizaciones de todo el mundo. Estas directrices ayudan a asegurar que refleja bien en sus muchos contribuidores y la comunidad de seguridad.

Para mantener la calidad de la WSTG, por favor sigue estas reglas generales.

1. Sé factual, específico, y asegura que los párrafos estén enfocados en su encabezado.
2. Asegura que la información sea creíble y actualizada. Proporciona enlaces y citas donde sea apropiado.
3. Evita duplicar contenido. Para referirte a contenido existente, enlázalo inline.

## Escribe para el Lector

Los lectores de la WSTG vienen de muchos países diferentes y tienen niveles variables de experiencia técnica. Escribe para una audiencia internacional con un fondo técnico básico. Usa palabras que probablemente sean entendidas por un hablante no nativo de inglés. Usa oraciones cortas que sean fáciles de entender.

La herramienta web [Hemingway](https://hemingwayapp.com/) puede ayudarte a escribir con claridad.

## Formateo

Usa formateo consistente para ayudarnos a revisar y publicar contenido, y ayudar a los lectores a digerir información. Escribe todo el contenido usando [sintaxis Markdown](https://guides.github.com/features/mastering-markdown/#examples).

Por favor sigue estas directrices adicionales para formateo.

### Plantilla de Artículo

Usamos una plantilla de artículo para ayudar a asegurar que los temas sean completos y fáciles de entender. Por favor usa los [materiales de plantilla](template) para estructurar nuevo contenido.

### Estructura de Carpeta del Proyecto

Al agregar artículos e imágenes, por favor coloca artículos en el directorio de sub-sección apropiado. Coloca imágenes en una carpeta `images/` dentro del directorio del artículo. Aquí hay un ejemplo de la estructura del proyecto:

```sh
document/
 ├───0_Foreword/
 │   └───0_Foreword.md
 ├───1_Frontispiece/
 │   ├───images/
 │   │   └───example.jpg
 │   └───1_Frontispiece.md
 ├───2_Introduction/
 │   ├───images/
 │   │   └───example.jpg
 │   └───2_Introduction.md
 ├───3_The_OWASP_Testing_Framework/
 │   ├───images/
 │   │   └───example.jpg
 │   └───3_The_OWASP_Testing_Framework.md
 ├───4_Web_Application_Security_Testing/
 │   ├───4.1_Introduction_and_Objectives/
 │   │   └───4.1_Testing_Introduction_and_Objectives.md
 │   ├───4.2_Information_Gathering/
 │   │   ├───images/
 │   │   │   └───example.jpg
 │   │   ├───4.2_Testing_Information_Gathering.md
 │   │   └───4.2.1_Conduct_Search_Engine_Discovery.md

```

### Resaltado de Sintaxis de Código

Usa fences de código con resaltado de sintaxis para snippets. Por ejemplo:

```md
    ```javascript
    if (isAwesome){
        return true
    }
    ```
```

### Imágenes con Caption

Pon captions a imágenes y figuras usando title case. Usa los números de sección y sub-sección, seguidos por la posición de la figura en el documento. Usa el formato `Figure <section>.<sub-section>-<position>: Caption Title`.

Por ejemplo, pon caption a la primera imagen mostrada en sección 4.8, sub-sección 19 como sigue:

```md
![SSTI XVWA Example](images/SSTI_XVWA.jpeg)\
*Figure 4.7.19-1: SSTI XVWA Example*
```

### Enlaces Inline

Agrega enlaces inline. Usa palabras en la oración para describirlos, o incluye su título específico. Por ejemplo:

```md
Este proyecto proporciona una [guía de estilo](style_guide.md). Algunas elecciones de estilo son tomadas de [Chicago Manual of Style](https://www.chicagomanualofstyle.org/).
```

### Referencias Inline

Para recursos donde un enlace no está disponible, como un whitepaper o libro, preferimos una referencia conversacional in-line en lugar de cualquier cita estilo académico. Trabaja el título del recurso así como su autor en tu texto. Por ejemplo:

> Hay tres casos posibles: solo la ballena existe, solo las petunias existen, o tanto la ballena como las petunias existen simultáneamente. Estas posibilidades son referenciadas en una serie de libros titulada *The Hitchhiker's Guide to the Galaxy,* por Douglas Adams.

Este formato tiene la ventaja de continuar el flujo del artículo y no invitar a los lectores a saltar de párrafo a párrafo, buscando un asterisco, o a otra ubicación para encontrar una lista de referencias. También es fácil de leer y mantener ya que aparece en solo un lugar.

### Negrita, Itálica, y Subrayado

No uses texto negrita, itálico, o subrayado para énfasis.

Puedes italicizar una palabra cuando refiriéndote a la palabra en sí, aunque la necesidad de esto en escritura técnica es rara. Para ejemplos, ver la sección [Usa Palabras Correctas](#usa-palabras-correctas). Usa asteriscos: `*italic*`.

## Lenguaje y Gramática

Para hacer la WSTG consistente y placentera de leer, por favor verifica tu ortografía (usamos inglés americano) y usa gramática apropiada.

Las secciones abajo describen elecciones de estilo específicas a seguir.

### Title Case

Usa title case para encabezados, siguiendo el [Chicago Manual of Style](https://www.chicagomanualofstyle.org/book/ed17/frontmatter/toc.html). La pestaña "Chicago" en el sitio web [Capitalize My Title](https://capitalizemytitle.com/#Chicago) puede ayudar.

### Voz Activa

Evita usar voz pasiva. Por ejemplo:

> Mal: "Vulnerabilidades son encontradas corriendo tests."  
> Bien: "Corre tests para encontrar vulnerabilidades."  

### Segunda Persona

No escribas en primera o tercera persona, como usando *I* o *he*. Cuando dando instrucción técnica, dirígete al lector en segunda persona. Usa un [sujeto cero o implícito](https://en.wikipedia.org/wiki/Subject_(grammar)#Forms_of_the_subject), o si debes, usa *you*.

> Mal: "He/she/un mono IT correría este código para testear..."  
> Mejor: "Correndo este código, puedes testear..."  
> Mejor: "Corre este código para testear..."

### Convenciones de Numeración

Para números de cero a diez, escribe la palabra. Para números más altos que diez, usa enteros. Por ejemplo:

> Un test automatizado roto encuentra 42 errores si lo corres diez veces.

Describe fracciones simples en palabras. Por ejemplo:

> La mitad de todos los desarrolladores de software gustan de petunias, y un tercio de ellos gustan de ballenas.

Cuando describiendo una magnitud aproximada de valor monetario, escribe la palabra completa y no abrevies. Por ejemplo:

> Mal: "Pruebas de seguridad salvan compañías $18M en cerveza cada año."  
> Bien: "Pruebas de seguridad salvan compañías dieciocho millones de dólares en cerveza cada año."

Para valor monetario específico, usa símbolos de moneda y enteros. Por ejemplo:

> Una cerveza cuesta $6.75 hoy, y $8.25 mañana.

### Abreviaturas

Explica abreviaturas la primera vez que aparecen en tu documento. Capitaliza las palabras apropiadas para indicar la forma abreviada. Por ejemplo:

> Este proyecto contiene el código fuente para la Guía de Pruebas de Seguridad Web (WSTG). La WSTG es un libro agradable y preciso.

### Listas y Puntuación

Usa listas con viñetas cuando el orden es poco importante. Usa listas numeradas para pasos secuenciales. Para cada línea, capitaliza la primera palabra. Si la línea es una oración o completa una oración, termina con un punto. Por ejemplo:

> Testear este escenario hará:
>
> - Hacer la aplicación más segura.
> - Mejorar la postura de seguridad general.
> - Mantener clientes felices.
>
> Para testear este escenario:
>
> 1. Copia el código.
> 2. Abre un terminal.
> 3. Corre el código como root.
>
> Aquí hay algunos alimentos para snackear mientras testeas.
>
> - Manzanas
> - Carne seca
> - Chocolate

Para listas en una oración, usa comas seriales u [Oxford commas](https://www.grammarly.com/blog/what-is-the-oxford-comma-and-why-do-people-care-so-much-about-it/). Por ejemplo:

> Testea la aplicación usando tests automatizados, revisión de código estático, y pruebas de penetración.

### Usa Palabras Correctas

La sección siguiente cubre algunas palabras frecuentemente mal usadas e instrucciones sobre cómo usarlas correctamente.

#### *and/or*

Mientras a veces usado en documentos legales, *and/or* lleva a ambigüedad y confusión en escritura técnica. En su lugar, usa *or*, que en el lenguaje inglés incluye *and*. Por ejemplo:

> Mal: "El código saldrá un número de error and/or descripción."  
> Bien: "El código saldrá un número de error or descripción."

La última oración no excluye la posibilidad de tener tanto un número de error como descripción.

Si necesitas especificar todos los resultados posibles, usa una lista:

> "El código saldrá un número de error, or una descripción, or ambos."

#### *frontend, backend*

Mientras es cierto que el lenguaje inglés evoluciona con el tiempo, estas no son aún palabras.

Cuando refiriéndote a sustantivos, usa *front end* y *back end*. Por ejemplo:

> Seguridad es igualmente importante en el front end como lo es en el back end.

Como adverbio descriptivo, usa los hyphenated *front-end* y *back-end*.

> Tanto desarrolladores front-end como back-end son responsables de la seguridad de la aplicación.

#### *whitebox*, *blackbox*, *greybox*

Estas no son palabras.

Como sustantivos, usa *white box*, *black box*, y *grey box*. Estos sustantivos rara vez aparecen en conexión con ciberseguridad.

> Mi gato disfruta saltando en esa grey box.

Como adverbios, usa los hyphenated *white-box*, *black-box*, y *grey-box*. No uses capitalización a menos que las palabras estén en un título.

> Mientras white-box testing involucra conocimiento de código fuente, black-box testing no. Un grey-box test está en algún lugar en medio.

#### *ie*, *eg*

Estas son letras.

La abreviatura *ie* se refiere al latín `id est`, que significa "en otras palabras." La abreviatura *eg* es para `exempli gratia`, traduciendo a "por ejemplo." Para usar estas en una oración:

> Escribe usando inglés apropiado, i.e. ortografía correcta y gramática. Usa palabras comunes sobre poco comunes, e.g. "learn" en lugar de "glean."

#### *etc*

Estas también son letras.

La frase latina *et cetera* traduce a "y el resto." Es abreviada y típicamente colocada al final de una lista que parece redundante completar:

> Autores WSTG gustan de colores arcoíris, como rojo, amarillo, verde, etc.

En escritura técnica, el uso de *etc* es problemático. Asume que el lector sabe de qué estás hablando, y pueden no. Violeta es uno de los colores del arcoíris, pero el ejemplo arriba no te dice explícitamente si violeta es un color que autores WSTG gustan.

Es mejor ser explícito y thorough que hacer suposiciones del lector. Solo usa *etc* para evitar completar una lista que fue dada en full anteriormente en el documento.

#### *...* (elipsis)

La marca de puntuación elipsis puede indicar que palabras han sido dejadas fuera de una cita:

> Linus Torvalds una vez dijo, "Una vez que realizas que documentación debería ser reída... THEN, y solo then, has alcanzado el nivel donde puedes leerla con seguridad e intentar usarla para implementar un driver."

Mientras la omisión no cambie el significado de la cita, este es uso aceptable de elipsis en la WSTG.

Todos los otros usos de elipsis, como indicar un pensamiento inacabado, no lo son.

#### *ex*

Mientras esta es una palabra, es likely no la palabra que estás buscando. La palabra *ex* tiene significado particular en los campos de finanzas y comercio, y puede referir a una persona si estás discutiendo tus relaciones pasadas. Ninguno de estos temas debería aparecer en la WSTG.

La abreviatura puede ser usada para significar "example" por escritores lazy. Por favor no seas lazy, y escribe *example* en su lugar.