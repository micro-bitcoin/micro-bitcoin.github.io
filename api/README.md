# Library API

## Header files

To use the library you need to include it's headers. Not all of them are necessary, it depends on what functionality you use in the code.

For example, our [tutorial](../tutorial/) was only using HD wallets and PSBT transactions, so all we needed was to include:

```cpp
#include "Bitcoin.h"
#include "PSBT.h"
```

But there are also other headers that have some more stuff:

```cpp
// hash functions and classes - sha256, sha512, HMAC and PBKDF2
#include "Hash.h"
// helper functions to convert from and to different encodings:
// hex, base64, base58, bech32 and base43.
#include "Conversion.h" 
// transaction format used in Electrum wallet
#include "Electrum.h"
```

## Creating class instances

You can create a class instance without any parameters, pass parameters according to the class constructor or pass a string. If the class doesn't have a human-readable representation the string passed to the constructor should be a hex representation of the serialization format.

For example, `HDPublicKey` class has a human readable representation, so I can instantiate it like this:

```cpp
HDPublicKey hd("xpub6Bo5UgSfGJuCSVMJW1Rh8DwjoQXRJgh8iZgzE2sVNV4knMQuNtqwcyPVvbvLUYpZAP9C6Nn1xaH4AD9u4cGUfj4BTMHUxFFZtA3WRsokiQM");
// or even like this:
HDPublicKey hd2 = "zpub6rTnwpKM9ZXmXdJWeEEJmuWwi2afCKnPfX1kHL9YbAdRBQitf12WcCjTWUhp6SNP8WjhMkp28GkG42NjV1ZZ7zaTBYTeqjqYQukfMWweE4w";
```

But `PublicKey` doesn't have a human-readable for, so we use a hex-encoded string:

```cpp
PublicKey pub("031f3ca913f17ad79cfbc5717c0bc000b3d93cdb71bbe4366cff86aa70eeafd3b8");
// assignmeng like pub = "031..." will not work here.
```

If we know we gonna need a global class instance we can create it without any parameters and then call `.parse()` function when we want to change it. For example, a global root key:

```cpp
HDPrivateKey root;
//... and then somewhere in your code
root.parse("xprv9s21ZrQH143K2V89ZYW7WYT3TfaR2jHmugDEnbhV6hi9EskpDwx5sQDs7xQFhxDUFkmMjq4BbGmLVBupM8mkBGUx2jn85pK1EypPTn8LEjB");
// or just assign it:
root = "xprv9s21ZrQH143K2V89ZYW7WYT3TfaR2jHmugDEnbhV6hi9EskpDwx5sQDs7xQFhxDUFkmMjq4BbGmLVBupM8mkBGUx2jn85pK1EypPTn8LEjB";
```

## Print and display

All classes can be printed or displayed. If the class has a human-readable form it will be used for printing. If not - hex encoding will be used.

For example, printing `PrivateKey` that has a human-readable format (WIF), and `PublicKey` that doesn't:

```cpp
PrivateKey pk = "KwoKWyR45S1rk4SPAwzofVfxgcNTHZ6b9o3ouxKQrUjASZBga5Be";
PublicKey pub = pk.publicKey();
Serial.println(pk); // will print KwoKWyR45S1rk4SPAwzofVfxgcNTHZ6b9o3ouxKQrUjASZBga5Be
Serial.println(pub); // will print 0377206b87cbf321089476a0c55bb624b468a07fb18e28d9960f58b2dfbe595a07
```

So all of them can be also converted to Strings. If you prefer to send them as raw bytes you can do that as well:

```cpp
byte buf[33];
// len will be number of bytes written to the buffer
size_t len = pub.serialize(buf, sizeof(buf));
// send raw bytes to the serial
Serial.write(buf, len);
```

The same applies to parsing — you can use `.parse(uint8_t * arr, size_t len)`.

## Networks

In some cases we need to use network-specific prefixes. For example HD keys have different prefix on mainnet and on testnet. Also addresses are different.

We define all these prefixes in `Network` struct and supported networks are `Mainnet`, `Testnet`, `Regtest` and `Signet`.

For example to change a network of the `HDPrivateKey` to Testnet use:
```cpp
HDPrivateKey hd("xprv9s21ZrQH143K2V89ZYW7WYT3TfaR2jHmugDEnbhV6hi9EskpDwx5sQDs7xQFhxDUFkmMjq4BbGmLVBupM8mkBGUx2jn85pK1EypPTn8LEjB");
hd.network = &Testnet;
Serial.println(hd.xpub()); // will print tpub...
```

The same with address functions. Printing segwit address for Signet:
```cpp
PublicKey pub("031f3ca913f17ad79cfbc5717c0bc000b3d93cdb71bbe4366cff86aa70eeafd3b8");
Serial.println(pub.segwitAddress(&Signet));
// prints sb1q798f6qusgndu2yunye7yq5xaq3pftgt4df9djv
```

Notice that we use a pointers here: `&Signet` and `&Testnet`.

## Classes overview

First, classes that are available when you `#include "Bitcoin.h"` header:

- [`PrivateKey`](keys.md#privatekey), [`PublicKey`](keys.md#publickey) and [`Signature`](keys.md#signature) are described in the [Individual keys and signature](keys.md) section.
- [`HDPrivateKey`](hdwallets.md#hdprivatekey), [`HDPublicKey`](hdwallets.md#hdpublickey) and [mnemonic functions](hdwallets.md#mnemonic) are in [HD wallets](hdwallets.md) section.
- [`Script`](scripts.md) - how to work with Bitcoin Scripts
- [`Transaction`](transaction.md) - raw Bitcoin transactions

Some functionality is moved to separate header files:

- [`PSBT`](psbt.md) - how to work with PSBT transactions, requires to include `"PSBT.h"`
- [`ElectrumTx`](electrum.md) - unsigned transaction format used in Electrum Wallet, requires `"Electrum.h"` header.
- [Conversion functions](conversion.md) - hex, base58, bech32, base64 and base43 defined in `"Conversion.h"`
- [Hash functions](hash.md) - sha256, sha512, rmd160 and hmac functions defined in `"Hash.h"`
