# XSS-CSRF

## TL;DR
- View Page Source - CTRL + U
- Developer Tools - F12

## Types
### Stored (Persistent) XSS
- The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)
### Reflected (Non-Persistent) XSS
- Requires user to visit link
- Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message).
- Reflected XSS vulnerabilities occur when our input reaches the back-end server and gets returned to us without being filtered or sanitized.
- Reflected XSS is the simplest variety of cross-site scripting. It arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.
### DOM-based XSS
- - Requires user to visit link
- DOM-based XSS (also known as DOM XSS) arises when an application contains some client-side JavaScript that processes data from an untrusted source in an unsafe way, usually by writing the data back to the DOM.
- Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)
- While reflected XSS sends the input data to the back-end server through HTTP requests, DOM XSS is completely processed on the client-side through JavaScript. DOM XSS occurs when JavaScript is used to change the page source through the Document Object Model (DOM).
- No HTTP requests are being made
- We usualy see that the input parameter in the URL is using a hashtag # for the item we added, which means that this is a client-side parameter that is completely processed on the browser. This indicates that the input is being processed at the client-side through JavaScript and never reaches the back-end; hence it is a DOM-based XSS.
- To further understand the nature of the DOM-based XSS vulnerability, we must understand the concept of the Source and Sink of the object displayed on the page. The Source is the JavaScript object that takes the user input, and it can be any input parameter like a URL parameter or an input field, as we saw above. On the other hand, the Sink is the function that writes the user input to a DOM Object on the page. If the Sink function does not properly sanitize the user input, it would be vulnerable to an XSS attack.

## XSS Testing Payloads
- ```<script>alert(window.origin)</script>```
## General Notes
- What is the difference between XSS and CSRF? XSS involves causing a web site to return malicious JavaScript, while CSRF involves inducing a victim user to perform actions they do not intend to do.
- What is the difference between XSS and SQL injection? XSS is a client-side vulnerability that targets other application users, while SQL injection is a server-side vulnerability that targets the application's database.
- Unfortunately, there's a slight hitch if you use Chrome. From version 92 onward (July 20th, 2021), cross-origin iframes are prevented from calling alert(). As these are used to construct some of the more advanced XSS attacks, you'll sometimes need to use an alternative PoC payload. In this scenario, we recommend the print() function.
- As XSS attacks execute JavaScript code within the browser, they are limited to the browser's JS engine (i.e., V8 in Chrome). They cannot execute system-wide JavaScript code to do something like system-level code execution. In modern browsers, they are also limited to the same domain of the vulnerable website. Nevertheless, being able to execute JavaScript in a user's browser may still lead to a wide variety of attacks, as mentioned above. In addition to this, if a skilled researcher identifies a binary vulnerability in a web browser (e.g., a Heap overflow in Chrome), they can utilize an XSS vulnerability to execute a JavaScript exploit on the target's browser, which eventually breaks out of the browser's sandbox and executes code on the user's machine.
- Many modern web applications utilize cross-domain IFrames to handle user input, so that even if the web form is vulnerable to XSS, it would not be a vulnerability on the main web application. This is why we are showing the value of window.origin in the alert box, instead of a static value like 1. In this case, the alert box would reveal the URL it is being executed on, and will confirm which form is the vulnerable one, in case an IFrame was being used.
- Some of the commonly used JavaScript functions to write to DOM objects are:
  - document.write()
  - DOM.innerHTML
  - DOM.outerHTML
- Furthermore, some of the jQuery library functions that write to DOM objects are:
  - add()
  - after()
  - append()
- Identifying advanced XSS vulnerabilities requires advanced code review skills. The most reliable method of detecting XSS vulnerabilities is manual code review, which should cover both back-end and front-end code. If we understand precisely how our input is being handled all the way until it reaches the web browser, we can write a custom payload that should work with high confidence.
-  XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in HTTP headers like the Cookie or User-Agent (i.e., when their values are displayed on the page).
-  We are unlikely to find any XSS vulnerabilities through payload lists or XSS tools for the more common web applications. This is because the developers of such web applications likely run their application through vulnerability assessment tools and then patch any identified vulnerabilities before release. For such cases, manual code review may reveal undetected XSS vulnerabilities, which may survive public releases of common web applications.
-  It would be wise to try running our HTML code locally to see how it looks and to ensure that it runs as expected, before we commit to it in our final payload.
-  JavaScript functions that allow changing raw text of HTML fields, like:
  - DOM.innerHTML
  - DOM.outerHTML
  - document.write()
  - document.writeln()
  - document.domain
- And the following jQuery functions:
  - html()
  - parseHTML()
  - add()
  - append()
  - prepend()
  - after()
  - insertAfter()
  - before()
  - insertBefore()
  - replaceAll()
  - replaceWith()
- As these functions write raw text to the HTML code, if any user input goes into them, it may include malicious JavaScript code, which leads to an XSS vulnerability.

## Tooling
- Nessus, Burp Pro, ZAP all have capabilities for detecting all 3 types of XSS vulnerabilties. These scanners usually do two types of scanning: A Passive Scan, which reviews client-side code for potential DOM-based vulnerabilities, and an Active Scan, which sends various types of payloads to attempt to trigger an XSS through payload injection in the page source.
- Such tools usually work by identifying input fields in web pages, sending various types of XSS payloads, and then comparing the rendered page source to see if the same payload can be found in it, which may indicate a successful XSS injection.
- Some of the common open-source tools that can assist us in XSS discovery are XSS Strike, Brute XSS, and XSSer.

## Defacing 
- Three HTML elements are usually utilized to change the main look of a web page:
  - Background Color document.body.style.background
  - Background document.body.background
  - Page Title document.title
  - 
## Prevention
- In addition to input validation, we should always ensure that we do not allow any input with JavaScript code in it, by escaping any special characters. For this, we can utilize the DOMPurify JavaScript library
- we should always ensure that we never use user input directly within certain HTML tags
  - Page Text DOM.innerHTML
- We should have XSS prevention measures on the back-end as well. This can be achieved with Input and Output Sanitization and Validation, Server Configuration, and Back-end Tools that help prevent XSS vulnerabilities.
- In general, effectively preventing XSS vulnerabilities is likely to involve a combination of the following measures:
  - Filter input on arrival. At the point where user input is received, filter as strictly as possible based on what is expected or valid input.
  - Encode data on output. At the point where user-controllable data is output in HTTP responses, encode the output to prevent it from being interpreted as active content. Depending on the output context, this might require applying combinations of HTML, URL, JavaScript, and CSS encoding.
  - Use appropriate response headers. To prevent XSS in HTTP responses that aren't intended to contain any HTML or JavaScript, you can use the Content-Type and X-Content-Type-Options headers to ensure that browsers interpret the responses in the way you intend.
  - Content Security Policy. As a last line of defense, you can use Content Security Policy (CSP) to reduce the severity of any XSS vulnerabilities that still occur.
- How do I prevent XSS in PHP? Filter your inputs with a whitelist of allowed characters and use type hints or type casting. Escape your outputs with htmlentities and ENT_QUOTES for HTML contexts, or JavaScript Unicode escapes for JavaScript contexts.
- How do I prevent XSS in Java? Filter your inputs with a whitelist of allowed characters and use a library such as Google Guava to HTML-encode your output for HTML contexts, or use JavaScript Unicode escapes for JavaScript contexts.

## Server Configuration
In addition to the above, there are certain back-end web server configurations that may help in preventing XSS attacks, such as:
  - Using HTTPS across the entire domain.
  - Using XSS prevention headers.
  - Using the appropriate Content-Type for the page, like X-Content-Type-Options=nosniff.
  - Using Content-Security-Policy options, like script-src 'self', which only allows locally hosted scripts.
  - Using the HttpOnly and Secure cookie flags to prevent JavaScript from reading cookies and only transport them over HTTPS.
    
- In addition to the above, having a good Web Application Firewall (WAF) can significantly reduce the chances of XSS exploitation, as it will automatically detect any type of injection going through HTTP requests and will automatically reject such requests. Furthermore, some frameworks provide built-in XSS protection, like ASP.NET.

## What can XSS be used for?
An attacker who exploits a cross-site scripting vulnerability is typically able to:

- Impersonate or masquerade as the victim user.
- Carry out any action that the user is able to perform.
- Read any data that the user is able to access.
- Capture the user's login credentials.
- Perform virtual defacement of the web site.
- Inject trojan functionality into the web site.

## Impact of XSS 
- In a brochureware application, where all users are anonymous and all information is public, the impact will often be minimal.
- In an application holding sensitive data, such as banking transactions, emails, or healthcare records, the impact will usually be serious.
- If the compromised user has elevated privileges within the application, then the impact will generally be critical, allowing the attacker to take full control of the vulnerable application and compromise all users and their data.

## How to find and test for XSS vulnerabilities
- Manually testing for reflected and stored XSS normally involves submitting some simple unique input (such as a short alphanumeric string) into every entry point in the application, identifying every location where the submitted input is returned in HTTP responses, and testing each location individually to determine whether suitably crafted input can be used to execute arbitrary JavaScript. In this way, you can determine the context in which the XSS occurs and select a suitable payload to exploit it.
- Manually testing for DOM-based XSS arising from URL parameters involves a similar process: placing some simple unique input in the parameter, using the browser's developer tools to search the DOM for this input, and testing each location to determine whether it is exploitable. However, other types of DOM XSS are harder to detect. To find DOM-based vulnerabilities in non-URL-based input (such as document.cookie) or non-HTML-based sinks (like setTimeout), there is no substitute for reviewing JavaScript code, which can be extremely time-consuming.
- Burp Suite's web vulnerability scanner combines static and dynamic analysis of JavaScript to reliably automate the detection of DOM-based vulnerabilities.

## Content Security Policy
- Content security policy (CSP) is a browser mechanism that aims to mitigate the impact of cross-site scripting and some other vulnerabilities. If an application that employs CSP contains XSS-like behavior, then the CSP might hinder or prevent exploitation of the vulnerability. Often, the CSP can be circumvented to enable exploitation of the underlying vulnerability.

## Dangling markup injection
- Dangling markup injection is a technique that can be used to capture data cross-domain in situations where a full cross-site scripting exploit is not possible, due to input filters or other defenses. It can often be exploited to capture sensitive information that is visible to other users, including CSRF tokens that can be used to perform unauthorized actions on behalf of the user.

