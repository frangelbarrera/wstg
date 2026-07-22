# Probar la Capacidad de Falsificar Solicitudes

|ID          |
|------------|
|WSTG-BUSL-02|

## Resumen

Falsificar solicitudes es un método que los atacantes usan para evadir la aplicación GUI frontend y directamente enviar información para el procesamiento del backend. El objetivo del atacante es enviar solicitudes HTTP POST/GET a través de un proxy de intercepción con valores de datos que no están soportados, protegidos, o esperados por la lógica de negocio de la aplicación. Algunos ejemplos de solicitudes falsificadas incluyen explotar parámetros adivinables o predecibles o exponer características y funcionalidades "ocultas" tales como habilitar depuración o presentar pantallas o ventanas especiales que son muy útiles durante el desarrollo pero podrían filtrar información o evadir la lógica de negocio.

Las vulnerabilidades relacionadas con la capacidad de falsificar solicitudes son únicas para cada aplicación y diferentes de la validación de datos de lógica de negocio en que su enfoque está en romper el flujo de trabajo de la lógica de negocio.

Las aplicaciones deberían tener verificaciones lógicas en su lugar para prevenir que el sistema acepte solicitudes falsificadas que podrían permitir a los atacantes la oportunidad de explotar la lógica de negocio, proceso, o flujo de la aplicación. La falsificación de solicitudes no es nada nueva; el atacante usa un proxy de intercepción para enviar solicitudes HTTP POST/GET a la aplicación. A través de falsificaciones de solicitud los atacantes podrían ser capaces de evadir la lógica de negocio o proceso encontrando, prediciendo y manipulando parámetros para hacer que la aplicación piense que un proceso o tarea ha tenido o no ha tenido lugar.

Además, las solicitudes falsificadas podrían permitir la subversión del flujo lógico programático o de negocio invocando características o funcionalidades "ocultas" tales como depuración inicialmente usada por desarrolladores y testers a veces referida como un ["Easter egg"](https://en.wikipedia.org/wiki/Easter_egg_(media)). "Un Easter egg es una broma interna intencional, mensaje oculto, o característica en un trabajo tal como un programa de computadora, película, libro, o crucigrama. Según el diseñador de juegos Warren Robinett, el término fue acuñado en Atari por personal que fueron alertados de la presencia de un mensaje secreto que había sido ocultado por Robinett en su juego ya ampliamente distribuido, Adventure. El nombre ha sido dicho que evoca la idea de una búsqueda tradicional de huevos de Pascua."

### Ejemplo 1

Suponer que un sitio de e-commerce de teatro permite a los usuarios seleccionar su boleto, aplicar un descuento de Senior de 10% por única vez en toda la venta, ver el subtotal y liquidar la venta. Si un atacante es capaz de ver a través de un proxy que la aplicación tiene un campo oculto (de 1 o 0) usado por la lógica de negocio para determinar si un descuento ya ha sido tomado o no. El atacante entonces es capaz de enviar el valor de 1 o "ningún descuento ha sido tomado" múltiples veces para aprovecharse del mismo descuento múltiples veces.

### Ejemplo 2

Suponer que un videojuego en línea paga tokens por puntos anotados por encontrar tesoro de piratas, piratas, y por cada nivel completado. Estos tokens pueden luego ser intercambiados por premios. Adicionalmente los puntos de cada nivel tienen un valor multiplicador igual al nivel. Si un atacante fue capaz de ver a través de un proxy que la aplicación tiene un campo oculto usado durante el desarrollo y pruebas para llegar rápidamente a los niveles más altos del juego, podrían llegar rápidamente a los niveles más altos y acumular puntos no ganados rápidamente.

Además, si un atacante fue capaz de ver a través de un proxy que la aplicación tiene un campo oculto usado durante el desarrollo y pruebas para habilitar un log que indicaba dónde estaban otros jugadores en línea, o tesoros ocultos en relación al atacante, entonces serían capaces de ir rápidamente a estas ubicaciones y anotar puntos.

## Objetivos de Prueba

- Revisar la documentación del proyecto buscando funcionalidad adivinable, predecible, u oculta de campos.
- Insertar datos lógicamente válidos para evadir el flujo de trabajo normal de lógica de negocio.

## Cómo Probar

### A Través de Identificación de Valores Adivinables

- Usando un proxy de intercepción observar el HTTP POST/GET buscando alguna indicación de que los valores se están incrementando a un intervalo regular o son fácilmente adivinables.
- Si se encuentra que algún valor es adivinable, este valor puede ser cambiado y se puede ganar visibilidad inesperada.

### A Través de Identificación de Opciones Ocultas

- Usando un proxy de intercepción observar el HTTP POST/GET buscando alguna indicación de características ocultas tales como debug que pueden ser activadas o activadas.
- Si se encuentra alguna, intentar adivinar y cambiar estos valores para obtener una respuesta o comportamiento diferente de la aplicación.

## Casos de Prueba Relacionados

- [Pruebas de Variables de Sesión Expuestas](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md)
- [Pruebas de Cross Site Request Forgery (CSRF)](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)
- [Pruebas de Enumeración de Cuentas y Cuenta de Usuario Adivinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)

## Remediación

La aplicación debe ser lo suficientemente inteligente y diseñada con lógica de negocio que prevendrá a los atacantes predecir y manipular parámetros para subvertir el flujo lógico programático o de negocio, o explotar funcionalidad oculta/no documentada tal como depuración.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [Easter egg](https://en.wikipedia.org/wiki/Easter_egg_(media))
- [Top 10 Software Easter Eggs](https://lifehacker.com/371083/top-10-software-easter-eggs)
