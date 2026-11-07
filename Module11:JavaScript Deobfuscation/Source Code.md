# Source Code

Most websites nowadays utilize JavaScript to perform their functions. While HTML is used to determine the website's main fields and parameters, and CSS is used to determine its design, JavaScript is used to perform any functions necessary to run the website. This happens in the background, and we only see the pretty front-end of the website and interact with it.

Even though all of this source code is available at the client-side, it is rendered by our browsers, so we do not often pay attention to the HTML source code. However, if we wanted to understand a certain page's client-side functionalities, we usually start by taking a look at the page's source code. This section will show how we can uncover the source code that contains all of this and understand its general usage.

## HTML

We will start by starting the exercise below, open Firefox in our PwnBox, and visit the url shown in the question:

<img width="2880" height="622" alt="image" src="https://github.com/user-attachments/assets/894f8393-b1a1-47c1-b37e-0945a9bdd5ec" />

As we can see, the website says Secret Serial Generator, without having any input fields or showing any clear functionality. So, our next step is to peek at its source code. We can do that by pressing `[CTRL + U]`, which should open the source view of the website:

<img width="1328" height="500" alt="image" src="https://github.com/user-attachments/assets/40da061a-0f85-4bf5-9922-3f3c338bbd7b" />

As we can see, we can view the HTML source code of the website.

The HTML source code can hold various information, such as comments, to further facilitate a better understanding of the code. Developers may leave sensitive information, which can be further taken advantage of. It is worth reading through the HTML comments.

## CSS

CSS code is either defined internally within the same HTML file between `<style>` elements, or defined externally in a separate `.css` file and referenced within the HTML code.

In this case, we see that the CSS is internally defined, as seen in the code snippet below:

<img width="742" height="365" alt="image" src="https://github.com/user-attachments/assets/17267ae6-cb6e-4095-a896-55b15cf4d198" />

If a page CSS style is externally defined, the external `.css` file is referred to with the `<link>` tag within the HTML head, as follows:

<img width="746" height="97" alt="image" src="https://github.com/user-attachments/assets/1ba3ce29-5ae9-4b8b-92e5-4d700e5ddc94" />

## JavaScript

The same concept applies to JavaScript. It can be internally written between `<script>` elements or written into a separate `.js` file and referenced within the HTML code.

We can see in our HTML source that the `.js` file is referenced externally:

<img width="748" height="63" alt="image" src="https://github.com/user-attachments/assets/bf0d5d33-43cb-4023-bc62-d322ffc1d512" />

We can check out the script by clicking on secret.js, which should take us directly into the script. When we visit it, we see that the code is very complicated and cannot be comprehended:

<img width="736" height="86" alt="image" src="https://github.com/user-attachments/assets/b419a99d-39b4-4641-bbf1-839779171095" />

The reason behind this is code obfuscation. What is it? How is it done? Where is it used?

## 🔑 Key Takeaway

HTML, CSS, and JavaScript make up key parts of a website's client-side code. Inspecting the HTML source can reveal comments and other information, while checking internal and external CSS and JavaScript files helps uncover how the website works. Obfuscated JavaScript may require further analysis to understand its functionality.
