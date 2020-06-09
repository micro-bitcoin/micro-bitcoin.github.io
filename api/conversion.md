# Conversions

Requires: `#include "Conversion.h"`

A few handy conversion functions are defined here.

```cpp
uint8_t arr[] = {1,2,3,4,5,6};
size_t len = sizeof(arr);
// hex conversion
String hex = toHex(arr, len);
// base58 conversion, without checksum
String b58 = toBase58(arr, len);
// base58 conversion, with checksum
String b58ck = toBase58Check(arr, len);
// base64 conversion
String b64 = toBase64(arr, len);
// base43 conversion, used in Electrum QR codes
String b43 = toBase43(arr,len);
```

Reverse functions have similar API:

```cpp
uint8_t arr[6];
size_t len = sizeof(arr);
size_t written;
// from hex
written = fromHex("0102030405060708", arr, len);
// from base58
written = fromBase58("W7LcTy7", arr, len);
// from base58 with checksum
written = fromBase58Check("4HUtbHhPSbmKj", arr, len);
// from base64
written = fromBase64("AQIDBAUG", arr, len);
// from base43
written = fromBase43("43D0VQUV", arr, len);
```

Also there are functions to convert little or big endian and varints:

- `uint64_t littleEndianToInt(arr, len)`
- `intToLittleEndian(num, arr, len)`
- `uint64_t bigEndianToInt(arr, len)`
- `intToBigEndian(num, arr, len)`
- `readVarInt(arr, len)`
- `writeVarInt(num, arr, len)`

