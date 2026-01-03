# Aprovechando Herramientas de Desarrollo

Este apéndice describe varios detalles para el uso de la funcionalidad de Herramientas de Desarrollo en navegador para ayudar en actividades de pruebas de seguridad.

Obviamente la funcionalidad en navegador no es un sustituto para: herramientas DAST (Pruebas Dinámicas de Seguridad de Aplicaciones), herramientas SAST (Pruebas Estáticas de Seguridad de Aplicaciones), o la experiencia de un probador, sin embargo, puede ser aprovechada para algunas actividades de pruebas y tareas relacionadas con producción de informes.

## Accediendo a Herramientas de Desarrollo

Abrir Herramientas de Desarrollo puede lograrse de varias maneras.

1. Via el atajo de teclado `F12`.
2. Via el atajo de teclado `ctrl` + `shift` + `i` en Windows.
3. Via el atajo de teclado `cmd` + `option` + `i` en Mac.
4. Via el menú contextual de clic derecho en la página web y luego seleccionando `Inspeccionar` en Google Chrome.
5. Via el menú contextual de clic derecho en la página web y luego seleccionando `Inspeccionar Elemento` en Mozilla Firefox.
6. Via el menú de triple punto 'kabob' en Google Chrome luego seleccionando `Más herramientas` y luego `Herramientas de Desarrollo`.
7. Via el menú de triple línea 'hamburger' (o 'pancake') en Mozilla Firefox luego seleccionando `Desarrollador Web` y luego `Alternar Herramientas`.

> NOTA: La mayoría de las instrucciones abajo asumen que Herramientas de Desarrollo ya está abierta o activa.

## Capacidades

| Funcionalidad         | Chrome* | Firefox | Safari |
|-----------------------|:-------:|:-------:|:------:|
| Cambio de User-Agent  | Y       | Y       | Y      |
| Editar/Reenviar Solicitudes | Y       | Y       | N      |
| Edición de Cookies    | Y       | Y       | N      |
| Edición de Almacenamiento Local | Y       | Y       | N      |
| Deshabilitar CSS      | Y       | Y       | Y      |
| Deshabilitar JavaScript | Y       | Y       | Y      |
| Ver Encabezados HTTP  | Y       | Y       | Y      |
| Capturas de Pantalla  | Y       | Y       | N      |
| Modo Offline          | Y       | Y       | N      |
| Codificación y Decodificación | Y       | Y       | Y      |
| Modo de Diseño Responsivo| Y       | Y       | Y      |

`*` Cualquier cosa que aplique a Google Chrome debería ser aplicable a todas las aplicaciones basadas en Chromium. (Lo que incluye el rebranding de Microsoft Edge alrededor de 2019/2020.)

## Cambio de User-Agent

### Pruebas Relacionadas

- [Pruebas para Debilidades de Caché del Navegador](../4-Web_Application_Security_Testing/04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)

### Google Chrome

1. Haz clic en el menú de triple punto 'kabob' en el lado derecho del panel de Herramientas de Desarrollo, selecciona `Más herramientas` luego selecciona `Condiciones de red`.
2. Desmarca la casilla de verificación "Seleccionar automáticamente".
3. Selecciona el user agent del menú desplegable o ingresa un user agent personalizado

![Menú desplegable de selección de User-Agent en Google Chrome](images/f_chrome_devtools_ua_switch.png)\
*Figura 6.F-1: Funcionalidad de Cambio de User-Agent en Herramientas de Desarrollo de Google Chrome*

### Mozilla Firefox

1. Navega a la página `about:config` de Firefox y haz clic en `¡Acepto el riesgo!`.
2. Ingresa `general.useragent.override` en el campo de búsqueda.
3. Busca `general.useragent.override`, si no puedes ver esta preferencia, busca una que muestre un conjunto de botones de radio `Boolean, Number, String` selecciona `String` luego haz clic en el botón de signo más `Add` en la página `about:config`.
4. Establece el valor de `general.useragent.override` a cualquier [User-Agent](https://developers.whatismybrowser.com/useragents/explore/) que puedas necesitar.

![Preferencia de configuración de User-Agent en Mozilla Firefox](images/f_firefox_ua_switch.png)\
*Figura 6.F-2: Funcionalidad de Cambio de User-Agent en Mozilla Firefox*

Después haz clic en el botón de bote de basura `Delete` a la derecha de la preferencia `general.useragent.override` para remover el override y volver al user agent default.

## Editar/Reenviar Solicitudes

### Pruebas Relacionadas

- [Pruebas de Autenticación](../4-Web_Application_Security_Testing/04-Authentication_Testing/README.md)
- [Pruebas de Autorización](../4-Web_Application_Security_Testing/05-Authorization_Testing/README.md)
- [Pruebas de Gestión de Sesión](../4-Web_Application_Security_Testing/06-Session_Management_Testing/README.md)
- [Pruebas de Validación de Entrada](../4-Web_Application_Security_Testing/07-Input_Validation_Testing/README.md)
- [Pruebas de Lógica de Negocio](../4-Web_Application_Security_Testing/10-Business_Logic_Testing/README.md)

### Mozilla Firefox

1. Selecciona la pestaña `Red`.
2. Realiza cualquier acción en la aplicación web.
3. Haz clic derecho en la solicitud HTTP de la lista y selecciona `Editar y Reenviar`.
4. Haz las modificaciones deseadas y haz clic en el botón `Enviar`.
5. Haz clic derecho en la solicitud modificada y selecciona `Abrir en Nueva Pestaña`.

### Google Chrome

1. Selecciona la pestaña `Red`.
2. Realiza cualquier acción en la aplicación web.
3. Haz clic derecho en la solicitud HTTP de la lista y selecciona `Copiar > Copiar como fetch`.
4. Pega el código JavaScript proporcionado en la pestaña `Consola`.
5. Haz cualquier modificación requerida, y luego presiona enter para enviar la solicitud.

## Edición de Cookies

### Pruebas Relacionadas

- [Pruebas de Autenticación](../4-Web_Application_Security_Testing/04-Authentication_Testing/README.md)
- [Pruebas de Autorización](../4-Web_Application_Security_Testing/05-Authorization_Testing/README.md)
- [Pruebas de Gestión de Sesión](../4-Web_Application_Security_Testing/06-Session_Management_Testing/README.md)
- [Pruebas para Atributos de Cookies](../4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

### Google Chrome

1. Haz clic en la pestaña `Aplicación`.
2. Expande `Cookies` bajo el encabezado `Almacenamiento`.
3. Selecciona el nombre de dominio relevante.
4. Haz doble clic en la columna `Valor` para editar cualquier valor de cookie.

> Nota: Las cookies pueden ser eliminadas una vez seleccionadas presionando la tecla `delete`, o desde el menú contextual de clic derecho.

### Mozilla Firefox

1. Haz clic en la pestaña `Almacenamiento`.
2. Expande la sección `Cookies`.
3. Selecciona el nombre de dominio relevante.
4. Haz doble clic en la columna `Valor` para editar cualquier valor de cookie.

> Nota: Las cookies pueden ser eliminadas una vez seleccionadas presionando la tecla `delete`, o con varias opciones del menú contextual de clic derecho.

![Funcionalidad de Edición de Cookies en Mozilla Firefox](images/f_firefox_cookie_edit.png)\
*Figura 6.F-3: Funcionalidad de Edición de Cookies en Mozilla Firefox*

## Edición de Almacenamiento Local

### Pruebas Relacionadas

- [Pruebas de Almacenamiento del Navegador](../4-Web_Application_Security_Testing/11-Client-side_Testing/12-Testing_Browser_Storage.md)

### Google Chrome

1. Haz clic en la pestaña `Aplicación`.
2. Expande `Almacenamiento Local` bajo el encabezado `Almacenamiento`.
3. Selecciona el nombre de dominio relevante.
4. Haz doble clic en la columna `Valor` para editar cualquier valor de cookie.
5. Haz doble clic en la Celda aplicable para editar la `Clave` o `Valor`.

> Nota: Editar `Almacenamiento de Sesión` o `Index DB` sigue esencialmente los mismos pasos.
>
> Nota: Los elementos pueden ser agregados o eliminados via el menú contextual de clic derecho.

### Mozilla Firefox

1. Haz clic en la pestaña `Almacenamiento`.
2. Expande la sección `Almacenamiento Local`.
3. Selecciona el nombre de dominio relevante.
4. Haz doble clic en la Celda aplicable para editar la `Clave` o `Valor`.

> Nota: Editar `Almacenamiento de Sesión` o `Index DB` sigue esencialmente los mismos pasos.
>
> Nota: Los elementos pueden ser agregados o eliminados via el menú contextual de clic derecho.

## Deshabilitar CSS

### Pruebas Relacionadas

- [Pruebas para Manipulación de Recursos del Lado del Cliente](../4-Web_Application_Security_Testing/11-Client-side_Testing/06-Testing_for_Client-side_Resource_Manipulation.md)

### General

Todos los navegadores principales soportan manipular CSS aprovechando la Consola de Herramientas de Desarrollo y funcionalidad JavaScript:

- Para remover todas las hojas de estilo externas: `$('style,link[rel="stylesheet"]').remove();`
- Para remover todas las hojas de estilo internas: `$('style').remove();`
- Para remover todos los estilos en línea: `Array.prototype.forEach.call(document.querySelectorAll('*'),function(el){el.removeAttribute('style');});`
- Para remover todo de la etiqueta head: `$('head').remove();`

## Deshabilitar JavaScript

### Google Chrome

1. Haz clic en el menú de triple punto 'kabob' en el lado derecho de la barra de herramientas de desarrollador web y haz clic en `Configuración`.
2. En la pestaña `Preferencias`, bajo la sección `Depurador`, marca la casilla de verificación `Deshabilitar JavaScript`.

### Mozilla Firefox

1. En la pestaña `Depurador` de herramientas de desarrollo, haz clic en el botón de engranaje de configuración en la esquina superior derecha de la barra de herramientas de desarrollador.
2. Selecciona `Deshabilitar JavaScript` del menú desplegable (este es un elemento de menú habilitar/deshabilitar; cuando JavaScript está deshabilitado, el elemento de menú tiene una marca de verificación).

## Ver Encabezados HTTP

### Pruebas Relacionadas

- [Recopilación de Información](../4-Web_Application_Security_Testing/01-Information_Gathering/README.md)

### Google Chrome

1. En la pestaña `Red` en Herramientas de Desarrollo selecciona cualquier URL o solicitud.
2. En el panel inferior derecho selecciona la pestaña `Encabezados`.

![Vista de Encabezados en Google Chrome](images/f_chrome_devtools_headers.png)\
*Figura 6.F-4: Vista de Encabezados en Google Chrome*

### Mozilla Firefox

1. En la pestaña `Red` en Herramientas de Desarrollo selecciona cualquier URL o solicitud.
2. En el panel inferior derecho selecciona la pestaña `Encabezados`.

![Vista de Encabezados en Mozilla Firefox](images/f_firefox_devtools_headers.png)\
*Figura 6.F-5: Vista de Encabezados en Mozilla Firefox*

## Capturas de Pantalla

### Pruebas Relacionadas

- [Reportes](../5-Reporting/README.md)

### Google Chrome

1. Presiona el botón `Alternar Barra de Herramientas de Dispositivo` o presiona `ctrl` + `shift` + `m`.
2. Haz clic en el menú de triple punto 'kabob' en la Barra de Herramientas de Dispositivo.
3. Selecciona `Capturar captura de pantalla` o `Capturar captura de pantalla de tamaño completo`.

### Mozilla Firefox

1. Presiona el botón de triple punto `elipsis` en la barra de direcciones.
2. Selecciona `Tomar una Captura de Pantalla`.
3. Selecciona la opción `Guardar página completa` o `Guardar visible`.

## Modo Offline

### Google Chrome

1. Navega a la pestaña `Red`.
2. En el menú desplegable `Throttle` selecciona `Offline`.

![Opción Offline en Google Chrome](images/f_chrome_devtools_offline.png)\
*Figura 6.F-6: Opción Offline en Google Chrome*

### Mozilla Firefox

1. Desde el menú de triple línea 'hamburger' (o 'pancake') selecciona `Desarrollador Web` y luego `Trabajar Offline`.

![Opción Offline en Mozilla Firefox](images/f_firefox_devtools_offline.png)\
*Figura 6.F-7: Opción Offline en Mozilla Firefox*

## Codificación y Decodificación

### Pruebas Relacionadas

- Muchos (quizás incluso la mayoría) tipos de [Pruebas de Seguridad de Aplicaciones Web](../4-Web_Application_Security_Testing/README.md) pueden beneficiarse de varios tipos de codificación.

### General

Todos los navegadores principales soportan codificar y decodificar cadenas de varias maneras aprovechando la Consola de Herramientas de Desarrollo y funcionalidad JavaScript:

- codificar base64: `btoa("string-to-encode")` & decodificar base64: `atob("string-to-decode")` - funciones JavaScript integradas que se usan para codificar una cadena a base64 y decodificar una cadena desde base64.
- codificar URL: `encodeURIComponent("string-to-encode")` & decodificar URL: `decodeURIComponent("string-to-decode")` - Codifica y decodifica entrada proporcionada por el usuario que será usada como parte de una URL, y codifica todos los caracteres que tienen significados especiales en una URL, incluyendo caracteres reservados.
- codificar URI: `encodeURI()` y decodificar URI: `decodeURI()` son funciones usadas para codificar y decodificar un URI completo, como parámetros de consulta, segmentos de ruta, o fragmentos, incluyendo caracteres especiales pero excluyendo los caracteres reservados como `:/?#[]@!$'()*+,;=` que tienen significados especiales en una URL.

## Modo de Diseño Responsivo

### Pruebas Relacionadas

- [Pruebas para Debilidades de Caché del Navegador](../4-Web_Application_Security_Testing/04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)
- [Pruebas para Autenticación Más Débil en Canal Alternativo](../4-Web_Application_Security_Testing/04-Authentication_Testing/10-Testing_for_Weaker_Authentication_in_Alternative_Channel.md)
- [Pruebas para Clickjacking](../4-Web_Application_Security_Testing/11-Client-side_Testing/09-Testing_for_Clickjacking.md)

### Google Chrome

1. Haz clic en el botón `Alternar barra de herramientas de dispositivo` o presiona `ctrl` + `shift` + `m`.

![Modo de Diseño Responsivo en Google Chrome](images/f_chrome_responsive_design_mode.png)\
*Figura 6.F-8: Modo de Diseño Responsivo en Google Chrome*

### Mozilla Firefox

1. Haz clic en el botón `Modo de Diseño Responsivo` o presiona `ctrl` + `shift` + `m`.

![Modo de Diseño Responsivo en Mozilla Firefox](images/f_firefox_responsive_design_mode.png)\
*Figura 6.F-9: Modo de Diseño Responsivo en Mozilla Firefox*

## Referencias

- [Pruebas de Seguridad de Aplicaciones Web con Navegadores](https://getmantra.com/web-app-security-testing-with-browsers/)
- [Black Hills Information Security - Webcast: ¡Herramientas Gratuitas! Cómo Usar Herramientas de Desarrollo y JavaScript en Pentests de Webapp](https://www.blackhillsinfosec.com/webcast-free-tools-how-to-use-developer-tools-and-javascript-in-webapp-pentests/)
- [Greg Malcolm - Chrome Developer Tools: Saqueando el Arsenal](https://github.com/gregmalcolm/wacky-wandas-wicked-weapons-frontend/blob/fix-it/README.md)
- [Lista de Cadenas de UserAgent](https://techblog.willshouse.com/2012/01/03/most-common-user-agents/)