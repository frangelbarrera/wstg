# Pruebas de debilidades en la caché del navegador

|ID          |
|------------|
|WSTG-ATHN-06|

## Summary

En esta fase, el probador verifica que la aplicación instruya correctamente al navegador para que no retenga datos sensibles.

Los navegadores pueden almacenar información con fines de caché e historial. La caché se utiliza para mejorar el rendimiento, de modo que la información previamente mostrada no necesite descargarse nuevamente. Los mecanismos de historial se utilizan para la comodidad del usuario, para que pueda ver exactamente lo que vio en el momento en que se recuperó el recurso. Si se muestra información sensible al usuario (como su dirección, detalles de tarjeta de crédito, número de seguridad social o nombre de usuario), entonces esta información podría almacenarse con fines de caché o historial, y por lo tanto ser recuperable examinando la caché del navegador o simplemente presionando el botón **Atrás** del navegador.

## Test Objectives

- Revisar si la aplicación almacena información sensible en el lado del cliente.
- Revisar si se puede acceder sin autorización.

## How to Test

### Browser History

Técnicamente, el botón **Atrás** es un historial y no una caché (ver [Caching in HTTP: History Lists](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13)). La caché y el historial son dos entidades diferentes. Sin embargo, comparten la misma debilidad de presentar información sensible previamente mostrada.

La primera y más simple prueba consiste en ingresar información sensible en la aplicación y cerrar sesión. Luego, el probador hace clic en el botón **Atrás** del navegador para verificar si se puede acceder a información sensible previamente mostrada mientras no está autenticado.

Si al presionar el botón **Atrás** el probador puede acceder a páginas anteriores pero no a nuevas, entonces no es un problema de autenticación, sino un problema de historial del navegador. Si estas páginas contienen datos sensibles, significa que la aplicación no prohibió al navegador almacenarlos.

La autenticación no necesariamente necesita estar involucrada en las pruebas. Por ejemplo, cuando un usuario ingresa su dirección de correo electrónico para suscribirse a un boletín, esta información podría ser recuperable si no se maneja correctamente.

El botón **Atrás** se puede detener para mostrar datos sensibles. Esto se puede hacer mediante:

- Entregar la página a través de HTTPS.
- Establecer `Cache-Control: must-revalidate`

### Browser Cache

Aquí los probadores verifican que la aplicación no filtre ningún dato sensible en la caché del navegador. Para hacerlo, pueden usar un proxy (como ZAP) y buscar en las respuestas del servidor que pertenecen a la sesión, verificando que para cada página que contiene información sensible el servidor instruyó al navegador para que no almacene en caché ningún dato. Tal directiva se puede emitir en los encabezados de respuesta HTTP con las siguientes directivas:

- `Cache-Control: no-cache, no-store`
- `Expires: 0`
- `Pragma: no-cache`

Estas directivas son generalmente robustas, aunque pueden ser necesarios indicadores adicionales para el encabezado `Cache-Control` para prevenir mejor archivos persistentemente vinculados en el sistema de archivos. Estos incluyen:

- `Cache-Control: must-revalidate, max-age=0, s-maxage=0`

```http
HTTP/1.1:
Cache-Control: no-cache
```

```html
HTTP/1.0:
Pragma: no-cache
Expires: "past date or illegal value (e.g., 0)"
```

Por ejemplo, si los probadores están probando una aplicación de comercio electrónico, deberían buscar todas las páginas que contengan un número de tarjeta de crédito u otra información financiera, y verificar que todas esas páginas apliquen la directiva `no-cache`. Si encuentran páginas que contienen información crítica pero que fallan en instruir al navegador para que no almacene en caché su contenido, saben que la información sensible se almacenará en el disco, y pueden verificarlo doblemente simplemente buscando la página en la caché del navegador.

La ubicación exacta donde se almacena esa información depende del sistema operativo del cliente y del navegador utilizado. Aquí hay algunos ejemplos:

- Mozilla Firefox:
    - Unix/Linux: `~/.cache/mozilla/firefox/`
    - Windows: `C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer:
    - `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome:
    - Windows: `C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
    - Unix/Linux: `~/.cache/google-chrome`

#### Reviewing Cached Information

Firefox proporciona funcionalidad para ver información almacenada en caché, lo que puede ser beneficioso para ti como probador. Por supuesto, la industria también ha producido varias extensiones y aplicaciones externas que puedes preferir o necesitar para Chrome, Internet Explorer o Edge.

Los detalles de la caché también están disponibles a través de herramientas de desarrollo en la mayoría de los navegadores modernos, como [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector#Cache_Storage), [Chrome](https://developers.google.com/web/tools/chrome-devtools/storage/cache) y Edge. Con Firefox también es posible usar la URL `about:cache` para verificar detalles de la caché.

#### Check Handling for Mobile Browsers

El manejo de directivas de caché puede ser completamente diferente para navegadores móviles. Por lo tanto, los probadores deberían iniciar una nueva sesión de navegación con cachés limpias y aprovechar características como el [Modo de dispositivo](https://developers.google.com/web/tools/chrome-devtools/device-mode) de Chrome o el [Modo de diseño responsivo](https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_Mode) de Firefox para volver a probar o probar por separado los conceptos descritos anteriormente.

Además, proxies personales como ZAP y Burp Suite permiten al probador especificar qué `User-Agent` debe enviarse por sus arañas/rastreadores. Esto podría configurarse para coincidir con una cadena de `User-Agent` de navegador móvil y usarse para ver qué directivas de caché se envían por la aplicación que se está probando.

### Gray-Box Testing

La metodología para las pruebas es equivalente al caso de caja negra, ya que en ambos escenarios los probadores tienen acceso completo a los encabezados de respuesta del servidor y al código HTML. Sin embargo, con pruebas de caja gris, el probador puede tener acceso a credenciales de cuenta que les permitirán probar páginas sensibles que son accesibles solo para usuarios autenticados.

## Tools

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)

## References

### Whitepapers

- [Caching in HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)