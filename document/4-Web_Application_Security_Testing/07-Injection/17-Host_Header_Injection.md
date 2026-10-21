# Inyección de cabecera Host

|ID          |
|------------|
|WSTG-INJT-17|

## Resumen

Un servidor web suele alojar varias aplicaciones web en la misma dirección IP y dirigir a cada aplicación mediante el host virtual correspondiente. En una solicitud HTTP entrante, los servidores web suelen enviar la solicitud al host virtual de destino según el valor proporcionado en la cabecera Host. Sin una validación adecuada de ese valor, el atacante puede proporcionar una entrada no válida para hacer que el servidor web:

- Envíe las solicitudes al primer host virtual de la lista.
- Redirija a un dominio controlado por el atacante.
- Envenene la caché web.
- Manipule la funcionalidad de restablecimiento de contraseña.
- Permita el acceso a hosts virtuales que no estaban destinados a ser accesibles desde el exterior.

## Objetivos de la prueba

- Evaluar si la cabecera Host se analiza dinámicamente en la aplicación.
- Eludir los controles de seguridad que dependen de la cabecera.

## Cómo probar

La prueba inicial consiste simplemente en proporcionar otro dominio (por ejemplo, `attacker.com`) en el campo de la cabecera Host. El impacto depende de cómo procese el servidor web el valor de la cabecera. El ataque es válido cuando el servidor web procesa la entrada y envía la solicitud a un host controlado por el atacante que se encuentra en el dominio proporcionado, en lugar de a un host virtual interno alojado en el servidor web.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

En el caso más sencillo, esto puede provocar una redirección 302 al dominio proporcionado.

```http
HTTP/1.1 302 Found
[...]
Location: https://www.attacker.com/login.php

```

Como alternativa, el servidor web puede enviar la solicitud al primer host virtual de la lista.

### Elusión mediante la cabecera X-Forwarded-Host

Si la inyección de cabecera Host se mitiga comprobando si se ha introducido una entrada no válida mediante la cabecera Host, se puede proporcionar el valor en la cabecera `X-Forwarded-Host`.

```http
GET / HTTP/1.1
Host: www.example.com
X-Forwarded-Host: www.attacker.com
[...]
```

Esto puede producir una salida del lado del cliente como la siguiente:

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

Una vez más, esto depende de cómo procese el servidor web el valor de la cabecera.

### Envenenamiento de caché web

Con esta técnica, un atacante puede manipular una caché web para que sirva contenido envenenado a cualquiera que lo solicite. Esto depende de que sea posible envenenar el proxy de caché de la propia aplicación, las CDN u otros proveedores posteriores. Como resultado, la víctima no podrá controlar si recibe el contenido malicioso al solicitar la aplicación vulnerable.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

Cuando una víctima visite la aplicación vulnerable, la caché web servirá lo siguiente:

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

### Envenenamiento del restablecimiento de contraseña

Es común que la funcionalidad de restablecimiento de contraseña incluya el valor de la cabecera Host al crear enlaces de restablecimiento que utilizan un token secreto generado. Si la aplicación procesa un dominio controlado por el atacante para crear un enlace de restablecimiento de contraseña, la víctima podría hacer clic en el enlace del correo electrónico y permitir que el atacante obtenga el token de restablecimiento y, de ese modo, restablezca la contraseña de la víctima.

El ejemplo siguiente muestra un enlace de restablecimiento de contraseña generado en PHP que usa el valor de `$_SERVER['HTTP_HOST']`, establecido según el contenido de la cabecera HTTP Host:

```php
$reset_url = "https://" . $_SERVER['HTTP_HOST'] . "/reset.php?token=" .$token;
send_reset_email($email,$rset_url);
```

Al enviar una solicitud HTTP a la página de restablecimiento de contraseña con una cabecera Host manipulada, podemos modificar el destino de la URL:

```http
POST /request_password_reset.php HTTP/1.1
Host: www.attacker.com
[...]

email=user@example.org
```

A continuación, el dominio especificado (`www.attacker.com`) se utilizará en el enlace de restablecimiento que se envía por correo electrónico al usuario. Cuando este haga clic en el enlace, el atacante podrá robar el token y comprometer su cuenta.

```text
... Fragmento del correo electrónico ...

Haga clic en el siguiente enlace para restablecer su contraseña:

https://www.attacker.com/reset.php?token=12345

... Fragmento del correo electrónico ...
```

### Acceso a hosts virtuales privados

En algunos casos, un servidor puede tener hosts virtuales que no están destinados a ser accesibles desde el exterior. Esto es más común en configuraciones DNS de [horizonte dividido](https://en.wikipedia.org/wiki/Split-horizon_DNS), en las que los servidores DNS internos y externos devuelven registros distintos para el mismo dominio.

Por ejemplo, una organización puede tener un único servidor web en su red interna que aloje tanto su sitio web público (en `www.example.org`) como su Intranet interna (en `intranet.example.org`, aunque ese registro solo exista en el servidor DNS interno). Aunque no sería posible acceder directamente a `intranet.example.org` desde fuera de la red (porque el dominio no se resolvería), podría ser posible acceder a la Intranet enviando desde el exterior una solicitud con la siguiente cabecera `Host`:

```http
Host: intranet.example.org
```

También se podría lograr agregando una entrada para `intranet.example.org` al archivo hosts con la dirección IP pública de `www.example.org`, o anulando la resolución DNS en la herramienta de prueba.

## Referencias

- [¿Qué es un ataque de cabecera Host?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [Ataque de cabecera Host](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
- [Ataques a la cabecera HTTP Host](https://portswigger.net/web-security/host-header)