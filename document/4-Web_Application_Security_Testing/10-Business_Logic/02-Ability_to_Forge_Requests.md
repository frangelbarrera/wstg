# Capacidad para falsificar solicitudes

|ID          |
|------------|
|WSTG-BUSL-02|

## Resumen

La falsificación de solicitudes es un método que usan los atacantes para eludir la interfaz gráfica de usuario (GUI) del frontend y enviar información directamente para que la procese el backend. El objetivo del atacante es enviar solicitudes HTTP POST/GET a través de un proxy de interceptación con valores de datos que la lógica de negocio de la aplicación no admite, no protege o no espera. Algunos ejemplos de solicitudes falsificadas incluyen explotar parámetros adivinables o predecibles, o exponer características y funcionalidades «ocultas», como habilitar la depuración o mostrar pantallas o ventanas especiales. Estas pueden ser muy útiles durante el desarrollo, pero también podrían filtrar información o eludir la lógica de negocio.

Las vulnerabilidades relacionadas con la capacidad de falsificar solicitudes son exclusivas de cada aplicación y se diferencian de los problemas de validación de datos de la lógica de negocio porque se centran en romper el flujo de trabajo de dicha lógica.

Las aplicaciones deben contar con comprobaciones lógicas que impidan que el sistema acepte solicitudes falsificadas que podrían permitir a los atacantes explotar la lógica, el proceso o el flujo de trabajo de la aplicación. La falsificación de solicitudes no es algo nuevo: el atacante usa un proxy de interceptación para enviar solicitudes HTTP POST/GET a la aplicación. Mediante la falsificación de solicitudes, los atacantes podrían eludir la lógica o el proceso de negocio al encontrar, predecir y manipular parámetros para hacer que la aplicación crea que un proceso o una tarea ha tenido lugar, o que no la ha tenido.

Además, las solicitudes falsificadas podrían permitir subvertir el flujo lógico programático o de negocio mediante la invocación de características o funcionalidades «ocultas», como la depuración, que los desarrolladores y evaluadores usaban inicialmente y que a veces se denominan ["huevos de Pascua"](https://en.wikipedia.org/wiki/Easter_egg_(media)). «Un huevo de Pascua es una broma interna intencional, un mensaje oculto o una característica de una obra, como un programa informático, una película, un libro o un crucigrama. Según el diseñador de videojuegos Warren Robinett, el término fue acuñado en Atari por el personal que recibió un aviso sobre la presencia de un mensaje secreto que Robinett había ocultado en su juego Adventure, ya ampliamente distribuido. Se dice que el nombre evoca la idea de una tradicional búsqueda de huevos de Pascua».

### Ejemplo 1

Supongamos que un sitio web de venta de entradas para un teatro permite a los usuarios seleccionar sus entradas, aplicar un descuento de tercera edad del 10 % por única vez al total de la compra, ver el subtotal y completar la compra. Si un atacante puede observar mediante un proxy que la aplicación tiene un campo oculto (con valor 1 o 0) que la lógica de negocio usa para determinar si ya se aplicó un descuento, podría enviar varias veces el valor 1, que significa «no se ha aplicado ningún descuento», para aprovechar varias veces el mismo descuento.

### Ejemplo 2

Supongamos que un videojuego en línea otorga fichas por los puntos obtenidos al encontrar el tesoro de los piratas, a los piratas y por cada nivel completado. Más adelante, estas fichas se pueden canjear por premios. Además, los puntos de cada nivel tienen un multiplicador igual al número del nivel. Si un atacante puede observar mediante un proxy que la aplicación tiene un campo oculto que se usa durante el desarrollo y las pruebas para avanzar rápidamente a los niveles más altos del juego, podría llegar a ellos rápidamente y acumular puntos que no se ha ganado.

Además, si un atacante puede observar mediante un proxy que la aplicación tiene un campo oculto que se usa durante el desarrollo y las pruebas para habilitar un registro que indicaba dónde estaban otros jugadores en línea o los tesoros ocultos con respecto al atacante, podría dirigirse rápidamente a esas ubicaciones y obtener puntos.

## Objetivos de prueba

- Revisar la documentación del proyecto en busca de campos con funcionalidad adivinable, predecible u oculta.
- Insertar datos lógicamente válidos para eludir el flujo de trabajo habitual de la lógica de negocio.

## Cómo probar

### Mediante la identificación de valores adivinables

- Usar un proxy de interceptación para observar las solicitudes HTTP POST/GET y buscar indicios de que los valores aumentan a intervalos regulares o son fáciles de adivinar.
- Si se descubre que un valor es adivinable, se puede cambiar para obtener una visibilidad inesperada.

### Mediante la identificación de opciones ocultas

- Usar un proxy de interceptación para observar las solicitudes HTTP POST/GET y buscar indicios de características ocultas, como una opción de depuración que se pueda activar.
- Si se encuentran, intentar adivinar y cambiar estos valores para obtener una respuesta o un comportamiento diferente de la aplicación.

## Casos de prueba relacionados

- [Variables de sesión expuestas](../06-Session_Management/04-Exposed_Session_Variables.md)
- [Falsificación de solicitudes entre sitios (CSRF)](../06-Session_Management/05-Cross_Site_Request_Forgery.md)
- [Enumeración de cuentas y cuentas de usuario adivinables](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md)

## Remediación

La aplicación debe diseñarse con una lógica de negocio capaz de impedir que los atacantes predigan y manipulen parámetros para subvertir el flujo lógico programático o de negocio, o exploten funcionalidades ocultas o no documentadas, como la depuración.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [Huevo de Pascua](https://en.wikipedia.org/wiki/Easter_egg_(media))
- [Los 10 mejores huevos de Pascua de software](https://lifehacker.com/371083/top-10-software-easter-eggs)