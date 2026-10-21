# Reverse Tabnabbing

|ID          |
|------------|
|WSTG-CLNT-14|

## Resumen

[Reverse Tabnabbing](https://owasp.org/www-community/attacks/Reverse_Tabnabbing) es un ataque que puede utilizarse para redirigir a los usuarios a páginas de phishing. Esto suele ser posible porque el atributo `target` de la etiqueta `<a>` está establecido en `_blank`, lo que hace que el enlace se abra en una pestaña nueva. Cuando no se utiliza el atributo `rel='noopener noreferrer'` en la misma etiqueta `<a>`, la página recién abierta puede influir en la página original y redirigirla a un dominio controlado por el atacante.

Como el usuario estaba en el dominio original cuando se abrió la pestaña nueva, es menos probable que note que la página ha cambiado, especialmente si la página de phishing es idéntica a la del dominio original. Por lo tanto, las credenciales introducidas en el dominio controlado por el atacante acabarán en manos de este.

Los enlaces abiertos mediante la función JavaScript `window.open` también son vulnerables a este ataque.

_NOTA: Este es un problema heredado que no afecta a los [navegadores modernos](https://caniuse.com/mdn-html_elements_a_implicit_noopener). Las versiones antiguas de navegadores populares (por ejemplo, las anteriores a Google Chrome 88), así como Internet Explorer, son vulnerables a este ataque._

### Ejemplo

Imaginemos una aplicación web que permite a los usuarios insertar una URL en su perfil. Si la aplicación es vulnerable a reverse tabnabbing, un usuario malicioso podrá proporcionar un enlace a una página que tenga el siguiente código:

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

Al hacer clic en el enlace, se abrirá una pestaña nueva y la pestaña original se redirigirá a "example.org". Si "example.org" se parece a la aplicación web vulnerable, es menos probable que el usuario note el cambio y más probable que introduzca información confidencial en la página.

## Cómo probar

- Compruebe el código fuente HTML de la aplicación para ver si los enlaces con `target="_blank"` utilizan las palabras clave `noopener` y `noreferrer` en el atributo `rel`. Si no las utilizan, es probable que la aplicación sea vulnerable a reverse tabnabbing. El enlace será explotable si apunta a un sitio de terceros que el atacante haya comprometido o si está controlado por el usuario.
- Busque áreas en las que un atacante pueda insertar enlaces, es decir, controlar el argumento `href` de una etiqueta `<a>`. Intente insertar un enlace a una página que tenga el código fuente del ejemplo anterior y compruebe si se redirige el dominio original. Esta prueba puede realizarse en IE si los demás navegadores no funcionan.

## Remediación

Se recomienda asegurarse de que el atributo HTML `rel` esté establecido con las palabras clave `noreferrer` y `noopener` en todos los enlaces.

## Referencias

- [Tabnabbing - Hoja de referencia de HTML5](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#tabnabbing)
- [La vulnerabilidad de target="_blank" con un ejemplo](https://dev.to/ben/the-targetblank-vulnerability-by-example)
- [Acerca de rel=noopener](https://mathiasbynens.github.io/rel-noopener/)
- [Target=”_blank” - la vulnerabilidad más subestimada de todas](https://medium.com/@jitbit/target-blank-the-most-underestimated-vulnerability-ever-96e328301f4c)
- [La vulnerabilidad de reverse tabnabbing afecta a IBM Business Automation Workflow e IBM Business Process Manager](https://www.ibm.com/support/pages/security-bulletin-reverse-tabnabbing-vulnerability-affects-ibm-business-automation-workflow-and-ibm-business-process-manager-bpm-cve-2020-4490-0)