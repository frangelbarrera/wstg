# Redirección de URL del lado del cliente

|ID          |
|------------|
|WSTG-CLNT-04|

## Resumen

Esta sección describe cómo comprobar si se producen redirecciones de URL del lado del cliente, también conocidas como redirecciones abiertas. Se trata de un fallo de validación de entradas que se produce cuando una aplicación acepta entradas controladas por el usuario que especifican un enlace a una URL externa que podría ser maliciosa. Este tipo de vulnerabilidad podría utilizarse para llevar a cabo un ataque de phishing o redirigir a una víctima a una página infectada.

Esta vulnerabilidad se produce cuando una aplicación acepta entradas no confiables que contienen un valor de URL y no las sanea. Este valor de URL podría hacer que la aplicación web redirigiera al usuario a otra página, como una página maliciosa controlada por el atacante.

Esta vulnerabilidad puede permitir que un atacante lleve a cabo con éxito una estafa de phishing y robe credenciales de usuario. Dado que la redirección se origina en la aplicación legítima, los intentos de phishing pueden parecer más confiables.

A continuación se muestra un ejemplo de una URL para un ataque de phishing.

```text
https://www.target.site?#redirect=www.fake-target.site
```

La víctima que visite esta URL será redirigida automáticamente a `fake-target.site`, donde un atacante podría colocar una página falsa que se parezca al sitio previsto para robar las credenciales de la víctima.

La redirección abierta también podría utilizarse para crear una URL que evite las comprobaciones de control de acceso de la aplicación y dirija al atacante a funciones privilegiadas a las que normalmente no podría acceder.

## Objetivos de las pruebas

- Identificar puntos de inyección que gestionen URLs o rutas.
- Evaluar los destinos a los que el sistema podría redirigir.

## Cómo probar

Cuando los evaluadores comprueban manualmente este tipo de vulnerabilidad, primero identifican si hay redirecciones del lado del cliente implementadas en el código del lado del cliente. Estas redirecciones pueden implementarse, por ejemplo en JavaScript, mediante el objeto `window.location`. Este objeto puede utilizarse para dirigir el navegador a otra página con solo asignarle una cadena. El siguiente fragmento lo demuestra:

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location='https://'+decodeURIComponent(redir);
}
```

En este ejemplo, el script no valida la variable `redir`, que contiene la entrada proporcionada por el usuario a través de la cadena de consulta. Como no se aplica ningún tipo de codificación, esta entrada no validada se pasa al objeto `windows.location`, lo que crea una vulnerabilidad de redirección de URL.

Esto significa que un atacante podría redirigir a la víctima a un sitio malicioso simplemente enviando la siguiente cadena de consulta:

```text
https://www.victim.site/?#www.malicious.site
```

Con una pequeña modificación, el fragmento de ejemplo anterior puede ser vulnerable a la inyección de JavaScript.

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location=decodeURIComponent(redir);
}
```

Esto puede explotarse enviando la siguiente cadena de consulta:

```text
https://www.victim.site/?#javascript:alert(document.cookie)
```

Al probar esta vulnerabilidad, tenga en cuenta que algunos caracteres se interpretan de manera diferente en distintos navegadores. Para obtener más información, consulte [XSS basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).