---
layout: layout.njk
title: </> htmx - hx-vals
---

## `hx-vals`

The `hx-vals` attribute allows you to add to the parameters that will be submitted with an AJAX request.  

By default, the value of this attribute is a list of name-expression values in [JSON (JavaScript Object Notation)](https://www.json.org/json-en.html) 
format.

If you wish for `hx-vals` to *evaluate* the values given, you can prefix the values with `javascript:` or `js:`.

```html
  <div hx-get="/example" hx-vals='{"myVal": "My Value"}'>Get Some HTML, Including A Value in the Request</div>

  <div hx-get="/example" hx-vals='js:{myVal: calculateValue()}'>Get Some HTML, Including a Dynamic Value from Javascript in the Request</div>
```

### Security Considerations

* By default, the value of `hx-vals` must be valid [JSON](https://developer.mozilla.org/en-US/docs/Glossary/JSON). 
  It is **not** dynamically computed.  If you use the `javascript:` prefix, be aware that you are introducing
   security considerations, especially when dealing with user input such as query strings or user-generated content, 
   which could introduce a [Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) vulnerability. 

### Notes

* `hx-vals` is inherited and can be placed on a parent element.
* A child declaration of a variable overrides a parent declaration.
* Input values with the same name will be overridden by variable declarations.
