# Pruebas de Primitivas Criptográficas Débiles

|ID          |
|------------|
|WSTG-CRYP-04|

## Resumen

Los usos incorrectos de algoritmos de cifrado pueden resultar en exposición de datos sensibles, fuga de claves, autenticación rota, sesión insegura, y ataques de suplantación. Hay algunos algoritmos de cifrado o hash conocidos por ser débiles y no se sugieren para uso tales como MD5 y RC4.

Además de las elecciones correctas de algoritmos de cifrado o hash seguros, los usos correctos de parámetros también importan para el nivel de seguridad. Por ejemplo, el modo ECB (Electronic Code Book) generalmente no debería usarse.

## Objetivos de Prueba

- Proporcionar una guía para la identificación de usos e implementaciones débiles de cifrado o hashing.

## Cómo Probar

### Lista de Verificación de Seguridad Básica

- Al usar AES128 o AES256, el IV (Vector de Inicialización) debe ser aleatorio e impredecible. Referirse a [FIPS 140-2, Security Requirements for Cryptographic Modules](https://csrc.nist.gov/publications/detail/fips/140/2/final), sección 4.9.1. pruebas de generador de números aleatorios. Por ejemplo, en Java, `java.util.Random` se considera un generador de números aleatorios débil. `java.security.SecureRandom` debería usarse en lugar de `java.util.Random`.
- Para cifrado asimétrico, usar Criptografía de Curva Elíptica (ECC) con una curva segura como `Curve25519` preferida.
    - Si ECC no puede usarse entonces usar cifrado RSA con una clave mínima de 2048 bits.
- Al usar RSA en firma, se recomienda el padding PSS.
- No deberían usarse algoritmos de hash/cifrado débiles tales como MD5, RC4, DES, Blowfish, SHA1. RSA o DSA de 1024-bit, ECDSA de 160-bit (curvas elípticas), 2TDEA de 80/112-bit (triple DES de dos claves)
- Requisitos mínimos de longitud de clave:

```text
Key exchange: Diffie–Hellman key exchange con mínimo 2048 bits
Message Integrity: HMAC-SHA2
Message Hash: SHA2 256 bits
Asymmetric encryption: RSA 2048 bits
Symmetric-key algorithm: AES 128 bits
Password Hashing: PBKDF2, Scrypt, Bcrypt
ECDH, ECDSA: 256 bits
```

- Usos de SSH, el modo CBC no debería usarse.
- Cuando se usa algoritmo de cifrado simétrico, el modo ECB (Electronic Code Book) no debería usarse.
- Cuando se usa PBKDF2 para hash de contraseña, el parámetro de iteración se recomienda que sea sobre 10000. [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5) también sugiere al menos 10,000 iteraciones de la función hash. Además, la función hash MD5 está prohibida de usarse con PBKDF2 tal como PBKDF2WithHmacMD5.

### Revisión de Código Fuente

- Buscar las siguientes palabras clave para identificar uso de algoritmos débiles: `MD4, MD5, RC4, RC2, DES, Blowfish, SHA-1, ECB`

- Para implementaciones Java, la siguiente API está relacionada con cifrado. Revisar los parámetros de la implementación de cifrado. Por ejemplo,

```java
SecretKeyFactory(SecretKeyFactorySpi keyFacSpi, Provider provider, String algorithm)
SecretKeySpec(byte[] key, int offset, int len, String algorithm)
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

- Para cifrado RSA, los siguientes modos de padding son sugeridos.

```text
RSA/ECB/OAEPWithSHA-1AndMGF1Padding (2048)
RSA/ECB/OAEPWithSHA-256AndMGF1Padding (2048)
```

- Buscar `ECB`, no se permite que sea usado en padding.
- Revisar si se usa diferente IV (Vector Inicial).

```java
// Usar un valor IV diferente para cada cifrado
byte[] newIv = ...;
s = new GCMParameterSpec(s.getTLen(), newIv);
cipher.init(..., s);
...
```

- Buscar `IvParameterSpec`, verificar si el valor IV se genera de manera diferente y aleatoria.

```java
 IvParameterSpec iv = new IvParameterSpec(randBytes);
 SecretKeySpec skey = new SecretKeySpec(key.getBytes(), "AES");
 Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
 cipher.init(Cipher.ENCRYPT_MODE, skey, iv);
```

- En Java, buscar MessageDigest para verificar si se usa algoritmo de hash débil (MD5 o CRC). Por ejemplo:

`MessageDigest md5 = MessageDigest.getInstance("MD5");`

- Para firma, SHA1 y MD5 no deberían usarse. Por ejemplo:

`Signature sig = Signature.getInstance("SHA1withRSA");`

- Buscar `PBKDF2`. Para generar el valor hash de contraseña, se sugiere usar `PBKDF2`. Revisar los parámetros para generar el valor hash de `PBKDF2`.

Las iteraciones deberían ser sobre **10000**, y el valor **salt** debería generarse como **valor aleatorio**.

```java
private static byte[] pbkdf2(char[] password, byte[] salt, int iterations, int bytes)
    throws NoSuchAlgorithmException, InvalidKeySpecException
  {
       PBEKeySpec spec = new PBEKeySpec(password, salt, iterations, bytes * 8);
       SecretKeyFactory skf = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM);
       return skf.generateSecret(spec).getEncoded();
   }
```

- Información sensible hardcoded:

```text
User related keywords: name, root, su, sudo, admin, superuser, login, username, uid
Key related keywords: public key, AK, SK, secret key, private key, passwd, password, pwd, share key, shared key, cryto, base64
Other common sensitive keywords: sysadmin, root, privilege, pass, key, code, master, admin, uname, session, token, Oauth, privatekey, shared secret
```

## Herramientas

- Escáneres de vulnerabilidades tales como Nessus, Nmap (scripts), o OpenVAS pueden escanear en busca de uso o aceptación de cifrado débil contra protocolo tales como SNMP, TLS, SSH, SMTP, etc.
- Usar herramienta de análisis de código estático para hacer revisión de código fuente tales como klocwork, Fortify, Coverity, CheckMark para los siguientes casos.

```text
CWE-261: Weak Cryptography for Passwords
CWE-323: Reusing a Nonce, Key Pair in Encryption
CWE-326: Inadequate Encryption Strength
CWE-327: Use of a Broken or Risky Cryptographic Algorithm
CWE-328: Reversible One-Way Hash
CWE-329: Not Using a Random IV with CBC Mode
CWE-330: Use of Insufficiently Random Values
CWE-347: Improper Verification of Cryptographic Signature
CWE-354: Improper Validation of Integrity Check Value
CWE-547: Use of Hard-coded, Security-relevant Constants
CWE-780: Use of RSA Algorithm without OAEP
```

## Referencias

- [NIST FIPS Standards](https://csrc.nist.gov/publications/fips)
- [Wikipedia: Initialization Vector](https://en.wikipedia.org/wiki/Initialization_vector)
- [Secure Coding - Generating Strong Random Numbers](https://www.securecoding.cert.org/confluence/display/java/MSC02-J.+Generate+strong+random+numbers)
- [Optimal Asymmetric Encryption Padding](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding)
- [Hoja de Referencia de Almacenamiento Criptográfico](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [Hoja de Referencia de Almacenamiento de Contraseñas](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Secure Coding - Do not use insecure or weak cryptographic algorithms](https://www.securecoding.cert.org/confluence/display/java/MSC61-J.+Do+not+use+insecure+or+weak+cryptographic+algorithms)
- [Insecure Randomness](https://owasp.org/www-community/vulnerabilities/Insecure_Randomness)
- [Insufficient Entropy](https://owasp.org/www-community/vulnerabilities/Insufficient_Entropy)
- [Insufficient Session-ID Length](https://owasp.org/www-community/vulnerabilities/Insufficient_Session-ID_Length)
- [Using a broken or risky cryptographic algorithm](https://owasp.org/www-community/vulnerabilities/Using_a_broken_or_risky_cryptographic_algorithm)
- [Javax.crypto.cipher API](https://docs.oracle.com/javase/8/docs/api/javax/crypto/Cipher.html)
- ISO 18033-1:2015 – Encryption Algorithms
- ISO 18033-2:2015 – Asymmetric Ciphers
- ISO 18033-3:2015 – Block Ciphers
