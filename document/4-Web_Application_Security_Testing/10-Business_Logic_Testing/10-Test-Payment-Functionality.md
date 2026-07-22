# Probar la Funcionalidad de Pago

|ID          |
|------------|
|WSTG-BUSL-10|

## Resumen

Muchas aplicaciones implementan funcionalidad de pago, incluyendo sitios de e-commerce, suscripciones, caridades, sitios de donación y casas de cambio. La seguridad de esta funcionalidad es crítica, ya que las vulnerabilidades podrían permitir a los atacantes robar a la organización, hacer compras fraudulentas, o incluso robar detalles de tarjetas de pago de otros usuarios. Estos problemas podrían resultar no solo en daño reputacional a la organización, sino también en pérdidas financieras significativas, tanto por pérdidas directas como por multas de reguladores de la industria.

## Objetivos de Prueba

- Determinar si la lógica de negocio para la funcionalidad de e-commerce es robusta.
- Entender cómo funciona la funcionalidad de pago.
- Determinar si la funcionalidad de pago es segura.

## Cómo Probar

### Métodos de Integración de Pasarela de Pago

Hay varias maneras diferentes en que las aplicaciones pueden integrar funcionalidad de pago, y el enfoque de prueba variará dependiendo de cuál se use. Los métodos más comunes son:

- Redirigir al usuario a una pasarela de pago de terceros.
- Cargar una pasarela de pago de terceros en un IFRAME en la aplicación.
- Tener un formulario HTML que hace una solicitud POST cross-domain a una pasarela de pago de terceros.
- Aceptar los detalles de la tarjeta directamente, y luego hacer un POST desde el backend de la aplicación a la API de la pasarela de pago.

### PCI DSS

El Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS, por sus siglas en inglés) es un estándar que las organizaciones están obligadas a seguir para procesar pagos con tarjetas de débito y crédito (aunque es importante notar que no es una ley). Una discusión completa de este estándar está fuera del alcance de esta guía (y de la mayoría de las pruebas de penetración) - pero es útil para los testers entender algunos puntos clave.

La idea errónea más común sobre PCI DSS es que solo aplica a sistemas que almacenan datos de tarjetahabiente (i.e., detalles de tarjetas de débito o crédito). Esto es incorrecto: aplica a cualquier sistema que "almacene, procese o transmita" esta información. Exactamente qué requisitos necesitan seguirse depende de cuál de los métodos de integración de pasarela de pago se use. La [guía de Visa Processing E-Commerce Payments](https://web.archive.org/web/2023/https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf) proporciona más detalles sobre esto, pero como un breve resumen:

| Método de Integración | Cuestionario de Autoevaluación (SAQ) |
|-----------------------|--------------------------------------|
| Redirect              | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| IFRAME                | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| Cross-domain POST     | [SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP-rev1_1.pdf) |
| Backend API           | [SAQ D](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-D_Merchant.pdf) |

Además de las diferencias en la superficie de ataque y el perfil de riesgo de cada enfoque, también hay una diferencia significativa en el número de requisitos entre SAQ A (22 requisitos) y SAQ D (329 requisitos) que la organización necesita cumplir. Como tal, vale la pena resaltar aplicaciones que no están usando una redirección o IFRAME, ya que representan riesgos técnicos y de cumplimiento aumentados.

### Manipulación de Cantidad

La mayoría de los sitios de e-commerce permiten a los usuarios añadir ítems a una canasta antes de que comiencen el proceso de checkout. Esta canasta debería llevar registro de qué ítems se han añadido, y la cantidad de cada ítem. La cantidad debería ser normalmente un entero positivo, pero si el sitio no valida esto apropiadamente entonces podría ser posible especificar una cantidad decimal de un ítem (tal como `0.1`), o una cantidad negativa (tal como `-1`). Dependiendo del procesamiento del backend, añadir cantidades negativas de un ítem podría resultar en un valor negativo, reduciendo el costo total de la canasta.

Habitualmente hay múltiples maneras de modificar los contenidos de la canasta que deberían probarse, tales como:

- Añadir una cantidad negativa de un ítem.
- Remover ítems repetidamente hasta que la cantidad sea negativa.
- Actualizar la cantidad a un valor negativo.

Algunos sitios también podrían proporcionar un menú desplegable de cantidades válidas (tales como ítems que deben comprarse en paquetes de 10), y podría ser posible manipular estas solicitudes para añadir otras cantidades de ítems.

Si los detalles completos de la canasta se pasan a la pasarela de pago (en lugar de simplemente pasar un valor total), también podría ser posible manipular los valores en esa etapa.

Finalmente, si la aplicación es vulnerable a [contaminación de parámetros HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md) entonces podría ser posible causar comportamiento inesperado pasando un parámetro múltiples veces, tal como:

```http
POST /api/basket/add
Host: example.org

item_id=1&quantity=5&quantity=4
```

### Manipulación de Precio

#### En la Aplicación

Al añadir un ítem a la canasta, la aplicación debería solo incluir el ítem y una cantidad, tal como la solicitud de ejemplo a continuación:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5
```

Sin embargo, en algunos casos la aplicación también podría incluir el precio, lo que significa que podría ser posible manipularlo:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5&price=2.00
```

Diferentes tipos de ítems podrían tener diferentes reglas de validación, por lo que cada tipo necesita probarse separadamente. Algunas aplicaciones también permiten a los usuarios añadir una donación opcional a caridad como parte de su compra, y esta donación puede usualmente ser un monto arbitrario. Si este monto no se valida, podría ser posible añadir un monto de donación negativo, lo cual entonces reduciría el valor total de la canasta.

#### En la Pasarela de Pago

Si el proceso de checkout se realiza en una pasarela de pago de terceros, entonces podría ser posible manipular los precios entre la aplicación y la pasarela.

La transferencia a la pasarela podría realizarse usando un POST cross-domain a la pasarela, como se muestra en el ejemplo HTML a continuación.

> Nota: Los detalles de la tarjeta no se incluyen en esta solicitud - al usuario se le pedirán en la pasarela de pago:

```html
<form action="https://example.org/process_payment" method="POST">
    <input type="hidden" id="merchant_id" value="123" />
    <input type="hidden" id="basket_id" value="456" />
    <input type="hidden" id="item_id" value="1" />
    <input type="hidden" id="item_quantity" value="5" />
    <input type="hidden" id="item_total" value="20.00" />
    <input type="hidden" id="shipping_total" value="2.00" />
    <input type="hidden" id="basket_total" value="22.00" />
    <input type="hidden" id="currency" value="GBP" />
    <input type="submit" id="submit" value="submit" />
</form>
```

Al modificar el formulario HTML o interceptar la solicitud POST, podría ser posible modificar los precios de los ítems, y efectivamente comprarlos por menos. Considerar que muchas pasarelas de pago rechazarán una transacción con un valor de cero, por lo que un total de 0.01 es más probable que tenga éxito. Sin embargo, algunas pasarelas de pago podrían aceptar valores negativos (usados para procesar reembolsos). Donde hay múltiples valores (tales como precios de ítems, un costo de envío, y el costo total de la canasta), todos estos deberían probarse.

Si la pasarela de pago usa un IFRAME en su lugar, podría ser posible realizar un tipo similar de ataque modificando la URL del IFRAME:

```html
<iframe src="https://example.org/payment_iframe?merchant_id=123&basket_total=22.00" />
```

> Nota: Las pasarelas de pago usualmente son operadas por terceros, y como tal podrían no estar incluidas en el alcance de las pruebas. Esto significa que mientras la manipulación de precios podría ser aceptable, otros tipos de ataques (tales como inyección SQL) no deberían realizarse sin aprobación escrita explícita.

#### Detalles de Transacción Cifrados

Para prevenir que la transacción sea manipulada, algunas pasarelas de pago cifrarán los detalles de la solicitud que se les hace. Por ejemplo, [PayPal](https://developer.paypal.com/api/nvp-soap/paypal-payments-standard/integration-guide/encryptedwebpayments/#link-usingewptoprotectmanuallycreatedpaymentbuttons) hace esto usando criptografía de clave pública.

Lo primero que intentar es hacer una solicitud no cifrada, ya que algunas pasarelas de pago permiten transacciones inseguras a menos que se hayan configurado específicamente para rechazarlas.

Si esto no funciona, entonces necesitas encontrar la clave pública que se usa para cifrar los detalles de la transacción, la cual podría estar expuesta en un respaldo de la aplicación, o si puedes encontrar una vulnerabilidad de directory traversal.

Alternativamente, es posible que la aplicación reutilice el mismo par de claves pública/privada para la pasarela de pago y su certificado digital. Puedes obtener la clave pública del servidor con el siguiente comando:

```bash
echo -e '\0' | openssl s_client -connect example.org:443 2>/dev/null | openssl x509 -pubkey -noout
```

Una vez que tienes esta clave, puedes intentar y crear una solicitud cifrada (basada en la documentación de la pasarela de pago), y enviarla a la pasarela para ver si se acepta.

#### Hashes Seguros

Otras pasarelas de pago usan un hash seguro (o un HMAC) de los detalles de la transacción para prevenir manipulación. Los detalles exactos de cómo se hace esto variarán entre proveedores (por ejemplo, [Adyen](https://docs.adyen.com/online-payments/classic-integrations/hosted-payment-pages/hmac-signature-calculation) usa HMAC-SHA256), pero normalmente incluirá los detalles de la transacción y un valor secreto. Por ejemplo, un hash podría calcularse como:

```php
$secure_hash = md5($merchant_id . $transaction_id . $items . $total_value . $secret)
```

Este valor entonces se añade a la solicitud POST que se envía a la pasarela de pago, y se verifica para asegurar que la transacción no haya sido manipulada.

Lo primero que intentar es remover el hash seguro, ya que algunas pasarelas de pago permiten transacciones inseguras a menos que se haya establecido una opción de configuración específica.

La solicitud POST debería contener todos los valores requeridos para calcular este hash, aparte de la clave secreta. Esto significa que si sabes cómo se calcula el hash (lo cual debería incluirse en la documentación de la pasarela de pago), entonces puedes intentar forzar bruscamente el secreto. Alternativamente, si el sitio está ejecutando una aplicación comercial, podría haber un secreto por defecto en los archivos de configuración o código fuente. Finalmente, si puedes encontrar un respaldo del sitio, o de otra manera obtener acceso a los archivos de configuración, podrías encontrar el secreto allí.

Si puedes obtener este secreto, puedes entonces manipular los detalles de la transacción, y luego generar tu propio hash seguro que será aceptado por la pasarela de pago.

#### Manipulación de Moneda

Si no es posible manipular los precios reales, podría ser posible cambiar la moneda que se usa, especialmente donde las aplicaciones soportan múltiples monedas. Por ejemplo, la aplicación podría validar que el precio es 10, pero si puedes cambiar la moneda para que pagues 10 USD en lugar de 10 GBP, esto te permitiría comprar ítems más baratamente.

#### Solicitudes Retrasadas en Tiempo

Si el valor de los ítems en el sitio cambia con el tiempo (por ejemplo en una casa de cambio), entonces podría ser posible comprar o vender a un precio viejo interceptando solicitudes usando un proxy local y retrasándolas. Para que esto sea explotable, el precio necesitaría o bien estar incluido en la solicitud, o vinculado a algo en la solicitud (tal como sesión o ID de transacción). El ejemplo a continuación muestra cómo esto potencialmente podría explotarse en una aplicación que permite a los usuarios comprar y vender oro:

- Ver el precio actual del oro en el sitio.
- Iniciar una solicitud de compra de 1oz de oro.
- Interceptar y congelar la solicitud.
- Esperar un minuto para verificar el precio del oro de nuevo:
    - Si aumenta, permitir que la transacción se complete, y comprar el oro por menos de su valor actual.
    - Si disminuye, soltar la solicitud.

Si el sitio permite al usuario hacer pagos usando criptomonedas (las cuales son usualmente mucho más volátiles), podría ser posible explotar esto obteniendo un precio fijo en esa criptomoneda, y luego esperar para ver si el valor sube o baja comparado con la moneda principal usada por el sitio.

### Códigos de Descuento

Si la aplicación soporta códigos de descuento, entonces hay varias verificaciones que deberían llevarse a cabo:

- ¿Los códigos son fácilmente adivinables (TEST, TEST10, SORRY, SORRY10, nombre de la empresa, etc)?
    - Si un código tiene un número, ¿se pueden encontrar más códigos incrementando el número?
- ¿Hay alguna protección contra fuerza bruta?
- ¿Se pueden aplicar múltiples códigos de descuento a la vez?
- ¿Se pueden aplicar códigos de descuento múltiples veces?
- ¿Puedes [inyectar caracteres comodín](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection) tales como `%` o `*`?
- ¿Los códigos de descuento están expuestos en el código fuente HTML o campos `<input>` ocultos en cualquier lugar de la aplicación?

Además de estos, las vulnerabilidades habituales tales como inyección SQL deberían probarse.

### Rompiendo Flujos de Pago

Si el proceso de checkout o pago en una aplicación involucra múltiples etapas (tales como añadir ítems a una canasta, ingresar códigos de descuento, ingresar detalles de envío, e ingresar información de facturación), entonces podría ser posible causar comportamiento no previsto realizando estos pasos fuera de la secuencia esperada. Por ejemplo, podrías intentar:

- Modificar la dirección de envío después de que se hayan ingresado los detalles de facturación para reducir costos de envío.
- Remover ítems después de ingresar detalles de envío, para evitar un valor mínimo de canasta.
- Modificar los contenidos de la canasta después de aplicar un código de descuento.
- Modificar los contenidos de una canasta después de completar el proceso de checkout.

También podría ser posible saltarse el proceso de pago completo para la transacción. Por ejemplo, si la aplicación redirige a una pasarela de pago de terceros, el flujo de pago podría ser:

- El usuario ingresa detalles en la aplicación.
- El usuario es redirigido a la pasarela de pago de terceros.
- El usuario ingresa los detalles de su tarjeta.
    - Si el pago es exitoso, es redirigido a `success.php` en la aplicación.
    - Si el pago no es exitoso, es redirigido a `failure.php` en la aplicación
- La aplicación actualiza su base de datos de órdenes, y procesa la orden si fue exitosa.

Dependiendo de si la aplicación realmente valida que el pago en la pasarela fue exitoso, podría ser posible navegar por fuerza bruta a la página `success.php` (posiblemente incluyendo un ID de transacción si se requiere uno), lo cual causaría que el sitio procese la orden como si el pago hubiera sido exitoso. Adicionalmente, podría ser posible hacer solicitudes repetidas a la página `success.php` para causar que una orden sea procesada múltiples veces.

### Explotando Tarifas de Procesamiento de Transacciones

Los comerciantes normalmente tienen que pagar tarifas por cada transacción procesada, las cuales típicamente se componen de una pequeña tarifa fija, y un porcentaje del valor total. Esto significa que recibir pagos muy pequeños (tales como $0.01) podría resultar en el comerciante perdiendo dinero realmente, ya que las tarifas de procesamiento de transacciones son mayores que el valor total de la transacción.

Este problema raramente es explotable en sitios de e-commerce, ya que el precio del ítem más barato es usualmente lo suficientemente alto para prevenirlo. Sin embargo, si el sitio permite a los clientes hacer pagos con montos arbitrarios (tales como donaciones), verificar que imponga un valor mínimo sensato.

### Tarjetas de Pago de Prueba

La mayoría de las pasarelas de pago tienen un conjunto definido de detalles de tarjetas de prueba, las cuales pueden ser usadas por desarrolladores durante pruebas y depuración. Estas deberían solo ser utilizables en versiones de desarrollo o sandbox de las pasarelas, pero podrían ser aceptadas en sitios en vivo si han sido mal configuradas.

Ejemplos de estos detalles de prueba para varias pasarelas de pago se listan a continuación:

- [Adyen - Test Card Numbers](https://docs.adyen.com/development-resources/test-cards-and-credentials/test-card-numbers)
- [Globalpay - Test Cards](https://developer.globalpay.com/resources/test-card-numbers)
- [Stripe - Basic Test Card Numbers](https://stripe.com/docs/testing#cards)

### Logística de Pruebas

Probar la funcionalidad de pago en aplicaciones puede introducir complejidad adicional, especialmente si se está probando un sitio en vivo. Áreas que necesitan considerarse incluyen:

- Obtener detalles de pago de tarjeta de prueba para la aplicación.
    - Si estos no están disponibles, entonces podría ser posible obtener una tarjeta pre-pagada o una alternativa.
- Mantener un registro de cualquier orden que se haga para que puedan cancelarse y reembolsarse.
- No colocar órdenes que no puedan cancelarse, o que causen otras acciones (tales como bienes siendo despachados inmediatamente desde un almacén).

#### Origen Igual a Destino

Si el origen de la transferencia es igual al destino, podría resultar en simplemente añadir valor a la cuenta sin que ocurra ninguna transferencia real. Este escenario debería probarse para asegurar que la aplicación previene tales operaciones.

#### Pagos o Transferencias en Dos Pasos

Para pagos o transferencias que requieren dos pasos (iniciación y confirmación), asegurar que las verificaciones se realicen durante ambas fases. Por ejemplo:

- Iniciar dos pagos separados.
- Confirmarlos individualmente.

Verificar que las verificaciones necesarias, tales como límites diarios o validaciones de saldo, se realicen durante la fase de confirmación. Fallar en hacerlo podría llevar a saldos negativos o evasión de límites.

#### Añadir Ítems Después de la Iniciación del Pago

Probar el escenario donde se inicia un pago, y se añaden ítems al carrito después. Confirmar el pago podría resultar en marcar los ítems añadidos como pagados, lo cual podría llevar a inconsistencias en el proceso de pago.

#### Condiciones de Carrera (Race Conditions)

Una condición de carrera ocurre cuando el comportamiento de una aplicación depende del tiempo o secuencia de eventos que no están propiamente controlados. En contextos de pago, esto típicamente se manifiesta cuando múltiples solicitudes concurrentes interactúan con estado compartido (tal como un saldo de cuenta, un conteo de inventario, o un código de cupón de un solo uso) sin sincronización adecuada, permitiendo a un atacante explotar la brecha de tiempo entre una verificación y la acción subsiguiente. Esto se conoce comúnmente como una falla de Tiempo-de-Verificación a Tiempo-de-Uso (TOCTOU, por sus siglas en inglés).

Las condiciones de carrera son particularmente impactantes en:

- Transacciones financieras (transferencias, pagos, retiros)
- Tokens o códigos de un solo uso (cupones, tarjetas de regalo, bonos de referencia)
- Asignación de recursos con disponibilidad limitada (inventario, asientos, registros)
- Transiciones de estado que deberían ocurrir exactamente una vez (activación de cuenta, flujos de trabajo de aprobación)

Para probar, enfocarse en endpoints donde un valor se verifica y luego se modifica, o donde una acción debería solo tener éxito una vez. Enviar múltiples solicitudes idénticas simultáneamente y observar si la aplicación procesa más de una exitosamente cuando solo una debería tener éxito.

Usando curl, enviar 20 solicitudes rápidas a un endpoint que cambia estado:

```bash
# Enviar 20 solicitudes casi concurrentes para redimir un cupón de un solo uso
for i in $(seq 1 20); do
  curl -s -X POST https://example.com/api/redeem-coupon \
    -H "Cookie: session=USER_SESSION" \
    -H "Content-Type: application/json" \
    -d '{"code":"SINGLE-USE-CODE"}' &
done
wait
```

Para un tiempo más preciso, usar [Burp Suite Turbo Intruder](https://github.com/PortSwigger/turbo-intruder) con el mecanismo de gate para sostener todas las solicitudes y liberarlas simultáneamente:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=20,
                           requestsPerConnection=1,
                           pipeline=False)
    for i in range(20):
        engine.queue(target.req, gate='race')
    engine.openGate('race')

def handleResponse(req, interesting):
    table.add(req)
```

Escenarios de prueba comunes incluyen:

- Confirmaciones de Pago Concurrentes:
  Iniciar múltiples solicitudes de confirmación (por ejemplo, `POST /confirm-payment`) simultáneamente para la misma orden. Esto podría resultar en que la misma orden se procese múltiples veces.

- Replay o Flooding de Callbacks:
  Intercepta la solicitud de callback de la pasarela (por ejemplo, a `success.php` o `/payment/callback`) y reprodúcela rápidamente en paralelo. Si el backend carece de verificaciones de idempotencia apropiadas, esto puede:
    - Disparar múltiples eventos de cumplimiento de orden (por ejemplo, envío, créditos).
    - Marcar la misma orden como "pagada" múltiples veces.
    - Causar inflación de saldo o errores de inventario.

- Operaciones de Saldo:
  Si una cuenta tiene 100 créditos, enviar múltiples solicitudes concurrentes que cada una gaste el saldo completo. Si el total debitado excede el saldo original (por ejemplo, el saldo va a -100), la aplicación es vulnerable.

- Tokens de Un Solo Uso:
  Para códigos de cupón, enlaces de invitación, o tokens de reseteo de contraseña, enviar múltiples solicitudes concurrentes que cada una intente usar el token. Si más de una tiene éxito, la aplicación falla en invalidar atómicamente el token.

Indicadores de vulnerabilidad incluyen: múltiples respuestas de éxito para una acción de un solo uso, saldos negativos después de débitos concurrentes, recursos asignados más allá de la capacidad establecida, o efectos secundarios duplicados (múltiples correos, doble cargo).

Para mitigar condiciones de carrera, envolver operaciones de verificación-y-modificación en transacciones de base de datos con bloqueo a nivel de fila (`SELECT ... FOR UPDATE`), implementar bloqueo optimista con columnas de versión, requerir claves de idempotencia para solicitudes que cambian estado, e invalidar tokens atómicamente usando operaciones como `DELETE FROM tokens WHERE token = ? RETURNING *`.

Ver también [CWE-362: Concurrent Execution using Shared Resource with Improper Synchronization](https://cwe.mitre.org/data/definitions/362.html).

#### Sistemas de Entradas Múltiples (Pagos en Lote)

En sistemas que soportan pagos en lote, probar escenarios donde el monto total permanece positivo, pero las entradas individuales incluyen valores negativos. Por ejemplo:

```plaintext
account_id_1 = $5
account_id_2 = -$4
Total = $1 pagado, pero $5 acreditados
```

Asegurar que la aplicación maneja correctamente tales casos y previene explotación.

## Casos de Prueba Relacionados

- [Pruebas de Contaminación de Parámetros HTTP](../07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md)
- [Pruebas de Inyección SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md)
- [Pruebas de Evasión de Flujos de Trabajo](06-Testing_for_the_Circumvention_of_Work_Flows.md)

## Remediación

- Evitar almacenar, transmitir o procesar detalles de tarjeta donde sea posible.
    - Usar una redirección o IFRAME para la pasarela de pago.
- Revisar la documentación de la pasarela de pago y usar todas las características de seguridad disponibles (tales como cifrado y hashes seguros).
- Manejar toda la información relacionada con precios del lado del servidor:
    - Las únicas cosas incluidas en solicitudes del lado del cliente deberían ser IDs de ítem y cantidades.
- Implementar validación de entrada apropiada y restricciones de lógica de negocio (tales como verificar números o valores de ítem negativos).
- Asegurar que el flujo de pago de la aplicación sea robusto y que los pasos no puedan realizarse fuera de secuencia.

## Referencias

- [Payment Card Industry Data Security Standard (PCI DSS)](https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-2-1.pdf)
- [Visa Processing E-Commerce Payments guidance](https://web.archive.org/web/2023/https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf)
