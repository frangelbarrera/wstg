# Pruebas de Padding Oracle

|ID          |
|------------|
|WSTG-CRYP-02|

## Resumen

Un padding oracle es una función de una aplicación que descifra datos cifrados proporcionados por el cliente, por ejemplo estado de sesión interno almacenado en el cliente, y filtra el estado de la validez del padding después del descifrado. La existencia de un padding oracle permite a un atacante descifrar datos cifrados y cifrar datos arbitrarios sin conocimiento de la clave usada para estas operaciones criptográficas. Esto puede llevar a fuga de datos sensibles o a vulnerabilidades de escalada de privilegios, si la aplicación asume integridad de los datos cifrados.

Los cifrados de bloque cifran datos solo en bloques de ciertos tamaños. Los tamaños de bloque usados por cifrados comunes son 8 y 16 bytes. Los datos donde el tamaño no coincide con un múltiplo del tamaño de bloque del cifrado usado tienen que ser paddeados de una manera específica para que el descifrador pueda eliminar el padding. Un esquema de padding comúnmente usado es PKCS#7. Llena los bytes restantes con el valor de la longitud del padding.

### Ejemplo 1

Si el padding tiene la longitud de 5 bytes, el valor de byte `0x05` se repite cinco veces después del texto plano.

Una condición de error está presente si el padding no coincide con la sintaxis del esquema de padding usado. Un padding oracle está presente si una aplicación filtra esta condición de error de padding específica para datos cifrados proporcionados por el cliente. Esto puede suceder exponiendo excepciones (por ejemplo `BadPaddingException` en Java) directamente, por diferencias sutiles en las respuestas enviadas al cliente o por otro canal lateral como comportamiento de tiempo.

Ciertos modos de operación de criptografía permiten ataques de bit-flipping, donde invertir un bit en el texto cifrado causa que el bit también se invierta en el texto plano. Invertir un bit en el n-ésimo bloque de datos cifrados en CBC causa que el mismo bit en el bloque (n+1)-ésimo se invierta en los datos descifrados. El n-ésimo bloque del texto cifrado descifrado se basura por esta manipulación.

El ataque de padding oracle permite a un atacante descifrar datos cifrados sin conocimiento de la clave de cifrado y cifrado usado enviando textos cifrados manipulados hábilmente al padding oracle y observando los resultados devueltos por él. Esto causa pérdida de confidencialidad de los datos cifrados. Por ejemplo, en el caso de datos de sesión almacenados del lado del cliente, el atacante puede ganar información sobre el estado interno y estructura de la aplicación.

Un ataque de padding oracle también permite a un atacante cifrar textos planos arbitrarios sin conocimiento de la clave y cifrado usado. Si la aplicación asume que la integridad y autenticidad de los datos descifrados está dada, un atacante podría ser capaz de manipular el estado de sesión interno y posiblemente ganar mayores privilegios.

## Objetivos de Prueba

- Identificar mensajes cifrados que dependen de padding.
- Intentar romper el padding de los mensajes cifrados y analizar los mensajes de error devueltos para análisis adicional.

## Cómo Probar

### Pruebas de Caja Negra

Primero deben identificarse los posibles puntos de entrada para padding oracles. Generalmente deben cumplirse las siguientes condiciones:

1. Los datos están cifrados. Buenos candidatos son valores que aparecen ser aleatorios.
2. Se usa un cifrado de bloque. La longitud del texto cifrado decodificado (Base64 se usa a menudo) es un múltiplo de tamaños de bloque de cifrado comunes como 8 o 16 bytes. Diferentes textos cifrados (por ejemplo recolectados por diferentes sesiones o manipulación de estado de sesión) comparten un divisor común en la longitud.

#### Ejemplo 2

`Dg6W8OiWMIdVokIDH15T/A==` resulta después de decodificar base64 en `0e 0e 96 f0 e8 96 30 87 55 a2 42 03 1f 5e 53 fc`. Esto parece ser aleatorio y de 16 bytes de largo.

Si se identifica tal valor de entrada candidato, debe verificarse el comportamiento de la aplicación a la manipulación bit a bit del valor cifrado. Normalmente este valor codificado en base64 incluirá el vector de inicialización (IV) prependado al texto cifrado. Dado un texto plano *`p`* y un cifrado con un tamaño de bloque *`n`*, el número de bloques será *`b = ceil( length(p) / n)`*. La longitud de la cadena cifrada será *`y=(b+1)*n`* debido al vector de inicialización. Para verificar la presencia del oracle, decodificar la cadena, invertir el último bit del segundo-último bloque *`b-1`* (el bit menos significativo del byte en *`y-n-1`*), recodificar y enviar. Luego, decodificar la cadena original, invertir el último bit del bloque *`b-2`* (el bit menos significativo del byte en *`y-2*n-1`*), recodificar y enviar.

Si se sabe que la cadena cifrada es un solo bloque (el IV se almacena en el servidor o la aplicación está usando una mala práctica de IV hardcoded), varios bit flips deben realizarse por turno. Un enfoque alternativo podría ser prependar un bloque aleatorio, e invertir bits para hacer que el último byte del bloque añadido tome todos los valores posibles (0 a 255).

Las pruebas y el valor base deberían al menos causar tres estados diferentes durante y después del descifrado:

- El texto cifrado se descifra, los datos resultantes son correctos.
- El texto cifrado se descifra, los datos resultantes están basura y causan alguna excepción o manejo de error en la lógica de la aplicación.
- El descifrado del texto cifrado falla debido a errores de padding.

Comparar las respuestas cuidadosamente. Buscar especialmente excepciones y mensajes que indiquen que algo está mal con el padding. Si tales mensajes aparecen, la aplicación contiene un padding oracle. Si los tres estados diferentes descritos anteriormente son observables implícitamente (diferentes mensajes de error, canales laterales de tiempo), hay una alta probabilidad de que haya un padding oracle presente en este punto. Intentar realizar el ataque de padding oracle para asegurar esto.

##### Ejemplo 3

- ASP.NET lanza `System.Security.Cryptography.CryptographicException: Padding is invalid and cannot be removed.` si el padding de un texto cifrado descifrado está roto.
- En Java se lanza una `javax.crypto.BadPaddingException` en este caso.
- Errores de descifrado o similares pueden ser posibles padding oracles.

> Una implementación segura verificará la integridad y causará solo dos respuestas: `ok` y `failed`. No hay canales laterales que puedan usarse para determinar estados de error internos.

### Pruebas de Caja Gris

Verificar que todos los lugares donde se descifran datos cifrados del cliente, que solo deberían ser conocidos por el servidor. Las siguientes condiciones deberían cumplirse por tal código:

1. La integridad del texto cifrado debería verificarse por un mecanismo seguro, como HMAC o modos de operación de cifrado autenticados como GCM o CCM.
2. Todos los estados de error durante el descifrado y procesamiento adicional se manejan uniformemente.

### Ejemplo 4

[Visualización del proceso de descifrado](https://erlend.oftedal.no/blog/poet/)

## Herramientas

- [Bletchley](https://code.blindspotsecurity.com/trac/bletchley)
- [PadBuster](https://github.com/GDSSecurity/PadBuster)
- [Poracle](https://github.com/iagox86/Poracle)
- [python-paddingoracle](https://github.com/mwielgoszewski/python-paddingoracle)

## Referencias

- [Wikipedia - Padding Oracle Attack](https://en.wikipedia.org/wiki/Padding_oracle_attack)
- [Juliano Rizzo, Thai Duong, "Practical Padding Oracle Attacks"](https://www.usenix.org/event/woot10/tech/full_papers/Rizzo.pdf)
