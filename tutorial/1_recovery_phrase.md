# Recovery phrases

You've seen phrases like this before: `add good charge eagle walk culture book inherit fan nature seek repair`. These recovery phrases became standards in the industry and almost every wallet uses them. We will stick to the standard ([BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)) and use them as well.

Fundamentally it is just a convenient way of representing some random number in a human-readable form. It contains a checksum and uses a fixed dictionary so if you make a mistake while typing it the wallet will spot it. Then this recovery phrase together with the password can be converted to a 64-byte seed used for key derivation in [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

Entropy corresponding to the recovery phrase above is `032c8c9a228f686b06639e52d26f0c5b`. We could generate it at random using TRNG on the board, mix some user entropy and environmental noise there to get good overall randomness. Initial entropy generation is the point where good random numbers are really important.

We can use built-in random number generator, read bytes from analog-to-digital converters ([`AnalogRead()`](https://www.arduino.cc/reference/en/language/functions/analog-io/analogread/)), get some noise from the microphone or camera or even ask the user to randomly click the buttons.

Whatever method we use let's say we ended up with a looong byte array or string with data. Let's convert it to a recovery phrase and print it.

```cpp
// our random data from good random sources or user
String entropy = "h3iq7fqj3f7yowu3849uqomfiq3984";
// generating 12 words (first parameter, 24 by default)
String phrase = generateMnemonic(12, entropy);
// phrase: habit opera fox human grow relax snow shoulder just knife tail guilt
Serial.println(phrase);
```

The string here can be of any length. It will be hashed to make sure it's uniformly random and part of this hash will be converted to a recovery phrase according to the standard.

If we have a byte array we can do the same.

```cpp
uint8_t entropy[] = {
	0x68,0x33,0x69,0x71,0x37,0x66,0x71,0x6a,0x33,0x66,0x37,0x79,0x6f,0x77,0x75,
	0x33,0x38,0x34,0x39,0x75,0x71,0x6f,0x6d,0x66,0x69,0x71,0x33,0x39,0x38,0x34
};
// just don't forget to pass the length of the data in the array
String phrase = generateMnemonic(12, entropy, 30);
// same phrase: habit opera fox human grow relax snow shoulder just knife tail guilt
Serial.println(phrase);
```

Keep in mind that you can only generate recovery phrases with 12, 15, 18 or 24 words. To maintain compatibility with other wallets better to only use 12 or 24 words.

This recovery phrase can be converted to a root extended private key (often also called as master private key). The standard allows us to use a passphrase as well. By default the password is empty, but can be arbitrary number of any characters.

```cpp
// using default empty password
HDPrivateKey root(phrase, "");
// now we can check how it is converted to xprv
Serial.println(root);
// xprv9s21ZrQH143K2Y2x3ehUA7JMErX2wPHiXwpciAR28Jsx5NK5Pw8Ured3p1toftWgA3nYxR53LHgZqHBgBUT4DsXFL4xzWr5Dpbr2D6kHfn9
```

The passphrase should never be stored on the device, it is an additional protection measure for hardware wallets that don't have strong hardware security.

Now when we have our recovery phrase and root key in the [next part](2_hdwallet.md) we will start using it and derive more keys.