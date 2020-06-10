# Change detection

Change detection is the most complicated part for any hardware wallet. There were plenty of attacks on different wallets due to this functionality ([Trezor](https://blog.trezor.io/details-of-the-multisig-change-address-issue-and-its-mitigation-6370ad73ed2a), [Ledger](https://sergeylappo.github.io/ledger-hack/)).

So we need to be extra careful when hiding one of the outputs.

I recommend checking that:
- all inputs have the same script type (no mixing of single-key and multisig)
- change output has the same script type as inputs
- all keys are derived from the same account xpub
- derivation indexes are not too different from each other

In the scope of our tutorial we will only verify that we can derive the public key and that the output address is the same as native segwit address of our public key.

In the last section we wrote the following code for printing the transaction:

```cpp
// going through all outputs
for(int i=0; i<psbt.tx.outputsNumber; i++){
  Serial.print(psbt.tx.txOuts[i].address(&Testnet));
  Serial.print(" -> ");
  // You can also use .btcAmount() function that returns a float in whole Bitcoins
  Serial.print(int(psbt.tx.txOuts[i].amount));
  Serial.println(" sat");
}
```

Now let's change it and check every output if it is a change output or not:

```cpp
// going through all outputs
for(int i=0; i<psbt.tx.outputsNumber; i++){
  Serial.print(psbt.tx.txOuts[i].address(&Testnet));
  // check if it is a change output
  if(psbt.txOutsMeta[i].derivationsLen > 0){ // there is derivation path
    // considering only single key for simplicity
    HDPublicKey pub = hd.derive(der.derivation, der.derivationLen).xpub();
    // as pub is HDPublicKey it will also generate correct address type
    if(pub.address() == psbt.tx.txOuts[i].address()){
      Serial.print(" (change) ");
    }
  }
  Serial.print(" -> ");
  // You can also use .btcAmount() function that returns a float in whole Bitcoins
  Serial.print(int(psbt.tx.txOuts[i].amount));
  Serial.println(" sat");
}
```

If we check the output on our transaction now we will see:

```
Transactions details:
Outputs:
tb1q09g9af9hgc2vfs6a5ywlq9e3tsyd2htyryqdw5 (change)  -> 199682 sat
tb1q35ea0m77yjke6uuf23etn8fqzyykufhvllnkqk -> 100000 sat
Fee: 318 sat
```

That's it, we have a minimal hardware wallet!

From now you can go to the [API documentation](../api/README.md) or checkout our collection of [recepies](../recepies/README.md) for some common use-cases. Have fun :)
