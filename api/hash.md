# Hashes

Header: `#include "Hash.h"`

We don't include `sha1` function because it causes name collisions with other libraries.

## One line hash functions

One-line hash functions have a common API - `hash(msg, msglen, output)` where `msg` is the buffer with your message, `msglen` - it's length, `output` is an output buffer where to write result. The function returns number of bytes written to the output buffer.

If you want to hash Arduino `String`, use `.c_str()` method to get access to internal buffer:

```cpp
String msg = "Hash me!";
sha256(msg.c_str(), msg.length(), out);
```

Examples for available one-liners:

```cpp
// message we gonna hash
char msg[] = "Hash me completely!";
// length of the message
size_t len = strlen(msg);
// buffer that's enough for all hash functions:
uint8_t arr[64];

// sha256 outputs 32 bytes
size_t outLen = sha256(msg, len, arr);
// print it in hex, toHex() requires to #include "Conversion.h"
Serial.println(toHex(arr, outLen));

// sha512 outputs 64 bytes
outLen = sha512(msg, len, arr);
// rmd160 outputs 20 bytes
outLen = rmd160(msg, len, arr);

// Also there are commonly used combinations of hash functions:
// hash160 is the same as rmd160(sha256(msg))
outLen = hash160(msg, len, arr);
// doubleSha is the same as sha256(sha256(msg))
outLen = doubleSha(msg, len, arr);
```

## One line HMAC functions

Similar to hash functions, one line HMAC functions have a common API - `hmac(key, keylen, msg, msglen, output)` where `key` and `keylen` are the buffer with the HMAC key and it's length, `msg` and `msglen` are the buffer with your message and it's length, `output` is an output buffer where to write result. The function returns number of bytes written to the output buffer.

```cpp
// key
char key[] = "My super secret key";
size_t keylen = strlen(key);
// message
char msg[] = "Hash me completely!";
size_t msglen = strlen(msg);
// buffer that's enough for all hash functions:
uint8_t arr[64];

// sha256 outputs 32 bytes
size_t outLen = sha256Hmac(key, keylen, msg, msglen, arr);
// print it in hex, toHex() requires to #include "Conversion.h"
Serial.println(toHex(arr, outLen));

// sha512 outputs 64 bytes
size_t outLen = sha512Hmac(key, keylen, msg, msglen, arr);
// print it in hex
Serial.println(toHex(arr, outLen));
```

## Classes

If you can't hash in one line because you are hashing streaming data or you have multiple pieces that you want to hash together, you can use hash classes.

All classes share the same API, for `SHA256` and `SHA512` there are additional HMAC-related functions, but otherwise it's all the same.

Classes are:
- `SHA256`
- `SHA512`
- `RMD160`
- `Hash160`
- `DoubleSha`

Let's take `SHA256` for example. This is how you hash or HMAC in several rounds:

```cpp
uint8_t data[] = {0x01, 0x02, 0x03};
char str[] = "umtz umtz umtz";
String s = "blah blah blah";
char key[] = "angry unicorn";

// create an instance of the hash function
SHA256 hash;
// and another one for HMAC
SHA256 hmac;
// init hmac with the key
hmac.beginHMAC(key, strlen(key));

// add 3 bytes from data to the hash
hash.write(data, 3);
// and to the hmac
hmac.write(data, 3);

// hash char array as well
hash.write(str, strlen(str));
// hmac Arduino string
hmac.write(s.c_str(), s.length());

// get the result of the hash
uint8_t result[32];
hash.end(result);
// and get the result of HMAC
hmac.endHMAC(result);
```
