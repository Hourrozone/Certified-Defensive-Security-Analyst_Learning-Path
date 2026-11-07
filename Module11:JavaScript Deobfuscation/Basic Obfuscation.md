# Basic Obfuscation

## Basic Obfuscation

Code obfuscation is usually not done manually, as there are many tools for various languages that do automated code obfuscation. Many online tools can be found to do so, though many malicious actors and professional developers develop their own obfuscation tools to make it more difficult to deobfuscate.

## Running JavaScript code

Let us take the following line of code as an example and attempt to obfuscate it:

<img width="740" height="62" alt="image" src="https://github.com/user-attachments/assets/6697b7cc-b3df-4bb7-b3e3-bf8977343b24" />

First, let us test running this code in cleartext, to see it work in action. We can go to JSConsole, paste the code and hit enter, and see its output:

<img width="1243" height="366" alt="image" src="https://github.com/user-attachments/assets/2f883681-6e29-4795-a76f-08d962d6bd43" />

We see that this line of code prints `HTB JavaScript Deobfuscation Module`, which is done using the `console.log()` function.

## Minifying JavaScript code

A common way of reducing the readability of a snippet of JavaScript code while keeping it fully functional is JavaScript minification. Code minification means having the entire code in a single (often very long) line. Code minification is more useful for longer code, as if our code only consisted of a single line, it would not look much different when minified.

Many tools can help us minify JavaScript code, like javascript-minifier. We simply copy our code, and click Minify, and we get the minified output on the right:

<img width="2510" height="1256" alt="image" src="https://github.com/user-attachments/assets/010c51ff-4337-4355-8d54-917adc09613f" />

Once again, we can copy the minified code to JSConsole, and run it, and we see that it runs as expected. Usually, minified JavaScript code is saved with the extension `.min.js`.

> **Note:** Code minification is not exclusive to JavaScript, and can be applied to many other languages, as can be seen on javascript-minifier.

## Packing JavaScript code

Now, let us obfuscate our line of code to make it more obscure and difficult to read. First, we will try BeautifyTools to obfuscate our code:

<img width="1390" height="926" alt="image" src="https://github.com/user-attachments/assets/3348e8fc-b3b1-472e-bd66-71da42ee26a4" />

<img width="737" height="131" alt="image" src="https://github.com/user-attachments/assets/2d339794-3b6a-4036-8086-95558d61d9cc" />

We see that our code became much more obfuscated and difficult to read. We can copy this code into https://jsconsole.com, to verify that it still does its main function:

<img width="2864" height="456" alt="image" src="https://github.com/user-attachments/assets/43dc5f99-83d5-4c49-a63f-1b6f94b4869e" />

We see that we get the same output.

> **Note:** The above type of obfuscation is known as "packing", which is usually recognizable from the six function arguments used in the initial function `"function(p,a,c,k,e,d)"`.

A packer obfuscation tool usually attempts to convert all words and symbols of the code into a list or a dictionary and then refer to them using the `(p,a,c,k,e,d)` function to re-build the original code during execution. The `(p,a,c,k,e,d)` can be different from one packer to another. However, it usually contains a certain order in which the words and symbols of the original code were packed to know how to order them during execution.

While a packer does a great job reducing the code's readability, we can still see its main strings written in cleartext, which may reveal some of its functionality. This is why we may want to look for better ways to obfuscate our code.

## 🔑 Key Takeaway

JavaScript obfuscation can involve minification, which reduces formatting and whitespace, or packing, which replaces words and symbols with dictionary references that are reconstructed during execution. Although packing makes code harder to read, cleartext strings may still reveal parts of its functionality.
