# Tutorial

To keep the tutorial hardware-independent we will only use `Serial`, both for "GUI" and for transactions. It will show you the logic behind the library and then you can add your favorite GUI library, buttons or whatever you use and make it more functional.

Any decent board with 32-bit microcontroller would work - esp32, for example [M5Stack](https://m5stack.com/) or [TTGO](https://www.aliexpress.com/store/2090076?spm=a2g0o.productlist.0.0.757b39deNF7ohU), stm32 boards, [Adafruit](https://www.adafruit.com/category/943) M0 or M4 series (better M4) and many others.

In the tutorial we will to the following:

- Generate a recovery phrase from entropy
- Derive root key from mnemonic and password
- Derive an account and print the master public key
- Print first 5 receiving addresses
- Parse PSBT transaction and outputs
- Check if we have a change output and hide it
- Sign PSBT transaction and print it

Let's start with the [recovery phrases](/tutorial/1_recovery_phrase.md)!