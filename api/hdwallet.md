# HD wallets

Requires: `#include "Bitcoin.h"`

Two main classes and mnemonic functions for [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) and [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) standard:

- [Mnemonic functions](#mnemonic)
- [`HDPrivateKey`](#hdprivatekey) - extended private key
- [`HDPublicKey`](#hdpublickey) - extended public key

For common derivation paths hd keys automatically detect the network and version according to [SLIP-132](https://github.com/satoshilabs/slips/blob/master/slip-0132.md). You can manually set the script `.type` of the key to a different type to change the prefix.

Automatically detected paths are:
- `m/84'/{coin}'/*'` - [BIP-84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki), coin type is `0` for mainnet, `1` for testnet. Used prefix - `zprv/zpub` for mainnet, `vprv/vpub` for testnet. Script type - `P2WPKH`.
- `m/49'/{coin}'/*'` - [BIP-49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki), coin type is `0` for mainnet, `1` for testnet. Used prefix - `yprv/ypub` for mainnet, `uprv/upub` for testnet. Script type - `P2SH_P2WPKH`.
- `m/48'/{coin}'/*'/2'` - native segwit multisig (`P2WSH`), prefix - `Zprv/Zpub` for mainnet, `Vprv/Vpub` for testnet.
- `m/48'/{coin}'/*'/1'` - nested segwit multisig (`P2WSH`), prefix - `Yprv/Ypub` for mainnet, `Uprv/Upub` for testnet.

## Usage example

This is an example how to generate a new recovery phrase, convert it to the root key, derive bip-84 and bip-48 accounts for testnet and derive individual public keys from xpub.

```cpp
// random data from good random sources or user input
String entropy = "h3iq7fqj3f7yowu3849uqomfiq3984";
// generating 12 words
String phrase = generateMnemonic(12, entropy);
// phrase: note upset stairs pupil copy want scorpion hard valid stamp weasel cloud

// creating root key from the recovery phrase and some password
HDPrivateKey root(phrase, "my super secure password");
// xprv9s21ZrQH143K3yDyvTZzUnGYiNt...8N8YipQSGdyLBcnJFDZeXF2b13N9

// deriving bip-84 account for testnet
HDPrivateKey bip84 = root.derive("m/84'/1'/0'");
// vprv9Lg1kCgJEUxjCY...a4DY8u5573ZmaMZgcK612W7AGVpE

// deriving bip-49 mainnet account using a derivation array
uint32_t derivation[] = {
    HARDENED_INDEX+49, 
    HARDENED_INDEX,
    HARDENED_INDEX
};
HDPrivateKey bip49 = root.derive(derivation, 3);
// yprvAK4VjBwvdAqGbx7uCSguwNhRf2...SR6UEVB8BQjdeUMDz2XJSno1yQH3jPh3hiN

// converting private to public key
HDPublicKey xpub = bip84.xpub();

// deriving first receiving public key, unsing child function
HDPublicKey first = xpub.child(0).child(0);
// address function of the HDPublicKey 
// will use stored .type and .network:
Serial.println(first.address());
// tb1qj86n7aj2u34a0x7ayr53sz7pvxyl0nm5vkaz79

// but if you convert it to a PublicKey this information is lost
PublicKey pub = first;
Serial.println(pub.address());
// 1EJkmwQwDvmRoZ54cV9WJ3ZdoVKBdbABEp
```

## Mnemonic

`generateMnemonic` function returns a [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) recovery phrase using english dictionary.

Number of words can be `12`, `15`, `18` and `24`. For better compatibility with other wallets only use `12` or `24`.

When external entropy is provided it is hashed using `sha256` hash function and part of the result is converted to the recovery phrase.

- `generateMnemonic(numWords)` - generates a new mnemonic using RAM-based RNG (gets some entopy from the initial state of the RAM). This RNG is pretty bad so use this function only for testing. In production write your own entropy generation function and provide it to the `generateMnemonic` function.
- `generateMnemonic(numWords, entropy, entropy_length)` - generates mnemonic from byte array.
- `generateMnemonic(numWords, string)` - generates mnemonic from the string.

Other mnemonic functions:

- `checkMnemonic(mnemonic)` - checks if mnemonic is valid
- `mnemonicToEntropy(mnemonic, array, arrayLen)` - converts mnemonic to entropy
- `mnemonicFromEntropy(array, arrayLen)` - converts entropy to mnemonic

## HDPrivateKey

`HDPrivateKey` is based on the [`PrivateKey`](keys.md#privatekey) class, so it has the same functionality - it can `.sign()`, you can get a `.publicKey()` from it etc.

You can instantiate `HDPrivateKey` either using it's human-readable form — base58 encoded string `xprv...`, or with a recovery phrase and a password:

```cpp
// using recovery phrase
HDPrivateKey hd1("habit opera fox human grow relax snow shoulder just knife tail guilt", "mypassword!");
// using xprv
HDPrivateKey hd2("xprv9s21ZrQH143K2Y2x3ehUA7JMErX2wPHiXwpciAR28Jsx5NK5Pw8Ured3p1toftWgA3nYxR53LHgZqHBgBUT4DsXFL4xzWr5Dpbr2D6kHfn9");
// using zprv and simple assignment
HDPrivateKey hd3 = "zprvAWgYBBk7JR8Gj8RBiNGiaHVManovpdGiNAs4GxCntKdiBZwXuFTc6mwKrRoyfhpWyL2ATNGAFcPfbrQocsH5pLtT4kMqgfiCN3yJzFxBzPb";
```

You can also create an empty key and fill it on later:

```cpp
HDPrivateKey hd;
// from recovery phrase and password
hd.fromMnemonic(mnemonic, password);
// from 64-byte seed
hd.fromSeed(seed);
// from string
hd.parse("zprvAWgYBBk7JR8Gj8RBiNGiaHVManovpdGiNAs4GxCntKdiBZwXuFTc6mwKrRoyfhpWyL2ATNGAFcPfbrQocsH5pLtT4kMqgfiCN3yJzFxBzPb");
// from raw serialized 78-byte array
hd.parse(raw, len);
```

You can change the type and the network of the private key if you want to:
```cpp
hd3.network = &Regtest;
hd3.type = P2SH_P2WPKH;
```

Available types that make a difference: `P2PKH` or `UNKNOWN_TYPE`, `P2WPKH`, `P2SH_P2WPKH`, `P2WSH`, `P2SH_P2WSH`. Corresponding [SLIP-132](https://github.com/satoshilabs/slips/blob/master/slip-0132.md) versions are used for serializations.

`.fingerprint()` function will give you a hex-encoded string with the fingerprint of the key (first 4 bytes of `hash160(pubkey)`).

You can derive hardened or non-hardened children from `HDPrivateKey`:

```cpp
// deriving first change key using a derivation string
HDPrivateKey child1 = hd.derive("m/48'/1'/0'/1/0")
// deriving with an array of derivations:
uint32_t der[] = {1,2,3}; // equivalent to "m/1/2/3"
HDPrivateKey child2 = hd.derive(der, 3);
// direct hardened child "m/3'":
HDPrivateKey hardened = hd.hardenedChild(3);
// non-hardened child "m/3":
HDPrivateKey nonhardered = hd.child(3);
```

To get corresponding `HDPublicKey` call:

```cpp
// the same network and script type will be in the xpub as well
HDPublicKey xpub = hd.xpub();
```

## HDPublicKey

`HDPublicKey` is based on the [`PublicKey`](keys.md#publickey) class, so it has the same functionality - it can `.verify()` signature, get `.segwitAddress()` and other address types etc.

It also has functionality very similar to [`HDPrivateKey`](#hdprivatekey), except hardened child derivations and mnemonic stuff.

```cpp
HDPublicKey xpub("zpub6rXFGRERF3SJyLPmfeocUwKUTTA5xdPiXm3Sgb73En1poyzzFrR7SbCN3GKJoujJTymFVn8G6Qqb6nsHj59VBDf4xxFwxys3A7TLoPvF5Fr");
// derive
HDPublicKey child = xpub.derive("m/0/0");
// change network and type
child.type = P2SH_P2WPKH;
child.network = &Testnet;
// print address
Serial.println(child.address());
```
