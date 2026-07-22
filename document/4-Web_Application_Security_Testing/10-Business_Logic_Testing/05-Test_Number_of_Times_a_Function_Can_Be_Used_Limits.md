# Probar Límites en el Número de Veces que una Función Puede Ser Usada

|ID          |
|------------|
|WSTG-BUSL-05|

## Resumen

Muchos de los problemas que las aplicaciones están resolviendo requieren límites en el número de veces que una función puede ser usada o una acción puede ser ejecutada. Las aplicaciones deben ser "lo suficientemente inteligentes" para no permitir al usuario exceder su límite en el uso de estas funciones ya que en muchos casos cada vez que se usa la función el usuario podría ganar algún tipo de beneficio que debe ser contabilizado para compensar apropiadamente al propietario. Por ejemplo: un sitio de eCommerce podría solo permitir a los usuarios aplicar un descuento una vez por transacción, o algunas aplicaciones podrían estar en un plan de suscripción y solo permitir a los usuarios descargar tres documentos completos mensualmente.

Las vulnerabilidades relacionadas con las pruebas de los límites de función son específicas de la aplicación y los casos de uso indebido deben ser creados que se esfuercen por ejercitar partes de las aplicaciones/funciones/acciones más del número permisible de veces.

Los atacantes podrían ser capaces de evadir la lógica de negocio y ejecutar una función más veces de lo "permisible" explotando la aplicación para beneficio personal.

### Ejemplo

Suponer que un sitio de eCommerce permite a los usuarios aprovechar cualquiera de muchos descuentos en su compra total y luego proceder a checkout y pago. ¿Qué pasa si el atacante navega de vuelta a la página de descuentos después de tomar y aplicar el descuento "permisible" uno? ¿Pueden aprovechar otro descuento? ¿Pueden aprovechar el mismo descuento múltiples veces?

## Objetivos de Prueba

- Identificar funciones que deben establecer límites a las veces que pueden ser llamadas.
- Evaluar si hay un límite lógico establecido en las funciones y si está propiamente validado.

## Cómo Probar

- Revisar la documentación del proyecto y usar pruebas exploratorias buscando funciones o características en la aplicación o sistema que no deberían ejecutarse más de una sola vez o número especificado de veces durante el flujo de trabajo de lógica de negocio.
- Para cada una de las funciones y características encontradas que solo deberían ejecutarse una sola vez o número especificado de veces durante el flujo de trabajo de lógica de negocio, desarrollar casos de abuso/uso indebido que puedan permitir a un usuario ejecutar más del número permisible de veces. Por ejemplo, ¿puede un usuario navegar hacia atrás y hacia adelante a través de las páginas múltiples veces ejecutando una función que solo debería ejecutarse una vez? o ¿puede un usuario cargar y descargar carritos de compra permitiendo descuentos adicionales?

## Casos de Prueba Relacionados

- [Pruebas de Enumeración de Cuentas y Cuenta de Usuario Adivinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)
- [Pruebas de Mecanismo de Bloqueo Débil](../04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism.md)

## Remediación

La aplicación debería establecer controles estrictos para prevenir el abuso de límites. Esto puede lograrse estableciendo un cupón para que ya no sea válido a nivel de base de datos, establecer un contador límite por usuario a nivel de backend o base de datos, ya que todos los usuarios deberían ser identificados a través de una sesión, cualquiera que sea mejor para el requisito de negocio.

## Referencias

- [Gold Trading Was Temporarily Halted On The CME This Morning](https://www.businessinsider.com/gold-halted-on-cme-for-stop-logic-event-2013-10)
