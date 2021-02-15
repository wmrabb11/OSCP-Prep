# XSS Guide
# Based on [this article from OWASP](https://owasp.org/www-community/attacks/xss/)

### Overview
Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. XSS attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end user. Flaws that allow these attacks to succeed are quite widespread and occur anywhere a web application uses input from a user within the ouptut it generates without validating or encoding it.


### Types

##### Stored
Stored attacks are those where the injected script is permanently stored on the target servers (like in a database). The victim then retrieves the malicious script from the server when it requests the stored information. Also known as persistent or Type-I XSS.

##### Reflected
Reflected attacks are those where the injected script is reflected off the web server, such as in an error message, search result, or any other response that includes some or all of the input sent to the server as a part of the request. Reflected attacks are delivered to victims via another route, such as in an e-mail message, or on some other website. When a user is tricked into clicking on a malicious link, submitting a specially crafted form, or even just browsing to a malicious site, the injected code travels to the vulnerable web site, which reflects the attack back to the user's browser. The browser then executes the code because it came from a "trusted" server. Reflected XSS is also known as Non-Persistent or Type-II XSS.


### Defense and Prevention [(Link)](https://portswigger.net/web-security/cross-site-scripting)
  - Filter input on arrival. At the point where user input is received, filter as strictly as possible based on what is expected or valid input
  - Encode data on output. At the point where user-controllable data is output in HTTP responses, encode the output to prevent it from being interpreted as active content. Depending on the output context, this might require applying combinations of HTML, URL, JS, and CSS encoding
  - Use appropriate response headers. To prevent XSS in HTTP responses that aren't intended to contain any HTML or JS, you can use the Content-Type and X-Content-Type-Options headers to ensure that browsers interpret the responses in the way you intend
  - Content Security Policy. As a last line of defense, you can use CSP to reduce the severity of any XSS vulns that may occur


### Google XSS Games [(Link)](https://xss-game.appspot.com)

##### Level 1
  - can be solved by simply typing "<script>alert()</script>" into the text box and submitting
  - this works because there is no sanitization of input, allowing our input to be rendered as HTML on the page and therefore calling our script

##### Level 2
  - looking at the code, we see our message is surrounded by "<blockquote>" tags
  - `<img src="#" onerror=alert('XSS')>`
  - "Execution Sink": some common JS functions are execution sinks which means that they will cause the browser to execute any scripts that appear in their input. Sometimes this fact is hidden by higher-level APIs which use one of these functions under the hood

##### Level 3
  - XSS going to be in the URL because "/level3/frame#3" -> "/level3/frame#2" -> the page is changing based on what we put in the URL
  - looking at the source, in the "chooseTab" function
  - when the HTML string is being created, there's no "parseInt" called when constructing the img URL
  - and the path is "<img src='/static/level3/cloud" + num + ".jpg' />" -> this means we can stop the src string with a single tick and whatever we add after that will be rendered as HTML
  - `https://xss-game.appspot.com/level3/frame#0' onerror=alert()>`

##### Level 4
  - templating used (i.e. `onload="startTimer('{{ timer }}')"`)
  - "timer" is a GET parameter -> we can control it
  - the "startTimer" function gets called in the "onload" event of the loading gif
  - make sure we close the startTimer call and leave room for the `')` that will be appended to our input and then loaded onto the page
  - `3');alert('XSS`

##### Level 5
  - on the signup page, `<a href="{{ next }}">Next</a>`
  - next is a GET parameter -> we can control it
  - found [this](https://www.developsec.com/2017/09/06/javascript-in-an-href-or-src-attribute/) article on using javascript in href tags
  - "HREF supports the javascript: prefix. That will allow us to bypass the current encodings we have performed. This is because with using the javascript: prefix, we are not trying to break out of the HREF attribute. We don't need to break out of the double quotes to create another attribute."
  - change the url in the target window to `https://xss-game.appspot.com/level5/frame/signup?next=javascript:alert('XSS')` and click "next"

##### Level 6
  - loads in a gadget, based on the "window.location.hash" element (after the pound sign)
  - then sets the script source to be the specified URL, there is a check to see if "http" or "https" is in the URL, so we can't use that
  - looking at the hints, there's reference to `google.com/jsapi?callback=foo`, which we can use to call a JS function
  - because the filter isn't case-sensitive we can use `htTps://google.com/jsapi?callback=alert`
