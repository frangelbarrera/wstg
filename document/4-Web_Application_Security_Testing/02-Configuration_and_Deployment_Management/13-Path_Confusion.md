# Confusión de rutas

|ID          |
|------------|
|WSTG-CONF-13|

## Resumen

La confusión de rutas ocurre cuando distintos componentes de la ruta de una solicitud (enrutadores, proxies, cachés, WAF y la propia aplicación) no coinciden en lo que identifica una ruta URL determinada, debido a la normalización de rutas, las barras finales, la distinción entre mayúsculas y minúsculas o las diferencias de codificación. Esta discrepancia permite que un atacante consiga que un componente interprete una ruta de una manera y otro la interprete de otra, eludiendo un control de seguridad que depende de la coincidencia de rutas (por ejemplo, una regla de control de acceso, un controlador de ruta o una clave de caché).

Por ejemplo, si las rutas no están configuradas correctamente y el objetivo también usa un CDN, un atacante puede explotar la discrepancia entre lo que sirve el origen y lo que la caché almacena o usa como clave para ejecutar ataques de engaño de caché web.

Nota sobre el alcance: esta prueba trata sobre la confusión de identidad de la propia cadena de ruta entre componentes que deberían coincidir; no trata sobre el uso de `../` o secuencias de recorrido codificadas para salir de un directorio (eso es recorrido de rutas, cubierto en las pruebas de validación de entradas/inyección, por ejemplo, WSTG-ATHZ-01 / pruebas de recorrido de directorios). Los hallazgos de confusión de rutas suelen implicar una ruta sintácticamente válida que no intenta salir del directorio previsto, pero que distintas capas interpretan de manera incoherente.

Por lo tanto, para prevenir otros ataques, el tester debería evaluar esta configuración.

## Objetivos de prueba

- Asegurarse de que las rutas de la aplicación estén configuradas y normalizadas de forma coherente en todos los componentes (aplicación, proxy, caché, CDN, WAF) que toman decisiones basadas en la ruta.
- Identificar casos en los que las barras finales, la distinción entre mayúsculas y minúsculas o las diferencias de codificación provoquen que se eluda un control de seguridad (reglas de autenticación/acceso, enrutamiento, generación de claves de caché) o que este se comporte de forma incoherente.

## Cómo probar

### Pruebas de caja negra

En un escenario de pruebas de caja negra, el tester debería reemplazar todas las rutas existentes por rutas que no existan y, luego, examinar el comportamiento y el código de estado del objetivo.

Por ejemplo, hay una ruta en la aplicación que corresponde a un panel y muestra el saldo de la cuenta del usuario (dinero, créditos de juego, etc.).

Supongamos que la ruta es `https://example.com/user/dashboard`; el tester debería probar las diferentes variantes que el desarrollador podría haber considerado para esta ruta. Para detectar vulnerabilidades de Web Cache Deception, el analista debería considerar una ruta como `https:// example.com/user/dashboard/non.js`. Si la información del panel está visible y el objetivo usa un CDN (u otra caché web), es probable que los ataques de Web Cache Deception sean aplicables.

Para cada ruta protegida o enrutada, pruebe también las siguientes variantes y compare el código de estado, el cuerpo de la respuesta y el control aplicado (si lo hay) con la ruta de referencia:

- Barra final: `/user/dashboard` frente a `/user/dashboard/`: compruebe si una regla de control de acceso, una ruta o una clave de caché se aplica solo a una de las variantes.
- Distinción entre mayúsculas y minúsculas: `/user/dashboard` frente a `/User/Dashboard`: compruebe si el origen y cualquier proxy, caché o WAF frontal normalizan las mayúsculas y minúsculas de forma coherente.
- Normalización de rutas / segmentos de punto que se mantienen dentro del alcance: `/user/dashboard/./` o `/user//dashboard` (barras repetidas); no se trata de salir del directorio mediante recorrido, sino de codificaciones alternativas del mismo recurso.
- Codificación porcentual de separadores de ruta o caracteres reservados: `/user%2Fdashboard`, `/user/dashboard%2F..%2F` (cuando el resultado decodificado sigue resolviéndose dentro del alcance); compruebe si el componente que aplica el control de seguridad decodifica antes o después de la coincidencia.
- Combinaciones de codificación y mayúsculas/minúsculas para determinar si un WAF o una capa de control de acceso compara la cadena sin procesar mientras la aplicación compara la cadena decodificada/normalizada (o viceversa).

Existe un hallazgo cuando cualquier variante produce un resultado distinto al de la ruta canónica (elude la autenticación, llega a otro controlador de ruta o se almacena en caché o se sirve), debido a que los componentes discrepan sobre la identidad de la ruta; no porque la ruta haya salido del directorio previsto.

### Pruebas de caja blanca

Examine la configuración de enrutamiento de la aplicación. La mayoría de las veces, los desarrolladores usan expresiones regulares en el enrutamiento de la aplicación.

En este ejemplo, en el archivo `urls.py` de una aplicación del framework Django, vemos un ejemplo de confusión de rutas. El desarrollador no usó la expresión regular correcta, lo que provocó una vulnerabilidad:

```python
    from django.urls import re_path
    from . import views

    urlpatterns = [

        re_path(r'.*^dashboard', views.path_confusion ,name = 'index'),

    ]
```

Si el usuario también abre en el navegador la ruta `https://example.com/dashboard/none.js`, puede mostrarse la información del panel del usuario y, si el objetivo usa un CDN o una caché web, puede llevarse a cabo un ataque de Web Cache Deception.

## Herramientas

- [Zed Attack Proxy](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Remediación

- Evite clasificar o gestionar el almacenamiento en caché según la extensión del archivo o la ruta (aproveche el tipo de contenido).
- Asegúrese de que los mecanismos de caché respeten los encabezados cache-control especificados por la aplicación.
- Implemente un manejo de errores «File Not Found» y redirecciones que cumplan con RFC.
- Normalice las rutas (mayúsculas y minúsculas, barra final, codificación y separadores repetidos) de forma coherente y a la misma forma canónica en todos los componentes que tomen decisiones de seguridad basadas en la ruta (aplicación, proxy inverso, caché, CDN, WAF).
- Realice la comparación de rutas relevante para la seguridad (control de acceso, enrutamiento) después de la normalización/decodificación, no sobre la cadena sin procesar de la línea de solicitud.

## Referencias

- [Cómo eludir las contramedidas contra el envenenamiento de caché web](https://portswigger.net/research/bypassing-web-cache-poisoning-countermeasures)
- [Almacenada en caché y confundida: el engaño de caché web en la práctica (USENIX Security 2020)](https://www.usenix.org/conference/usenixsecurity20/presentation/mirheidari)
- [Ataque de engaño de caché web](https://omergil.blogspot.com/2017/02/web-cache-deception-attack.html)