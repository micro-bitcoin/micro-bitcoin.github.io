# HD wallets

Back in the days we were generating all private keys randomly. And it was awful — we had to make backups very often, backup size was growing, we had to reuse addresses... Dark times.

[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) solved this. Using small amount of initial entropy and magic of hash functions we are able to derive any amount of private keys. And even better, in some cases we can use *extended public keys* to generate new public keys without any knowledge about private keys (only if non-hardened derivation is used).

Derivation paths are also kinda standartized. We have [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) for legacy addresses, [BIP-49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki) for nested segwit and [BIP-84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki) for native segwit. Some of the wallets don't use these standards for derivation (i.e. Bitcoin Core, Green Wallet), others do.

To check that we are deriving everything right we can use [this nice tool](https://iancoleman.io/bip39/) by Ian Coleman.

Let's use [BIP-84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki) and generate an extended public key for testnet. We can import this extended public key to Bitcoin Core or Electrum and watch our balance.

```cpp
// last time we used mnemonic, let's use xprv this time
HDPrivateKey root("xprv9s21ZrQH143K2Y2x3ehUA7JMErX2wPHiXwpciAR28Jsx5NK"
                  "5Pw8Ured3p1toftWgA3nYxR53LHgZqHBgBUT4DsXFL4xzWr5Dpbr2D6kHfn9");
// derive account according to BIP-84 for testnet
HDPrivateKey account = root.derive("m/84'/1'/0'");
// print account and it's xpub
Serial.println(account);
// vprv9KsKcyEuS2MvjhWTH3e...Q2oRLWJhnrZSYQJ9F1Z4
Serial.println(account.xpub());
// vpub5Yrg2UmoGPvDxBavP5B...6t3uP285Lq5iFsrfkoKu
```

As you see it starts with `vpub`, not `tpub` that you would expect. That's because there is another commonly used non-BIP standard - [SLIP-132](https://github.com/satoshilabs/slips/blob/master/slip-0132.md). It is used in many wallest like Trezor, Electrum, Blue Wallet and others.

When you are deriving children keys the library automagically detect what prefix to use. If a standartized path is used that you will see corresponding prefix.

Try, for example `m/48'/0'/0'/2` - it is one of the standard multisig paths, so you will get `Zpub...`.

But Bitcoin Core doesn't understand these funny prefixes. So if we want to use Core as our watch-only wallet we need to get xpub. It can be done by setting a script type of the HD wallet:

```cpp
account.type = UNKNOWN_TYPE;
Serial.println(account.xpub());
// now will be tpubDCtqA3wKH2xdvaA1Vyjp...3XsUNYxVRxR4HFn9j
```

The following script types have slip-132 prefixes:
- `P2SH_P2WPKH` for `yprv`/`ypub`, `uprv`/`upub` on testnet
- `P2WPKH` for `zprv`/`zpub`, `vprv`/`vpub` on testnet
- `P2SH_P2WSH` for `Yprv`/`Ypub` (path `m/48'/0'/0'/1'`)
- `P2WSH` for `Zprv`/`Zpub` (path `m/48'/0'/0'/2'`)
- `UNKNOWN_TYPE`, `P2PKH` or any other script type for `xpub`/`xprv`

We can continue from the account and derive individual public keys. As now we don't need to do hardened derivation we can use `HDPublicKey` class that doesn't store private keys.

```cpp
HDPublicKey xpub = account.xpub();
// every child is also an xpub
HDPublicKey first = xpub.derive("m/0/0");
Serial.println(first);
// tpubDHEx9BBgd4TE3xeYFd3ssSWcG...GjT2fLmtsV9iqUjJaq2jC

// but it can be converted to a PublicKey as well
PublicKey pub = first;
Serial.println(pub);
// 0249494abd7edb336b96de2724a3d0131c331c115d3dec022304956f7186a94280
```

Now we have everything needed to print our receiving addresses. We will do it in the [next part](3_addresses.md).