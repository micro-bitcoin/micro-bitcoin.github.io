# Addresses

Let's generate first 5 receiving addresses from our xpub. First we need to derive public keys, and then generate corresponding scripts or addresses directly.

Common single-key bitcoin scripts are:
- legacy pay-to-pubkey-hash
- pay-to-witness-pubkey-hash that use bech32 encoding for addresses
- pay-to-witness-pubkey-hash nested in pay-to-script-hash for backward compatibility with legacy wallets

```cpp
// this is out account xpub from previous part
HDPublicKey xpub("vpub5Yrg2UmoGPvDxBavP5BUmdByVpZMxnKf2EcaMRXNVNBxQMLwjK4BbfdPS5qotmByvnmUtPVAUMsfsP9RJrmRDWk6t3uP285Lq5iFsrfkoKu");
// deriving first 5 public keys and printing their addresses
HDPublicKey pub;
for(int i=0; i<5; i++){
    // deriving in a different manner
    pub = xpub.child(0).child(i);
    Serial.println(pub.address());
}
```

This will print our first addresses with a correct type - native segwit. Note that if we would define `pub` as `PublicKey` it would print legacy addresses for mainnet by default, and to get native segwit address for testnet we would need to slightly change the code:

```cpp
// deriving first 5 public keys and printing their addresses
PublicKey pub;
for(int i=0; i<5; i++){
    // deriving in a different manner
    pub = xpub.child(0).child(i);
    Serial.println(pub.segwitAddress(&Testnet));
}
```

Here we passed an argument to the function - the network we want address for.
Available networks are: `Mainnet`, `Testnet`, `Regtest` and `Signet`.

We can check that the addresses are derived correctly [here](https://iancoleman.io/bip39/).

If you would want to get a nested segwit address just call `.nestedSegwitAddress()` function.

We can print addresses now, it's time to make Bitcoin Core watching our addresses and create a transaction for us! To the [next part](4_bitcoin_core.md).
