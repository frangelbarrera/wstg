# Hoja de Trucos para Evaluación REST

## Acerca de Servicios Web RESTful

Los Servicios Web son una implementación de tecnología web usada para comunicación máquina a máquina. Como tal, se usan para comunicación inter-aplicación, Web 2.0 y Mashups y por aplicaciones de escritorio y móviles para llamar a un servidor.

Los servicios web RESTful (a menudo llamados simplemente REST) son una variante ligera de Servicios Web basada en el patrón de diseño RESTful. En práctica, los servicios web RESTful utilizan solicitudes HTTP que son similares a llamadas HTTP regulares en contraste con otras tecnologías de Servicios Web como SOAP que utilizan un protocolo complejo.

## Propiedades clave relevantes de servicios web RESTful

- Uso de métodos HTTP (`GET`, `POST`, `PUT` y `DELETE`) como el verbo primario para la operación solicitada.
- Especificaciones de parámetros no estándar:
    - Como parte de la URL.
    - En headers.
- Parámetros y respuestas estructurados usando JSON o XML en valores de parámetros, cuerpo de solicitud o cuerpo de respuesta. Estos son requeridos para comunicar información útil para máquina.
- Autenticación y gestión de sesión personalizada, a menudo utilizando tokens de seguridad personalizados: esto es necesario ya que la comunicación máquina a máquina no permite secuencias de login.
- Falta de documentación formal. Un [estándar propuesto para describir servicios web RESTful llamado WADL](https://www.w3.org/Submission/wadl/) fue enviado por Sun Microsystems pero nunca fue oficialmente adaptado.

## El desafío de probar la seguridad de servicios web RESTful

- Inspeccionar la aplicación no revela la superficie de ataque, I.e. las URLs y estructura de parámetros usados por el servicio web RESTful. Las razones son:
    - Ninguna aplicación utiliza todas las funciones y parámetros disponibles expuestos por el servicio
    - Aquellos usados son a menudo activados dinámicamente por código del lado del cliente y no como enlaces en páginas.
    - La aplicación cliente a menudo no es una aplicación web y no permite inspección del enlace activador o incluso código relevante.
- Los parámetros son no estándar haciendo difícil determinar qué es solo parte de la URL o un header constante y qué es un parámetro worth [fuzzing](https://owasp.org/www-community/Fuzzing).
- Como interfaz de máquina el número de parámetros usados puede ser muy grande, por ejemplo una estructura JSON puede incluir docenas de parámetros. [fuzzing](https://owasp.org/www-community/Fuzzing) cada uno significativamente alarga el tiempo requerido para pruebas.
- Mecanismos de autenticación personalizados requieren ingeniería inversa y hacen herramientas populares no útiles ya que no pueden rastrear una sesión de login.

## Cómo pentestar un servicio web RESTful

Determinar la superficie de ataque a través de documentación - Las pruebas de penetración RESTful podrían estar mejor si algún nivel de pruebas white box es permitido y puedes obtener información sobre el servicio.

Esta información asegurará cobertura más completa de la superficie de ataque. Tal información a buscar:

- Descripción formal del servicio - Mientras que para otros tipos de servicios web como SOAP una descripción formal, usualmente en WSDL está a menudo disponible, este no es el caso para REST. Dicho eso, ya sea WSDL 2.0 o WADL pueden describir REST y a veces se usan.
- Una guía de desarrollador para usar el servicio puede ser menos detallada pero comúnmente se encuentra, y podría incluso ser considerada *black box*.
- Fuente de aplicación o configuración - en muchos frameworks, incluyendo dotNet, la definición del servicio REST podría ser fácilmente obtenida de archivos de configuración en lugar de código.

Recopilar solicitudes completas usando un [proxy](https://www.zaproxy.org/) - mientras siempre un paso importante de pruebas de penetración, esto es más importante para aplicaciones basadas en REST ya que la UI de la aplicación puede no dar pistas sobre la superficie de ataque real.

Nota que el proxy debe ser capaz de recopilar solicitudes completas y no solo URLs ya que los servicios REST utilizan más que solo parámetros GET.

Analizar solicitudes recopiladas para determinar la superficie de ataque:

- Buscar parámetros no estándar:
    - Buscar headers HTTP anormales - aquellos serían muchas veces parámetros basados en header.
    - Determinar si un segmento URL tiene un patrón repetitivo a través de URLs. Tales patrones pueden incluir una fecha, un número o una cadena como ID e indicar que el segmento URL es un parámetro embebido en URL.
        - Por ejemplo: `https://server/srv/2013-10-21/use.php`
    - Buscar valores de parámetros estructurados - aquellos pueden ser JSON, XML o una estructura no estándar.
    - Si el último elemento de una URL no tiene extensión, puede ser un parámetro. Esto es especialmente cierto si la tecnología de aplicación normalmente usa extensiones o si un segmento anterior sí tiene extensión.
        - Por ejemplo: `https://server/svc/Grid.asmx/GetRelatedListItems`
    - Buscar segmentos URL altamente variables - un solo segmento URL que tiene muchos valores puede ser parámetro y no un directorio físico.
        - Por ejemplo si la URL `https://server/src/XXXX/page` se repite con cientos de valores para `XXXX`, chances `XXXX` es un parámetro.

Verificar parámetros no estándar: en algunos casos (pero no todos), establecer el valor de un segmento URL sospechoso de ser un parámetro a un valor esperado a ser inválido puede ayudar a determinar si es un elemento de path o un parámetro. Si un elemento de path, el servidor web devolverá un mensaje *404*, mientras que para un valor inválido a un parámetro la respuesta sería un mensaje de nivel aplicación ya que el valor es legal al nivel de servidor web.

Analizando solicitudes recopiladas para optimizar [fuzzing](https://owasp.org/www-community/Fuzzing) - después de identificar parámetros potenciales para fuzz, analizar los valores recopilados para cada uno para determinar:

- Valores válidos vs. inválidos, para que [fuzzing](https://owasp.org/www-community/Fuzzing) pueda enfocarse en valores inválidos marginales.
    - Por ejemplo enviando *0* para un valor encontrado a ser siempre un entero positivo.
- Secuencias permitiendo fuzz más allá del rango presumiblemente asignado al usuario actual.

Por último, cuando [fuzzing](https://owasp.org/www-community/Fuzzing), no olvides emular el mecanismo de autenticación usado.

## Recursos Relacionados

- [Hoja de Trucos de Seguridad REST](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) - el otro lado de esta hoja de trucos
- [Servicios RESTful, punto ciego de seguridad web](https://www.youtube.com/watch?v=pWq4qGLAZHI) - una presentación de video elaborando en la mayoría de los temas en esta hoja de trucos.