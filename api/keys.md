# Individual keys and signatures

Requires: `#include "Bitcoin.h"`

Three main classes for elliptic curve math:

- [`PrivateKey`](#privatekey) is a single key that is basically a 32-byte number.
- [`PublicKey`](#publickey) is a point on the curve defined as `privkey*G` where G is a generator point of our `secp256k1` curve.
- [`Signaure`](#ecdsa-signature) is a ECDSA signature.
- [`SchnorrSignaure`](#schnorrsignature) is a Schnorr signature.

## Usage example

In this example we demonstrate all three classes working together:

```cpp
// 32-byte secret
uint8_t secret[] = {
    0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 
    0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f,
    0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 
    0x18, 0x19, 0x1a, 0x1b, 0x1c, 0x1d, 0x1e, 0x1f
};
// set private key with scalar `secret`, compressed, for Testnet
// only secret is required, compressed is true by default, newtork is Mainnet
PrivateKey pk(secret, true, &Testnet);
// get corresponding pubkey
PublicKey pub = pk.publicKey();
// define a message to sign
char msg[] = "I created Bitcoin!";
// hash it, you may need to #include "Hash.h" to get hash functions
uint8_t hash[32];
// hash it to the hash array
sha256(msg, strlen(msg), hash);
// sign the hash of the message
Signature sig = pk.sign(hash);
// verify the signature
if(pub.verify(sig, hash)){
    Serial.println("Ok, I trust you.");
}else{
    Serial.println("No Craig, you are not Satoshi!");
}
```

## PrivateKey

A human-readable form of the `PrivateKey` is `WIF` - wallet import format. It is a base58 encoded string containing the scalar itself as well as information about the network and a flag that defines whether to use compressed public key or uncompressed.

`PrivateKey` can be constructed in several ways:

Using WIF string:
```cpp
PrivateKey pk("L3j6cTgf584owQWKbKD4B9DYWqBXWfr3JpVNFxKMdfJcJHN33A2A");
```

Providing a byte array with a 32-byte scalar: 
```cpp
PrivateKey pk(secret);
```

Optionally you can pass a compressed flag and network to the constructor: 
```cpp
PrivateKey pk(secret, false, &Testnet);
```
This will create a `PrivateKey` that will use uncompressed form for public key and `Testnet` as a network.

You can also create a private key without any parameters and call `.parse()` or `.setSecret()` methods later:

```cpp
PrivateKey pk;
// parse from WIF
pk.parse("L3j6cTgf584owQWKbKD4B9DYWqBXWfr3JpVNFxKMdfJcJHN33A2A");
// set secret value manually
pk.setSecret(secret);
// change compressed flag
pk.compressed = false;
// change network to regtest
pk.network = &Regtest;
```

To get a public key from the private key just call `.publicKey()` method. You can also get corresponding address for the private key. As private key contains information about the network it will print addresses for corresponding network:

```cpp
// get public key from private
PublicKey pub = pk.publicKey();
// print legacy address
Serial.println(pk.legacyAddress());
// print native segwit address
Serial.println(pk.segwitAddress());
// print nested segwit address
Serial.println(pk.nestedSegwitAddress());
```

To sign a message with the private key pass it's 32-byte hash to the `.sign()` method to get ECDSA signature, or `.schnorr_sign()` method to get `SchnorrSignature`:

```cpp
Signature sig = pk.sign(msghash);
SchnorrSignature schnorrsig = pk.schnorr_sign(msghash);
```

## PublicKey

`PublicKey` is a point on the curve. So it stores information about `x` and `y` coordinates of the point.

This class doesn't have a human-readable form. It also doesn't store any information about the network, but it stores information whether it should be serialized in compressed or uncompressed form.

Serialization format of the `PublicKey` is called `SEC` which stands for "Standards for Efficient Cryptography". Serialization of the public key in uncompressed form is 65 bytes long and looks like `04<x><y>`. In compressed form it is almost two times smaller and looks like `03<x>` or `02<x>`. The first byte defines whether the `y` coordinate is odd or even.

`PublicKey` can be constructed in several ways:

Using SEC byte array or corresponding hex string:
```cpp
// using hex-encoded string
PublicKey pub("02bac193519b1f8e062279e99a264973a30a7f70452756bbce29cd0ed091114457");
// using byte array
PublicKey pub2(sec);
```

Alternatively you can provide a 64-byte array with `<x><y>` coordinates, and a boolean `compressed` flag:
```cpp
// provide 64-byte array with <x><y> and set compressed to true
PublicKey pub(xydata, true);
```

You can also get a public key from private key:

```cpp
// get public key from private
PublicKey pub = pk.publicKey();
```

To get an address for the public key you need to pass a network as an argument. By default `Mainnet` is used:

```cpp
// print legacy address for mainnet
Serial.println(pub.legacyAddress());
// print native segwit address for signet
Serial.println(pub.segwitAddress(&Signet));
// print nested segwit address for testnet
Serial.println(pub.nestedSegwitAddress(&Testnet));
```

To verify a `Signature` against public key pass a signature and a message to the `.verify()` function for ECDSA signature or `.schnorr_verify()` for `SchnorrSignature`:

```cpp
if(pub.verify(sig, msghash)){
  // ok, signature is fine
}
if(pub.schnorr_verify(schnorrsig, msghash)){
  // ok, signature is fine
}
```

## ECDSA `Signature`

[ECDSA signature](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) also doesn't have a human-readable form. `DER` format is used for serialization which stands for "Distinguished Encoding Rules".

`Signature` class can be constructed using two byte arrays with `r` and `s` components of the signature, or by passing a byte array or hex-encoded string with it's `DER` encoding:

```cpp
// using `r` and `s` 32-byte arrays
Signature sig1(r, s);
// using der byte array
Signature sig2(der);
// using hex string
Signature sig3("304402202cb5f8e23c931e84be602add1e9405bb1833780e92cd8f3cbeee94c7cc26102702203cf5dfb6692f7d11ac2d0a290e9886c30836188a39ff01d1e950bb02062a16d5");
```

As `DER` encoding has a variable length maybe you want to know the length of the signature to allocate enough memory. It can be done using `.length()` method that will return number of bytes required for signature serialization.

## `SchnorrSignature`

Schnorr signature is always encoded as `64` byte array, containing `r` and `s` components. So it's `.length()` method will always return `64`.
