# Hashes

Header: `#include "Hash.h"`

We don't include `sha1` function because it causes name collisions with other libraries.

## Hash functions

Hash functions defined in Hash.h file. 

Single line functions:

- sha256
- sha256Hmac
- sha512
- sha512Hmac
- rmd160
- hash160 - rmd160(sha256(m))
- doubleSha

And corresponding hash classes (with HMAC support):

- SHA256
- SHA512
- RMD160
- Hash160
- DoubleSha
