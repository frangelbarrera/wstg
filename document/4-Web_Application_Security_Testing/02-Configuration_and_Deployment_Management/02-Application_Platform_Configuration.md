# Configuración de la plataforma de aplicación

|ID          |
|------------|
|WSTG-CONF-02|

## Resumen

La configuración adecuada de los elementos individuales que componen una arquitectura de aplicación es importante para prevenir errores que podrían comprometer la seguridad de toda la arquitectura.

Revisar y probar las configuraciones son tareas fundamentales para crear y mantener una arquitectura. Esto se debe a que varios sistemas suelen incluir configuraciones genéricas que quizá no se ajusten bien a las tareas que se supone que deben realizar en los sitios específicos donde se instalan.

Aunque la instalación típica de un servidor web y de aplicaciones incluye muchas funciones (como ejemplos de aplicaciones, documentación y páginas de prueba), todo lo que no sea esencial debería eliminarse antes del despliegue para evitar su explotación después de la instalación.

## Objetivos de las pruebas

- Asegurarse de que los archivos predeterminados, de ejemplo y conocidos se hayan eliminado de la plataforma, la imagen de contenedor o el paquete sin servidor desplegados.
- Validar que en los entornos de producción no queden habilitados código de depuración, endpoints de depuración, modos de depuración ni módulos o dependencias que no se utilicen.
- Revisar los mecanismos de registro establecidos para la aplicación, incluidos los datos confidenciales que podrían capturar y quién o qué puede acceder a ellos o conservarlos.
- Confirmar que se hayan aplicado los ajustes de seguridad de producción específicos del framework, el contenedor y el entorno sin servidor, en lugar de dejar los valores predeterminados inseguros.

## Cómo probar

### Pruebas de caja negra

#### Archivos y directorios de ejemplo y conocidos

En una instalación predeterminada, muchos servidores web, servidores de aplicaciones y frameworks proporcionan aplicaciones de ejemplo, estructuras iniciales y páginas de prueba para que los desarrolladores puedan confirmar que el servidor funciona correctamente justo después de la instalación o de ejecutar `init`. Históricamente, esta clase de problema se ejemplificaba con casos como CVE-1999-0449 (denegación de servicio en IIS cuando se había instalado el sitio de ejemplo Exair), CAN-2002-1744 (vulnerabilidad de recorrido de directorios en CodeBrws.asp en Microsoft IIS 5.0) o CAN-2003-1172 (recorrido de directorios en el ejemplo view-source de Apache Cocoon). El problema subyacente (contenido de ejemplo o estructura inicial que se incluye en el entorno de producción y no se elimina) no ha cambiado; solo cambian los objetivos:

- Estructuras iniciales de frameworks que se dejan en su lugar: páginas de bienvenida predeterminadas de Django/Rails/Laravel, páginas de error o «marca blanca» predeterminadas de Spring Boot, páginas de inicio predeterminadas de create-react-app/Next.js/Vite o un `index.html` sin modificar de una plantilla inicial. Cada una puede revelar el framework y la versión exactos en uso y, en ocasiones, rutas de la estructura inicial que hayan quedado.
- Documentación o consolas de API generadas automáticamente que siguen accesibles en producción: Swagger UI/OpenAPI (`/swagger-ui`, `/swagger.json`, `/v2/api-docs`), GraphQL Playground/GraphiQL o la reflexión de gRPC. A menudo exponen el esquema completo, incluidas operaciones no documentadas o reservadas para administradores, y algunas permiten ejecutar solicitudes directamente («try it out») sin comprobaciones de autorización adicionales.
- Clases de archivos de ejemplo heredados: `phpinfo.php`, `test.php`, `info.php` y scripts predeterminados de instalación o configuración de CMS (por ejemplo, `wp-admin/install.php` y `installation/` de Joomla) que siguen accesibles después de la puesta en producción.

Los escáneres CGI/web que cuentan con una base de datos de archivos y directorios de ejemplo conocidos permiten comprobarlo rápidamente. El descubrimiento automatizado debería ir seguido de una revisión manual del sitio para detectar estructuras iniciales específicas de frameworks que los escáneres genéricos no identifican.

#### Endpoints y modos de depuración

La mayoría de los frameworks modernos incluyen un modo de depuración o desarrollo que no es seguro para producción, ya que está diseñado para ofrecer al desarrollador la máxima visibilidad (trazas completas de pila, ejecución de código en el navegador y volcados de variables de entorno), a costa de exponer exactamente la misma información a un atacante si se deja habilitado:

- Django: `DEBUG=True` muestra trazas completas de pila, valores de variables locales y la lista de aplicaciones y ajustes instalados ante cualquier excepción no controlada. Provoca un error (por ejemplo, solicita una ruta inexistente o envía una entrada mal formada) y comprueba si la respuesta muestra una página de depuración de Django en lugar de un error 404/500 genérico.
- Flask/Werkzeug: si está habilitado (`app.run(debug=True)` o `FLASK_DEBUG=1`), el depurador de Werkzeug expone una consola de Python en el navegador ante excepciones no controladas. Aunque en teoría está protegido por un PIN, a menudo este puede deducirse a partir de información que también aparece en la página de error o de valores predecibles del propio host en versiones anteriores de Werkzeug. Cualquier depurador de Werkzeug accesible en producción debe considerarse un hallazgo crítico (ejecución remota de código), no una mera divulgación de información.
- Spring Boot Actuator: endpoints como `/actuator/env`, `/actuator/heapdump`, `/actuator/httptrace` y `/actuator/mappings` pueden exponer variables de entorno y secretos, un volcado completo del heap (que puede contener credenciales y datos de sesión) y el enrutamiento interno. Comprueba si los endpoints de Actuator son accesibles en producción y, de ser así, si `management.endpoints.web.exposure.include` se ha limitado en lugar de dejar un valor predeterminado demasiado amplio.
- Rails: `config.consider_all_requests_local = true` (valor predeterminado en desarrollo) muestra páginas de error detalladas con fragmentos de código fuente y datos de solicitudes y sesiones. Confirma que sea `false` en la configuración del entorno de producción.
- Express/Node.js: controladores de errores predeterminados que devuelven `err.stack` al cliente, o middleware de desarrollo integrado en un framework (por ejemplo, middleware de servidor de desarrollo de un empaquetador) que se deja montado en una compilación de producción.
- Next.js/React y otras herramientas de compilación para SPA: las compilaciones de producción que siguen incluyendo mapas de código fuente de JavaScript (archivos `.map`) o dejan accesible un servidor de desarrollo permiten reconstruir el código fuente original a partir de los paquetes minimizados.
- Introspección de GraphQL: aunque no haya una interfaz de Playground, responder a la consulta de introspección (`{ __schema { types { name } } }`) en producción revela el esquema completo, incluidos campos y mutaciones que no se exponen en la superficie documentada de la API.

Como prueba general, provoca deliberadamente un error de aplicación (entrada mal formada, parámetro ausente o excepción forzada) en cada componente o framework distinto en uso y confirma que la respuesta sea un error genérico que no revele información, en lugar de una página de depuración del framework.

#### Revisión de comentarios

Es muy común que los programadores agreguen comentarios al desarrollar aplicaciones web grandes. Sin embargo, los comentarios incluidos en línea en el código HTML podrían revelar información interna que no debería estar disponible para un atacante. En ocasiones, se comenta una parte del código fuente cuando una funcionalidad ya no es necesaria, pero ese comentario se filtra involuntariamente en las páginas HTML que se devuelven a los usuarios.

Se deben revisar los comentarios para determinar si se filtra información a través de ellos. Esta revisión solo puede realizarse exhaustivamente mediante un análisis del contenido estático y dinámico del servidor web y búsquedas de archivos. Puede ser útil recorrer el sitio de forma automática o guiada y almacenar todo el contenido recuperado. Después, se puede buscar en ese contenido para analizar los comentarios HTML que haya en el código.

#### Configuración del sistema

Se pueden usar diversas herramientas, documentos o listas de comprobación para ofrecer a los profesionales de TI y seguridad una evaluación detallada de la conformidad de los sistemas objetivo con distintas líneas base o referencias de configuración. Algunas de estas herramientas son:

- [CIS-CAT Lite](https://learn.cisecurity.org/cis-cat-lite)
- [Attack Surface Analyzer de Microsoft](https://github.com/microsoft/AttackSurfaceAnalyzer)
- [National Checklist Program de NIST](https://ncp.nist.gov/repository)

### Pruebas de caja gris

#### Revisión de la configuración

La configuración del servidor web o de aplicaciones desempeña un papel importante en la protección del contenido del sitio y debe revisarse cuidadosamente para detectar errores comunes de configuración. Naturalmente, la configuración recomendada varía según la política del sitio y la funcionalidad que deba proporcionar el software del servidor. Sin embargo, en la mayoría de los casos deberían seguirse las directrices de configuración (proporcionadas por el proveedor del software o por terceros) para determinar si el servidor se ha protegido adecuadamente.

No es posible indicar de manera genérica cómo debería configurarse un servidor; sin embargo, deben tenerse en cuenta algunas directrices comunes:

- Habilita solo los módulos del servidor (extensiones ISAPI en el caso de IIS) que necesite la aplicación. Esto reduce la superficie de ataque, ya que el servidor disminuye en tamaño y complejidad al deshabilitar módulos de software. También evita que las vulnerabilidades que puedan aparecer en el software del proveedor afecten al sitio si solo están presentes en módulos que ya se han deshabilitado. El mismo principio se aplica a las pilas modernas de middleware y módulos: revisa los módulos del servidor web (módulos de Apache/Nginx e IIS), el middleware del framework de la aplicación y las listas de paquetes o dependencias instalados (`npm ls`, `pip list`, etc.) para detectar cualquier elemento incluido por una plantilla predeterminada que en realidad no se utilice. Una dependencia sin usar sigue siendo una superficie de ataque y una carga de mantenimiento de parches, aunque el código de la aplicación nunca la invoque.
- Gestiona los errores del servidor (40x o 50x) con páginas personalizadas, en lugar de usar las páginas predeterminadas del servidor web. Asegúrate en particular de que los errores de la aplicación no se devuelvan al usuario final y de que no se filtre código a través de ellos, ya que esto ayudaría a un atacante. Es muy común olvidar este punto, puesto que los desarrolladores necesitan esta información en los entornos previos a producción.
- Asegúrate de que el software del servidor se ejecute con privilegios mínimos en el sistema operativo. Esto evita que un error en el software del servidor comprometa directamente todo el sistema, aunque un atacante podría elevar sus privilegios después de ejecutar código como el servidor web.
- Asegúrate de que el software del servidor registre correctamente tanto los accesos legítimos como los errores.
- Asegúrate de que el servidor esté configurado para gestionar correctamente las sobrecargas y prevenir ataques de denegación de servicio. Comprueba que el rendimiento del servidor se haya ajustado adecuadamente.
- Nunca concedas a identidades que no sean administrativas (con excepción de `NT SERVICE\WMSvc`) acceso de lectura o escritura a applicationHost.config, redirection.config y administration.config.
- Nunca compartas applicationHost.config, redirection.config ni administration.config en la red.
- No almacenes información confidencial en archivos de configuración legibles.
- Cifra los datos confidenciales de configuración cuando sea necesario.
- Restringe el acceso a la configuración compartida y protégela adecuadamente.

#### Registro

El registro es una parte importante de la seguridad de las aplicaciones. Facilita la supervisión, la respuesta ante incidentes y las investigaciones. Revisa tanto el registro de la plataforma como el de la aplicación para determinar si se registran los eventos relevantes para la seguridad, si los registros son útiles y si el proceso de registro introduce riesgos adicionales.

Durante la revisión, comprueba lo siguiente:

- Se registran los eventos relevantes para la seguridad (autenticación, fallos de autorización, acciones administrativas y fallos de validación de entradas).
- Las entradas de registro contienen suficiente contexto para facilitar la investigación.
- Datos confidenciales en los registros: comprueba que no se registren contraseñas, tokens o claves de API, identificadores de sesión, números completos de tarjetas de pago ni otros datos personales regulados. Una causa sorprendentemente común es dejar habilitado en producción el registro detallado o de depuración (por ejemplo, un framework que registra los cuerpos completos de solicitudes y respuestas, o una biblioteca que registra objetos de excepción que incluyen credenciales pasadas como argumentos) y las integraciones de terceros para seguimiento de errores o APM que reenvían datos sin procesar de solicitudes (encabezados y cuerpos) fuera del host de forma predeterminada.
- Los datos registrados se gestionan de forma segura antes de almacenarlos y mostrarlos (por ejemplo, las entradas no se muestran más adelante sin codificar en una interfaz de visualización de registros, lo que podría habilitar XSS almacenado mediante inyección en los registros).
- Acceso a los registros: restringido al personal o los servicios autorizados. Comprueba tanto los propios archivos o flujos de registros (permisos de archivo; consulta [Permisos de archivos](09-File_Permissions.md)) como cualquier plataforma centralizada de registros (SIEM, servicio de registro en la nube) para detectar permisos de lectura demasiado amplios o falta de aislamiento por inquilino en sistemas multiinquilino.
- Retención: confirma que exista un periodo de retención definido y que se aplique. Tanto «los registros se conservan durante muy poco tiempo para investigar un incidente» como «los registros se conservan indefinidamente y se convierten en un repositorio cada vez mayor de datos confidenciales» son hallazgos que conviene comunicar.
- Los registros están protegidos contra alteraciones (mediante almacenamiento de solo anexado o de escritura única, o controles de integridad equivalentes cuando sea viable).
- Los fallos de registro no exponen información confidencial de depuración al usuario final (un fallo del subsistema de registro no debería provocar que el error no registrado se muestre directamente en la respuesta).
- Los registros se pueden centralizar o correlacionar en entornos distribuidos. Los contenedores, las funciones sin servidor y los nodos con balanceo de carga necesitan una forma coherente de correlacionar una solicitud o traza en los distintos componentes. Comprueba que exista esa posibilidad, en lugar de que cada componente registre datos de forma aislada y sin un identificador compartido de solicitud o traza.

Las pruebas deben confirmar que los registros sean utilizables y coherentes, y que respalden eficazmente las actividades de detección y respuesta sin convertirse ellos mismos en una fuente de exposición de datos confidenciales.

#### Imágenes de contenedor

Cuando la aplicación se despliega como una imagen de contenedor, se aplica a lo que se incorpora en la imagen el mismo principio de «eliminar lo que no se necesita» que se usa en el fortalecimiento tradicional de servidores. La mayoría de estas comprobaciones requieren la imagen en sí (caja gris); las excepciones se indican. Para los campos relacionados de `securityContext` en tiempo de ejecución (`runAsNonRoot`, `readOnlyRootFilesystem`, capacidades, `seccompProfile` y espacios de nombres del host), consulta la [Lista de comprobación de SecurityContext y fortalecimiento de contenedores](09-File_Permissions.md#securitycontext-and-container-hardening-checklist) en [Permisos de archivos](09-File_Permissions.md).

- Revisa el `Dockerfile` y la compilación de la imagen para detectar archivos predeterminados o de ejemplo que hayan quedado, secretos usados durante la compilación (claves de API o credenciales pasadas como argumentos de compilación y conservadas en una capa intermedia, aunque se hayan eliminado de la capa final) y herramientas de compilación innecesarias (compiladores, cachés de gestores de paquetes y shells) que una compilación de varias etapas debería haber dejado en la etapa de compilación. Caja gris: ejecuta `docker history --no-trunc <image>` o una herramienta de inspección de capas (por ejemplo, `dive`) en la imagen real. Para que haya un hallazgo, el secreto o las herramientas deben ser visibles en una capa conservada; no basta con sospecharlo a partir del código fuente del `Dockerfile`.
- Prefiere imágenes base mínimas (slim/distroless/scratch-based) para la capa final de ejecución. Caja gris: compara la imagen base indicada en la etapa final del `Dockerfile` con la lista real de paquetes instalados en la imagen (`docker run --rm <image> sh -c "..."` cuando haya un shell, o inspección de capas cuando no lo haya).
- Confirma que las variables de entorno del modo de depuración o desarrollo no estén establecidas como valores predeterminados de la imagen (por ejemplo, `ENV FLASK_DEBUG=1`, `ENV NODE_ENV=development`). Caja negra: provoca un error de aplicación en el contenedor en ejecución y comprueba si aparece una página de depuración o una traza de pila (la misma prueba que en [Endpoints y modos de depuración](#endpoints-y-modos-de-depuración)); esto demuestra que el valor predeterminado surtió efecto durante la ejecución, ya que una sobreescritura en la capa de orquestación podría haber corregido un valor predeterminado inseguro de la imagen. Caja gris: confirma con `docker inspect --format '{{.Config.Env}}' <image>` que el valor predeterminado de la imagen sea seguro por sí mismo, independientemente de cualquier sobreescritura.
- Confirma que el contenedor no se ejecute como root de forma predeterminada y que la imagen no incluya `.git`, `.env` u otros archivos confidenciales copiados mediante un `COPY . .` amplio sin un `.dockerignore` correspondiente. Caja gris: usa `docker inspect --format '{{.Config.User}}' <image>` (un valor vacío o `0` significa root) e inspecciona las capas o el sistema de archivos para encontrar los archivos no deseados.

Ejemplo de hallazgo: `docker history --no-trunc` en una imagen publicada muestra una capa de compilación con `RUN echo $DB_PASSWORD > /tmp/debug.log`, que filtra la contraseña de la base de datos de producción a cualquiera que pueda descargar la imagen, aunque la capa final ya no contenga el archivo. La evidencia es el valor visible en el historial de esa capa, no simplemente que el `Dockerfile` use un argumento de compilación con un nombre similar.

#### Entornos de ejecución sin servidor

Las funciones sin servidor (AWS Lambda, Azure Functions, Google Cloud Functions/Cloud Run functions) trasladan el lugar en el que aparecen los errores de configuración, pero siguen siendo aplicables las mismas categorías de configuración incorrecta. Las consolas y CLI de cada proveedor son diferentes (AWS, Azure y GCP exponen esta configuración de maneras distintas); adapta el comando o la ruta de consola al proveedor real incluido en el alcance, en lugar de asumir que existe una opción universal.

- Código predeterminado o de ejemplo: confirma que se haya eliminado el código de estructura inicial generado por `sam init`, `serverless create` o plantillas equivalentes (controladores de ejemplo y cargas de evento de muestra), en vez de desplegarlo junto con la función real. Caja negra: invoca las rutas o los endpoints de cualquier controlador de ejemplo que encuentres y confirma que devuelvan datos de la aplicación o un error, no contenido provisional de la estructura inicial.
- Registro detallado o de depuración y respuestas de error: muchos frameworks sin servidor registran la carga completa del evento o devuelven una traza de pila completa en la respuesta HTTP de forma predeterminada en el modo local o de desarrollo. Caja negra: provoca un error de aplicación mediante el punto de entrada público de la función y comprueba si la respuesta HTTP contiene una traza de pila. Caja gris: revisa los registros de la función (por ejemplo, CloudWatch Logs) para detectar cuerpos completos de solicitudes o respuestas, según las indicaciones anteriores sobre datos confidenciales en los registros.
- Permisos demasiado amplios de IAM o del rol de ejecución: el rol de ejecución de una función debe limitarse a los recursos y acciones que necesita. En la mayoría de las evaluaciones, solo caja gris: lee la política asociada al rol de ejecución (por ejemplo, `aws iam get-role-policy`) y compara las acciones y los recursos concedidos con los que realmente invoca el código de la función. Si se confirma de forma independiente un fallo de ejecución de código o de inyección en la función, la evidencia de caja negra (lectura o escritura satisfactoria de un recurso al que el alcance previsto no debería permitir el acceso) eleva esta observación de configuración a un hallazgo demostrado.
- Variables de entorno con secretos: confirma que los secretos se obtengan de un gestor de secretos o almacén de parámetros en tiempo de ejecución, en lugar de almacenarse en texto sin formato en la configuración de la función. Caja gris: consulta las variables de entorno configuradas para la función mediante la consola o CLI del proveedor y comprueba si los valores parecen secretos sin procesar o referencias (ARN/ID de recurso) a una entrada de un gestor de secretos.
- Consolas de prueba o invocación: confirma que la función de «test invoke» nativa de la consola esté protegida por los mismos controles de acceso que la propia consola o cuenta, y que no se haya dejado accesible públicamente ningún endpoint HTTP de prueba asociado. Caja negra: intenta acceder directamente a cualquier ruta HTTP de prueba o invocación que encuentres.

Ejemplo de hallazgo: el rol de ejecución de una función tiene concedido `s3:*` en todos los buckets, en lugar de `s3:GetObject` en uno (confirmado mediante caja gris con la política de IAM del rol), y se usa un error de recorrido de rutas detectado de forma independiente en el propio código de la función (confirmado mediante caja negra) para leer o escribir en un bucket no relacionado. Esto demuestra que un error contenido se convirtió en una vulnerabilidad de lectura y escritura de S3 en toda la cuenta debido a que el alcance del rol era demasiado amplio.

#### Fortalecimiento de producción específico de cada framework

Confirma que se hayan aplicado los ajustes de fortalecimiento de producción documentados para cada framework en uso, en lugar de dejar simplemente los valores predeterminados. La tabla siguiente presenta un ajuste representativo por framework como punto de partida para la comprobación, no como una prueba completa de fortalecimiento ni como una lista de comprobación que se pueda dar por cumplida y dejar de lado. El nombre exacto, el valor predeterminado e incluso la disponibilidad de un ajuste pueden cambiar entre versiones principales de un mismo framework, y una implementación puede seguir siendo insegura aunque todos los ajustes indicados sean «correctos» si se pasó por alto otro elemento de la lista de comprobación del framework.

| Framework | Ajuste representativo que se debe comprobar |
|-----------|---------------------|
| Django | `DEBUG = False`, `ALLOWED_HOSTS` restringido a nombres de host reales |
| Flask | `FLASK_DEBUG=0` / `debug=False`, `app.config['ENV'] = 'production'` |
| Ruby on Rails | `RAILS_ENV=production`, `config.consider_all_requests_local = false` |
| Spring Boot | `management.endpoints.web.exposure.include` restringido a lo necesario; `server.error.include-stacktrace=never` |
| Express/Node.js | `NODE_ENV=production` (muchas bibliotecas de middleware cambian su comportamiento, incluido el nivel de detalle de los errores, según este valor) |
| Next.js/otros frameworks SPA | Se sirve una compilación de producción (no un servidor de desarrollo); los mapas de código fuente se excluyen o se restringe su acceso |

Para cada framework incluido en el alcance:

- Considera la entrada de la tabla como una comprobación representativa para orientar la prueba, no como prueba de que la implementación esté protegida. Sigue la lista de comprobación actual de despliegue en producción publicada oficialmente por el framework (enlazada en Referencias) para comprobar todos los elementos.
- Verifica la configuración efectiva en tiempo de ejecución, no solo el archivo de configuración del código fuente: una variable de entorno, una marca de línea de comandos o una sobreescritura en la capa de orquestación (variable de entorno del contenedor, ConfigMap/Secret de Kubernetes o ajuste del panel de control de PaaS) puede cambiar el valor que realmente se aplica durante la ejecución, aunque el archivo registrado en el repositorio parezca correcto. Siempre que sea viable, confírmalo mediante el comportamiento de la aplicación en ejecución (por ejemplo, si muestra una página de depuración ante un error o expone el endpoint), en lugar de basarte únicamente en la revisión estática de los archivos.
- No deduzcas que un ajuste correcto implica que el resto del framework esté protegido; podrían faltar otros ajustes no relacionados de la misma lista de comprobación.

## Referencias

- Apache
    - [Documentación de Apache HTTP Server](https://httpd.apache.org/docs/current/)
    - [Ajuste del rendimiento](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
- Microsoft IIS
    - [Proteger y fortalecer Internet Information Services](https://learn.microsoft.com/en-us/training/modules/secure-harden-internet-information-services/)
    - [Referencias de CIS para Microsoft IIS](https://www.cisecurity.org/benchmark/microsoft_iis/)
- General
    - [Hoja de referencia de registro](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Guía para la gestión de registros de seguridad informática, NIST
    - [PCI DSS](https://www.pcisecuritystandards.org/document_library) Requisito 10, PCI Security Standards Council
    - [Módulos de mejora de la seguridad de CERT: protección de servidores web públicos](https://www.sei.cmu.edu/library/securing-public-web-servers/)
- Contenedores y entornos sin servidor
    - [Referencia CIS para Docker](https://www.cisecurity.org/benchmark/docker)
    - [OWASP Serverless Top 10](https://owasp.org/www-project-serverless-top-10/)
    - [Prácticas recomendadas de seguridad para AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html)
- Fortalecimiento de producción de frameworks
    - [Lista de comprobación de despliegue de Django](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/)
    - [Opciones de despliegue de Flask](https://flask.palletsprojects.com/en/stable/deploying/)
    - [Guía de seguridad de Ruby on Rails](https://guides.rubyonrails.org/security.html)
    - [Spring Boot Actuator: funciones listas para producción](https://docs.spring.io/spring-boot/reference/actuator/index.html)