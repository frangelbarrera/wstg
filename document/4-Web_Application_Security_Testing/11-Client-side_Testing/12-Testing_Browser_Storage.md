# Pruebas de Almacenamiento del Navegador

|ID          |
|------------|
|WSTG-CLNT-12|

## Resumen

Los navegadores proporcionan los siguientes mecanismos de almacenamiento del lado del cliente para que los desarrolladores almacenen y recuperen datos:

- LocalStorage
- SessionStorage
- IndexedDB
- Web SQL (Obsoleto)
- Cookies

Estos mecanismos de almacenamiento pueden ser vistos y editados usando las herramientas de desarrollador del navegador, tales como [Google Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/storage/localstorage) o el [Inspector de Almacenamiento de Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector).

> Nota: Mientras que la caché también es una forma de almacenamiento, se cubre en una [sección separada](../04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md) que cubre sus propias peculiaridades y preocupaciones.

## Objetivos de Prueba

- Determinar si el sitio está almacenando datos sensibles en el almacenamiento del lado del cliente.
- El manejo del código de los objetos de almacenamiento debería ser examinado en busca de posibilidades de ataques de inyección, tales como el uso de entrada no validada o bibliotecas vulnerables.

## Cómo Probar

### LocalStorage

`window.localStorage` es una propiedad global que implementa la [API de Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) y proporciona almacenamiento **persistente** de pares clave-valor en el navegador.

Tanto las claves como los valores solo pueden ser cadenas, por lo que cualquier valor que no sea cadena debe convertirse primero a cadena antes de almacenarlo, usualmente hecho vía [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Las entradas a `localStorage` persisten incluso cuando se cierra la ventana del navegador, con la excepción de ventanas en modo Privado/Incógnito.

La capacidad máxima de almacenamiento de `localStorage` varía entre navegadores.

#### Listar Todas las Entradas Clave-Valor

```javascript
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### SessionStorage

`window.sessionStorage` es una propiedad global que implementa la [API de Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) y proporciona almacenamiento **efímero** de pares clave-valor en el navegador.

Tanto las claves como los valores solo pueden ser cadenas, por lo que cualquier valor que no sea cadena debe convertirse primero a cadena antes de almacenarlo, usualmente hecho vía [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Las entradas a `sessionStorage` son efímeras porque se limpian cuando se cierra la pestaña/ventana del navegador.

La capacidad máxima de almacenamiento de `sessionStorage` varía entre navegadores.

#### Listar Todas las Entradas Clave-Valor

```javascript
for (let i = 0; i < sessionStorage.length; i++) {
  const key = sessionStorage.key(i);
  const value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### IndexedDB

IndexedDB es una base de datos transaccional orientada a objetos destinada a datos estructurados. Una base de datos IndexedDB puede tener múltiples object stores y cada object store puede tener múltiples objetos.

En contraste con localStorage y sessionStorage, IndexedDB puede almacenar más que solo cadenas. Cualquier objeto soportado por el [algoritmo de clonado estructurado](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) puede ser almacenado en IndexedDB.

Un ejemplo de un objeto JavaScript complejo que puede ser almacenado en IndexedDB, pero no en localStorage/sessionStorage son las [CryptoKeys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey).

La recomendación del W3C sobre [Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/) [recomienda](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage) que las CryptoKeys que necesitan ser persistidas en el navegador se almacenen en IndexedDB. Al probar una página web, buscar cualquier CryptoKey en IndexedDB y verificar si están establecidas como `extractable: true` cuando deberían haber sido establecidas como `extractable: false` (i.e. asegurar que el material de clave subyacente nunca se exponga durante operaciones criptográficas).

#### Imprimir Todo el Contenido de IndexedDB

```javascript
const dumpIndexedDB = dbName => {
  const DB_VERSION = 1;
  const req = indexedDB.open(dbName, DB_VERSION);
  req.onsuccess = function() {
    const db = req.result;
    const objectStoreNames = db.objectStoreNames || [];

    console.log(`[*] Database: ${dbName}`);

    Array.from(objectStoreNames).forEach(storeName => {
      const txn = db.transaction(storeName, 'readonly');
      const objectStore = txn.objectStore(storeName);

      console.log(`\t[+] ObjectStore: ${storeName}`);

      // Imprimir todas las entradas en objectStore con nombre `storeName`
      objectStore.getAll().onsuccess = event => {
        const items = event.target.result || [];
        items.forEach(item => console.log(`\t\t[-] `, item));
      };
    });
  };
};

indexedDB.databases().then(dbs => dbs.forEach(db => dumpIndexedDB(db.name)));
```

### Web SQL

Web SQL está obsoleto desde el 18 de noviembre de 2010 y se recomienda que los desarrolladores web no lo usen.

### Cookies

Las cookies son un mecanismo de almacenamiento de pares clave-valor que se usa principalmente para gestión de sesiones pero los desarrolladores web todavía pueden usarlas para almacenar datos de cadena arbitrarios.

Las cookies se cubren extensamente en el escenario de [pruebas de atributos de Cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

#### Listar Todas las Cookies

```javascript
console.log(window.document.cookie);
```

### Objeto Window Global

A veces los desarrolladores web inicializan y mantienen estado global que está disponible solo durante la vida runtime de la página asignando atributos personalizados al objeto `window` global. Por ejemplo:

```javascript
window.MY_STATE = {
  counter: 0,
  flag: false,
};
```

Cualquier dato adjuntado al objeto `window` se perderá cuando la página se refresque o cierre.

#### Listar Todas las Entradas en el Objeto Window

```javascript
(() => {
  // crear un iframe y añadirlo al body para cargar un objeto window limpio
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  // obtener la lista actual de propiedades en window
  const currentWindow = Object.getOwnPropertyNames(window);

  // filtrar la lista contra las propiedades que existen en el window limpio
  const results = currentWindow.filter(
    prop => !iframe.contentWindow.hasOwnProperty(prop)
  );

  // remover iframe
  document.body.removeChild(iframe);

  // log de entradas clave-valor que son diferentes
  results.forEach(key => console.log(`${key}: ${window[key]}`));
})();
```

_(Versión modificada de este [snippet](https://stackoverflow.com/a/17246535/3099132))_

### Implicaciones de Seguridad

Al revisar mecanismos de almacenamiento del navegador, los testers deberían evaluar si los datos sensibles están innecesariamente expuestos en el lado del cliente. Las aplicaciones modernas, especialmente las SPAs, frecuentemente almacenan tokens de autenticación o estado de aplicación en el almacenamiento del navegador, lo cual puede introducir riesgos de seguridad.

Preocupaciones comunes incluyen:

- Tokens de autenticación (por ejemplo, JWTs) almacenados en `localStorage` o `sessionStorage`, los cuales son accesibles vía JavaScript y pueden ser expuestos a través de XSS.
- Tokens o identificadores de sesión que persisten después del logout.
- Datos de negocio sensibles almacenados en IndexedDB o localStorage sin un requisito claro.
- Material criptográfico almacenado como extraíble cuando debería estar protegido.

El almacenamiento inadecuado del lado del cliente puede aumentar el impacto de ataques del lado del cliente tales como XSS basado en DOM.

### Guía General de Pruebas

Además de enumerar entradas de almacenamiento, los testers deberían:

- Inspeccionar el almacenamiento del navegador usando herramientas de desarrollador (pestaña Application/Storage).
- Identificar tokens de autenticación, identificadores de sesión o datos de negocio sensibles.
- Intentar acceder a valores almacenados vía la consola JavaScript.
- Verificar si las entradas de almacenamiento se limpian después del logout o expiración de sesión.
- Evaluar si los datos almacenados podrían ser aprovechados en una cadena de ataques del lado del cliente.

### Cadena de Ataque

Siguiendo la identificación de cualquiera de los vectores de ataque anteriores, una cadena de ataque puede formarse con diferentes tipos de ataques del lado del cliente, tales como ataques de [XSS basado en DOM](01-Testing_for_DOM-based_Cross_Site_Scripting.md).

## Remediación

Las aplicaciones deberían almacenar datos sensibles en el lado del servidor, y no en el lado del cliente, de manera segura siguiendo las mejores prácticas.

## Referencias

- [LocalStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [SessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Web Crypto API: Key Storage](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)
- [Web SQL](https://www.w3.org/TR/webdatabase/)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

Para más recursos de OWASP sobre la API de HTML5 Web Storage, ver la [Hoja de Referencia de Gestión de Sesión](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#html5-web-storage-api).
