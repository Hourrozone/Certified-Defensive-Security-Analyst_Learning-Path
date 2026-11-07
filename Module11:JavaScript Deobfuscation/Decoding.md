# Decoding

After doing the exercise in the previous section, we got a strange block of text that seems to be encoded:

<img width="742" height="103" alt="image" src="https://github.com/user-attachments/assets/67d3138c-8efb-47ca-a757-67acbe0e3efd" />

This is another important aspect of obfuscation that we referred to in **More Obfuscation** in the **Advanced Obfuscation** section. Many techniques can further obfuscate the code and make it less readable by humans and less detectable by systems. For that reason, you will very often find obfuscated code containing encoded text blocks that get decoded upon execution. We will cover 3 of the most commonly used text encoding methods:

- base64
- hex
- rot13

## Base64

base64 encoding is usually used to reduce the use of special characters, as any characters encoded in base64 would be represented in alpha-numeric characters, in addition to `+` and `/` only. Regardless of the input, even if it is in binary format, the resulting base64 encoded string would only use them.

### Spotting Base64

base64 encoded strings are easily spotted since they only contain alpha-numeric characters. However, the most distinctive feature of base64 is its padding using `=` characters. The length of base64 encoded strings has to be in a multiple of 4. If the resulting output is only 3 characters long, for example, an extra `=` is added as padding, and so on.

### Base64 Encode

To encode any text into base64 in Linux, we can echo it and pipe it with `'|'` to base64:

<img width="725" height="96" alt="image" src="https://github.com/user-attachments/assets/5cbcaddc-1ccd-4b61-af16-320e9d445c65" />

### Base64 Decode

If we want to decode any base64 encoded string, we can use `base64 -d`, as follows:

<img width="742" height="100" alt="image" src="https://github.com/user-attachments/assets/bba56dad-f19b-4c0b-bb22-1926d995dfe2" />

## Hex

Another common encoding method is hex encoding, which encodes each character into its hex order in the ASCII table. For example, a is 61 in hex, b is 62, c is 63, and so on. You can find the full ASCII table in Linux using the `man ascii` command.

### Spotting Hex

Any string encoded in hex would be comprised of hex characters only, which are 16 characters only: `0-9` and `a-f`. That makes spotting hex encoded strings just as easy as spotting base64 encoded strings.

### Hex Encode

To encode any string into hex in Linux, we can use the `xxd -p` command:

<img width="746" height="111" alt="image" src="https://github.com/user-attachments/assets/fa496926-2766-4c2f-9d63-6c84996f6aa7" />

### Hex Decode

To decode a hex encoded string, we can use the `xxd -p -r` command:

<img width="743" height="124" alt="image" src="https://github.com/user-attachments/assets/4dbefaa3-a0a3-443d-81c4-20c7ddd504d9" />

## Caesar/Rot13

Another common -and very old- encoding technique is a Caesar cipher, which shifts each letter by a fixed number. For example, shifting by 1 character makes a become b, and b becomes c, and so on. Many variations of the Caesar cipher use a different number of shifts, the most common of which is rot13, which shifts each character 13 times forward.

### Spotting Caesar/Rot13

Even though this encoding method makes any text looks random, it is still possible to spot it because each character is mapped to a specific character. For example, in rot13, `http://www` becomes `uggc://jjj`, which still holds some resemblances and may be recognized as such.

### Rot13 Encode

There isn't a specific command in Linux to do rot13 encoding. However, it is fairly easy to create our own command to do the character shifting:

<img width="739" height="111" alt="image" src="https://github.com/user-attachments/assets/43244ac0-49d2-4212-a8b2-117a72b2d8cf" />

### Rot13 Decode

We can use the same previous command to decode rot13 as well:

<img width="745" height="109" alt="image" src="https://github.com/user-attachments/assets/4b8e1378-4f4c-4284-986b-12dd726d5d12" />

Another option to encode/decode rot13 would be using an online tool, like rot13.

## Other Types of Encoding

There are hundreds of other encoding methods we can find online. Even though these are the most common, sometimes we will come across other encoding methods, which may require some experience to identify and decode.

If you face any similar types of encoding, first try to determine the type of encoding, and then look for online tools to decode it.

Some tools can help us automatically determine the type of encoding, like Cipher Identifier. Try the encoded strings above with Cipher Identifier, to see if it can correctly identify the encoding method.

Other than encoding, many obfuscation tools utilize encryption, which is encoding a string using a key, which may make the obfuscated code very difficult to reverse engineer and deobfuscate, especially if the decryption key is not stored within the script itself.

## 🔑 Key Takeaway

Encoded strings are commonly used in obfuscated code to hide readable text. Recognizing patterns such as Base64 padding, hexadecimal characters, and ROT13 substitutions can help identify and decode them. Encryption differs from encoding because it requires a key to decrypt the data.
