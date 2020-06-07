
## Elliptic curve math

- ECScalar - a 256-bit number modulo N (curve order)
- ECPoint - a point on an elliptic curve (secp256k1)


## Key management:

Classes:

- PrivateKey - a single private key class. Stores the 32-byte secret and network information (mainnet or testnet). Can be exported / imported as WIF. Inherits functionality from ECScalar - you can add, multiply, divide them, multiply PublicKey by it etc.
- PublicKey - a single public key class. Inherits functionality from ECPoint class - you can add them, multiply by scalar etc.
- HDPrivateKey - HD wallet private key with bip32 functionality. For common derivation pathes (bip44/49/84) automatically detects the script types. Can derive children and hardened children, can be derived from mnemonic or seed.
- HDPublicKey - An HD public key corresponding to a particular HD private key. Can derive children, but not hardened.

Handy functions:

- `generateMnemonic(int strength)` - generates a new mnemonic using RNG
- `generateMnemonic(const uint8_t * entropy_data, size_t length)` - generates mnemonic from byte array
- `generateMnemonic(const char * entropy_string)` - generates mnemonic from the string

## Other Bitcoin classes

- Signature - ECDSA signature
- Script
- Witness
- Tx - transaction class
- TxIn
- TxOut

Extra classes:

- PSBT - partially signed bitcoin transaction ([bip174](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki))
- ElectrumTx - unsigned electrum transaction, poorly implemented, consider using PSBT instead.
