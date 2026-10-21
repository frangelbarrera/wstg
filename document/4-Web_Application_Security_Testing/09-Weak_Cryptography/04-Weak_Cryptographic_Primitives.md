# Primitivas criptográficas débiles

|ID          |
|------------|
|WSTG-CRYP-04|

## Resumen

El uso incorrecto de algoritmos de cifrado puede provocar la exposición de datos sensibles, la filtración de claves, la vulneración de la autenticación, sesiones inseguras y ataques de suplantación. Se sabe que algunos algoritmos de cifrado o hash son débiles y no se recomienda su uso, como MD5 y RC4.

Además de elegir algoritmos seguros de cifrado o hash, el uso correcto de los parámetros también influye en el nivel de seguridad. Por ejemplo, por lo general no debería usarse el modo ECB (Electronic Code Book).

## Objetivos de prueba

- Proporcionar una guía para identificar usos e implementaciones débiles de cifrado o hash.

## Cómo probar

### Lista de verificación de seguridad básica

- Al usar AES128 o AES256, el IV (vector de inicialización) debe ser aleatorio e impredecible. Consulta [FIPS 140-3, Security Requirements for Cryptographic Modules](https://csrc.nist.gov/pubs/fips/140-3/final), sección sobre pruebas de generación de números aleatorios. Por ejemplo, en Java, `java.util.Random` se considera un generador de números aleatorios débil. En su lugar, debería usarse `java.security.SecureRandom`.
- Para el cifrado asimétrico, se recomienda usar criptografía de curva elíptica (ECC) con una curva segura, como `Curve25519`.
    - Si no se puede usar ECC, entonces se debe usar cifrado RSA con una clave de al menos 2048 bits.
- Al usar RSA para firmas, se recomienda el padding PSS.
- No deberían usarse algoritmos débiles de hash o cifrado, como MD5, RC4, DES, Blowfish y SHA1, ni RSA o DSA de 1024 bits, ECDSA de 160 bits (curvas elípticas) o 2TDEA de 80/112 bits (triple DES de dos claves).
- Requisitos mínimos de longitud de clave:

```text
Intercambio de claves: intercambio de claves Diffie-Hellman con un mínimo de 2048 bits
Integridad de mensajes: HMAC-SHA2
Hash de mensajes: SHA2 de 256 bits
Cifrado asimétrico: RSA de 2048 bits
Algoritmo de clave simétrica: AES de 128 bits
Hash de contraseñas: PBKDF2, Scrypt, Bcrypt
ECDH, ECDSA: 256 bits
```

- En SSH, no debería usarse el modo CBC.
- Cuando se use un algoritmo de cifrado simétrico, no debería usarse el modo ECB (Electronic Code Book).
- Cuando se use PBKDF2 para generar el hash de una contraseña, se recomienda que el parámetro de iteraciones sea superior a 10000. [NIST](https://pages.nist.gov/800-63-4/sp800-63b.html) también sugiere realizar al menos 10,000 iteraciones de la función hash. Además, está prohibido usar la función hash MD5 con PBKDF2, por ejemplo, PBKDF2WithHmacMD5.

### Revisión del código fuente

- Busca las siguientes palabras clave para identificar el uso de algoritmos débiles: `MD4, MD5, RC4, RC2, DES, Blowfish, SHA-1, ECB`

- En las implementaciones Java, la siguiente API está relacionada con el cifrado. Revisa los parámetros de la implementación de cifrado. Por ejemplo:

```java
SecretKeyFactory(SecretKeyFactorySpi keyFacSpi, Provider provider, String algorithm)
SecretKeySpec(byte[] key, int offset, int len, String algorithm)
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

- Para el cifrado RSA, se recomiendan los siguientes modos de padding.

```text
RSA/ECB/OAEPWithSHA-1AndMGF1Padding (2048)
RSA/ECB/OAEPWithSHA-256AndMGF1Padding (2048)
```

- Busca `ECB`: no se permite usarlo en el padding.
- Revisa si se usa un IV (vector de inicialización) diferente.

```java
// Usar un valor IV diferente para cada cifrado
byte[] newIv = ...;
s = new GCMParameterSpec(s.getTLen(), newIv);
cipher.init(..., s);
...
```

- Busca `IvParameterSpec` y comprueba si el valor del IV se genera de manera diferente y aleatoria.

```java
 IvParameterSpec iv = new IvParameterSpec(randBytes);
 SecretKeySpec skey = new SecretKeySpec(key.getBytes(), "AES");
 Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
 cipher.init(Cipher.ENCRYPT_MODE, skey, iv);
```

- En Java, busca MessageDigest para comprobar si se usa un algoritmo hash débil (MD5 o CRC). Por ejemplo:

`MessageDigest md5 = MessageDigest.getInstance("MD5");`

- Para las firmas, no deberían usarse SHA1 ni MD5. Por ejemplo:

`Signature sig = Signature.getInstance("SHA1withRSA");`

- Busca `PBKDF2`. Se recomienda usar `PBKDF2` para generar el valor hash de una contraseña. Revisa los parámetros usados para generar el valor hash de `PBKDF2`.

Las iteraciones deberían ser superiores a **10000**, y el valor de **salt** debería generarse de forma **aleatoria**.

```java
private static byte[] pbkdf2(char[] password, byte[] salt, int iterations, int bytes)
    throws NoSuchAlgorithmException, InvalidKeySpecException
  {
       PBEKeySpec spec = new PBEKeySpec(password, salt, iterations, bytes * 8);
       SecretKeyFactory skf = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM);
       return skf.generateSecret(spec).getEncoded();
   }
```

- Información sensible codificada de forma fija:

```text
Palabras clave relacionadas con usuarios: name, root, su, sudo, admin, superuser, login, username, uid
Palabras clave relacionadas con claves: public key, AK, SK, secret key, private key, passwd, password, pwd, share key, shared key, crypto, base64
Otras palabras clave sensibles comunes: sysadmin, root, privilege, pass, key, code, master, admin, uname, session, token, Oauth, privatekey, shared secret
```

## Herramientas

- Los escáneres de vulnerabilidades, como Nessus, Nmap (scripts) u OpenVAS, pueden analizar el uso o la aceptación de cifrado débil en protocolos como SNMP, TLS, SSH, SMTP, etc.
- Usa herramientas de análisis estático de código para revisar el código fuente, como klocwork, Fortify, Coverity o CheckMark, en los siguientes casos.

```text
CWE-261: Weak Cryptography for Passwords
CWE-323: Reusing a Nonce, Key Pair in Encryption
CWE-326: Inadequate Encryption Strength
CWE-327: Use of a Broken or Risky Cryptographic Algorithm
CWE-328: Reversible One-Way Hash
CWE-329: Not Using a Random IV with CBC Mode
CWE-330: Use of Insufficiently Random Values
CWE-347: Improper Verification of Cryptographic Signature
CWE-354: Improper Validation of Integrity Check Value
CWE-547: Use of Hard-coded, Security-relevant Constants
CWE-780: Use of RSA Algorithm without OAEP
```

## Referencias

- [Estándares NIST FIPS](https://csrc.nist.gov/publications/fips)
- [Wikipedia: vector de inicialización](https://en.wikipedia.org/wiki/Initialization_vector)
- [Programación segura: generación de números aleatorios sólidos](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/miscellaneous-msc/msc02-j)
- [Optimal Asymmetric Encryption Padding](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding)
- [Hoja de referencia de almacenamiento criptográfico](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [Hoja de referencia de almacenamiento de contraseñas](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Programación segura: no usar algoritmos criptográficos inseguros o débiles](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/recommendations/miscellaneous-msc/msc61-j)
- [Aleatoriedad insegura](https://owasp.org/www-community/vulnerabilities/Insecure_Randomness)
- [Entropía insuficiente](https://owasp.org/www-community/vulnerabilities/Insufficient_Entropy)
- [Longitud insuficiente del ID de sesión](https://owasp.org/www-community/vulnerabilities/Insufficient_Session-ID_Length)
- [Uso de un algoritmo criptográfico roto o arriesgado](https://owasp.org/www-community/vulnerabilities/Using_a_broken_or_risky_cryptographic_algorithm)
- [API Javax.crypto.cipher](https://docs.oracle.com/javase/8/docs/api/javax/crypto/Cipher.html)
- ISO 18033-1:2015 - Algoritmos de cifrado
- ISO 18033-2:2015 - Cifrados asimétricos
- ISO 18033-3:2015 - Cifrados de bloque