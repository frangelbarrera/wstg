# Pruebas de inyección de cabecera Host

|ID          |
|------------|
|WSTG-INPV-17|

## Resumen

Un servidor web normalmente aloja varias aplicaciones web en la misma dirección IP, refiriéndose a cada aplicación a través del host virtual. En una solicitud HTTP entrante, los servidores web a menudo despachan la solicitud al host virtual objetivo basado en el valor proporcionado en la cabecera Host. Sin una validación adecuada del valor de la cabecera, el atacante puede proporcionar entrada inválida para causar que el servidor web:

- Despache solicitudes al primer host virtual de la lista.
- Realice una redirección a un dominio controlado por el atacante.
- Realice envenenamiento de caché web.
- Manipule la funcionalidad de restablecimiento de contraseña.
- Permita el acceso a hosts virtuales que no estaban destinados a ser accesibles externamente.

## Objetivos de la prueba

- Evaluar si la cabecera Host se está analizando dinámicamente en la aplicación.
- Eludir controles de seguridad que dependen de la cabecera.

## Cómo probar

La prueba inicial es tan simple como proporcionar otro dominio (es decir, `attacker.com`) en el campo de la cabecera Host. Es cómo el servidor web procesa el valor de la cabecera lo que dicta el impacto. El ataque es válido cuando el servidor web procesa la entrada para enviar la solicitud a un host controlado por el atacante que reside en el dominio proporcionado, y no a un host virtual interno que reside en el servidor web.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

En el caso más simple, esto puede causar una redirección 302 al dominio proporcionado.

```http
HTTP/1.1 302 Found
[...]
Location: https://www.attacker.com/login.php

```

Alternativamente, el servidor web puede enviar la solicitud al primer host virtual de la lista.

### Elusión de cabecera X-Forwarded-Host

En el caso de que la inyección de cabecera Host se mitigue mediante la verificación de entrada inválida inyectada a través de la cabecera Host, puede proporcionar el valor a la cabecera `X-Forwarded-Host`.

```http
GET / HTTP/1.1
Host: www.example.com
X-Forwarded-Host: www.attacker.com
[...]
```

Potencialmente produciendo salida del lado del cliente como:

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

Una vez más, esto depende de cómo el servidor web procesa el valor de la cabecera.

### Envenenamiento de caché web

Usando esta técnica, un atacante puede manipular una caché web para servir contenido envenenado a cualquiera que lo solicite. Esto depende de la capacidad de envenenar el proxy de caché ejecutado por la aplicación misma, CDN o otros proveedores descendentes. Como resultado, la víctima no tendrá control sobre recibir el contenido malicioso al solicitar la aplicación vulnerable.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

Lo siguiente se servirá desde la caché web, cuando una víctima visite la aplicación vulnerable.

```html
[...]
<link src="https://www.attacker.com/link" />
[...]
```

### Envenenamiento de restablecimiento de contraseña

Es común que la funcionalidad de restablecimiento de contraseña incluya el valor de la cabecera Host al crear enlaces de restablecimiento de contraseña que usan un token secreto generado. Si la aplicación procesa un dominio controlado por el atacante para crear un enlace de restablecimiento de contraseña, la víctima puede hacer clic en el enlace en el correo electrónico y permitir que el atacante obtenga el token de restablecimiento, restableciendo así la contraseña de la víctima.

El ejemplo a continuación muestra un enlace de restablecimiento de contraseña que se genera en PHP usando el valor de `$_SERVER['HTTP_HOST']`, que se establece basado en el contenido de la cabecera HTTP Host:

```php
$reset_url = "https://" . $_SERVER['HTTP_HOST'] . "/reset.php?token=" .$token;
send_reset_email($email,$rset_url);
```

Al hacer una solicitud HTTP a la página de restablecimiento de contraseña con una cabecera Host manipulada, podemos modificar a dónde apunta la URL:

```http
POST /request_password_reset.php HTTP/1.1
Host: www.attacker.com
[...]

email=user@example.org
```

El dominio especificado (`www.attacker.com`) se usará entonces en el enlace de restablecimiento, que se envía por correo electrónico al usuario. Cuando el usuario hace clic en este enlace, el atacante puede robar el token y comprometer su cuenta.

```text
... Fragmento de correo electrónico ...

Haga clic en el siguiente enlace para restablecer su contraseña:

https://www.attacker.com/reset.php?token=12345

... Fragmento de correo electrónico ...
```

### Acceso a hosts virtuales privados

En algunos casos, un servidor puede tener hosts virtuales que no están destinados a ser accesibles externamente. Esto es más común con una configuración DNS de horizonte dividido (donde los servidores DNS internos y externos devuelven registros diferentes para el mismo dominio).

Por ejemplo, una organización puede tener un único servidor web en su red interna, que aloja tanto su sitio web público (en `www.example.org`) como su Intranet interna (en `intranet.example.org`, pero ese registro solo existe en el servidor DNS interno). Aunque no sería posible navegar directamente a `intranet.example.org` desde fuera de la red (ya que el dominio no se resolvería), puede ser posible acceder a la Intranet haciendo una solicitud desde fuera con la siguiente cabecera `Host`:

```http
Host: intranet.example.org
```

Esto también se podría lograr agregando una entrada para `intranet.example.org` a su archivo hosts con la dirección IP pública de `www.example.org`, o anulando la resolución DNS en su herramienta de prueba.

## Referencias

- [¿Qué es un ataque de cabecera Host?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [Ataque de cabecera Host](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
- [Ataques de cabecera HTTP Host](https://portswigger.net/web-security/host-header)