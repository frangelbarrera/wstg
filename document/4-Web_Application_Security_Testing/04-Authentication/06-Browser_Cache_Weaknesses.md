# Pruebas de debilidades en la caché del navegador

|ID          |
|------------|
|WSTG-ATHN-06|

## Resumen

En esta fase, el probador verifica que la aplicación instruya correctamente al navegador para que no retenga datos sensibles.

Los navegadores pueden almacenar información con fines de caché e historial. La caché se utiliza para mejorar el rendimiento, de modo que la información que ya se mostró no tenga que descargarse nuevamente. Los mecanismos de historial se utilizan para comodidad del usuario, para que pueda ver exactamente lo que vio en el momento en que se recuperó el recurso. Si se muestra información sensible al usuario (como su dirección, los datos de su tarjeta de crédito, su número de Seguro Social o su nombre de usuario), esta información podría almacenarse con fines de caché o historial y, por lo tanto, recuperarse examinando la caché del navegador o simplemente presionando el botón **Atrás** del navegador.

## Objetivos de las pruebas

- Revisar si la aplicación almacena información sensible en el lado del cliente.
- Revisar si se puede acceder sin autorización.

## Cómo probar

### Historial del navegador

Técnicamente, el botón **Atrás** corresponde al historial y no a la caché (ver [Caching in HTTP: History Lists](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13)). La caché y el historial son dos entidades diferentes. Sin embargo, comparten la misma debilidad: presentar información sensible que ya se mostró.

La primera y más sencilla prueba consiste en ingresar información sensible en la aplicación y cerrar sesión. Luego, el probador hace clic en el botón **Atrás** del navegador para verificar si se puede acceder a la información sensible que ya se mostró mientras no se está autenticado.

Si al presionar el botón **Atrás** el probador puede acceder a páginas anteriores, pero no a páginas nuevas, entonces no se trata de un problema de autenticación, sino de un problema del historial del navegador. Si estas páginas contienen datos sensibles, significa que la aplicación no impidió que el navegador los almacenara.

No es necesario que las pruebas impliquen autenticación. Por ejemplo, cuando un usuario ingresa su dirección de correo electrónico para suscribirse a un boletín, esta información podría recuperarse si no se maneja correctamente.

Se puede impedir que el botón **Atrás** muestre datos sensibles mediante lo siguiente:

- Entregar la página a través de HTTPS.
- Establecer `Cache-Control: no-store`

`Cache-Control: no-store` indica al navegador que no almacene la respuesta en ninguna caché ni en el historial, lo que ayuda a impedir que los usuarios puedan acceder a datos sensibles al volver atrás después de cerrar sesión.

### Caché del navegador

En esta sección, los probadores verifican que la aplicación no filtre datos sensibles en la caché del navegador. Para ello, pueden usar un proxy (como ZAP) y buscar en las respuestas del servidor que pertenecen a la sesión, comprobando que, para cada página que contiene información sensible, el servidor haya indicado al navegador que no almacene ningún dato en caché. Esta directiva puede incluirse en los encabezados de respuesta HTTP con la siguiente directiva:

- `Cache-Control: no-store`

Cuando se usa `Cache-Control: no-store`, las directivas adicionales, como `must-revalidate` y `max-age`, o el encabezado `Expires`, generalmente son innecesarias en los navegadores modernos.

```http
HTTP/1.1
Cache-Control: no-store
```

Por ejemplo, si los probadores están probando una aplicación de comercio electrónico, deberían buscar todas las páginas que contengan un número de tarjeta de crédito u otra información financiera y comprobar que todas esas páginas apliquen la directiva `no-store`. Si encuentran páginas que contienen información crítica, pero que no indican al navegador que no almacene su contenido en caché, sabrán que la información sensible se almacenará en el disco y podrán confirmarlo buscando la página en la caché del navegador.

La ubicación exacta donde se almacena esa información depende del sistema operativo del cliente y del navegador utilizado. Estos son algunos ejemplos:

- Mozilla Firefox:
    - Unix/Linux: `~/.cache/mozilla/firefox/`
    - Windows: `C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer:
    - `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome:
    - Windows: `C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
    - Unix/Linux: `~/.cache/google-chrome`

#### Revisar la información almacenada en caché

Firefox ofrece una funcionalidad para ver la información almacenada en caché, que puede resultar útil para el probador. Además, el sector ha desarrollado diversas extensiones y aplicaciones externas que se pueden preferir o necesitar para Chrome, Internet Explorer o Edge.

Los detalles de la caché también están disponibles mediante las herramientas de desarrollo de la mayoría de los navegadores modernos, como [Firefox](https://firefox-source-docs.mozilla.org/devtools-user/storage_inspector/index.html), [Chrome](https://developer.chrome.com/docs/devtools/storage/cache) y Edge. En Firefox también es posible usar la URL `about:cache` para consultar los detalles de la caché.

#### Comprobar el manejo de los navegadores móviles

El manejo de las directivas de caché puede ser completamente distinto en los navegadores móviles. Por lo tanto, los probadores deberían iniciar una nueva sesión de navegación con las cachés limpias y aprovechar funciones como el [modo de dispositivo](https://developer.chrome.com/docs/devtools/device-mode) de Chrome o el [modo de diseño adaptable](https://firefox-source-docs.mozilla.org/devtools-user/responsive_design_mode/index.html) de Firefox para volver a probar o probar por separado los conceptos descritos anteriormente.

Además, los proxies personales, como ZAP y Burp Suite, permiten al probador especificar qué `User-Agent` deben enviar sus arañas o rastreadores. Se puede configurar para que coincida con una cadena `User-Agent` de un navegador móvil y utilizarla para ver qué directivas de caché envía la aplicación que se está probando.

### Pruebas de caja gris

La metodología de prueba es equivalente a la del caso de caja negra, ya que en ambos escenarios los probadores tienen acceso completo a los encabezados de respuesta del servidor y al código HTML. Sin embargo, en las pruebas de caja gris, el probador puede tener acceso a credenciales de cuenta que le permitirán probar páginas sensibles accesibles solo para usuarios autenticados.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)

## Referencias

### Documentos técnicos

- [MDN - Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control)
- [Euthanize Pragma no-cache](https://www.veggiespam.com/euthanize-pragma-no-cache/)
- [Caching in HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)