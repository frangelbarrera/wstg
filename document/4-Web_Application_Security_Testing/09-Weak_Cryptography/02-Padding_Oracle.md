# Oráculo de padding

|ID          |
|------------|
|WSTG-CRYP-02|

## Resumen

Un oráculo de padding es una función de una aplicación que descifra datos cifrados proporcionados por el cliente, por ejemplo, el estado interno de una sesión almacenado en el cliente, y filtra información sobre la validez del padding después del descifrado. La existencia de un oráculo de padding permite a un atacante descifrar datos cifrados y cifrar datos arbitrarios sin conocer la clave usada para estas operaciones criptográficas. Esto puede provocar la filtración de datos sensibles o vulnerabilidades de escalada de privilegios si la aplicación presupone la integridad de los datos cifrados.

Los cifrados de bloque cifran datos en bloques de determinados tamaños. Los tamaños de bloque usados por los cifrados comunes son de 8 y 16 bytes. Los datos cuyo tamaño no es múltiplo del tamaño de bloque del cifrado usado deben rellenarse de una manera específica para que el descifrador pueda eliminar el padding. Un esquema de padding de uso común es PKCS#7. Este rellena los bytes restantes con el valor correspondiente a la longitud del padding.

### Ejemplo 1

Si el padding tiene una longitud de 5 bytes, el valor de byte `0x05` se repite cinco veces después del texto plano.

Se produce una condición de error si el padding no coincide con la sintaxis del esquema de padding usado. Hay un oráculo de padding si una aplicación filtra esta condición de error de padding específica para los datos cifrados proporcionados por el cliente. Esto puede ocurrir al exponer directamente excepciones (por ejemplo, `BadPaddingException` en Java), debido a diferencias sutiles en las respuestas enviadas al cliente o mediante otro canal lateral, como el comportamiento temporal.

Ciertos modos de operación criptográficos permiten ataques de inversión de bits, en los que invertir un bit del texto cifrado hace que también se invierta ese bit en el texto plano. Invertir un bit en el n-ésimo bloque de datos cifrados con CBC hace que se invierta ese mismo bit en el bloque (n+1)-ésimo de los datos descifrados. Esta manipulación corrompe el n-ésimo bloque del texto cifrado descifrado.

El ataque de oráculo de padding permite a un atacante descifrar datos cifrados sin conocer la clave de cifrado ni el cifrado usado, mediante el envío de textos cifrados manipulados con habilidad al oráculo de padding y la observación de los resultados que devuelve. Esto provoca una pérdida de confidencialidad de los datos cifrados. Por ejemplo, en el caso de los datos de sesión almacenados en el cliente, el atacante puede obtener información sobre el estado y la estructura internos de la aplicación.

Un ataque de oráculo de padding también permite a un atacante cifrar textos planos arbitrarios sin conocer la clave ni el cifrado usados. Si la aplicación presupone que los datos descifrados son íntegros y auténticos, un atacante podría manipular el estado interno de la sesión y posiblemente obtener privilegios más elevados.

## Objetivos de las pruebas

- Identificar mensajes cifrados que dependan del padding.
- Intentar romper el padding de los mensajes cifrados y analizar los mensajes de error devueltos para realizar un análisis más detallado.

## Cómo probar

### Pruebas de caja negra

Primero deben identificarse los posibles puntos de entrada de los oráculos de padding. Por lo general, deben cumplirse las siguientes condiciones:

1. Los datos están cifrados. Los valores que parecen aleatorios son buenos candidatos.
2. Se usa un cifrado de bloque. La longitud del texto cifrado decodificado (a menudo se usa Base64) es múltiplo de tamaños de bloque comunes, como 8 o 16 bytes. La longitud de distintos textos cifrados (por ejemplo, obtenidos en diferentes sesiones o mediante la manipulación del estado de la sesión) tiene un divisor común.

#### Ejemplo 2

`Dg6W8OiWMIdVokIDH15T/A==` da como resultado `0e 0e 96 f0 e8 96 30 87 55 a2 42 03 1f 5e 53 fc` al decodificarlo desde base64. Parece aleatorio y tiene 16 bytes de longitud.

Si se identifica un valor de entrada candidato, debe verificarse el comportamiento de la aplicación ante la manipulación bit a bit del valor cifrado. Normalmente, este valor codificado en base64 incluirá el vector de inicialización (IV) antepuesto al texto cifrado. Dado un texto plano *`p`* y un cifrado con un tamaño de bloque *`n`*, el número de bloques será *`b = ceil( length(p) / n)`*. La longitud de la cadena cifrada será *`y=(b+1)*n`* debido al vector de inicialización. Para verificar si hay un oráculo, decodifique la cadena, invierta el último bit del penúltimo bloque *`b-1`* (el bit menos significativo del byte en *`y-n-1`*), vuelva a codificarla y envíela. A continuación, decodifique la cadena original, invierta el último bit del bloque *`b-2`* (el bit menos significativo del byte en *`y-2*n-1`*), vuelva a codificarla y envíela.

Si se sabe que la cadena cifrada consta de un solo bloque (el IV se almacena en el servidor o la aplicación usa la mala práctica de un IV codificado de forma fija), deben probarse varios cambios de bits, uno a la vez. Un enfoque alternativo sería anteponer un bloque aleatorio e invertir bits para que el último byte del bloque añadido adopte todos los valores posibles (de 0 a 255).

Las pruebas y el valor base deberían producir al menos tres estados diferentes durante el descifrado y después de este:

- El texto cifrado se descifra y los datos resultantes son correctos.
- El texto cifrado se descifra, los datos resultantes quedan corrompidos y provocan alguna excepción o la gestión de errores en la lógica de la aplicación.
- El descifrado del texto cifrado falla debido a errores de padding.

Compare las respuestas con atención. Busque especialmente excepciones y mensajes que indiquen que hay algún problema con el padding. Si aparecen mensajes de este tipo, la aplicación contiene un oráculo de padding. Si los tres estados diferentes descritos anteriormente pueden observarse de forma implícita (mediante distintos mensajes de error o canales laterales temporales), es muy probable que haya un oráculo de padding en ese punto. Intente realizar el ataque de oráculo de padding para confirmarlo.

##### Ejemplo 3

- ASP.NET genera `System.Security.Cryptography.CryptographicException: Padding is invalid and cannot be removed.` si el padding de un texto cifrado descifrado está dañado.
- En Java se genera una `javax.crypto.BadPaddingException` en este caso.
- Los errores de descifrado o similares pueden indicar posibles oráculos de padding.

> Una implementación segura comprobará la integridad y solo producirá dos respuestas: `ok` y `failed`. No habrá canales laterales que puedan usarse para determinar estados de error internos.

### Pruebas de caja gris

Verifique todos los lugares en los que se descifran datos cifrados del cliente que solo deberían ser conocidos por el servidor. El código correspondiente debería cumplir las siguientes condiciones:

1. La integridad del texto cifrado debería verificarse mediante un mecanismo seguro, como HMAC o modos de operación de cifrado autenticado, como GCM o CCM.
2. Todos los estados de error durante el descifrado y el procesamiento posterior se gestionan de manera uniforme.

### Ejemplo 4

[Visualización del proceso de descifrado](https://erlend.oftedal.no/blog/poet/)

## Herramientas

- [PadBuster](https://github.com/GDSSecurity/PadBuster)
- [Poracle](https://github.com/iagox86/Poracle)
- [python-paddingoracle](https://github.com/mwielgoszewski/python-paddingoracle)

## Referencias

- [Wikipedia - Ataque de oráculo de padding](https://en.wikipedia.org/wiki/Padding_oracle_attack)
- [Juliano Rizzo, Thai Duong, "Practical Padding Oracle Attacks"](https://www.usenix.org/event/woot10/tech/full_papers/Rizzo.pdf)