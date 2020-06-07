# Configuring Bitcoin Core

You can use whatever wallet you want to watch your addresses, as soon as it can create PSBT transactions. There is also a minimal support of Electrum transactions, but it is not complete as there is no good documentation on the format.

In this tutorial we will use Bitcoin Core. First we need to create our watch-only wallet, let's call it `watchman`:

```bash
bitcoin-cli -testnet createwallet "watchman" true
```

Now we need to create a [wallet descriptor](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md) and import it.

In the descriptor we need to provide not only xpub, but also a fingerprint of the root key and derivation path in the form: `[fingerprint/derivation]xpub`.

Descriptor for native segwit addresses looks like this: `wpkh([fingerprint/<derivation/path>]xpub/0/*)#checksum` for receiving addresses and `.../1/*` for change addresses.

Let's write some code for our wallet to get our receiving descriptor. This function is defined in `PSBT.h` so don't forget to `#include "PSBT.h"`.

```cpp
String desc = "wpkh([";
// add fingerprint
desc += root.fingerprint();
// add derivation path. We need to remove leading `m`
desc += "/84h/1h/0h]";
// now pub xpub in normal form
xpub.type = UNKNOWN_TYPE;
desc += xpub.toString();
desc += "/0/*)";
// and add a checksum
desc += String("#")+descriptorChecksum(desc);
Serial.println(desc);
```

We need to do the same for change descriptor as well, just changing `/0/*` to `/1/*`.

Now we get two descriptors with checksums that we can form in a command to Bitcoin Core:

```bash
bitcoin-cli -testnet -rpcwallet="watchman" importmulti '[{"desc": "wpkh([a4a0eeeb/84h/1h/0h]tpubDCtqA3wKH2xdvaA1VyjpPiiMAswuqQoiQ6FKGmNHbHhcFqe7JrEPzEcViXqmR5T4XyXzzvC9vnQpL8sA97SghRZbgL3XsUNYxVRxR4HFn9j/0/*)#40r4znhh", "internal": false, "range": [0, 1000], "timestamp": "now", "keypool": true, "watchonly": true}, {"desc": "wpkh([a4a0eeeb/84h/1h/0h]tpubDCtqA3wKH2xdvaA1VyjpPiiMAswuqQoiQ6FKGmNHbHhcFqe7JrEPzEcViXqmR5T4XyXzzvC9vnQpL8sA97SghRZbgL3XsUNYxVRxR4HFn9j/1/*)#ymx5lx80", "internal": true, "range": [0, 1000], "timestamp": "now", "keypool": true, "watchonly": true}]'
```

Notice that first descriptor is marked as external (`"internal":false`) and second as internal. Also as it is a new wallet without any history we've set `rescan` option to `false`. 

That's it! Bitcoin Core is watching 1000 of our addresses. Let's check everything is right and ask Core for the first address:

```bash
bitcoin-cli -testnet -rpcwallet="watchman" getnewaddress "" bech32
```

We should get one of the addresses we printed [before](3_addresses.md). We can send some money to it, wait until it is confirmed and ask Core to create PSBT transaction.

```bash
bitcoin-cli -testnet -rpcwallet="watchman" walletcreatefundedpsbt '[]' '[{"tb1q35ea0m77yjke6uuf23etn8fqzyykufhvllnkqk":0.001}]' 0 '{"includeWatching":true}' true
```

If you are getting an error `Insufficient funds` just wait until your transaction is confirmed.

I got the folowing PSBT:

```
cHNidP8BAHECAAAAAbvujNVeCWqb0qj49V0FE4yNqzxNp/alDjJf6tfrDmLvAAAAAAD+////AgIMAwAAAAAAFgAUeVBepLdGFMTDXaEd8BcxXAjVXWSghgEAAAAAABYAFI0z1+/eJK2dc4lUcrmdIBEJbibsAAAAAAABAR/gkwQAAAAAABYAFBFV/d7WeO0DA8PtpNcne1avYrsIIgYCSUlKvX7bM2uW3icko9ATHDMcEV097AIjBJVvcYapQoAYpKDu61QAAIABAACAAAAAgAAAAAAAAAAAACICA0FKXUUYI9eFz3Kf5iDa4Iz4fGUp1/a27bGx4zBNKl1mGKSg7utUAACAAQAAgAAAAIABAAAAAAAAAAAA
```

We will sign it in the [next part](5_psbt.md).