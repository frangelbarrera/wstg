# Descripción general de las pruebas de WebAssembly

## Introducción a WebAssembly

WebAssembly (abreviado *Wasm*) es un formato de instrucciones binarias para una máquina virtual basada en pila. Wasm está diseñado como un destino de compilación portable para lenguajes de programación, lo que permite su implementación en la web para aplicaciones de cliente y servidor. Permite ejecutar en la web, a velocidades casi nativas, código escrito en varios lenguajes, como C, C++, Rust y Go.

La adopción de WebAssembly ha estado impulsada por la necesidad de obtener un alto rendimiento en aplicaciones web (por ejemplo, edición de video, motores de juegos y bibliotecas criptográficas) y por el deseo de portar aplicaciones de escritorio heredadas a la web sin reescribirlas en JavaScript.

Al igual que ocurre con la introducción de cualquier nueva arquitectura, WebAssembly conlleva fallas y vulnerabilidades propias. Aunque la máquina virtual de Wasm está aislada del entorno anfitrión, el código que se ejecuta *dentro* del módulo Wasm conserva las vulnerabilidades de seguridad de memoria inherentes al lenguaje de origen. Además, el límite entre el anfitrión y el módulo Wasm puede proporcionar una vía sin restricciones para comprometer la aplicación si no se protege adecuadamente.

Este capítulo busca orientar al investigador de seguridad sobre los conceptos necesarios para probar aplicaciones WebAssembly y examinar sus tecnologías principales, entornos de ejecución y paradigmas de vulnerabilidad.

## Historia

Gracias a sus ventajas de rendimiento, WebAssembly ha evolucionado rápidamente, pasando de ser una función específica de los navegadores a una tecnología versátil que se ejecuta en servidores, redes de borde y dispositivos integrados.

WebAssembly se anunció por primera vez en 2015 como un esfuerzo conjunto de ingenieros de Mozilla, Google, Microsoft y Apple. Para 2017, todos los principales motores de navegador habían publicado e implementado el Producto Mínimo Viable (MVP), lo que estableció a Wasm como el cuarto lenguaje que se ejecuta de forma nativa en los navegadores (junto con HTML, CSS y JavaScript).

En diciembre de 2019, el Consorcio World Wide Web (W3C) publicó la Especificación del núcleo de WebAssembly como estándar web oficial. Aproximadamente en el mismo período, se introdujo la Interfaz del sistema de WebAssembly (WASI). WASI estandarizó la forma en que los módulos WebAssembly interactúan con los sistemas operativos, impulsando Wasm más allá del navegador hacia servicios de backend, microservicios y plataformas de computación sin servidor en el borde (como Cloudflare Workers y Fastly Compute).

## ¿Qué entorno de WebAssembly?

Antes de hacer suposiciones sobre la superficie de ataque, conviene conocer el entorno en el que opera el módulo WebAssembly. El investigador de seguridad puede encontrarse con los siguientes entornos:

- **Wasm del lado del cliente (navegador):** El archivo `.wasm` se descarga y ejecuta en el navegador del usuario. Depende de un «código de enlace» de JavaScript para interactuar con el DOM y las API web.
- **Del lado del servidor:** El módulo Wasm se ejecuta en el backend, donde procesa solicitudes de API o realiza cálculos intensivos. Las vulnerabilidades en este entorno pueden provocar directamente el compromiso del servidor.
- **Computación sin servidor / en el borde:** Las instancias de Wasm se inician en nodos de borde para interceptar y procesar solicitudes HTTP. En estos entornos se suelen reutilizar instancias de memoria para reducir la latencia, lo que introduce riesgos de filtración de estado entre inquilinos.
- **WASI independiente:** El módulo se ejecuta de forma nativa en el sistema operativo mediante entornos de ejecución como Wasmtime o Wasmer, bajo un modelo de seguridad basado en capacidades.

## Componentes principales de WebAssembly

La arquitectura de WebAssembly se basa en varios conceptos fundamentales. Comprenderlos es esencial para analizar cómo se manifiestan las vulnerabilidades.

### Módulos

Un módulo WebAssembly es un archivo binario `.wasm` compilado. No mantiene estado y contiene el código de bytes compilado, las firmas de tipos y los segmentos de datos. En formato de texto, se representa como WebAssembly Text Format (WAT).

### Memoria lineal

A diferencia de las aplicaciones nativas, que utilizan montículos y pilas administrados por el sistema operativo, una instancia de WebAssembly almacena sus datos de ejecución en una única matriz contigua de bytes denominada **Memoria lineal**. Esta memoria se expone al anfitrión como un `ArrayBuffer`. La pila, el montículo, las cadenas literales y las variables globales del módulo Wasm comparten este espacio de memoria plano y no administrado.

### Tablas

Una tabla es una matriz de valores opacos, normalmente referencias a funciones. Como WebAssembly aplica la integridad del flujo de control (CFI) y no permite la ejecución directa de código desde la memoria lineal, los punteros a funciones en el código fuente (por ejemplo, en C, C++ o Rust) y los destinos de llamadas dinámicas se compilan en índices enteros que apuntan a posiciones específicas de una tabla. Posteriormente, las llamadas indirectas (`call_indirect`) se ejecutan mediante estos índices.

### Importaciones y exportaciones

WebAssembly no puede interactuar de forma autónoma con el mundo exterior.

- **Importaciones:** Funciones o variables proporcionadas por el anfitrión (por ejemplo, una función de JS como `console.log`) que puede invocar el módulo Wasm.
- **Exportaciones:** Funciones o instancias de memoria dentro del módulo Wasm que se ponen a disposición del anfitrión.

## El paradigma de seguridad

WebAssembly está aislado de forma inherente; un módulo no puede salir de su máquina virtual para leer archivos arbitrarios o sockets de red, a menos que el anfitrión importe explícitamente esas capacidades.

Sin embargo, **no se garantiza la seguridad de la memoria interna.** Cuando se compila código heredado de C/C++ a WebAssembly, cambian algunas suposiciones sobre las protecciones nativas:

- **No hay ASLR dentro del módulo:** Los desplazamientos dentro de la memoria lineal son deterministas, aunque el proceso anfitrión utilice ASLR.
- **Los protectores de pila dependen de la compilación:** Es posible que no haya canarios, salvo que se habiliten explícitamente en la cadena de herramientas.
- **La memoria lineal no es ejecutable, pero no está protegida contra modificaciones:** La corrupción de memoria puede alterar datos que influyen en el flujo de control (por ejemplo, los índices de función utilizados en `call_indirect`) y en la lógica de la aplicación.

Por consiguiente, las vulnerabilidades clásicas de corrupción de memoria (desbordamientos de búfer, uso después de liberar y cadenas de formato) persisten tras la compilación. Un atacante puede sobrescribir datos adyacentes en la memoria lineal para manipular la lógica de la aplicación, alterar los índices de la tabla de llamadas indirectas (`ret2win`) o corromper búferes de salida que consume el anfitrión. Esto puede convertir una vulnerabilidad de Wasm en XSS basado en DOM o en ejecución remota de código (RCE) del lado del servidor.

## Referencias

- [Especificación del núcleo de WebAssembly del W3C](https://webassembly.github.io/spec/core/)
- [Interfaz del sistema de WebAssembly (WASI)](https://wasi.dev/)
- [Documentación web de MDN: conceptos de WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/Guides/Concepts)