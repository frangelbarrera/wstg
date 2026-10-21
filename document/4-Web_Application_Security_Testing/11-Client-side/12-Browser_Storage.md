# Almacenamiento del navegador

|ID          |
|------------|
|WSTG-CLNT-12|

## Resumen

Los navegadores proporcionan los siguientes mecanismos de almacenamiento del lado del cliente para que los desarrolladores almacenen y recuperen datos:

- LocalStorage
- SessionStorage
- IndexedDB
- Cookies

Estos mecanismos de almacenamiento pueden verse y editarse mediante las herramientas de desarrollo del navegador, como [Google Chrome DevTools](https://developer.chrome.com/docs/devtools/storage/localstorage) o el [Inspector de almacenamiento de Firefox](https://firefox-source-docs.mozilla.org/devtools-user/storage_inspector/index.html).

> Nota: Aunque la caché también es una forma de almacenamiento, se aborda en una [sección aparte](../04-Authentication/06-Browser_Cache_Weaknesses.md), donde se tratan sus particularidades y preocupaciones.

## Objetivos de las pruebas

- Determinar si el sitio almacena datos sensibles en el almacenamiento del lado del cliente.
- Examinar el código que maneja los objetos de almacenamiento para detectar posibilidades de ataques de inyección, como el uso de entradas no validadas o bibliotecas vulnerables.

## Cómo realizar las pruebas

### LocalStorage

`window.localStorage` es una propiedad global que implementa la [API de Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) y proporciona almacenamiento **persistente** de pares clave-valor en el navegador.

Tanto las claves como los valores solo pueden ser cadenas, por lo que todo valor que no sea una cadena debe convertirse primero a una cadena antes de almacenarlo; normalmente, esto se hace mediante [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Las entradas de `localStorage` persisten incluso cuando se cierra la ventana del navegador, excepto en las ventanas abiertas en modo privado o de incógnito.

La capacidad máxima de almacenamiento de `localStorage` varía según el navegador.

#### Enumerar todas las entradas clave-valor

```javascript
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### SessionStorage

`window.sessionStorage` es una propiedad global que implementa la [API de Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) y proporciona almacenamiento **efímero** de pares clave-valor en el navegador.

Tanto las claves como los valores solo pueden ser cadenas, por lo que todo valor que no sea una cadena debe convertirse primero a una cadena antes de almacenarlo; normalmente, esto se hace mediante [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

Las entradas de `sessionStorage` son efímeras porque se eliminan al cerrar la pestaña o ventana del navegador.

La capacidad máxima de almacenamiento de `sessionStorage` varía según el navegador.

#### Enumerar todas las entradas clave-valor

```javascript
for (let i = 0; i < sessionStorage.length; i++) {
  const key = sessionStorage.key(i);
  const value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### IndexedDB

IndexedDB es una base de datos transaccional y orientada a objetos, destinada a datos estructurados. Una base de datos IndexedDB puede tener varios almacenes de objetos y cada almacén de objetos puede contener varios objetos.

A diferencia de localStorage y sessionStorage, IndexedDB puede almacenar más que cadenas. En IndexedDB se puede almacenar cualquier objeto compatible con el [algoritmo de clonado estructurado](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm).

Un ejemplo de objeto JavaScript complejo que se puede almacenar en IndexedDB, pero no en localStorage/sessionStorage, son las [CryptoKeys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey).

La recomendación del W3C sobre [Web Crypto API](https://www.w3.org/TR/WebCryptoAPI/) [recomienda](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage) que las CryptoKeys que deban persistir en el navegador se almacenen en IndexedDB. Al probar una página web, busque CryptoKeys en IndexedDB y compruebe si están configuradas como `extractable: true` cuando deberían estarlo como `extractable: false` (es decir, asegúrese de que el material de clave privada subyacente nunca quede expuesto durante las operaciones criptográficas).

#### Imprimir todo el contenido de IndexedDB

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

      // Print all entries in objectStore with name `storeName`
      objectStore.getAll().onsuccess = event => {
        const items = event.target.result || [];
        items.forEach(item => console.log(`\t\t[-] `, item));
      };
    });
  };
};

indexedDB.databases().then(dbs => dbs.forEach(db => dumpIndexedDB(db.name)));
```

### Cookies

Las cookies son un mecanismo de almacenamiento de pares clave-valor que se utiliza principalmente para la gestión de sesiones, aunque los desarrolladores web también pueden usarlas para almacenar datos de cadena arbitrarios.

Las cookies se tratan ampliamente en el escenario de [pruebas de los atributos de las cookies](../06-Session_Management/02-Cookies_Attributes.md).

#### Enumerar todas las cookies

```javascript
console.log(window.document.cookie);
```

### Objeto global Window

A veces, los desarrolladores web inicializan y mantienen un estado global que solo está disponible durante el tiempo de ejecución de la página, asignando atributos personalizados al objeto global `window`. Por ejemplo:

```javascript
window.MY_STATE = {
  counter: 0,
  flag: false,
};
```

Todos los datos asociados al objeto `window` se pierden cuando se actualiza o se cierra la página.

#### Enumerar todas las entradas del objeto Window

```javascript
(() => {
  // create an iframe and append to body to load a clean window object
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  // get the current list of properties on window
  const currentWindow = Object.getOwnPropertyNames(window);

  // filter the list against the properties that exist in the clean window
  const results = currentWindow.filter(
    prop => !iframe.contentWindow.hasOwnProperty(prop)
  );

  // remove iframe
  document.body.removeChild(iframe);

  // log key-value entries that are different
  results.forEach(key => console.log(`${key}: ${window[key]}`));
})();
```

_(Versión modificada de este [fragmento](https://stackoverflow.com/a/17246535/3099132))_

### Implicaciones de seguridad

Al revisar los mecanismos de almacenamiento del navegador, los evaluadores deben determinar si los datos sensibles están innecesariamente expuestos en el lado del cliente. Las aplicaciones modernas, especialmente las SPA, suelen almacenar tokens de autenticación o el estado de la aplicación en el almacenamiento del navegador, lo que puede introducir riesgos de seguridad.

Entre las preocupaciones habituales se incluyen:

- Tokens de autenticación (por ejemplo, JWT) almacenados en `localStorage` o `sessionStorage`, a los que se puede acceder mediante JavaScript y que pueden quedar expuestos a través de XSS.
- Tokens o identificadores de sesión que persisten después del cierre de sesión.
- Datos empresariales sensibles almacenados en IndexedDB o localStorage sin una necesidad clara.
- Material criptográfico almacenado como extraíble cuando debería estar protegido.

El almacenamiento inadecuado del lado del cliente puede aumentar el impacto de ataques del lado del cliente, como XSS basado en DOM.

### Guía general para las pruebas

Además de enumerar las entradas de almacenamiento, los evaluadores deben:

- Inspeccionar el almacenamiento del navegador mediante las herramientas de desarrollo (pestaña Application/Storage).
- Identificar tokens de autenticación, identificadores de sesión o datos empresariales sensibles.
- Intentar acceder a los valores almacenados mediante la consola de JavaScript.
- Verificar si las entradas de almacenamiento se eliminan al cerrar sesión o al expirar la sesión.
- Evaluar si los datos almacenados podrían aprovecharse en una cadena de ataques del lado del cliente.

### Cadena de ataque

Tras identificar cualquiera de los vectores de ataque anteriores, se puede formar una cadena de ataque con distintos tipos de ataques del lado del cliente, como ataques de [XSS basado en DOM](01-DOM-based_Cross_Site_Scripting.md).

## Mitigación

Las aplicaciones deben almacenar los datos sensibles de forma segura en el lado del servidor, y no en el lado del cliente, siguiendo las prácticas recomendadas.

## Referencias

- [LocalStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [SessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Web Crypto API: almacenamiento de claves](https://www.w3.org/TR/WebCryptoAPI/#concepts-key-storage)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)

Para obtener más recursos de OWASP sobre la API HTML5 Web Storage, consulte la [hoja de referencia de gestión de sesiones](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#html5-web-storage-api).