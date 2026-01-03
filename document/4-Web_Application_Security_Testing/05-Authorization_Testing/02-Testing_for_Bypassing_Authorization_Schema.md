# Pruebas de omisión del esquema de autorización

|ID          |
|------------|
|WSTG-ATHZ-02|

## Resumen

Este tipo de prueba se centra en verificar cómo se ha implementado el esquema de autorización para cada rol o privilegio para acceder a funciones y recursos reservados.

Para cada rol específico que el evaluador tenga durante la evaluación y para cada función y solicitud que la aplicación ejecute durante la fase posterior a la autenticación, es necesario verificar:

- ¿Es posible acceder a ese recurso incluso si el usuario no está autenticado?
- ¿Es posible acceder a ese recurso después del cierre de sesión?
- ¿Es posible acceder a funciones y recursos que deberían ser accesibles para un usuario que tenga un rol o privilegio diferente?

Intente acceder a la aplicación como un usuario administrativo y rastree todas las funciones administrativas.

- ¿Es posible acceder a funciones administrativas si el evaluador está conectado como un usuario no administrativo?
- ¿Es posible utilizar estas funciones administrativas como un usuario con un rol diferente y para quien esa acción debería ser denegada?

## Objetivos de la prueba

- Evaluar si es posible el acceso no autenticado, horizontal o vertical.

## Cómo probar

- Acceder a recursos y realizar operaciones sin iniciar sesión. - Solicitud directa de página ([navegación forzada](https://owasp.org/www-community/attacks/Forced_browsing))
- Acceder a recursos y realizar operaciones horizontalmente.
- Acceder a recursos y realizar operaciones verticalmente.

### Pruebas de acceso no autenticado básico

#### Usando un navegador manualmente

Cuando una aplicación web no aplica correctamente los mecanismos de control de acceso, los recursos sensibles quedan expuestos, permitiendo a usuarios no autenticados verlos. Por ejemplo, si un usuario solicita directamente una página diferente mediante navegación forzada, esa página puede no verificar la autorización del usuario anónimo antes de conceder acceso. Intente acceder directamente a una página protegida a través de la barra de direcciones en su navegador para probar usando este método.

![Solicitud directa a página protegida](images/Basm-directreq.jpg)\
*Figura 4.5.2-1: Solicitud directa a página protegida*

#### Usando automatización

Este proceso se puede automatizar si tiene una lista de todos los puntos finales con herramientas como ffuf, gobuster, ZAP y Burp Suite Intruder.

Para ZAP, usar un complemento para [Pruebas de control de acceso](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/) permite a los evaluadores determinar qué partes de la aplicación están disponibles para usuarios anónimos e identificar posibles problemas de control de acceso.

Para Burp Suite, herramientas integradas como Intruder y una serie de complementos, incluyendo Autorize, ayudan al evaluador a automatizar las pruebas de autorización.

### Pruebas de omisión horizontal del esquema de autorización

Para cada función, rol específico o solicitud que la aplicación ejecute, es necesario verificar:

- ¿Es posible acceder a recursos que deberían ser accesibles para un usuario que tenga una identidad diferente con el mismo rol o privilegio?
- ¿Es posible operar funciones en recursos que deberían ser accesibles para un usuario que tenga una identidad diferente?

Para cada rol:

1. Registre o genere dos usuarios con privilegios idénticos.
2. Establezca y mantenga dos sesiones diferentes activas (una para cada usuario).
3. Para cada solicitud, cambie los parámetros relevantes y el identificador de sesión del token uno al token dos y diagnostique las respuestas para cada token.
4. Una aplicación se considerará vulnerable si las respuestas son las mismas, contienen los mismos datos privados o indican una operación exitosa en el recurso o datos de otros usuarios.

Por ejemplo, suponga que la función `viewSettings` es parte de cada menú de cuenta de la aplicación con el mismo rol, y es posible acceder a ella solicitando la siguiente URL: `https://www.example.com/account/viewSettings`. Entonces, la siguiente solicitud HTTP se genera al llamar a la función `viewSettings`:

```http
POST /account/viewSettings HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=USER_SESSION

username=example_user
```

Respuesta válida y legítima:

```html
HTTP1.1 200 OK
[other HTTP headers]

{
  "username": "example_user",
  "email": "example@email.com",
  "address": "Example Address"
}
```

El atacante puede intentar ejecutar esa solicitud con el mismo parámetro `username`:

```html
POST /account/viewCCpincode HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ATTACKER_SESSION

username=example_user
```

Si la respuesta del atacante contiene los datos del `example_user`, entonces la aplicación es vulnerable a ataques de movimiento lateral, donde un usuario puede leer o escribir datos de otros usuarios.

### Pruebas de acceso a funciones administrativas

Por ejemplo, suponga que la función `addUser` es parte del menú administrativo de la aplicación, y es posible acceder a ella solicitando la siguiente URL `https://www.example.com/admin/addUser`.

Entonces, la siguiente solicitud HTTP se genera al llamar a la función `addUser`:

```http
POST /admin/addUser HTTP/1.1
Host: www.example.com
[...]

userID=fakeuser&role=3&group=grp001
```

Otras preguntas o consideraciones irían en la siguiente dirección:

- ¿Qué sucede si un usuario no administrativo intenta ejecutar esa solicitud?
- ¿Se creará el usuario?
- Si es así, ¿puede el nuevo usuario usar sus privilegios?

### Pruebas de acceso a recursos asignados a un rol diferente

Varias aplicaciones configuran controles de recursos basados en roles de usuario. Tomemos un ejemplo de currículos o CV (curriculum vitae) subidos en un formulario de carreras a un bucket S3.

Como usuario normal, intente acceder a la ubicación de esos archivos. Si puede recuperarlos, modificarlos o eliminarlos, entonces la aplicación es vulnerable.

### Pruebas de manejo de cabeceras de solicitud especiales

Algunas aplicaciones admiten cabeceras no estándar como `X-Original-URL` o `X-Rewrite-URL` para permitir anular la URL de destino en solicitudes con la especificada en el valor de la cabecera.

Este comportamiento se puede aprovechar en una situación en la que la aplicación está detrás de un componente que aplica restricciones de control de acceso basadas en la URL de la solicitud.

El tipo de restricción de control de acceso basada en la URL de la solicitud puede ser, por ejemplo, bloquear el acceso desde internet a una consola de administración expuesta en `/console` o `/admin`.

Para detectar el soporte para la cabecera `X-Original-URL` o `X-Rewrite-URL`, se pueden aplicar los siguientes pasos.

#### 1. Enviar una solicitud normal sin ninguna cabecera X-Original-Url o X-Rewrite-Url

```http
GET / HTTP/1.1
Host: www.example.com
[...]
```

#### 2. Enviar una solicitud con una cabecera X-Original-Url apuntando a un recurso no existente

```html
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
[...]
```

#### 3. Enviar una solicitud con una cabecera X-Rewrite-Url apuntando a un recurso no existente

```html
GET / HTTP/1.1
Host: www.example.com
X-Rewrite-URL: /donotexist2
[...]
```

Si la respuesta para cualquiera de las solicitudes contiene marcadores de que el recurso no se encontró, esto indica que la aplicación admite las cabeceras de solicitud especiales. Estos marcadores pueden incluir el código de estado de respuesta HTTP 404, o un mensaje de "recurso no encontrado" en el cuerpo de la respuesta.

Una vez validado el soporte para la cabecera `X-Original-URL` o `X-Rewrite-URL`, entonces se puede aprovechar el intento de omisión contra la restricción de control de acceso enviando la solicitud esperada a la aplicación pero especificando una URL "permitida" por el componente frontend como la URL de solicitud principal y especificando la URL de destino real en la cabecera `X-Original-URL` o `X-Rewrite-URL` dependiendo de cuál esté soportada. Si ambas están soportadas, pruebe una después de la otra para verificar para cuál cabecera la omisión es efectiva.

#### 4. Otras cabeceras a considerar

A menudo, los paneles de administración o bits relacionados con la administración solo son accesibles para clientes en redes locales, por lo tanto, puede ser posible abusar de varias cabeceras HTTP relacionadas con proxy o reenvío para obtener acceso. Algunas cabeceras y valores a probar son:

- Cabeceras:
    - `X-Forwarded-For`
    - `X-Forward-For`
    - `X-Remote-IP`
    - `X-Originating-IP`
    - `X-Remote-Addr`
    - `X-Client-IP`
- Valores
    - `127.0.0.1` (o cualquier cosa en los espacios de direcciones `127.0.0.0/8` o `::1/128`)
    - `localhost`
    - Cualquier dirección [RFC1918](https://tools.ietf.org/html/rfc1918):
        - `10.0.0.0/8`
        - `172.16.0.0/12`
        - `192.168.0.0/16`
    - Direcciones locales de enlace: `169.254.0.0/16`

Nota: Incluir un elemento de puerto junto con la dirección o nombre de host también puede ayudar a omitir protecciones de borde como firewalls de aplicaciones web, etc.
Por ejemplo: `127.0.0.4:80`, `127.0.0.4:443`, `127.0.0.4:43982`

## Remediation

Emplee los principios de menor privilegio en los usuarios, roles y recursos para asegurar que no ocurra acceso no autorizado.

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
    - [Complemento ZAP: Pruebas de control de acceso](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)
- [Port Swigger Burp Suite](https://portswigger.net/burp)
    - [Extensión Burp: AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix/)
    - [Extensión Burp: Autorize](https://github.com/Quitten/Autorize)

## Referencias

[OWASP Application Security Verification Standard 4.0.1](https://github.com/OWASP/ASVS/tree/master/4.0), v4.0.1-1, v4.0.1-4, v4.0.1-9, v4.0.1-16