# Pruebas de solicitudes HTTP entrantes

|ID          |
|------------|
|WSTG-INPV-16|

## Resumen

Esta sección describe cómo monitorear todas las solicitudes HTTP entrantes/salientes tanto en el lado del cliente como en el lado del servidor. El propósito de esta prueba es verificar si hay envío de solicitudes HTTP innecesarias o sospechosas en segundo plano.

La mayoría de las herramientas de pruebas de seguridad web (es decir, AppScan, BurpSuite, ZAP) actúan como proxy HTTP. Esto requerirá cambios en el proxy en la aplicación del lado del cliente o navegador. Las técnicas de prueba enumeradas a continuación se centran principalmente en cómo podemos monitorear las solicitudes HTTP sin cambios en el lado del cliente, lo que estará más cerca del escenario de uso en producción.

## Objetivos de la prueba

- Monitorear todas las solicitudes HTTP entrantes y salientes al servidor web para inspeccionar cualquier solicitud sospechosa.
- Monitorear el tráfico HTTP sin cambios en el proxy del navegador del usuario final o la aplicación del lado del cliente.

## Cómo probar

### Proxy inverso

Hay situaciones en las que nos gustaría monitorear todas las solicitudes HTTP entrantes en el servidor web, pero no podemos cambiar la configuración en el navegador o la aplicación del lado del cliente. En este escenario, podemos configurar un proxy inverso en el servidor web para monitorear todas las solicitudes entrantes/salientes en el servidor web.

Para la plataforma Windows, se recomienda Fiddler Classic. Proporciona no solo monitoreo, sino que también puede editar/responder las solicitudes HTTP. Consulte [esta referencia para cómo configurar Fiddler como proxy inverso](https://docs.telerik.com/fiddler/configure-fiddler/tasks/usefiddlerasreverseproxy)

Para la plataforma Linux, se puede usar Charles Web Debugging Proxy.

Los pasos de prueba:

1. Instalar Fiddler o Charles en el servidor web
2. Configurar Fiddler o Charles como proxy inverso
3. Capturar el tráfico HTTP
4. Inspeccionar el tráfico HTTP
5. Modificar las solicitudes HTTP y reproducir las solicitudes modificadas para pruebas

### Reenvío de puertos

El reenvío de puertos es otra forma de permitirnos interceptar solicitudes HTTP sin cambios en el lado del cliente. También puede usar Charles como proxy SOCKS para actuar como reenvío de puertos o usar herramientas de reenvío de puertos. Permitirá reenviar todo el tráfico capturado del lado del cliente al puerto del servidor web.

El flujo de prueba será:

1. Instalar Charles o el reenvío de puertos en otra máquina o servidor web
2. Configurar Charles como proxy SOCKS como reenvío de puertos.

### Captura de tráfico de red a nivel TCP

Esta técnica monitorea todo el tráfico de red a nivel TCP. Se pueden usar herramientas como TCPDump o WireShark. Sin embargo, estas herramientas no nos permiten editar el tráfico capturado y enviar solicitudes HTTP modificadas para pruebas. Para reproducir el tráfico capturado (paquetes PCAP), se puede usar Ostinato.

Los pasos de prueba serán:

1. Activar TCPDump o WireShark en el servidor web para capturar tráfico de red
2. Monitorear los archivos capturados (PCAP)
3. Editar archivos PCAP con la herramienta Ostinato según sea necesario
4. Reproducir las solicitudes HTTP

Se recomiendan Fiddler o Charles ya que estas herramientas pueden capturar tráfico HTTP y también editar fácilmente/reproducir las solicitudes HTTP modificadas. Además, si el tráfico web es HTTPS, Wireshark necesitará importar la clave privada del servidor web para inspeccionar el cuerpo del mensaje HTTPS. De lo contrario, el cuerpo del mensaje HTTPS del tráfico capturado estará encriptado.

## Herramientas

- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPProxy](https://grinder.sourceforge.net/g3/tcpproxy.html)
- [Charles Web Debugging Proxy](https://www.charlesproxy.com/)
- [WireShark](https://www.wireshark.org/)
- [PowerEdit-Pcap](https://sourceforge.net/projects/powereditpcap/)
- [pcapteller](https://github.com/BlackArch/pcapteller)
- [replayproxy](https://github.com/sparrowt/replayproxy)
- [Ostinato](https://ostinato.org/)

## Referencias

- [Charles Web Debugging Proxy](https://www.charlesproxy.com/)
- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPDUMP](https://www.tcpdump.org/)
- [Ostinato](https://ostinato.org/)