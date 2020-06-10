# Electrum

Requires: `#include "Electrum.h"`

Using with [Electrum wallet](https://electrum.org/)

Currently only works with single-sig transactions.

## Usage example

Keep in mind that Electrum transaction:
- only contains two last indexes for key derivation, so you need to sign transaction with account key, not with the root key
- doesn't have any derivation information for change output, so to detect change address you need to bruteforce addresses or keep a lookup table of addresses

Here is an example how to parse, display and sign Electrum transaction

```cpp
// heart sentence debate adult dizzy city almost since illness common walnut nuclear
HDPrivateKey key("vprv9L6pyxQ546fA7tGh21XpUAw7qAcjtoRwUu4r3AsXkQSZNTKVENtmtdwvukdEKyufFjqFE2NWUDuTZNQ4ZLvh6kxcawQguwWtGbz2KsqsyCd");

// segwit
String hex_tx = "45505446ff0002000000000101a97ff281232b6c599a356f157fdbea1f593f05d7e06db6053c04fd8f2e323afd0000000000fdffffff02a0860100000000001600145ed209b2d8ff40528206014d734c23627ad432a61dd431020000000016001488e54c917dc106a79093abd6f016aedcf84093b5feffffffff4a5b33020000000000000201ff53ff045f1cf60395fb52d1800000003e0828c7242312021bfc688606d99d0df8f45866517a69a427c6e283bad9402602dcb433d0c1e04bb69016d6da9ee2ce246166ebad6bf3e6110eb9c3e9b6080011000000005aab1a00";

ElectrumTx tx;

// trying to parse
tx.parse(hex_tx);
if(!tx){
  Serial.println("Can't parse tx");
  return;
}
// print outputs
for(int i=0; i<tx.tx.outputsNumber; i++){
  Serial.print(tx.tx.txOuts[i].address(key.network));
  Serial.print(" -> ");
  // Serial can't print uint64_t, so convert to int
  Serial.print(int(tx.tx.txOuts[i].amount));
  Serial.println(" sat");
}
// fee
Serial.print("Fee: ");
Serial.print(int(tx.fee()));
Serial.println(" sat");

tx.sign(key); 
// print signed transaction - ready to broadcast
Serial.println(tx);
```

When transaction is signed it will be serialized as a raw signed Bitcoin transaction, so you can broadcast it using Electrum or any block explorer.