
## Burp SQL Injection via bypassing XML encoding


<@hex_entities>1 
UNION SELECT username || '-' || password FROM users
<@/hex_entities>

## Lab: Reflected XSS into HTML context with nothing encoded

This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

To solve the lab, perform a cross-site scripting attack that calls the alert function. 

Manipulate the request entry. Rather than entering typical information enter:

<script>alert('XSS');</script>

## Lab: Stored XSS into HTML context with nothing encoded
 
 This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the alert function when the blog post is viewed. 

Since this webpage allows for comments that are expected to me non-malicious the comments are stored on the webpage for everyone who views it. Enter the same script as the last lab:

<script>alert('XSS');</script>


## Lab: DOM XSS in `document.write` sink using source

This lab contains a [DOM-based cross-site scripting](https://portswigger.net/web-security/cross-site-scripting/dom-based) vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.

To solve this lab, perform a [cross-site scripting](https://portswigger.net/web-security/cross-site-scripting) attack that calls the `alert` function.

* Web page is using a tracking pixel

`"><svg onload=alert(1)>`

enter this^ into the field to use the first closing bracket as a bypass in that it closes the first html img tag and adds a new line of html

<img src="/resources/images/tracker.gif?searchTerms=zinger">

