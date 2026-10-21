# Funcionalidad de Pago

|ID          |
|------------|
|WSTG-BUSL-10|

## Resumen

Muchas aplicaciones implementan funcionalidades de pago, incluidos sitios de comercio electrónico, suscripciones, organizaciones benéficas, sitios de donaciones y casas de cambio. La seguridad de estas funcionalidades es crítica, ya que las vulnerabilidades podrían permitir a los atacantes robar a la organización, realizar compras fraudulentas o incluso robar los datos de tarjetas de pago de otros usuarios. Estos problemas podrían causar no solo daños a la reputación de la organización, sino también pérdidas financieras significativas, tanto por pérdidas directas como por multas impuestas por los organismos reguladores del sector.

## Objetivos de Prueba

- Determinar si la lógica de negocio de la funcionalidad de comercio electrónico es robusta.
- Entender cómo funciona la funcionalidad de pago.
- Determinar si la funcionalidad de pago es segura.

## Cómo Probar

### Métodos de Integración de Pasarelas de Pago

Hay varias formas de integrar funcionalidades de pago en las aplicaciones, y el enfoque de prueba variará según cuál se utilice. Los métodos más comunes son:

- Redirigir al usuario a una pasarela de pago de terceros.
- Cargar una pasarela de pago de terceros en un IFRAME de la aplicación.
- Usar un formulario HTML que realice una solicitud POST entre dominios a una pasarela de pago de terceros.
- Aceptar directamente los datos de la tarjeta y, a continuación, realizar una solicitud POST desde el backend de la aplicación a la API de la pasarela de pago.

### PCI DSS

El Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS) es un estándar que las organizaciones deben cumplir para procesar pagos con tarjetas de débito y crédito (aunque es importante señalar que no es una ley). Un análisis completo de este estándar queda fuera del alcance de esta guía (y de la mayoría de las pruebas de penetración), pero es útil que los evaluadores conozcan algunos puntos clave.

La idea errónea más común sobre PCI DSS es que solo se aplica a los sistemas que almacenan datos de titulares de tarjetas (es decir, datos de tarjetas de débito o crédito). Esto es incorrecto: se aplica a cualquier sistema que «almacene, procese o transmita» esta información. Los requisitos que deben cumplirse dependen de cuál de los métodos de integración de la pasarela de pago se utilice. La [guía de Visa sobre el procesamiento de pagos de comercio electrónico](https://web.archive.org/web/2023/https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf) ofrece más detalles al respecto; a modo de breve resumen:

| Método de integración | Cuestionario de autoevaluación (SAQ) |
|----------------------|-------------------------------------|
| Redirect | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| IFRAME | [SAQ A](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-A.pdf) |
| Cross-domain POST | [SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP-rev1_1.pdf) |
| Backend API | [SAQ D](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2_1-SAQ-D_Merchant.pdf) |

Además de las diferencias en la superficie de ataque y el perfil de riesgo de cada enfoque, también hay una diferencia significativa entre la cantidad de requisitos que la organización debe cumplir: SAQ A tiene 22 requisitos y SAQ D, 329. Por ello, conviene señalar las aplicaciones que no utilizan una redirección o un IFRAME, ya que representan mayores riesgos técnicos y de cumplimiento.

### Manipulación de Cantidad

La mayoría de los sitios de comercio electrónico permiten a los usuarios añadir artículos a un carrito antes de iniciar el proceso de compra. El carrito debe llevar un registro de los artículos añadidos y de la cantidad de cada uno. La cantidad normalmente debe ser un entero positivo, pero si el sitio no la valida correctamente, podría ser posible especificar una cantidad decimal de un artículo (como `0.1`) o una cantidad negativa (como `-1`). Según el procesamiento del backend, añadir cantidades negativas de un artículo podría dar como resultado un valor negativo, lo que reduciría el costo total del carrito.

Por lo general, hay varias formas de modificar el contenido del carrito que deberían probarse, por ejemplo:

- Añadir una cantidad negativa de un artículo.
- Eliminar artículos repetidamente hasta que la cantidad sea negativa.
- Actualizar la cantidad a un valor negativo.

Algunos sitios también pueden ofrecer un menú desplegable con cantidades válidas (por ejemplo, artículos que deben comprarse en paquetes de 10), y podría ser posible manipular estas solicitudes para añadir otras cantidades.

Si se pasan los detalles completos del carrito a la pasarela de pago (en lugar de pasar simplemente un valor total), también podría ser posible manipular los valores en esa etapa.

Por último, si la aplicación es vulnerable a la [contaminación de parámetros HTTP](../07-Injection/04-HTTP_Parameter_Pollution.md), podría ser posible provocar un comportamiento inesperado pasando un parámetro varias veces, por ejemplo:

```http
POST /api/basket/add
Host: example.org

item_id=1&quantity=5&quantity=4
```

### Manipulación de Precios

#### En la Aplicación

Al añadir un artículo al carrito, la aplicación solo debería incluir el artículo y una cantidad, como en la solicitud de ejemplo siguiente:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5
```

Sin embargo, en algunos casos la aplicación también podría incluir el precio, lo que permitiría posiblemente manipularlo:

```http
POST /api/basket/add HTTP/1.1
Host: example.org

item_id=1&quantity=5&price=2.00
```

Los distintos tipos de artículos pueden tener reglas de validación diferentes, por lo que cada tipo debe probarse por separado. Algunas aplicaciones también permiten a los usuarios añadir una donación opcional a una organización benéfica como parte de su compra; normalmente, esta donación puede ser de cualquier importe. Si no se valida, podría ser posible añadir un importe de donación negativo, lo que reduciría el valor total del carrito.

#### En la Pasarela de Pago

Si el proceso de compra se realiza en una pasarela de pago de terceros, podría ser posible manipular los precios entre la aplicación y la pasarela.

La transferencia a la pasarela podría realizarse mediante una solicitud POST entre dominios, como se muestra en el siguiente ejemplo HTML.

> Nota: Los datos de la tarjeta no se incluyen en esta solicitud; se le pedirán al usuario en la pasarela de pago:

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

Al modificar el formulario HTML o interceptar la solicitud POST, podría ser posible modificar los precios de los artículos y, en la práctica, comprarlos por menos. Hay que tener en cuenta que muchas pasarelas de pago rechazarán una transacción cuyo valor sea cero, por lo que es más probable que se acepte un total de 0.01. Sin embargo, algunas pasarelas de pago podrían aceptar valores negativos (que se utilizan para procesar reembolsos). Si hay varios valores (como los precios de los artículos, el costo de envío y el costo total del carrito), deberían probarse todos.

Si, en cambio, la pasarela de pago utiliza un IFRAME, podría ser posible realizar un ataque similar modificando la URL del IFRAME:

```html
<iframe src="https://example.org/payment_iframe?merchant_id=123&basket_total=22.00" />
```

> Nota: Las pasarelas de pago suelen estar gestionadas por terceros y, por ello, podrían quedar fuera del alcance de las pruebas. Esto significa que, si bien la manipulación de precios podría ser aceptable, no deberían realizarse otros tipos de ataques (como la inyección SQL) sin aprobación explícita por escrito.

#### Detalles de Transacciones Cifrados

Para evitar que se manipule la transacción, algunas pasarelas de pago cifran los detalles de la solicitud que reciben. Por ejemplo, [PayPal](https://developer.paypal.com/api/nvp-soap/paypal-payments-standard/integration-guide/encryptedwebpayments/#link-usingewptoprotectmanuallycreatedpaymentbuttons) lo hace mediante criptografía de clave pública.

Lo primero que se debe probar es enviar una solicitud sin cifrar, ya que algunas pasarelas de pago permiten transacciones inseguras si no se han configurado específicamente para rechazarlas.

Si esto no funciona, hay que encontrar la clave pública que se utiliza para cifrar los detalles de la transacción. Esta podría estar expuesta en una copia de seguridad de la aplicación o, si se encuentra una vulnerabilidad de recorrido de directorios.

También es posible que la aplicación reutilice el mismo par de claves pública/privada para la pasarela de pago y su certificado digital. Se puede obtener la clave pública del servidor con el siguiente comando:

```bash
echo -e '\0' | openssl s_client -connect example.org:443 2>/dev/null | openssl x509 -pubkey -noout
```

Una vez obtenida esta clave, se puede intentar crear una solicitud cifrada (basada en la documentación de la pasarela de pago) y enviarla a la pasarela para comprobar si la acepta.

#### Hashes Seguros

Otras pasarelas de pago utilizan un hash seguro (o un HMAC) de los detalles de la transacción para evitar manipulaciones. La forma exacta de hacerlo varía según el proveedor (por ejemplo, [Adyen](https://docs.adyen.com/online-payments/classic-integrations/hosted-payment-pages/hmac-signature-calculation) utiliza HMAC-SHA256), pero normalmente incluirá los detalles de la transacción y un valor secreto. Por ejemplo, un hash podría calcularse así:

```php
$secure_hash = md5($merchant_id . $transaction_id . $items . $total_value . $secret)
```

Este valor se añade a la solicitud POST que se envía a la pasarela de pago y se verifica para asegurar que la transacción no haya sido manipulada.

Lo primero que se debe probar es eliminar el hash seguro, ya que algunas pasarelas de pago permiten transacciones inseguras si no se ha establecido una opción de configuración específica.

La solicitud POST debería contener todos los valores necesarios para calcular este hash, excepto la clave secreta. Esto significa que, si se sabe cómo se calcula el hash (lo que debería estar indicado en la documentación de la pasarela de pago), se puede intentar forzar por fuerza bruta el secreto. Otra posibilidad es que, si el sitio ejecuta una aplicación lista para usar, haya un secreto predeterminado en los archivos de configuración o en el código fuente. Por último, si se encuentra una copia de seguridad del sitio o se obtiene acceso a los archivos de configuración por otros medios, quizá se pueda encontrar allí el secreto.

Si se obtiene este secreto, se pueden manipular los detalles de la transacción y generar un hash seguro propio que la pasarela de pago aceptará.

#### Manipulación de Moneda

Si no es posible manipular los precios reales, podría ser posible cambiar la moneda utilizada, especialmente si la aplicación admite varias monedas. Por ejemplo, la aplicación podría validar que el precio es 10, pero si se puede cambiar la moneda para pagar 10 USD en lugar de 10 GBP, se podrían comprar los artículos a un precio menor.

#### Solicitudes Retrasadas

Si el valor de los artículos del sitio cambia con el tiempo (por ejemplo, en una casa de cambio), podría ser posible comprar o vender al precio anterior interceptando las solicitudes con un proxy local y retrasándolas. Para que esto sea explotable, el precio tendría que estar incluido en la solicitud o vinculado a algo que esta contenga (como la sesión o el ID de transacción). El ejemplo siguiente muestra cómo podría explotarse esto en una aplicación que permite comprar y vender oro:

- Consultar el precio actual del oro en el sitio.
- Iniciar una solicitud para comprar 1oz de oro.
- Interceptar y congelar la solicitud.
- Esperar un minuto y volver a consultar el precio del oro:
    - Si aumenta, permitir que se complete la transacción y comprar el oro por menos de su valor actual.
    - Si disminuye, descartar la solicitud.

Si el sitio permite al usuario realizar pagos con criptomonedas (que suelen ser mucho más volátiles), podría ser posible explotar esto obteniendo un precio fijo en esa criptomoneda y esperando para ver si su valor sube o baja en comparación con la moneda principal que utiliza el sitio.

### Códigos de Descuento

Si la aplicación admite códigos de descuento, deberían realizarse varias comprobaciones:

- ¿Son fáciles de adivinar los códigos (TEST, TEST10, SORRY, SORRY10, nombre de la empresa, etc.)?
    - Si un código contiene un número, ¿se pueden encontrar más códigos incrementando ese número?
- ¿Hay protección contra ataques de fuerza bruta?
- ¿Se pueden aplicar varios códigos de descuento a la vez?
- ¿Se pueden aplicar los códigos de descuento varias veces?
- ¿Se pueden [inyectar caracteres comodín](../07-Injection/05-SQL_Injection.md#sql-wildcard-injection) como `%` o `*`?
- ¿Se exponen códigos de descuento en el código fuente HTML o en campos `<input>` ocultos de la aplicación?

Además de estas comprobaciones, deberían probarse las vulnerabilidades habituales, como la inyección SQL.

### Alteración de los Flujos de Pago

Si el proceso de compra o pago de una aplicación tiene varias etapas (como añadir artículos al carrito, introducir códigos de descuento, proporcionar los datos de envío e introducir la información de facturación), podría ser posible provocar un comportamiento no previsto al realizar estos pasos fuera de la secuencia esperada. Por ejemplo, se podría intentar:

- Modificar la dirección de envío después de introducir los datos de facturación para reducir los costos de envío.
- Eliminar artículos después de introducir los datos de envío para evitar un valor mínimo del carrito.
- Modificar el contenido del carrito después de aplicar un código de descuento.
- Modificar el contenido del carrito después de completar el proceso de compra.

También podría ser posible omitir todo el proceso de pago de la transacción. Por ejemplo, si la aplicación redirige a una pasarela de pago de terceros, el flujo de pago podría ser el siguiente:

- El usuario introduce sus datos en la aplicación.
- Se redirige al usuario a la pasarela de pago de terceros.
- El usuario introduce los datos de su tarjeta.
    - Si el pago se realiza correctamente, se le redirige a `success.php` en la aplicación.
    - Si el pago no se realiza correctamente, se le redirige a `failure.php` en la aplicación.
- La aplicación actualiza su base de datos de pedidos y procesa el pedido si el pago se realizó correctamente.

Dependiendo de si la aplicación valida realmente que el pago en la pasarela se haya realizado correctamente, podría ser posible acceder directamente a la página `success.php` (posiblemente incluyendo un ID de transacción, si es necesario), lo que haría que el sitio procesara el pedido como si el pago se hubiera realizado correctamente. Además, podría ser posible enviar solicitudes repetidas a la página `success.php` para que un pedido se procese varias veces.

### Explotación de las Comisiones por Procesamiento de Transacciones

Los comerciantes normalmente deben pagar comisiones por cada transacción procesada, que suelen componerse de una pequeña tarifa fija y un porcentaje del valor total. Esto significa que recibir pagos muy pequeños (como $0.01) podría hacer que el comerciante pierda dinero, ya que las comisiones por procesamiento de transacciones superan el valor total de la transacción.

Este problema rara vez es explotable en sitios de comercio electrónico, ya que el precio del artículo más barato suele ser lo suficientemente alto como para impedirlo. Sin embargo, si el sitio permite a los clientes realizar pagos por importes arbitrarios (como donaciones), hay que comprobar que imponga un importe mínimo razonable.

### Tarjetas de Pago de Prueba

La mayoría de las pasarelas de pago tienen un conjunto definido de datos de tarjetas de prueba que los desarrolladores pueden utilizar durante las pruebas y la depuración. Estos datos solo deberían poder utilizarse en versiones de desarrollo o de pruebas aisladas (sandbox) de las pasarelas, pero podrían aceptarse en sitios en producción si se han configurado incorrectamente.

A continuación se muestran ejemplos de estos datos de prueba para varias pasarelas de pago:

- [Adyen - Números de tarjetas de prueba](https://docs.adyen.com/development-resources/test-cards-and-credentials/test-card-numbers)
- [Globalpay - Tarjetas de prueba](https://developer.globalpayments.com/resources/test-cards)
- [Stripe - Números básicos de tarjetas de prueba](https://docs.stripe.com/testing#cards)

### Logística

Probar la funcionalidad de pago de las aplicaciones puede añadir complejidad, especialmente si se prueba un sitio en producción. Entre los aspectos que deben tenerse en cuenta se incluyen:

- Obtener datos de tarjetas de prueba para la aplicación.
    - Si no están disponibles, quizá sea posible obtener una tarjeta prepaga u otra alternativa.
- Mantener un registro de los pedidos realizados para poder cancelarlos y reembolsarlos.
- No realizar pedidos que no se puedan cancelar o que provoquen otras acciones (como el envío inmediato de productos desde un almacén).

#### Origen Igual al Destino

Si el origen de una transferencia es igual al destino, podría simplemente añadirse valor a la cuenta sin que se produzca ninguna transferencia real. Este escenario debería probarse para asegurar que la aplicación impida estas operaciones.

#### Pagos o Transferencias en Dos Pasos

En los pagos o transferencias que requieren dos pasos (iniciación y confirmación), hay que asegurar que se realicen comprobaciones durante ambas fases. Por ejemplo:

- Iniciar dos pagos distintos.
- Confirmarlos por separado.

Hay que verificar que las comprobaciones necesarias, como los límites diarios o las validaciones de saldo, se realicen durante la fase de confirmación. Si no se hace, podrían producirse saldos negativos o eludir los límites.

#### Añadir Artículos Después de Iniciar el Pago

Probar el escenario en el que se inicia un pago y después se añaden artículos al carrito. La confirmación del pago podría hacer que los artículos añadidos se marquen como pagados, lo que podría provocar inconsistencias en el proceso de pago.

#### Condiciones de Carrera

Una condición de carrera ocurre cuando el comportamiento de una aplicación depende de la temporización o la secuencia de eventos que no están controlados correctamente. En el contexto de los pagos, suele manifestarse cuando varias solicitudes concurrentes interactúan con un estado compartido (como el saldo de una cuenta, la cantidad de artículos en inventario o un código de cupón de un solo uso) sin una sincronización adecuada, lo que permite a un atacante explotar el intervalo de tiempo entre una comprobación y la acción posterior. Esto se conoce comúnmente como una falla de tiempo de comprobación a tiempo de uso (Time-of-Check to Time-of-Use, TOCTOU).

Las condiciones de carrera tienen un impacto particular en:

- Transacciones financieras (transferencias, pagos, retiros)
- Tokens o códigos de un solo uso (cupones, tarjetas de regalo, bonificaciones por recomendación)
- Asignación de recursos con disponibilidad limitada (inventario, asientos, inscripciones)
- Transiciones de estado que deberían producirse exactamente una vez (activación de cuentas, flujos de aprobación)

Para probarlas, hay que centrarse en los endpoints donde se comprueba un valor y luego se modifica, o donde una acción solo debería tener éxito una vez. Enviar varias solicitudes idénticas simultáneamente y observar si la aplicación procesa correctamente más de una cuando solo debería procesar una.

Con curl, enviar 20 solicitudes rápidas a un endpoint que modifica el estado:

```bash
# Send 20 near-concurrent requests to redeem a single-use coupon
for i in $(seq 1 20); do
  curl -s -X POST https://example.com/api/redeem-coupon \
    -H "Cookie: session=USER_SESSION" \
    -H "Content-Type: application/json" \
    -d ‘{"code":"SINGLE-USE-CODE"}’ &
done
wait
```

Para lograr una temporización más precisa, se puede usar [Burp Suite Turbo Intruder](https://github.com/PortSwigger/turbo-intruder) con el mecanismo gate, que permite retener todas las solicitudes y liberarlas simultáneamente:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=20,
                           requestsPerConnection=1,
                           pipeline=False)
    for i in range(20):
        engine.queue(target.req, gate=’race’)
    engine.openGate(‘race’)

def handleResponse(req, interesting):
    table.add(req)
```

Entre los escenarios de prueba habituales se incluyen:

- Confirmaciones de pago concurrentes:
  Iniciar simultáneamente varias solicitudes de confirmación (por ejemplo, `POST /confirm-payment`) para el mismo pedido. Esto podría hacer que el mismo pedido se procese varias veces.

- Repetición o inundación de callbacks:
  Interceptar la solicitud de callback de la pasarela (por ejemplo, a `success.php` o `/payment/callback`) y reproducirla rápidamente en paralelo. Si el backend carece de comprobaciones de idempotencia adecuadas, esto puede:
    - Activar varios eventos de gestión del pedido (por ejemplo, envíos o créditos).
    - Marcar el mismo pedido como «pagado» varias veces.
    - Provocar la inflación de saldos o errores de inventario.

- Operaciones de saldo:
  Si una cuenta tiene 100 créditos, enviar varias solicitudes concurrentes que gasten cada una el saldo completo. Si el total debitado supera el saldo original (por ejemplo, si el saldo llega a -100), la aplicación es vulnerable.

- Tokens de un solo uso:
  En el caso de códigos de cupón, enlaces de invitación o tokens de restablecimiento de contraseña, enviar varias solicitudes concurrentes que intenten usar el token. Si más de una tiene éxito, la aplicación no invalida el token de forma atómica.

Entre los indicios de vulnerabilidad se incluyen varias respuestas de éxito para una acción de un solo uso, saldos negativos después de débitos concurrentes, recursos asignados por encima de la capacidad indicada o efectos secundarios duplicados (varios correos electrónicos, cargos duplicados).

Para mitigar las condiciones de carrera, se deben incluir las operaciones de comprobación y modificación en transacciones de base de datos con bloqueo a nivel de fila (`SELECT ... FOR UPDATE`), implementar bloqueo optimista con columnas de versión, exigir claves de idempotencia para las solicitudes que modifican el estado e invalidar tokens de forma atómica mediante operaciones como `DELETE FROM tokens WHERE token = ? RETURNING *`.

Véase también [CWE-362: Ejecución concurrente usando un recurso compartido con sincronización inadecuada](https://cwe.mitre.org/data/definitions/362.html).

#### Sistemas con Múltiples Entradas (Pagos en Lote)

En los sistemas que admiten pagos en lote, hay que probar escenarios en los que el importe total siga siendo positivo, pero algunas entradas individuales tengan valores negativos. Por ejemplo:

```plaintext
account_id_1 = $5
account_id_2 = -$4
Total = $1 paid, but $5 credited
```

Hay que asegurar que la aplicación gestione correctamente estos casos e impida su explotación.

## Casos de Prueba Relacionados

- [Contaminación de Parámetros HTTP](../07-Injection/04-HTTP_Parameter_Pollution.md)
- [Inyección SQL](../07-Injection/05-SQL_Injection.md)
- [la elusión de los flujos de trabajo](06-Circumvention_of_Work_Flows.md)

## Remediación

- Evitar almacenar, transmitir o procesar datos de tarjetas siempre que sea posible.
    - Usar una redirección o un IFRAME para la pasarela de pago.
- Revisar la documentación de la pasarela de pago y utilizar todas las funciones de seguridad disponibles (como el cifrado y los hashes seguros).
- Gestionar en el servidor toda la información relacionada con los precios:
    - Las únicas cosas incluidas en las solicitudes del lado del cliente deberían ser los ID de los artículos y las cantidades.
- Implementar una validación de entradas adecuada y restricciones de lógica de negocio (como comprobar si hay números o valores negativos de artículos).
- Asegurar que el flujo de pago de la aplicación sea robusto y que los pasos no puedan realizarse fuera de secuencia.

## Referencias

- [Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS)](https://www.pcisecuritystandards.org/document_library/)
- [Guía de Visa sobre el procesamiento de pagos de comercio electrónico](https://web.archive.org/web/2023/https://www.visa.co.uk/dam/VCOM/regional/ve/unitedkingdom/PDF/risk/processing-e-commerce-payments-guide-73-17337.pdf)