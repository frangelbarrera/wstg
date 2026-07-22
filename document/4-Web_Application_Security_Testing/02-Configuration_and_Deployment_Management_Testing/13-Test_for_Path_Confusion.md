# Probar Path Confusion

|ID          |
|------------|
|WSTG-CONF-13|

## Resumen

La configuración apropiada de las rutas de aplicación es importante porque, si las rutas no se configuran correctamente, permiten a un atacante explotar otras vulnerabilias en una etapa posterior usando esta mala configuración.

Por ejemplo, si las rutas no se configuran correctamente y el objetivo también usa un CDN, el atacante puede usar esta mala configuración para ejecutar ataques de engaño de caché web.

Como resultado, para prevenir otros ataques, esta configuración debería ser evaluada por el tester.

## Objetivos de Prueba

- Asegurar que las rutas de aplicación estén configuradas correctamente.

## Cómo Probar

### Pruebas de Caja Negra

En un escenario de pruebas de caja negra, el tester debería reemplazar todas las rutas existentes con rutas que no existen, y luego examinar el comportamiento y código de estado del objetivo.

Por ejemplo, hay una ruta en la aplicación que es un dashboard y muestra la cantidad del saldo de cuenta del usuario (dinero, créditos de juego, etc).

Asumir que la ruta es `https://example.com/user/dashboard`, el tester debería probar los diferentes modos que el desarrollador podría haber considerado para esta ruta. Para vulnerabilidades de Web Cache Deception el analista debería considerar una ruta tal como `https://example.com/user/dashboard/non.js` si la información del dashboard es visible, y el objetivo usa un CDN (u otra caché web), entonces los ataques de Web Cache Deception son probablemente aplicables.

### Pruebas de Caja Blanca

Examinar la configuración de enrutamiento de la aplicación, la mayoría del tiempo, los desarrolladores usan expresiones regulares en el enrutamiento de la aplicación.

En este ejemplo, en el archivo `urls.py` de una aplicación del framework Django, vemos un ejemplo de Path Confusion. El desarrollador no usó la expresión regular correcta resultando en una vulnerabilidad:

```python
    from django.urls import re_path
    from . import views

    urlpatterns = [

        re_path(r'.*^dashboard', views.path_confusion ,name = 'index'),

    ]
```

Si la ruta `https://example.com/dashboard/none.js` también es abierta por el usuario en el navegador, la información del dashboard del usuario puede mostrarse, y si el objetivo usa un CDN o caché web, un ataque de Web Cache Deception puede implementarse.

## Herramientas

- [Zed Attack Proxy](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Remediación

- Abstenerse de clasificar/manejar caché basado en extensión de archivo o ruta (aprovechar content-type).
- Asegurar que los mecanismos de caché se adhieran a los encabezados cache-control especificados por la aplicación.
- Implementar manejo de File Not Found y redirecciones que cumplan con RFC.

## Referencias

- [Bypassing Web Cache Poisoning Countermeasures](https://portswigger.net/research/bypassing-web-cache-poisoning-countermeasures)
- [Path confusion: Web cache deception threatens user information online](https://portswigger.net/daily-swig/path-confusion-web-cache-deception-threatens-user-information-online)
- [Web Cache Deception Attack](https://omergil.blogspot.com/2017/02/web-cache-deception-attack.html)
