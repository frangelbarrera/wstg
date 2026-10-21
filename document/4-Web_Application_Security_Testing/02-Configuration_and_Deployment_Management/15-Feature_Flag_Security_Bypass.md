# Evasión de seguridad mediante indicadores de funcionalidad

| ID         |
|------------|
|WSTG-CONF-15|

## Resumen

Los indicadores de funcionalidad se utilizan comúnmente para controlar las funciones de una aplicación y habilitar la entrega progresiva. Permiten activar, desactivar o desplegar gradualmente funciones sin necesidad de implementar toda la aplicación. Las aplicaciones modernas utilizan cada vez más indicadores de funcionalidad y mecanismos de apagado de emergencia para controlar funciones relevantes para la seguridad, como la autenticación, la autorización, la detección de fraude, la limitación de tasa y los controles de riesgo, ya sea mediante servicios de indicadores de funcionalidad (LaunchDarkly, Split, Flagsmith, ConfigCat) o soluciones personalizadas.

Cuando los controles de seguridad dependen de indicadores de funcionalidad, los estados incoherentes de estos pueden introducir vulnerabilidades. Un control puede estar habilitado en un servicio y deshabilitado en otro, o una reversión puede restaurar el código de la aplicación sin restaurar la configuración de seguridad correspondiente. También es posible manipular los indicadores del lado del cliente para revelar funciones que deberían permanecer protegidas.

Un patrón de falla común se produce cuando un mecanismo de apagado de emergencia desactiva la lógica de cumplimiento, pero deja una afirmación de confianza. Por ejemplo, un control de prevención del fraude puede desactivarse por problemas de rendimiento, mientras un servicio de backend sigue confiando en un parámetro `fraud_checked=true` que el control (ahora desactivado) establecía anteriormente. Un atacante puede reproducir una solicitud capturada previamente que contiene ese parámetro para que el backend trate una solicitud no verificada como si hubiera superado la comprobación antifraude.

Esta prueba evalúa si los controles de seguridad se aplican de forma coherente en todos los estados de los indicadores de funcionalidad, incluidos la habilitación, la deshabilitación, la reversión, el despliegue parcial, la implementación canary y las fallas del servicio de indicadores de funcionalidad.

## Objetivos de la prueba

- Identificar los indicadores de funcionalidad y los mecanismos de apagado de emergencia que controlan funciones relevantes para la seguridad.
- Determinar si un cliente no autorizado puede manipular los estados de los indicadores de funcionalidad y verificar que los controles de seguridad del backend sigan aplicándose independientemente del estado de los indicadores del lado del cliente.
- Verificar que los controles de seguridad se apliquen de forma coherente durante las transiciones de estado de los indicadores, las reversiones, los despliegues escalonados y en todos los servicios o instancias; esto incluye comprobar si pueden reutilizarse afirmaciones de seguridad obsoletas o estados de sesión después de una transición.
- Evaluar el comportamiento seguro ante fallas cuando el servicio de indicadores de funcionalidad no está disponible.
- Identificar información confidencial expuesta mediante las configuraciones de indicadores de funcionalidad e indicadores obsoletos que puedan exponer rutas de código obsoletas o sin parches.

## Cómo realizar las pruebas

Las pruebas de indicadores de funcionalidad pueden realizarse como **pruebas de caja negra**, infiriendo el comportamiento controlado por los indicadores mediante la comparación de diferencias en las respuestas entre las etapas del despliegue, la reproducción de solicitudes capturadas y el análisis temporal; o como **pruebas de caja gris**, cuando el evaluador tiene visibilidad del sistema de gestión de indicadores y puede cambiar directamente los estados para evaluar la coherencia de la aplicación de los controles. Cuando sea posible, combine ambos enfoques: las pruebas de caja negra confirman qué puede explotar realmente un atacante externo, mientras que las pruebas de caja gris confirman si el backend aplica los controles independientemente del estado del indicador.

### Identificar indicadores de funcionalidad relevantes para la seguridad

Identifique los indicadores de funcionalidad, los mecanismos de apagado de emergencia y los valores de configuración que controlan comportamientos relevantes para la seguridad, como la autenticación, la autenticación multifactor, la autorización, la detección de fraude, la limitación de tasa, la autenticación basada en riesgos, la recuperación de cuentas, las funciones administrativas y la supervisión de seguridad.

#### Analizar recursos del lado del cliente

Inspeccione los paquetes de JavaScript y otros recursos del lado del cliente en busca de patrones comunes de indicadores de funcionalidad:

```text
featureFlags
isFeatureEnabled
featureEnabled
flags
killSwitch
securityFeature
```

Herramientas como `source-map-explorer` o webpack Bundle Analyzer pueden ayudar a localizar código relacionado con indicadores dentro de paquetes minificados.

#### Supervisar el tráfico de red y las respuestas de la API

Utilice un proxy de interceptación (Burp Suite, ZAP) para capturar las solicitudes realizadas a servicios de indicadores de funcionalidad o puntos de conexión de configuración internos (por ejemplo, llamadas a `app.launchdarkly.com`, `/api/config` o `/api/flags`). Revise las cargas útiles de las respuestas para identificar nombres de indicadores, valores predeterminados y reglas de segmentación, y compruebe si algún punto de conexión devuelve el conjunto completo de indicadores evaluados para una sesión, incluidos los que no guardan relación con la página actual. Las cargas útiles de indicadores demasiado amplias son una fuente común de divulgación de información (véase más abajo).

### Probar la manipulación de indicadores del lado del cliente

Cuando los indicadores se evalúen o almacenen en caché del lado del cliente (por ejemplo, en un objeto JavaScript, una cookie o un valor de almacenamiento local), utilice un proxy para modificar el valor del indicador y reproducir la solicitud:

```http
PUT /api/user/settings HTTP/1.1
Host: example.com
Content-Type: application/json

{"flags": {"betaAdminPanel": true}}
```

- **Vulnerable:** La aplicación expone funciones ocultas o confía en el valor modificado del indicador sin una comprobación equivalente en el servidor.
- **Seguro:** El valor del lado del cliente no afecta a lo que el servidor autoriza; el servidor vuelve a evaluar el indicador de forma independiente.

### Verificar que la autorización del backend sea independiente

Para cada indicador relevante para la seguridad identificado, confirme que ocultar una función en la interfaz de usuario conlleve una restricción equivalente allí donde dicha función esté implementada (un punto de conexión de API, una llamada a un servicio de backend, un controlador de mensajes, etc.). Intente invocar directamente la función subyacente mientras el indicador está deshabilitado para el usuario actual; por ejemplo:

```http
GET /api/admin/reports HTTP/1.1
Host: example.com
Authorization: Bearer {low-privilege-token}
```

**Resultado esperado:** El servidor debe aplicar la autorización independientemente del estado del indicador del lado del cliente: se debe denegar el acceso a un usuario no autorizado (por ejemplo, con `401 Unauthorized` o `403 Forbidden`) incluso si el indicador se manipula del lado del cliente. Si, por diseño, la función está deshabilitada globalmente, debe permanecer inaccesible para todos los usuarios, en lugar de limitarse a ocultar el elemento de la interfaz mientras la función subyacente sigue siendo accesible.

### Probar el comportamiento durante las transiciones de los indicadores y las reversiones

- **Evasión del mecanismo de apagado de emergencia:** Deshabilite un indicador relevante para la seguridad (comprobación antifraude, límite de tasa, autenticación reforzada) y reproduzca una solicitud capturada *antes* de deshabilitar el indicador, que contenga un parámetro, token o atributo de sesión establecido previamente por ese control (por ejemplo, `fraud_checked=true`). Confirme que el backend no siga confiando en esa afirmación obsoleta.
- **Reversión:** Simule u observe una reversión de la aplicación y confirme que también se restaure toda configuración de seguridad vinculada a la versión revertida, en lugar de dejarla en su estado anterior (potencialmente deshabilitado).
- **Despliegue parcial / canary:** Cuando varias instancias o servicios puedan estar ejecutándose simultáneamente con distintos estados de los indicadores, envíe solicitudes repetidas y compare las respuestas de las distintas instancias para detectar incoherencias en la aplicación de los controles.
- **Segundo escenario: mecanismo de apagado de emergencia para MFA:** Deshabilite la verificación reforzada/MFA para un subconjunto de usuarios mediante un mecanismo de apagado de emergencia. Confirme que un servicio posterior no siga confiando en un atributo de sesión existente (por ejemplo, `mfa_verified=true`) que solo era válido porque antes se aplicaba MFA. Si se confía en el atributo después de deshabilitar el control, un atacante con una sesión obsoleta puede eludir por completo la verificación reforzada.

### Evaluar el comportamiento seguro ante fallas cuando el servicio de indicadores no está disponible

Bloquee o limite el acceso al servicio de indicadores de funcionalidad (por ejemplo, mediante reglas de proxy o el bloqueo de DNS en un entorno de pruebas) y observe el comportamiento de la aplicación cuando no pueda recuperar el estado de los indicadores.

- **Vulnerable:** La aplicación falla en modo abierto y establece los indicadores relevantes para la seguridad en un estado habilitado o permisivo de forma predeterminada.
- **Resultado esperado:** Los controles relevantes para la seguridad siguen aplicándose o recurren a una configuración segura documentada cuando el servicio de indicadores de funcionalidad no está disponible.

### Analizar las configuraciones de indicadores de funcionalidad en busca de divulgación de información

Revise cualquier configuración de indicadores que el cliente pueda recuperar (mediante una respuesta de API, un paquete JS o un panel de administración expuesto) para identificar:

- Nombres de funciones aún no publicadas o de servicios internos.
- Reglas de segmentación que revelen segmentos internos de usuarios, cuentas de empleados o grupos de prueba.
- Valores de configuración (URL, descripciones de funciones) que expongan detalles de implementación.

### Identificar indicadores de funcionalidad obsoletos

Busque en el código base (cuando esté disponible) o solicite una lista de indicadores cuyo estado ya no se cambia activamente. Los indicadores obsoletos que controlan rutas de código que ya no se utilizan aún pueden ser accesibles y hacer referencia a lógica que no ha recibido parches de seguridad posteriores. Confirme si esas rutas siguen siendo accesibles en la aplicación en ejecución y, de ser así, si todavía se protegen correctamente.

## Remediación

- Aplique todas las comprobaciones de autorización relevantes para la seguridad en el backend, independientemente de cualquier estado de los indicadores proporcionado o visible para el cliente.
- Asegúrese de que los mecanismos de apagado de emergencia eliminen la confianza en cualquier parámetro, token o atributo de sesión que solo era válido porque el control asociado estaba activo.
- Vincule la configuración de seguridad al mismo control de despliegue y versiones que el código de la aplicación, de modo que las reversiones restauren ambos de forma conjunta.
- Asegúrese de que los servicios de indicadores de funcionalidad utilicen una alternativa segura y documentada para los indicadores relevantes para la seguridad cuando no estén disponibles.
- Evite exponer al cliente la configuración completa de los indicadores; devuelva solo los que sean pertinentes para el usuario y el contexto actuales.
- Audite periódicamente y elimine los indicadores obsoletos, junto con las rutas de código que controlan, una vez completado el despliegue.
- Aplique la evaluación de indicadores de forma coherente en todos los servicios e instancias que gestionan una solicitud.

## Herramientas

- [Burp Suite](https://portswigger.net/burp)
- [ZAP](https://www.zaproxy.org/)
- [Herramientas para desarrolladores del navegador](../../6-Appendix/F-Leveraging_Dev_Tools.md)
- [source-map-explorer](https://github.com/danvk/source-map-explorer) / webpack Bundle Analyzer (para localizar la lógica de los indicadores en paquetes JS minificados)

## Referencias

- [Conmutadores de funcionalidad (también conocidos como indicadores de funcionalidad) - Martin Fowler](https://martinfowler.com/articles/feature-toggles.html)
- [Seguridad de cuentas de LaunchDarkly](https://launchdarkly.com/docs/home/account/secure)
- [OWASP ASVS - Configuración](https://owasp.org/projects/asvs)
- [CWE-284: Control de acceso inadecuado](https://cwe.mitre.org/data/definitions/284.html)
- [CWE-863: Autorización incorrecta](https://cwe.mitre.org/data/definitions/863.html)