<img width="2233" height="838" alt="image" src="https://github.com/user-attachments/assets/773d0487-498b-43c3-98ab-d3d029b94de7" /># Deobfuscation

Now that we understand how code obfuscation works let's start our learning towards deobfuscation. Just as there are tools to obfuscate code automatically, there are tools to beautify and deobfuscate the code automatically.

## Beautify

We see that the current code we have is all written in a single line. This is known as Minified JavaScript code. In order to properly format the code, we need to Beautify our code. The most basic method for doing so is through our Browser Dev Tools.

For example, if we were using Firefox, we can open the browser debugger with `CTRL+SHIFT+Z`, and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the `{ }` button at the bottom, which will Pretty Print the script into its proper JavaScript formatting:

<img width="975" height="426" alt="image" src="https://github.com/user-attachments/assets/a26af725-1618-43fb-a689-f67fb9f2d2a0" />

Furthermore, we can utilize many online tools or code editor plugins, like Prettier or Beautifier. Let us copy the `secret.js` script:

<img width="733" height="170" alt="image" src="https://github.com/user-attachments/assets/bbb0de1b-5e84-4768-bf1c-1a2040a42345" />

We can see that both websites do a good job in formatting the code:

<img width="2233" height="838" alt="image" src="https://github.com/user-attachments/assets/46ffb38d-77c9-46b8-9a1d-63c891ea627d" />

<img width="1393" height="660" alt="image" src="https://github.com/user-attachments/assets/80c1d5ec-e48f-4697-bc48-c5c449a8069d" />

However, the code is still not very easy to read. This is because the code we are dealing with was not only minified but obfuscated as well. So, simply formatting or beautifying the code will not be enough. For that, we will require tools to deobfuscate the code.

## Deobfuscate

We can find many good online tools to deobfuscate JavaScript code and turn it into something we can understand. One good tool is **UnPacker**. Let's try copying our above-obfuscated code and run it in UnPacker by clicking the UnPack button.

> **Tip:** Ensure you do not leave any empty lines before the script, as it may affect the deobfuscation process and give inaccurate results.

<img width="718" height="503" alt="image" src="https://github.com/user-attachments/assets/3b918980-ef04-4220-babb-a4750867858a" />

We can see that this tool does a much better job in deobfuscating the JavaScript code and gave us an output we can understand:

<img width="740" height="203" alt="image" src="https://github.com/user-attachments/assets/0c8584be-f19f-40a8-bede-d261a21bf865" />

As previously mentioned, the above-used method of obfuscation is packing. Another way of unpacking such code is to find the return value at the end and use `console.log` to print it instead of executing it.

## Reverse Engineering

Though these tools are doing a good job so far in clearing up the code into something we can understand, once the code becomes more obfuscated and encoded, it would become much more difficult for automated tools to clean it up. This is especially true if the code was obfuscated using a custom obfuscation tool.

We would need to manually reverse engineer the code to understand how it was obfuscated and its functionality for such cases. If you are interested in knowing more about advanced JavaScript Deobfuscation and Reverse Engineering, you can check out the Secure Coding 101 module, which should thoroughly cover this topic.

## 🔑 Key Takeaway

JavaScript deobfuscation starts with beautifying minified code, followed by using unpacking tools to reverse common obfuscation techniques. When automated tools are insufficient, manual reverse engineering is necessary to understand custom or more complex obfuscation.
