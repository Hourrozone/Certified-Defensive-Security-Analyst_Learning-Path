# Advanced Obfuscation

So far, we have been able to make our code obfuscated and more difficult to read. However, the code still contains strings in cleartext, which may reveal its original functionality. In this section, we will try a couple of tools that should completely obfuscate the code and hide any remnants of its original functionality.

## Obfuscator

Let's visit https://obfuscator.io. Before we click obfuscate, we will change String Array Encoding to Base64, as seen below:

<img width="2242" height="578" alt="image" src="https://github.com/user-attachments/assets/e6942d2b-a4d8-45bf-8203-6c709383c013" />

Now, we can paste our code and click obfuscate:

<img width="2262" height="752" alt="image" src="https://github.com/user-attachments/assets/b9ab4085-db2f-48a7-afdb-bcdf0fcb4f48" />

We get the following code:

<img width="735" height="475" alt="image" src="https://github.com/user-attachments/assets/a766a6f4-645a-409b-86ec-9bd148fedec7" />

This code is obviously more obfuscated, and we can't see any remnants of our original code. We can now try running it in https://jsconsole.com to verify that it still performs its original function. Try playing with the obfuscation settings in https://obfuscator.io to generate even more obfuscated code, and then try rerunning it in https://jsconsole.com to verify it still performs its original function.

## More Obfuscation

Now we should have a clear idea of how code obfuscation works. There are still many variations of code obfuscation tools, each of which obfuscates the code differently. Take the following JavaScript code, for example:

<img width="740" height="342" alt="image" src="https://github.com/user-attachments/assets/cb1ca123-f8fa-4926-9c80-770ef001e12c" />

We can still run this code, and it would still perform its original function:

<img width="2880" height="864" alt="image" src="https://github.com/user-attachments/assets/2d0563b6-ac76-455b-a59f-50efe13889d1" />

> **Note:** The above code was snipped as the full code is too long, but the full code should successfully run.

We can try obfuscating code using the same tool in JSF, and then rerunning it. We will notice that the code may take some time to run, which shows how code obfuscation could affect the performance, as previously mentioned.

There are many other JavaScript obfuscators, like JJ Encode or AA Encode. However, such obfuscators usually make code execution/compilation very slow, so it is not recommended to be used unless for an obvious reason, like bypassing web filters or restrictions.

## 🔑 Key Takeaway

Advanced JavaScript obfuscation tools, such as obfuscator.io, can encode strings and hide recognizable elements of the original code. Different obfuscation techniques offer varying levels of concealment, but heavily obfuscated code may take longer to execute.
