# Working with PSBT transactions

Partially Signed Bitcoin Transaction format is a standard for unsigned bitcoin transactions defined in [BIP-174](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki). It was developed specifically for hardware wallets that have very limited resources and doesn't have enough knowledge about current state of the blockchain, UTXO set, used keys etc.

PSBT transaction is basically a raw bitcoin transaction with additional metadata that hardware wallet can use to sign the transaction and to display relevant information to the user. Normally it includes derivation pathes for all inputs and change outputs, scriptpubkeys and amounts of previous outputs we are spending, sometimes it may have redeem or witness script and so on. It is very well documented in the [BIP](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki) so it makes sense to go through it and read.

Normally PSBT transaction is in `base64` encoding, so we parse a transaction with `.fromBase64()` method:

```cpp
PSBT psbt;
psbt.parseBase64("cHNidP8BAHECAAAAAbvujNVeCWqb0qj49V0FE4yNqzxNp/alDjJf6tfrDmLvAAAAAA"
    "D+////AgIMAwAAAAAAFgAUeVBepLdGFMTDXaEd8BcxXAjVXWSghgEAAAAAABYAFI0z1+/eJK2dc4lUc"
    "rmdIBEJbibsAAAAAAABAR/gkwQAAAAAABYAFBFV/d7WeO0DA8PtpNcne1avYrsIIgYCSUlKvX7bM2uW"
    "3icko9ATHDMcEV097AIjBJVvcYapQoAYpKDu61QAAIABAACAAAAAgAAAAAAAAAAAACICA0FKXUUYI9e"
    "Fz3Kf5iDa4Iz4fGUp1/a27bGx4zBNKl1mGKSg7utUAACAAQAAgAAAAIABAAAAAAAAAAAA");
```

Before signing the transaction we need to display all the necessary information about it. At least the addresses and amounts:

```cpp
Serial.println("Transactions details:");
Serial.println("Outputs:");
// going through all outputs
for(int i=0; i<psbt.tx.outputsNumber; i++){
  Serial.print(psbt.tx.txOuts[i].address(&Testnet));
  Serial.print(" -> ");
  // You can also use .btcAmount() function that returns a float in whole Bitcoins
  Serial.print(int(psbt.tx.txOuts[i].amount));
  Serial.println(" sat");
}
Serial.print("Fee: ");
// Arduino can't print 64-bit ints so we need to convert it to int
Serial.print(int(psbt.fee()));
Serial.println(" sat");
```

Assuming the user confirmed the transaction we sign it using our root key:

```cpp
psbt.sign(root);
Serial.println(psbt.toBase64());
```

Currenlty when PSBT is serializes the library removes all metadata except signatures, so to get final transaction we need to ask Bitcoin Core to combine information from both and finalize:

```bash
bitcoin-cli -testnet combinepsbt '["cHNidP8BAHECAAAAAbvujNVeCWqb0qj49V0FE4yNqzxNp/alDjJf6tfrDmLvAAAAAAD+////AgIMAwAAAAAAFgAUeVBepLdGFMTDXaEd8BcxXAjVXWSghgEAAAAAABYAFI0z1+/eJK2dc4lUcrmdIBEJbibsAAAAAAABAR/gkwQAAAAAABYAFBFV/d7WeO0DA8PtpNcne1avYrsIIgYCSUlKvX7bM2uW3icko9ATHDMcEV097AIjBJVvcYapQoAYpKDu61QAAIABAACAAAAAgAAAAAAAAAAAACICA0FKXUUYI9eFz3Kf5iDa4Iz4fGUp1/a27bGx4zBNKl1mGKSg7utUAACAAQAAgAAAAIABAAAAAAAAAAAA","cHNidP8BAHECAAAAAbvujNVeCWqb0qj49V0FE4yNqzxNp/alDjJf6tfrDmLvAAAAAAD+////AgIMAwAAAAAAFgAUeVBepLdGFMTDXaEd8BcxXAjVXWSghgEAAAAAABYAFI0z1+/eJK2dc4lUcrmdIBEJbibsAAAAAAAiAgJJSUq9ftsza5beJySj0BMcMxwRXT3sAiMElW9xhqlCgEgwRQIhAKMya43HJtU1HyjzDUkzh7v7EOGyZ6uE6WpfP+9SC7/fAiAd3jjiEkmRnNDSaGcc7+VGB7opg45mIv+fXWtXzMyA2QEAAAA="]'
```

```bash
bitcoin-cli -testnet finalizepsbt "cHNidP8BAHECAAAAAbvujNVeCWqb0qj49V0FE4yNqzxNp/alDjJf6tfrDmLvAAAAAAD+////AgIMAwAAAAAAFgAUeVBepLdGFMTDXaEd8BcxXAjVXWSghgEAAAAAABYAFI0z1+/eJK2dc4lUcrmdIBEJbibsAAAAAAABAR/gkwQAAAAAABYAFBFV/d7WeO0DA8PtpNcne1avYrsIIgICSUlKvX7bM2uW3icko9ATHDMcEV097AIjBJVvcYapQoBIMEUCIQCjMmuNxybVNR8o8w1JM4e7+xDhsmerhOlqXz/vUgu/3wIgHd444hJJkZzQ0mhnHO/lRge6KYOOZiL/n11rV8zMgNkBIgYCSUlKvX7bM2uW3icko9ATHDMcEV097AIjBJVvcYapQoAYpKDu61QAAIABAACAAAAAgAAAAAAAAAAAACICA0FKXUUYI9eFz3Kf5iDa4Iz4fGUp1/a27bGx4zBNKl1mGKSg7utUAACAAQAAgAAAAIABAAAAAAAAAAAA"
```

And now we can broadcast:

```bash
bitcoin-cli -testnet sendrawtransaction "02000000000101bbee8cd55e096a9bd2a8f8f55d05138c8dab3c4da7f6a50e325fead7eb0e62ef0000000000feffffff02020c03000000000016001479505ea4b74614c4c35da11df017315c08d55d64a0860100000000001600148d33d7efde24ad9d73895472b99d2011096e26ec02483045022100a3326b8dc726d5351f28f30d493387bbfb10e1b267ab84e96a5f3fef520bbfdf02201dde38e21249919cd0d268671cefe54607ba29838e6622ff9f5d6b57cccc80d901210249494abd7edb336b96de2724a3d0131c331c115d3dec022304956f7186a9428000000000"
```

We just signed our first transaction with a hardware wallet and broadcasted it with Bitcoin Core! 

But it's not enough - at the moment we see both outputs when transaction is printed. Would be nice to detect and verify which of the addresses is a change address and remove it from the list.

We will do it in the [last section](6_change.md).