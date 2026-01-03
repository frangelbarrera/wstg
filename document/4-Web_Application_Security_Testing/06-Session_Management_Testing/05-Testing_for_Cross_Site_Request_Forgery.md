# Prueba de Falsificación de Solicitud entre Sitios

|ID          |
|------------|
|WSTG-SESS-05|

## Summary

La Falsificación de Solicitud entre Sitios ([CSRF](https://owasp.org/www-community/attacks/csrf)) es un ataque que fuerza a un usuario final a ejecutar acciones no intencionadas en una aplicación web en la que están actualmente autenticados. Con un poco de ayuda de ingeniería social (como enviar un enlace por correo electrónico o chat), un atacante puede forzar a los usuarios de una aplicación web a ejecutar acciones de su elección. Un exploit CSRF exitoso puede comprometer los datos y operaciones del usuario final cuando apunta a un usuario normal. Si el usuario final objetivo es la cuenta de administrador, un ataque CSRF puede comprometer toda la aplicación web.

CSRF se basa en:

1. El comportamiento del navegador web con respecto al manejo de información relacionada con la sesión, como cookies e información de autenticación HTTP.
2. El conocimiento del atacante de URLs válidas de aplicaciones web, solicitudes o funcionalidad.
3. La gestión de sesiones de aplicaciones que depende únicamente de información conocida por el navegador.
4. La existencia de etiquetas HTML cuya presencia causa acceso inmediato a un recurso HTTP[S]; por ejemplo, la etiqueta de imagen `img`.

Los puntos 1, 2 y 3 son esenciales para que la vulnerabilidad esté presente, mientras que el punto 4 facilita la explotación real, pero no es estrictamente requerido.

1. Los navegadores envían automáticamente información utilizada para identificar una sesión de usuario. Suponga que *site* es un sitio que aloja una aplicación web, y el usuario *victim* acaba de autenticarse en *site*. En respuesta, *site* envía a *victim* una cookie que identifica las solicitudes enviadas por *victim* como pertenecientes a la sesión autenticada de *victim*. Una vez que el navegador recibe la cookie establecida por *site*, la enviará automáticamente junto con cualquier solicitud posterior dirigida a *site*.
2. Si la aplicación no utiliza información relacionada con la sesión en las URLs, entonces las URLs de la aplicación, sus parámetros y valores legítimos pueden ser identificados. Esto puede lograrse mediante análisis de código o accediendo a la aplicación y tomando nota de formularios y URLs incrustados en HTML o JavaScript.
3. "Conocido por el navegador" se refiere a información como cookies o información de autenticación basada en HTTP (como Autenticación Básica y no autenticación basada en formularios), que son almacenadas por el navegador y posteriormente presentes en cada solicitud dirigida hacia un área de aplicación que solicita esa autenticación. Las vulnerabilidades discutidas a continuación se aplican a aplicaciones que dependen completamente de este tipo de información para identificar una sesión de usuario.

Por simplicidad, considere URLs accesibles vía GET (aunque la discusión se aplica también a solicitudes POST). Si *victim* ya se ha autenticado, enviar otra solicitud hace que la cookie se envíe automáticamente con ella. La figura a continuación ilustra al usuario accediendo a una aplicación en `www.example.com`.

![Session Riding](images/Session_riding.GIF)\
*Figure 4.6.5-1: Session Riding*

La solicitud GET podría ser enviada por el usuario de varias formas diferentes:

- Usando la aplicación web
- Escribiendo la URL directamente en el navegador
- Siguiendo un enlace externo que apunta a la URL

Estas invocaciones son indistinguibles por la aplicación. En particular, la tercera puede ser bastante peligrosa. Hay una serie de técnicas y vulnerabilidades que pueden disfrazar las propiedades reales de un enlace. El enlace puede estar incrustado en un mensaje de correo electrónico, aparecer en un sitio malicioso al que el usuario es atraído, o aparecer en contenido alojado por un tercero (como otro sitio o correo HTML) y apuntar a un recurso de la aplicación. Si el usuario hace clic en el enlace, dado que ya están autenticados en la aplicación web en *site*, el navegador emitirá una solicitud GET a la aplicación web, acompañada de información de autenticación (la cookie de ID de sesión). Esto resulta en una operación válida siendo realizada en la aplicación web que el usuario no espera; por ejemplo, una transferencia de fondos en una aplicación de banca web.

Al usar una etiqueta como `img`, como se especifica en el punto 4 arriba, ni siquiera es necesario que el usuario siga un enlace particular. Suponga que el atacante envía al usuario un correo electrónico induciéndolo a visitar una URL que se refiere a una página que contiene el siguiente HTML (simplificado en exceso).

```html
<html>
    <body>
...
<img src="https://www.company.example/action" width="0" height="0">
...
    </body>
</html>
```

Cuando el navegador muestra esta página, intentará mostrar la imagen especificada de dimensión cero (por lo tanto, invisible) desde `https://www.company.example` también. Esto resulta en una solicitud siendo enviada automáticamente a la aplicación web alojada en *site*. No es importante que la URL de la imagen no se refiera a una imagen adecuada, ya que su presencia activará la solicitud `action` especificada en el campo `src` de todos modos. Esto sucede siempre que la descarga de imágenes no esté deshabilitada en el navegador. La mayoría de los navegadores no tienen descargas de imágenes deshabilitadas ya que eso inutilizaría la mayoría de las aplicaciones web más allá de la usabilidad.

El problema aquí es una consecuencia de:

- Etiquetas HTML en la página que resultan en ejecución automática de solicitudes HTTP (`img` siendo una de esas).
- El navegador no tiene forma de saber que el recurso referenciado por `img` no es una imagen legítima.
- La carga de imágenes que ocurre independientemente de la ubicación de la fuente de la imagen alegada, es decir, el formulario y la imagen misma no necesitan estar ubicados en el mismo host o incluso el mismo dominio.

El hecho de que el contenido HTML no relacionado con la aplicación web pueda referirse a componentes en la aplicación, y el hecho de que el navegador componga automáticamente una solicitud válida hacia la aplicación, permite este tipo de ataque. No hay forma de prohibir este comportamiento a menos que se haga imposible para el atacante interactuar con la funcionalidad de la aplicación.

En entornos integrados de correo/navegador, simplemente mostrar un mensaje de correo electrónico que contenga la referencia de imagen resultaría en la ejecución de la solicitud a la aplicación web con la cookie del navegador asociada. Los mensajes de correo electrónico pueden referenciar URLs de imágenes aparentemente válidas como:

```html
<img src="https://[attacker]/picture.gif" width="0" height="0">
```

En este ejemplo, `[attacker]` es un sitio controlado por el atacante. Al utilizar un mecanismo de redirección, el sitio malicioso puede usar `https://[attacker]/picture.gif` para dirigir a la víctima a `https://[thirdparty]/action` y activar la `action`.

Las cookies no son el único ejemplo involucrado en este tipo de vulnerabilidad. Las aplicaciones web cuya información de sesión es suministrada enteramente por el navegador son vulnerables también. Esto incluye aplicaciones que dependen únicamente de mecanismos de autenticación HTTP, ya que la información de autenticación es conocida por el navegador y se envía automáticamente en cada solicitud. Esto no incluye autenticación basada en formularios, que ocurre solo una vez y genera alguna forma de información relacionada con la sesión, usualmente una cookie.

Supongamos que la víctima está conectada a una consola de gestión web de firewall. Para iniciar sesión, un usuario debe autenticarse y la información de sesión se almacena en una cookie.

Supongamos que la consola de gestión web de firewall tiene una función que permite a un usuario autenticado eliminar una regla especificada por su ID numérico, o todas las reglas en la configuración si el usuario especifica `*` (una característica peligrosa en la realidad, pero que hace un ejemplo más interesante). La página de eliminación se muestra a continuación. Supongamos que el formulario – por simplicidad – emite una solicitud GET. Para eliminar la regla número uno:

```text
https://[target]/fwmgt/delete?rule=1
```

Para eliminar todas las reglas:

```text
https://[target]/fwmgt/delete?rule=*
```

Este ejemplo es intencionalmente ingenuo, pero muestra de manera simplificada los peligros de CSRF.

![Session Riding Firewall Management](images/Session_Riding_Firewall_Management.gif)\
*Figure 4.6.5-2: Session Riding Firewall Management*

Usando el formulario mostrado en la figura anterior, ingresar el valor `*` y hacer clic en el botón Eliminar enviará la siguiente solicitud GET:

```text
https://www.company.example/fwmgt/delete?rule=*
```

Esto eliminaría todas las reglas del firewall.

![Session Riding Firewall Management 2](images/Session_Riding_Firewall_Management_2.gif)\
*Figure 4.6.5-3: Session Riding Firewall Management 2*

El usuario también podría haber logrado los mismos resultados enviando manualmente la URL:

```text
https://[target]/fwmgt/delete?rule=*
```

O siguiendo un enlace que apunta, directa o vía redirección, a la URL anterior. O, nuevamente, accediendo a una página HTML con una etiqueta `img` incrustada apuntando a la misma URL.

En todos estos casos, si el usuario está actualmente conectado a la aplicación de gestión de firewall, la solicitud tendrá éxito y modificará la configuración del firewall. Uno puede imaginar ataques dirigidos a aplicaciones sensibles y realizando ofertas automáticas en subastas, transferencias de dinero, pedidos, cambiando la configuración de componentes de software críticos, etc.

Una cosa interesante es que estas vulnerabilidades pueden ejercerse detrás de un firewall; es decir, es suficiente que el enlace siendo atacado sea alcanzable por la víctima y no directamente por el atacante. En particular, puede ser cualquier servidor web de intranet; por ejemplo, en el escenario de gestión de firewall mencionado antes, que es poco probable que esté expuesto a internet.

Aplicaciones auto-vulnerables, es decir, aplicaciones que se usan tanto como vector de ataque como objetivo (como aplicaciones de correo web), empeoran las cosas. Dado que los usuarios están conectados cuando leen sus mensajes de correo electrónico, una aplicación vulnerable de este tipo puede permitir a los atacantes realizar acciones como eliminar mensajes o enviar mensajes que parecen originarse de la víctima.

## Test Objectives

- Determinar si es posible iniciar solicitudes en nombre de un usuario que no son iniciadas por el usuario.

## How to Test

Audite la aplicación para determinar si su gestión de sesiones es vulnerable. Si la gestión de sesiones depende únicamente de valores del lado del cliente (información disponible para el navegador), entonces la aplicación es vulnerable. "Valores del lado del cliente" se refiere a cookies y credenciales de autenticación HTTP (Autenticación Básica y otras formas de autenticación HTTP; no autenticación basada en formularios, que es autenticación a nivel de aplicación).

Los recursos accesibles vía solicitudes HTTP GET son fácilmente vulnerables, aunque las solicitudes POST pueden automatizarse vía JavaScript y son vulnerables también; por lo tanto, el uso de POST solo no es suficiente para corregir la ocurrencia de vulnerabilidades CSRF.

En caso de POST, se puede usar la siguiente muestra.

1. Cree una página HTML similar a la mostrada a continuación
2. Aloje el HTML en un sitio malicioso o de terceros
3. Envíe el enlace para la página a las víctimas e indúzcalas a hacer clic en él.

```html
<html>
<body onload='document.CSRF.submit()'>

<form action='https://targetWebsite/Authenticate.jsp' method='POST' name='CSRF'>
    <input type='hidden' name='name' value='Hacked'>
    <input type='hidden' name='password' value='Hacked'>
</form>

</body>
</html>
```

En caso de aplicaciones web en las que los desarrolladores están utilizando JSON para la comunicación navegador a servidor, puede surgir un problema con el hecho de que no hay parámetros de consulta con el formato JSON, que son un must con formularios de auto-envío. Para eludir este caso, podemos usar un formulario de auto-envío con cargas útiles JSON incluyendo entrada oculta para explotar CSRF. Tendremos que cambiar el tipo de codificación (`enctype`) a `text/plain` para asegurar que la carga útil se entregue tal cual. El código de exploit se verá como el siguiente:

```html
<html>
  <body>
   <script>history.pushState('', '', '/')</script>
    <form action='https://victimsite.com' method='POST' enctype='text/plain'>
      <input type='hidden' name='{"name":"hacked","password":"hacked","padding":"'value='something"}' />
      <input type='submit' value='Submit request' />
    </form>
  </body>
</html>
```

La solicitud POST será como sigue:

```http
POST / HTTP/1.1
Host: victimsite.com
Content-Type: text/plain

{"name":"hacked","password":"hacked","padding":"=something"}
```

Cuando estos datos se envían como una solicitud POST, el servidor aceptará felizmente los campos de nombre y contraseña e ignorará el de relleno ya que no lo necesita.

## Remediation

- Consulte la [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) para medidas de prevención.

## Tools

- [ZAP](https://www.zaproxy.org/)
- [CSRF Tester](https://wiki.owasp.org/index.php/Category:OWASP_CSRFTester_Project)
- [Pinata-csrf-tool](https://code.google.com/archive/p/pinata-csrf-tool/)

## References

- [Peter W: "Cross-Site Request Forgeries"](https://web.archive.org/web/20160303230910/http://www.tux.org/~peterw/csrf.txt)
- [Thomas Schreiber: "Session Riding"](https://web.archive.org/web/20160304001446/https://www.securenet.de/papers/Session_Riding.pdf)
- [Oldest known post](https://web.archive.org/web/20000622042229/https://www.zope.org/Members/jim/ZopeSecurity/ClientSideTrojan)
- [Cross-site Request Forgery FAQ](https://www.cgisecurity.com/csrf-faq.html)
- [A Most-Neglected Fact About Cross Site Request Forgery (CSRF)](https://yehg.net/lab/pr0js/view.php/A_Most-Neglected_Fact_About_CSRF.pdf)
- [Multi-POST CSRF](https://www.lanmaster53.com/2013/07/17/multi-post-csrf/)
- [SANS Pen Test Webcast: Complete Application pwnage via Multi POST XSRF](https://www.youtube.com/watch?v=EOs5PZiiwug)