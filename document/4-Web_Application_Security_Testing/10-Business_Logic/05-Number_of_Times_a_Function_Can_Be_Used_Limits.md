# Límites en el número de veces que se puede usar una función

|ID          |
|------------|
|WSTG-BUSL-05|

## Resumen

Muchos de los problemas que resuelven las aplicaciones requieren limitar el número de veces que se puede usar una función o ejecutar una acción. Las aplicaciones deben ser «lo suficientemente inteligentes» para no permitir que el usuario supere el límite de uso de estas funciones, ya que en muchos casos cada uso puede proporcionarle algún tipo de beneficio que debe contabilizarse para compensar adecuadamente al propietario. Por ejemplo, un sitio de comercio electrónico podría permitir a los usuarios aplicar un descuento solo una vez por transacción, o algunas aplicaciones podrían ofrecer un plan de suscripción que permita a los usuarios descargar tres documentos completos al mes.

Las vulnerabilidades relacionadas con las pruebas de los límites de uso de las funciones dependen de cada aplicación, y deben crearse casos de uso indebido que intenten ejecutar partes de la aplicación, funciones o acciones más veces de las permitidas.

Los atacantes podrían evadir la lógica de negocio y ejecutar una función más veces de las «permitidas», explotando la aplicación para obtener un beneficio personal.

### Ejemplo

Supongamos que un sitio de comercio electrónico permite a los usuarios aprovechar cualquiera de varios descuentos en el total de su compra y luego continuar con el proceso de pago. ¿Qué ocurre si el atacante vuelve a la página de descuentos después de obtener y aplicar el único descuento «permitido»? ¿Puede aprovechar otro descuento? ¿Puede aprovechar varias veces el mismo descuento?

## Objetivos de prueba

- Identificar las funciones cuyo número de llamadas debe estar limitado.
- Evaluar si las funciones tienen un límite lógico establecido y si este se valida correctamente.

## Cómo probar

- Revisar la documentación del proyecto y realizar pruebas exploratorias para buscar funciones o características de la aplicación o el sistema que no deberían ejecutarse más de una vez o más del número especificado de veces durante el flujo de trabajo de la lógica de negocio.
- Para cada función o característica identificada que solo debería ejecutarse una vez o un número especificado de veces durante el flujo de trabajo de la lógica de negocio, desarrollar casos de abuso o uso indebido que puedan permitir que un usuario la ejecute más veces de las permitidas. Por ejemplo, ¿puede un usuario navegar varias veces hacia atrás y hacia delante por las páginas y ejecutar una función que solo debería ejecutarse una vez? ¿Puede cargar y descargar carritos de compra para obtener descuentos adicionales?

## Casos de prueba relacionados

- [Enumeración de cuentas y cuentas de usuario fáciles de adivinar](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md)
- [Mecanismo de bloqueo débil](../04-Authentication/03-Weak_Lock_Out_Mechanism.md)

## Remediación

La aplicación debería establecer controles estrictos para prevenir el abuso de los límites. Esto puede lograrse haciendo que un cupón deje de ser válido en el nivel de la base de datos, o estableciendo un contador por usuario en el backend o en la base de datos, ya que todos los usuarios deberían identificarse mediante una sesión. La opción elegida dependerá de los requisitos del negocio.

## Referencias

- [Gold Trading Was Temporarily Halted On The CME This Morning](https://www.businessinsider.com/gold-halted-on-cme-for-stop-logic-event-2013-10)