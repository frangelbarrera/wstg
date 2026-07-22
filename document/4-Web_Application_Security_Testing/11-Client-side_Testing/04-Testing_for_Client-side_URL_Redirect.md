# Pruebas de Redirección de URL del Lado del Cliente

|ID          |
|------------|
|WSTG-CLNT-04|

## Resumen

Esta sección describe cómo verificar la redirección de URL del lado del cliente, también conocida como redirección abierta. Es un fallo de validación de entrada que existe cuando una aplicación acepta entrada controlada por el usuario que especifica un enlace que lleva a una URL externa que podría ser maliciosa. Este tipo de vulnerabilidad podría ser usada para lograr un ataque de phishing o redirigir a una víctima a una página de infección.

Esta vulnerabilidad ocurre cuando una aplicación acepta entrada no confiable que contiene un valor de URL y no lo sanea. Este valor de URL podría causar que la aplicación web redirija al usuario a otra página, tal como una página maliciosa controlada por el atacante.

Esta vulnerabilidad podría permitir a un atacante lanzar exitosamente una estafa de phishing y robar credenciales de usuario. Dado que la redirección se origina en la aplicación real, los intentos de phishing podrían tener una apariencia más confiable.

Aquí hay un ejemplo de una URL de ataque de phishing.

```text
https://www.target.site?#redirect=www.fake-target.site
```

La víctima que visite esta URL será automáticamente redirigida a `fake-target.site`, donde un atacante podría colocar una página falsa que se asemeje al sitio previsto, para robar las credenciales de la víctima.

La redirección abierta también podría ser usada para diseñar una URL que evite las verificaciones de control de acceso de la aplicación y reenvíe al atacante a funciones privilegiadas a las que normalmente no podría acceder.

## Objetivos de Prueba

- Identificar puntos de inyección que manejan URLs o rutas.
- Evaluar las ubicaciones a las que el sistema podría redirigir.

## Cómo Probar

Cuando los testers verifican manualmente este tipo de vulnerabilidad, primero identifican si hay redirecciones del lado del cliente implementadas en el código del lado del cliente. Estas redirecciones pueden ser implementadas, para dar un ejemplo de JavaScript, usando el objeto `window.location`. Esto puede ser usado para dirigir el navegador a otra página simplemente asignándole una cadena. Esto se demuestra en el siguiente fragmento:

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location='https://'+decodeURIComponent(redir);
}
```

En este ejemplo, el script no realiza ninguna validación de la variable `redir` que contiene la entrada proporcionada por el usuario vía la cadena de consulta. Como no se aplica ninguna forma de codificación, esta entrada no validada se pasa al objeto `windows.location`, creando una vulnerabilidad de redirección de URL.

Esto implica que un atacante podría redirigir a la víctima a un sitio malicioso simplemente enviando la siguiente cadena de consulta:

```text
https://www.victim.site/?#www.malicious.site
```

Con una ligera modificación, el fragmento de ejemplo anterior puede ser vulnerable a inyección de JavaScript.

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location=decodeURIComponent(redir);
}
```

Esto puede ser explotado enviando la siguiente cadena de consulta:

```text
https://www.victim.site/?#javascript:alert(document.cookie)
```

Al probar esta vulnerabilidad, considerar que algunos caracteres son tratados de manera diferente por diferentes navegadores. Como referencia, ver [XSS basado en DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).
