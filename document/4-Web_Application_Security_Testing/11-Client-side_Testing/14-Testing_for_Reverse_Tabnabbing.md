# Pruebas de Reverse Tabnabbing

|ID          |
|------------|
|WSTG-CLNT-14|

## Resumen

[Reverse Tabnabbing](https://owasp.org/www-community/attacks/Reverse_Tabnabbing) es un ataque que puede ser usado para redirigir usuarios a páginas de phishing. Esto usualmente se vuelve posible debido a que el atributo `target` de la etiqueta `<a>` se establece a `_blank` lo cual causa que el enlace se abra en una nueva pestaña. Cuando el atributo `rel='noopener noreferrer'` no se usa en la misma etiqueta `<a>`, la página recién abierta puede influir en la página original y redirigirla a un dominio controlado por el atacante.

Dado que el usuario estaba en el dominio original cuando se abrió la nueva pestaña, es menos probable que note que la página ha cambiado, especialmente si la página de phishing es idéntica al dominio original. Cualquier credencial ingresada en el dominio controlado por el atacante terminará así en posesión del atacante.

Los enlaces abiertos vía la función JavaScript `window.open` también son vulnerables a este ataque.

_NOTA: Este es un problema heredado que no afecta a [navegadores modernos](https://caniuse.com/mdn-html_elements_a_implicit_noopener). Versiones antiguas de navegadores populares (Por ejemplo, versiones anteriores a Google Chrome 88) así como Internet Explorer son vulnerables a este ataque._

### Ejemplo

Imaginar una aplicación web donde se permite a los usuarios insertar una URL en su perfil. Si la aplicación es vulnerable a reverse tabnabbing, un usuario malicioso podrá proporcionar un enlace a una página que tiene el siguiente código:

```html
<html>
 <body>
  <script>
    window.opener.location = "https://example.org";
  </script>
<b>Error loading...</b>
 </body>
</html>
```

Hacer clic en el enlace abrirá una nueva pestaña mientras la pestaña original se redirigirá a "example.org". Suponer que "example.org" se ve similar a la aplicación web vulnerable, el usuario es menos probable que note el cambio y es más probable que ingrese información sensible en la página.

## Cómo Probar

- Verificar el código fuente HTML de la aplicación para ver si los enlaces con `target="_blank"` están usando las palabras clave `noopener` y `noreferrer` en el atributo `rel`. Si no, es probable que la aplicación sea vulnerable a reverse tabnabbing. Tal enlace se vuelve explotable si apunta a un sitio de terceros que ha sido comprometido por el atacante, o si es controlado por el usuario.
- Buscar áreas donde un atacante pueda insertar enlaces, i.e. controlar el argumento `href` de una etiqueta `<a>`. Intentar insertar un enlace a una página que tiene el código fuente dado en el ejemplo anterior, y ver si el dominio original se redirige. Esta prueba puede hacerse en IE si otros navegadores no funcionan.

## Remediación

Se recomienda asegurarse que el atributo HTML `rel` esté establecido con las palabras clave `noreferrer` y `noopener` para todos los enlaces.

## Referencias

- [Tabnabbing - Hoja de Referencia HTML5](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#tabnabbing)
- [La vulnerabilidad target="_blank" con ejemplo](https://dev.to/ben/the-targetblank-vulnerability-by-example)
- [Sobre rel=noopener](https://mathiasbynens.github.io/rel-noopener/)
- [Target="_blank" — la vulnerabilidad más subestimada de todas](https://medium.com/@jitbit/target-blank-the-most-underestimated-vulnerability-ever-96e328301f4c)
- [Vulnerabilidad de reverse tabnabbing afecta IBM Business Automation Workflow e IBM Business Process Manager](https://www.ibm.com/support/pages/security-bulletin-reverse-tabnabbing-vulnerability-affects-ibm-business-automation-workflow-and-ibm-business-process-manager-bpm-cve-2020-4490-0)
