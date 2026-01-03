# Pruebas de inyección de plantillas del lado del servidor

|ID          |
|------------|
|WSTG-INPV-18|

## Resumen

Las aplicaciones web comúnmente utilizan tecnologías de plantillas del lado del servidor (Jinja2, Twig, FreeMaker, etc.) para generar respuestas HTML dinámicas. Las vulnerabilidades de inyección de plantillas del lado del servidor (SSTI) ocurren cuando la entrada del usuario se incrusta en una plantilla de manera insegura y resulta en la ejecución remota de código en el servidor. Cualquier característica que admita marcado avanzado proporcionado por el usuario puede ser vulnerable a SSTI, incluyendo páginas wiki, reseñas, aplicaciones de marketing, sistemas CMS, etc. Algunos motores de plantillas emplean varios mecanismos (por ejemplo, sandbox, listas de permitidos, etc.) para protegerse contra SSTI.

### Ejemplo - Twig

El siguiente ejemplo es un extracto del proyecto [Extreme Vulnerable Web Application](https://github.com/s4n7h0/xvwa).

```php
public function getFilter($name)
{
        [snip]
        foreach ($this->filterCallbacks as $callback) {
        if (false !== $filter = call_user_func($callback, $name)) {
            return $filter;
        }
    }
    return false;
}
```

En la función getFilter, el `call_user_func($callback, $name)` es vulnerable a SSTI: el parámetro `name` se obtiene de la solicitud HTTP GET y se ejecuta por el servidor:

![SSTI XVWA Example](images/SSTI_XVWA.jpeg)\
*Figura 4.7.18-1: Ejemplo SSTI XVWA*

### Ejemplo - Flask/Jinja2

El siguiente ejemplo utiliza Flask y el motor de plantillas Jinja2. La función `page` acepta un parámetro 'name' de una solicitud HTTP GET y renderiza una respuesta HTML con el contenido de la variable `name`:

```python
@app.route("/page")
def page():
    name = request.values.get('name')
    output = Jinja2.from_string('Hello ' + name + '!').render()
    return output
```

Este fragmento de código es vulnerable a XSS, pero también es vulnerable a SSTI. Usando lo siguiente como payload en el parámetro `name`:

```bash
$ curl -g 'https://www.target.com/page?name={{7*7}}'
Hello 49!
```

## Objetivos de la prueba

- Detectar puntos de vulnerabilidad de inyección de plantillas.
- Identificar el motor de plantillas.
- Construir el exploit.

## Cómo probar

Las vulnerabilidades SSTI existen ya sea en contexto de texto o de código. En contexto de texto plano, los usuarios pueden usar 'texto' de forma libre con código HTML directo. En contexto de código, la entrada del usuario también puede colocarse dentro de una declaración de plantilla (por ejemplo, en un nombre de variable).

### Identificar vulnerabilidad de inyección de plantillas

El primer paso en las pruebas de SSTI en contexto de texto plano es construir expresiones de plantilla comunes utilizadas por varios motores de plantillas como payloads y monitorear las respuestas del servidor para identificar qué expresión de plantilla fue ejecutada por el servidor.

Ejemplos de expresiones de plantilla comunes:

```text
a{{bar}}b
a{{7*7}}
{var} ${var} {{var}} <%var%> [% var %]
```

En este paso se recomienda una lista extensa de [cadenas/payloads de prueba de expresiones de plantilla](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection).

Las pruebas de SSTI en contexto de código son ligeramente diferentes. Primero, el probador construye la solicitud que resulta en respuestas del servidor en blanco o con error. En el ejemplo a continuación, el parámetro HTTP GET se inserta en la variable `personal_greeting` en una declaración de plantilla:

```text
personal_greeting=username
Hello user01
```

Usando el siguiente payload - la respuesta del servidor es en blanco "Hello":

```text
personal_greeting=username<tag>
Hello
```

En el siguiente paso es salir de la declaración de plantilla e inyectar una etiqueta HTML después de ella usando el siguiente payload

```text
personal_greeting=username}}<tag>
Hello user01 <tag>
```

### Identificar el motor de plantillas

Basado en la información del paso anterior, ahora el probador tiene que identificar qué motor de plantillas se utiliza proporcionando varias expresiones de plantilla. Basado en las respuestas del servidor, el probador deduce el motor de plantillas utilizado. Este enfoque manual se discute en mayor detalle en [este](https://portswigger.net/blog/server-side-template-injection?#Identify) artículo de PortSwigger. Para automatizar la identificación de la vulnerabilidad SSTI y el motor de plantillas, varias herramientas están disponibles, incluyendo [Tplmap](https://github.com/epinna/tplmap) o la extensión [Backslash Powered Scanner Burp Suite](https://github.com/PortSwigger/backslash-powered-scanner).

### Construir el exploit de RCE

El objetivo principal en este paso es identificar para obtener mayor control en el servidor con un exploit de RCE estudiando la documentación de la plantilla e investigando. Las áreas clave de interés son:

- **Para autores de plantillas** secciones que cubren sintaxis básica.
- Secciones de **consideraciones de seguridad**.
- Listas de métodos integrados, funciones, filtros y variables.
- Listas de extensiones/plugins.

El probador también puede identificar qué otros objetos, métodos y propiedades pueden exponerse enfocándose en el objeto `self`. Si el objeto `self` no está disponible y la documentación no revela los detalles técnicos, se recomienda una fuerza bruta del nombre de la variable. Una vez identificado el objeto, el siguiente paso es recorrer el objeto para identificar todos los métodos, propiedades y atributos que son accesibles a través del motor de plantillas. Esto podría llevar a otros tipos de hallazgos de seguridad, incluyendo escaladas de privilegios, divulgación de información sobre contraseñas de aplicación, claves API, configuraciones y variables de entorno, etc.

## Herramientas

- [Tplmap](https://github.com/epinna/tplmap)
- [Extensión Backslash Powered Scanner Burp Suite](https://github.com/PortSwigger/backslash-powered-scanner)
- [Lista de cadenas/payloads de prueba de expresiones de plantilla](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

## Referencias

- [James Kettle: Server-Side Template Injection:RCE for the modern webapp (whitepaper)](https://portswigger.net/kb/papers/serversidetemplateinjection.pdf)
- [Server-Side Template Injection](https://portswigger.net/blog/server-side-template-injection)
- [Exploring SSTI in Flask/Jinja2](https://www.lanmaster53.com/2016/03/exploring-ssti-flask-jinja2/)
- [Server-Side Template Injection: from detection to Remote shell](https://www.okiok.com/server-side-template-injection-from-detection-to-remote-shell/)
- [Extreme Vulnerable Web Application](https://github.com/s4n7h0/xvwa)
- [Exploiting SSTI in Thymeleaf](https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/)