---
layout: layout.njk
title: </> htmx - Documentation
---
<div class="row">
<div class="2 col nav">

**Contents**

<div id="contents">

* [introduction](#introduction)
* [installing](#installing)
* [ajax](#ajax)
  * [triggers](#triggers)
    * [trigger filters](#trigger-filters)
    * [trigger modifiers](#trigger-modifiers)
    * [special events](#special-events)
    * [polling](#polling)
    * [load polling](#load_polling)
  * [targets](#targets)
  * [indicators](#indicators)
  * [swapping](#swapping)
  * [css transitions](#css_transitions)
  * [parameters](#parameters)
  * [confirming](#confirming)
* [inheritance](#inheritance)
* [boosting](#boosting)
* [websockets & SSE](#websockets-and-sse)
* [history](#history)
* [requests & responses](#requests)
* [validation](#validation)
* [animations](#animations)
* [extensions](#extensions)
* [events & logging](#events)
* [debugging](#debugging)
* [hyperscript](#hyperscript)
* [3rd party integration](#3rd-party)
* [security](#security)
* [configuring](#config)

</div>

</div>
<div class="10 col">

## <a name="introduction"></a>[Htmx in a Nutshell](#introduction)

Htmx is a library that allows you to access modern browser features directly from HTML, rather than using
javascript.

To understand htmx, first lets take a look at an anchor tag:

``` html
  <a href="/blog">Blog</a>
```

This anchor tag tells a browser:

> "When a user clicks on this link, issue an HTTP GET request to '/blog' and load the response content
>  into the browser window".

With that in mind, consider the following bit of HTML:

``` html
  <button hx-post="/clicked"
       hx-trigger="click"
       hx-target="#parent-div"
       hx-swap="outerHTML">
    Click Me!
  </button>
```

This tells htmx:

> "When a user clicks on this button, issue an HTTP POST request to '/clicked' and use the content from the response
>  to replace the element with the id `parent-div` in the DOM"

Htmx extends and generalizes the core idea of HTML as a hypertext, opening up many more possibilities directly
within the language:

* Now any element, not just anchors and forms, can issue an HTTP request
* Now any event, not just clicks or form submissions, can trigger requests
* Now any [HTTP verb](https://en.wikipedia.org/wiki/HTTP_Verbs), not just `GET` and `POST`, can be used
* Now any element, not just the entire window, can be the target for update by the request

Note that when you are using htmx, on the server side you typically respond with *HTML*, not *JSON*.  This keeps you firmly
within the [original web programming model](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm),
using [Hypertext As The Engine Of Application State](https://en.wikipedia.org/wiki/HATEOAS)
without even needing to really understand that concept.

It's worth mentioning that, if you prefer, you can use the `data-` prefix when using htmx:

``` html
  <a data-hx-post="/click">Click Me!</a>
```

## <a name="installing"></a> [Installing](#installing)

Htmx is a dependency-free javascript library.

It can be used via [NPM](https://www.npmjs.com/) as "`htmx.org`" or downloaded or included from
[unpkg](https://unpkg.com/browse/htmx.org/) or your other favorite NPM-based CDN:

``` html
    <script src="https://unpkg.com/htmx.org@1.6.0"></script>
```

For added security, you can load the script using [Subresource Integrity (SRI)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity).

``` html
    <script src="https://unpkg.com/htmx.org@1.6.0" integrity="sha384-G4dtlRlMBrk5fEiRXDsLjriPo8Qk5ZeHVVxS8KhX6D7I9XXJlNqbdvRlp9/glk5D" crossorigin="anonymous"></script>
```

## <a name="ajax"></a> [AJAX](#ajax)

The core of htmx is a set of attributes that allow you to issue AJAX requests directly from HTML:

| Attribute | Description |
|-----------|-------------|
| [hx-get](/attributes/hx-get) | Issues a `GET` request to the given URL|
| [hx-post](/attributes/hx-post) | Issues a `POST` request to the given URL
| [hx-put](/attributes/hx-put) | Issues a `PUT` request to the given URL
| [hx-patch](/attributes/hx-patch) | Issues a `PATCH` request to the given URL
| [hx-delete](/attributes/hx-delete) | Issues a `DELETE` request to the given URL


Each of these attributes takes a URL to issue an AJAX request to.  The element will issue a request of the specified
type to the given URL when the element is [triggered](#triggers):

```html
  <div hx-put="/messages">
    Put To Messages
  </div>
```

This tells the browser:

> When a user clicks on this div, issue a PUT request to the URL /messages and load the response into the div

### <a name="triggers"></a> [Triggering Requests](#triggers)

By default, AJAX requests are triggered by the "natural" event of an element:

* `input`, `textarea` & `select` are triggered on the `change` event
* `form` is triggered on the `submit` event
* everything else is triggered by the `click` event

If you want different behavior you can use the [hx-trigger](/attributes/hx-trigger)
attribute to specify which event will cause the request.

Here is a `div` that posts to `/mouse_entered` when a mouse enters it:

```html
   <div hx-post="/mouse_entered" hx-trigger="mouseenter">
      [Here Mouse, Mouse!]
   </div>
```

#### <a name="trigger-modifiers"></a> [Trigger Modifiers](#trigger-modifiers)

A trigger can also have a few additional modifiers that change its behavior.  For example, if you want a request to only
 happen once, you can use the `once` modifier for the trigger:

```html
   <div hx-post="/mouse_entered" hx-trigger="mouseenter once">
     [Here Mouse, Mouse!]
   </div>
```

Other modifiers you can use for triggers are:

* `changed` - only issue a request if the value of the element has changed
*  `delay:<time interval>` - wait the given amount of time (e.g. `1s`) before
issuing the request.  If the event triggers again, the countdown is reset.
*  `throttle:<time interval>` - wait the given amount of time (e.g. `1s`) before
issuing the request.  Unlike `delay` if a new event occurs before the time limit is hit the event will be discarded,
so the request will trigger at the end of the time period.
*  `from:<CSS Selector>` - listen for the event on a different element.  This can be used for things like keyboard shortcuts.

You can use these attributes to implement many common UX patterns, such as [Active Search](/examples/active-search):

```html
   <input type="text" name="q"
          hx-get="/trigger_delay"
          hx-trigger="keyup changed delay:500ms"
          hx-target="#search-results"
          placeholder="Search..."/>
    <div id="search-results"></div>
```

This input will issue a request 500 milliseconds after a key up event if the input has been changed and inserts the results
into the `div` with the id `search-results`.

Multiple triggers can be specified in the [hx-trigger](/attributes/hx-trigger) attribute, separated by commas.

#### <a name="trigger-filters"></a> [Trigger Filters](#trigger-filters)

You may also apply trigger filters by using square brackets after the event name, enclosing a javascript expression that
will be evaluated.  If the expression evaluates to `true` the event will trigger, otherwise it will not.

Here is an example that triggers only on a Control-Click of the element

```html
<div hx-get="/clicked" hx-trigger="click[ctrlKey]">Control Click Me</div>
```

Properties like `ctrlKey` will be resolved against the triggering event first, then the global scope.

#### <a name="special-events"></a> [Special Events](#special-events)

htmx provides a few special events for use in [hx-trigger](/attributes/hx-trigger):

* `load` - fires once when the element is first loaded
* `revealed` - fires once when an element first scrolls into the viewport
* `intersect` - fires once when an element first intersects the viewport.  This supports two additional options:
    * `root:<selector>` - a CSS selector of the root element for intersection
    * `threshold:<float>` - a floating point number between 0.0 and 1.0, indicating what amount of intersection to fire the event on

You can also use custom events to trigger requests if you have an advanced use case.

#### <a name="polling"></a> [Polling](#polling)

If you want an element to poll the given URL rather than wait for an event, you can use the `every` syntax
with the [`hx-trigger`](/attributes/hx-trigger/) attribute:

```html
  <div hx-get="/news" hx-trigger="every 2s">
  </div>
```

This tells htmx

> Every 2 seconds, issue a GET to /news and load the response into the div

If you want to stop polling from a server response you can respond with the HTTP response code [`286`](https://en.wikipedia.org/wiki/86_(term))
and the element will cancel the polling.

#### <a name="load_polling"></a> [Load Polling](#load_polling)

Another technique that can be used to achieve polling in htmx is "load polling", where an element specifies
an `load` trigger along with a delay, and replaces itself with the response:

```html
<div hx-get="/messages"
     hx-trigger="load delay:1s"
     hx-swap="outerHTML">

</div>
```

If the `/messages` end point keeps returning a div set up this way, it will keep "polling" back to the URL every
second.

Load polling can be useful in situations where a poll has an end point at which point the polling terminates, such as
when you are showing the user a [progress bar](/examples/progress-bar).

### <a name="indicators"></a> [Request Indicators](#indicators)

When an AJAX request is issued it is often good to let the user know that something is happening since the browser
will not give them any feedback.  You can accomplish this in htmx by using `htmx-indicator` class.

The `htmx-indicator` class is defined so that the opacity of any element with this class is 0 by default, making it invisible
but present in the DOM.

When htmx issues a request, it will put a `htmx-request` class onto an element (either the requesting element or
another element, if specified).  The `htmx-request` class will cause a child element with the `htmx-indicator` class
on it to transition to an opacity of 1, showing the indicator.

```html
  <button hx-get="/click">
      Click Me!
     <img class="htmx-indicator" src="/spinner.gif"/>
  </button>
```

Here we have a button.  When it is clicked the `htmx-request` class will be added to it, which will reveal the spinner
gif element.  (I like [SVG spinners](http://samherbert.net/svg-loaders/) these days.)

While the `htmx-indicator` class uses opacity to hide and show the progress indicator, if you would prefer another mechanism
you can create your own CSS transition like so:

```css
    .htmx-indicator{
        display:none;
    }
    .htmx-request .my-indicator{
        display:inline;
    }
    .htmx-request.my-indicator{
        display:inline;
    }
```

If you want the `htmx-request` class added to a different element, you can use the [hx-indicator](/attributes/hx-indicator)
attribute with a CSS selector to do so:

```html
  <div>
      <button hx-get="/click" hx-indicator="#indicator">
        Click Me!
      </button>
      <img id="indicator" class="htmx-indicator" src="/spinner.gif"/>
  </div>
```

Here we call out the indicator explicitly by id.  Note that we could have placed the class on the parent `div` as well
and had the same effect.

### <a name="targets"></a> [Targets](#targets)

If you want the response to be loaded into a different element other than the one that made the request, you can
use the  [hx-target](/attributes/hx-target) attribute, which takes a CSS selector.  Looking back at our Live Search example:

```html
   <input type="text" name="q"
          hx-get="/trigger_delay"
          hx-trigger="keyup delay:500ms changed"
          hx-target="#search-results"
          placeholder="Search..."/>
    <div id="search-results"></div>
```

You can see that the results from the search are going to be loaded into `div#search-results`, rather than into the
input tag.

### <a name="swapping"></a> [Swapping](#swapping)

htmx offers a few different ways to swap the HTML returned into the DOM.  By default, the content replaces the
`innerHTML` of the target element.  You can modify this by using the [hx-swap](/attributes/hx-swap) attribute
with any of the following values:

| Name | Description
|------|-------------
| `innerHTML` | the default, puts the content inside the target element
| `outerHTML` | replaces the entire target element with the returned content
| `afterbegin` | prepends the content before the first child inside the target
| `beforebegin` | prepends the content before the target in the targets parent element
| `beforeend` | appends the content after the last child inside the target
| `afterend` | appends the content after the target in the targets parent element
| `none` | does not append content from response (out of band items will still be processed)

#### <a name="css_transitions"></a>[CSS Transitions](#css_transitions)

htmx makes it easy to use [CSS Transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions) without 
javascript.  Consider this HTML content:

```html
  <div id="div1">Original Content</div>
```

Imagine this content is replaced by htmx via an ajax request with this new content:

```html
  <div id="div1" class="red">New Content</div>
```

Note two things: 

* The div has the *same* id in the original an in the new content
* The `red` class has been added to the new content

Given this situation, we can write a CSS transition from the old state to the new state:

```css
.red {
  color: red;
  transition: all ease-in 1s ;
}
```

When htmx swaps in this new content, it will do so in such a way that the CSS transition will apply to the new content,
giving you a nice, smooth transition to the new state.

So, in summary, all you need to do to use CSS transitions for an element is keep its `id` stable across requests!

You can see the [Animation Examples](/examples/animations) for more details and live demonstrations.

##### Details

To understand how CSS transitions actually work in htmx, you must understand the underlying swap & settle model that htmx uses.

When new content is received from a server, before the content is swapped in, the existing
content of the page is examined for elements that match by the `id` attribute.  If a match
is found for an element in the new content, the attributes of the old content are copied
onto the new element before the swap occurs.  The new content is then swapped in, but with the
*old* attribute values.  Finally, the new attribute values are swapped in, after a "settle" delay
(100ms by default).  A little crazy, but this is what allowes CSS transitions to work without any javascript by
the developer.

#### <a name="oob_swaps"></a>[Out of Band Swaps](#oob_swaps)

If you want to swap content from a response directly into the DOM by using the `id` attribute you can use the
[hx-swap-oob](/attributes/hx-swap-oob) attribute in the *response* html:

```html
  <div id="message" hx-swap-oob="true">Swap me directly!</div>
  Additional Content
```

In this response, `div#message` would be swapped directly into the matching DOM element, while the additional content
would be swapped into the target in the normal manner.

You can use this technique to "piggy-back" updates on other requests.

Note that out of band elements must be in the top level of the response, and not children of the top level elements.

#### Selecting Content To Swap

If you want to select a subset of the response HTML to swap into the target, you can use the [hx-select](/attributes/hx-select)
attribute, which takes a CSS selector and selects the matching elements from the response.

#### Preserving Content During A Swap

If there is content that you wish to be preserved across swaps (e.g. a video player that you wish to remain playing
even if a swap occurs) you can use the [hx-preserve](/attributes/hx-preserve)
attribute on the elements you wish to be preserved.

### <a name="parameters"></a> [Parameters](#parameters)

By default, an element that causes a request will include its value if it has one.  If the element is a form it
will include the values of all inputs within it.

Additionally, if the element causes a non-`GET` request, the values of all the inputs of the nearest enclosing form
will be included.

If you wish to include the values of other elements, you can use the [hx-include](/attributes/hx-include) attribute
with a CSS selector of all the elements whose values you want to include in the request.

If you wish to filter out some parameters you can use the [hx-params](/attributes/hx-params) attribute.

Finally, if you want to programatically modify the parameters, you can use the [htmx:configRequest](/events#htmx:configRequest)
event.

#### <a name="files"></a> [File Upload](#files)

If you wish to upload files via an htmx request, you can set the [hx-encoding](/attributes/hx-encoding) attribute to
`multipart/form-data`.  This will use a `FormData` object to submit the request, which will properly include the file
in the request.

Note that depending on your server-side technology, you may have to handle requests with this type of body content very
differently.

Note that htmx fires a `htmx:xhr:progress` event periodically based on the standard `progress` event during upload,
which you can hook into to show the progress of the upload.

#### <a name="extra-values"></a> [Extra Values](#extra-values)

You can include extra values in a request using the [hx-vals](/attributes/hx-vals) (name-expression pairs in JSON format) and
[hx-vars](/attributes/hx-vars) attributes (comma-separated name-expression pairs that are dynamically computed).

### <a name="confirming"></a> [Confirming Requests](#confirming)

Often you will want to confirm an action before issuing a request.  htmx supports the [`hx-confirm`](/attributes/hx-confirm)
attribute, which allows you to confirm an action using a simple javascript dialog:

```html
<button hx-delete="/account" hx-confirm="Are you sure you wish to delete your account?">
  Delete My Account
</button>
```

Using events you can implement more sophisticated confirmation dialogs.  The [confirm example](/examples/confirm/)
shows how to use [sweetalert2](https://sweetalert2.github.io/) library for confirmation of htmx actions.

## <a name="inheritance"></a>[Attribute Inheritance](#inheritance)

Most attributes in htmx are inherited: they apply to the element they are on as well as any children elements.  This
allows you to "hoist" attributes up the DOM to avoid code duplication.  Consider the following htmx:

```html
<button hx-delete="/account" hx-confirm="Are you sure?">
  Delete My Account
</button>
<button hx-put="/account" hx-confirm="Are you sure?">
  Update My Account
</button>
```

Here we have a duplicate `hx-confirm` attribute.  We can hoist this attribute to a parent element:

```html
<div hx-confirm="Are you sure?">
    <button hx-delete="/account">
      Delete My Account
    </button>
    <button hx-put="/account">
      Update My Account
    </button>
</div>
```

This `hx-confirm` attribute will now apply to all htmx-powered elements within it.

Sometimes you wish to undo this inheritance.  Consider if we had a cancel button to this group, but didn't want it to
be confirmed.  We could add an `unset` directive on it like so:

```html
<div hx-confirm="Are you sure?">
    <button hx-delete="/account">
      Delete My Account
    </button>
    <button hx-put="/account">
      Update My Account
    </button>
    <button hx-confirm="unset" hx-get="/">
      Cancel
    </button>
</div>
```

The top two buttons would then show a confirm dialog, but the bottom cancel button would not.

## <a name="boosting"></a>[Boosting](#boosting)

Htmx supports "boosting" regular HTML anchors and forms with the [hx-boost](/attributes/hx-boost) attribute.  This
attribute will convert all anchor tags and forms into AJAX requests that, by default, target the body of the page.

Here is an example:

```html
<div hx-boost="true">
    <a href="/blog">Blog</a>
</div>
```

The anchor tag in this div will issue an AJAX `GET` request to `/blog` and swap the response into the `body` tag.

This functionality is somewhat similar to [Turbolinks](https://github.com/turbolinks/turbolinks) and allows you to use
htmx for [progressive enhancement](https://en.wikipedia.org/wiki/Progressive_enhancement).

### <a name="websockets-and-sse"></a> [Web Sockets & SSE](#websockets-and-sse)

Htmx has experimental support for declarative use of both
[WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)
and  [Server Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events).

These features are under active development, so please let us know if you are willing to experiment with them.

#### <a name="websockets">[WebSockets](#websockets)

If you wish to establish a `WebSocket` connection in htmx, you use the [hx-ws](/attributes/hx-ws) attribute:

```html
  <div hx-ws="connect:wss:/chatroom">
    <div id="chat_room">
      ...
    </div>
    <form hx-ws="send:submit">
        <input name="chat_message">
    </form>
  </div>
```

The `connect` declaration established the connection, and the `send` declaration tells the form to submit values to the
socket on `submit`.

More details can be found on the [hx-ws attribute page](/attributes/hx-ws)

#### <a name="sse"></a> [Server Sent Events](#sse)

[Server Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) are
a way for servers to send events to browsers.  It provides a higher-level mechanism for communication between the
server and the browser than websockets.

If you want an element to respond to a Server Sent Event via htmx, you need to do two things:

1. Define an SSE source.  To do this, add a [hx-sse](/attributes/hx-sse) attribute on a parent element with
a `connect <url>` declaration that specifies the URL from which Server Sent Events will be received.

2. Define elements that are descendents of this element that are triggered by server sent events using the
`hx-trigger="sse:<event_name>"` syntax

Here is an example:

```html
    <body hx-sse="connect:/news_updates">
        <div hx-trigger="sse:new_news" hx-get="/news"></div>
    </body>
```

Depending on your implementation, this may be more efficient than the polling example above since the server would
notify the div if there was new news to get, rather than the steady requests that a poll causes.

## <a name="history"></a> [History Support](#history)

Htmx provides a simple mechanism for interacting with the [browser history API](https://developer.mozilla.org/en-US/docs/Web/API/History_API):

If you want a given element to push its request URL into the browser navigation bar and add the current state of the page
to the browser's history, include the [hx-push-url](/attributes/hx-push-url) attribute:

```html
    <a hx-get="/blog" hx-push-url="true">Blog</a>
```

When a user clicks on this link, htmx will snapshot the current DOM and store it before it makes a request to /blog.
It then does the swap and pushes a new location onto the history stack.

When a user hits the back button, htmx will retrieve the old content from storage and swap it back into the target,
 simulating "going back" to the previous state.  If the location is not found in the cache, htmx will make an ajax
 request to the given URL, with the header `HX-History-Restore-Request` set to true, and expects back the HTML needed
 for the entire page.  Alternatively, if the `htmx.config.refreshOnHistoryMiss` config variable is set to true, it will
 issue a hard browser refresh.

**NOTE:** If you push a URL into the history, you **must** be able to navigate to that URL and get a full page back!
A user could copy and paste the URL into an email, or new tab.  Additionally, htmx will need the entire page when restoring
history if the page is not in the history cache.

### Specifying History Snapshot Element

By default, htmx will use the `body` to take and restore the history snapshot from.  This is usually the right thing, but
if you want to use a narrower element for snapshotting you can use the [hx-history-elt](/attributes/hx-history-elt)
attribute to specify a different one.

Careful: this element will need to be on all pages or restoring from history won't work reliably.

## <a name="requests">[Requests &amp; Responses](#requests)

Htmx expects responses to the AJAX requests it makes to be HTML, typically HTML fragments (although a full HTML
document, matched with a [hx-select](/attributes/hx-select) tag can be useful too).  Htmx will then swap the returned
HTML into the document at the target specified and with the swap strategy specified.

Sometimes you might want to do nothing in the swap, but still perhaps trigger a client side event ([see below](#response-headers)).
For this situation you can return a `204 - No Content` response code, and htmx will ignore the content of the response.

In the event of an error response from the server (e.g. a 404 or a 501), htmx will trigger the [`htmx:responseError`](/events#htmx:responseError)
event, which you can handle.

In the event of a connection error, the `htmx:sendError` event will be triggered.

### <a name="request-header"></a> [Request Headers](#request-headers)

htmx includes a number of useful headers in requests:

| Header | Description
|--------|--------------
| `HX-Request` | will be set to "true"
| `HX-Trigger` | will be set to the id of the element that triggered the request
| `HX-Trigger-Name` | will be set to the name of the element that triggered the request
| `HX-Target` | will be set to the id of the target element
| `HX-Prompt` | will be set to the value entered by the user when prompted via [hx-prompt](/attributes/hx-prompt)

### <a name="response-header"></a> [Response Headers](#response-headers)

htmx supports some htmx-specific response headers:

* `HX-Push` - pushes a new URL into the browser’s address bar
* `HX-Redirect` - triggers a client-side redirect to a new location
* `HX-Refresh` - if set to "true" the client side will do a full refresh of the page
* `HX-Trigger` - triggers client side events
* `HX-Trigger-After-Swap` - triggers client side events after the swap step
* `HX-Trigger-After-Settle` - triggers client side events after the settle step

For more on the `HX-Trigger` headers, see [`HX-Trigger` Response Headers](/headers/hx-trigger).

### <a name="request-operations"></a> [Request Order of Operations](#request-operations)

The order of operations in a htmx request are:

* The element is triggered and begins a request
  * Values are gathered for the request
  * The `htmx-request` class is applied to the appropriate elements
  * The request is then issued asynchronously via AJAX
    * Upon getting a response the target element is marked with the `htmx-swapping` class
    * An optional swap delay is applied (see the [hx-swap](/attributes/hx-swap) attribute)
    * The actual content swap is done
        * the `htmx-swapping` class is removed from the target
        * the `htmx-added` class is added to each new piece of content
        * the `htmx-settling` class is applied to the target
        * A settle delay is done (default: 100ms)
        * The DOM is settled
        * the `htmx-settling` class is removed from the target
        * the `htmx-added` class is removed from each new piece of content

You can use the `htmx-swapping` and `htmx-settling` classes to create
[CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions) between pages.

## <a name="validation">[Validation](#validation)

Htmx integrates with the [HTML5 Validation API](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation)
and will not issue a request if a validatable input is invalid.  This is true for both AJAX requests as well as
WebSocket sends.

Htmx fires events around validation that can be used to hook in custom validation and error handling:

* `htmx:validation:validate` - called before an elements `checkValidity()` method is called.  May be used to add in
   custom validation logic
* `htmx:validation:failed` - called when `checkValidity()` returns false, indicating an invalid input
* `htmx:validation:halted` - called when a request is not issued due to validation errors.  Specific errors may be found
  in the `event.details.errors` object

### Validation Example

Here is an example of an input that uses the `htmx:validation:validate` event to require that an input have the value
`foo`, using hyperscript:

```html
<form hx-post="/test">
  <input _="on htmx:validation:validate
               if my.value != 'foo'
                  call me.setCustomValidity('Please enter the value foo')
               else
                  call me.setCustomValidity('')"
         name="example">
</form>
```

Note that all client side validations must be re-done on the server side, as they can always be bypassed.

## <a name="animations"></a> [Animations](#animations)

Htmx allows you to use [CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)
in many situations using only HTML and CSS.

Please see the [Animation Guide](/examples/animations) for more details on the options available.

## <a name="extensions"></a> [Extensions](#extensions)

Htmx has an extension mechanism that allows you to customize the libraries' behavior.  Extensions [are
defined in javascript](/extensions#defining) and then used via the [`hx-ext`](/attributes/hx-ext) attribute:

```html
<div hx-ext="debug">
  <button hx-post="/example">This button used the debug extension</button>
  <button hx-post="/example" hx-ext="ignore:debug">This button does not</button>
</div>
```

If you are interested in adding your own extension to htmx, please [see the extension docs](/extensions)

### Included Extensions

Htmx includes some extensions that are tested against the htmx code base.  Here are a few:

| Extension | Description
|-----------|-------------
| [`json-enc`](/extensions/json-enc) | use JSON encoding in the body of requests, rather than the default `x-www-form-urlencoded`
| [`morphdom-swap`](/extensions/morphdom-swap) | an extension for using the [morphdom](https://github.com/patrick-steele-idem/morphdom) library as the swapping mechanism in htmx.
| [`client-side-templates`](/extensions/client-side-templates) | support for client side template processing of JSON responses
| [`path-deps`](/extensions/path-deps) | an extension for expressing path-based dependencies [similar to intercoolerjs](http://intercoolerjs.org/docs.html#dependencies)
| [`class-tools`](/extensions/class-tools) | an extension for manipulating timed addition and removal of classes on HTML elements

See the [extensions page](/extensions#included) for a complete list.

## <a name="events"></a> [Events & Logging](#events)

Htmx has an extensive [events mechanism](https://htmx.org/reference/#events), which doubles as the logging system.

If you want to register for a given htmx event you can use

```js
   document.body.addEventListener('htmx:load', function(evt) {
      myJavascriptLib.init(evt.details.elt);
   });
```
 
or, if you would prefer, you can use the following htmx helper:

```javascript
  htmx.on("htmx:load", function(evt) {
        myJavascriptLib.init(evt.details.elt);
  });
```

The `htmx:load` event is fired every time an element is loaded into the DOM by htmx, and is effectively the equivalent 
 to the normal `load` event.  

Some common uses for htmx events are:

## <a name="init_3rd_party_with_events">[Initialize A 3rd Party Library With Events](#init_3rd_party_with_events)

Using the `htmx:load` event to initialize content is so common that htmx provides a helper function:

```javascript
  htmx.onLoad(function(target) {
        myJavascriptLib.init(target);
  });
```
This does the same thing as the first example, but is a little cleaner.

## <a name="config_request_with_events">[Configure a Request With Events](#config_request_with_events)

You can handle the [`htmx:configRequest`](/events/#htmx:configRequest) event in order to modify an AJAX request before it is issued:

```javascript
document.body.addEventListener('htmx:configRequest', function(evt) {
    evt.detail.parameters['auth_token'] = getAuthToken(); // add a new parameter into the request
    evt.detail.headers['Authentication-Token'] = getAuthToken(); // add a new header into the request
});
```

Here we add a parameter and header to the request before it is sent.

## <a name="modifying_swapping_behavior_with_events">[Modifying Swapping Behavior With Events](#modifying_swapping_behavior_with_events)

You can handle the [`htmx:beforeSwap`](/events/#htmx:beforeSwap) event in order to modify the swap behavior of htmx:

```javascript
document.body.addEventListener('htmx:beforeSwap', function(evt) {
    if(evt.detail.xhr.status === 404){
        // alert the user when a 404 occurs (maybe use a nicer mechanism than alert())
        alert("Error: Could Not Find Resource");
    } else if(evt.detail.xhr.status === 422){
        // allow 422 responses to swap as we are using this as a signal that
        // a form was submitted with bad data and want to rerender with the
        // errors
        evt.detail.shouldSwap = true;
    } else if(evt.detail.xhr.status === 418){
        // if the response code 418 (I'm a teapot) is returned, retarget the
        // content of the response to the element with the id `teapot`
        evt.detail.shouldSwap = true;        
        evt.detail.target = htmx.find("#teapot");
    }
});
```

Here we handle a few [400-level error response codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) 
that would normally not do a swap in htmx.

## <a name="event_naming">[Event Naming](#event_naming)

Note that all events are fired with two different names

* Camel Case
* Kebab Case

So, for example, you can listen for `htmx:afterSwap` or for `htmx:after-swap`.  This facilitates interoperability
with other libraries.  [Alpine.js](https://github.com/alpinejs/alpine/), for example, requires kebab case.

### Logging

If you set a logger at `htmx.logger`, every event will be logged.  This can be very useful for troubleshooting:

```javascript
    htmx.logger = function(elt, event, data) {
        if(console) {
            console.log(event, elt, data);
        }
    }
```

## <a name="debugging"></a> [Debugging](#debugging)

Declarative and event driven programming with htmx (or any other declartive language) can be a wonderful and highly productive
activity, but one disadvantage when compared with imperative approaches is that it can be tricker to debug.
 
Figuring out why something *isn't* happening, for example, can be difficult if you don't know the tricks.  

Well, here are the tricks:
 
The first debugging tool you can use is the `htmx.logAll()` method.  This will log every event that htmx triggers and
will allow you to see exactly what the library is doing.  

```javascript
  htmx.logAll();
```

Of course, that won't tell you why htmx *isn't* doing something.  You might also not know *what* events a DOM
element is firing to use as a trigger.  To address this, you can use the 
[`monitorEvents()`](https://developers.google.com/web/updates/2015/05/quickly-monitor-events-from-the-console-panel) method available in the
browser console:

```javascript
  monitorEvents(htmx.find("#theElement"));
```

This will spit out all events that are occuring on the element with the id `theElement` to the console, and allow you
to see exactly what is going on with it.

Note that this *only* works from the console, you cannot embed it in a script tag on your page.

Finally, push come shove, you might want to just debug `htmx.js` by loading up the unminimized version.  It's 
about 2500 lines of javascript, so not an insurmountable amount of code.  You would most likely want to set a break
point in the `issueAjaxRequest()` and `handleAjaxResponse()` methods to see what's going on.

And always feel free to jump on the [Discord](https://htmx.org/discord) if you need help.

## <a name="hyperscript"></a>[hyperscript](#hyperscript)

Hyperscript is an experimental front end scripting language designed to be expressive and easily embeddable directly in HTML
for handling custom events, etc.  The language is inspired by [HyperTalk](http://hypercard.org/HyperTalk%20Reference%202.4.pdf),
javascript, [gosu](https://gosu-lang.github.io/) and others.

You can explore the language more fully on its main website:

<http://hyperscript.org>

Hyperscript is *not* required when using htmx, anything you can do in hyperscript can be done in vanilla JS or with
 another javascript library like jQuery, but the two technologies were designed with one another in mind and play
 well together.

### Installing Hyperscript
 
 To use hyperscript in combination with htmx, you need to [install the hyperscript library](https://unpkg.com/browse/hyperscript.org/)
 either via a CDN or locally.  See the [hyperscript website](https://hyperscript.org) for the latest version of the
 library.  
 
 When hyperscript is included, it will automatically integrate with htmx and begin processing all hyperscripts embedded
 in your HTML.

### Events & Hyperscript

Hyperscript was designed to help address features and functionality from intercooler.js that are not implemented in htmx
directly, in a more flexible and open manner.  One of its prime features is the ability to respond to arbitrary events
on a DOM element, using the `on` syntax:

```html
<div _="on htmx:afterSettle log 'Settled!'">
 ...
</div>
```

This will log `Settled!` to the console when the `htmx:afterSettle` event is triggered.

#### intercooler.js features & hyperscript implementations

Below are some examples of intercooler features and the hyperscript equivalent.

##### `ic-remove-after`

Intercooler provided the [`ic-remove-after`](http://intercoolerjs.org/attributes/ic-remove-after.html) attribute
for removing an element after a given amount of time.

In hyperscript you can implement this, as well as fade effect, like so:

```html
<div _="on load wait 5s then transition opacity to 0 then remove me">
  Here is a temporary message!
</div>
```

##### `ic-post-errors-to`

Intercooler provided the [`ic-post-errors-to`](http://intercoolerjs.org/attributes/ic-post-errors-to.html) attribute
for posting errors that occured during requests and responses.

In hyperscript similar functionality is implemented like so:

```html
<body _="on htmx:error(errorInfo) fetch /errors {method:'POST', body:{errorInfo:errorInfo} as JSON} ">
  ...
</body>
```

##### `ic-switch-class`

Intercooler provided the [`ic-switch-class`](http://intercoolerjs.org/attributes/ic-switch-class.html) attribute, which
let you switch a class between siblings.

In hyperscript you can implement similar functionality like so:

```html
<div hx-target="#content" _="on htmx:beforeOnLoad take .active from .tabs for event.target">
    <a class="tabs active" hx-get="/tabl1" >Tab 1</a>
    <a class="tabs" hx-get="/tabl2">Tab 2</a>
    <a class="tabs" hx-get="/tabl3">Tab 3</a>
</div>
<div id="content">Tab 1 Content</div>
```

## <a name="3rd-party"></a>[3rd Party Javascript](#3rd-party)

Htmx integrates fairly well with third party libraries.  If the library fires events on the DOM, you can use those events to
trigger requests from htmx.  

A good example of this is the [SortableJS demo](/examples/sortable/):

```html
<form class="sortable" hx-post="/items" hx-trigger="end">
  <div class="htmx-indicator">Updating...</div>
  <div><input type='hidden' name='item' value='1'/>Item 1</div>
  <div><input type='hidden' name='item' value='2'/>Item 2</div>
  <div><input type='hidden' name='item' value='2'/>Item 3</div>
</form>
```

With Sortable, as with most javascript libraries, you need to initialize content at some point.  

In jquery you might do this like so:

```javascript
$(document).ready(function() {
    var sortables = document.body.querySelectorAll(".sortable");
    for (var i = 0; i < sortables.length; i++) {
      var sortable = sortables[i];
      new Sortable(sortable, {
          animation: 150,
          ghostClass: 'blue-background-class'
      });
    }
});
```

In htmx, you would instead use the `htmx.onLoad` function, and you would select only from the newly loaded content,
rather than the entire document:

```js
htmx.onLoad(function(content) {
    var sortables = content.querySelectorAll(".sortable");
    for (var i = 0; i < sortables.length; i++) {
      var sortable = sortables[i];
      new Sortable(sortable, {
          animation: 150,
          ghostClass: 'blue-background-class'
      });
    }
})
```

This will ensure that as new content is added to the DOM by htmx, sortable elements are properly initialized.

If javascript adds content to the DOM that has htmx attributes on it, you need to make sure that this content 
is initialized with the `htmx.process()` function.

For example, if you were to fetch some data and put it into a div using the `fetch` API, and that HTML had 
htmx attributes in it, you would need to add a call to `htmx.process()` like this:

```ecmascript 6
  let myDiv = document.getElementById('my-div')
  fetch('http://example.com/movies.json')
    .then(response => response.text())
    .then(data => { myDiv.innerHTML = data; htmx.process(myDiv); } );
```

## <a name="security"></a>[Security](#security)

htmx allows you to define logic directly in your DOM.  This has a number of advantages, the
largest being [Locality of Behavior](https://htmx.org/essays/locality-of-behaviour/) making your system 
more coherent.

One concern with this approach, however, is security. This is especially the case if you are injecting user-created
content into your site without any sort of HTML escaping discipline.  

You should, of course, escape all 3rd party untrusted content that is injected into your site to prevent, among other issues, [XSS attacks](https://en.wikipedia.org/wiki/Cross-site_scripting). Attributes starting with `hx-` and `data-hx`, as well as inline `<script>` tags should be filtered.

It is important to understand that htmx does *not* require inline scripts or `eval()` for most of its features. You (or your security team) may use a [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) that intentionally disallows inline scripts and the use of `eval()`. This, however, will have *no effect* on htmx functionality, which will still be able to execute JavaScript code placed in htmx attributes and may be a security concern.

To address this, if you don't want a particular part of the DOM to allow for htmx functionality, you can place the
`hx-disable` or `data-hx-disable` attribute on the enclosing element of that area.  

This will prevent htmx from executing within that area in the DOM:

```html
  <div hx-disable>
    <%= user_content %>
  </div>
```

This approach allows you to enjoy the benefits of [Locality of Behavior](https://htmx.org/essays/locality-of-behaviour/) 
while still providing additional safety if your HTML-escaping discipline fails. 

## <a name="config"></a>[Configuring htmx](#config)

Htmx has some configuration options that can be accessed either programatically or declaratively.  They are
listed below:

<div class="info-table">

| Config Variable | Info |
|-----------------|-------
|  `htmx.config.historyEnabled` | defaults to `true`, really only useful for testing
|  `htmx.config.historyCacheSize` | defaults to 10
|  `htmx.config.refreshOnHistoryMiss` | defaults to `false`, if set to `true` htmx will issue a full page refresh on history misses rather than use an AJAX request
|  `htmx.config.defaultSwapStyle` | defaults to `innerHTML`
|  `htmx.config.defaultSwapDelay` | defaults to 0
|  `htmx.config.defaultSettleDelay` | defaults to 100
|  `htmx.config.includeIndicatorStyles` | defaults to `true` (determines if the indicator styles are loaded)
|  `htmx.config.indicatorClass` | defaults to `htmx-indicator`
|  `htmx.config.requestClass` | defaults to `htmx-request`
|  `htmx.config.addedClass` | defaults to `htmx-added`
|  `htmx.config.settlingClass` | defaults to `htmx-settling`
|  `htmx.config.swappingClass` | defaults to `htmx-swapping`
|  `htmx.config.allowEval` | defaults to `true`
|  `htmx.config.useTemplateFragments` | defaults to `false`, HTML template tags for parsing content from the server (not IE11 compatible!)
|  `htmx.config.wsReconnectDelay` | defaults to `full-jitter`
|  `htmx.config.disableSelector` | defaults to `[disable-htmx], [data-disable-htmx]`, htmx will not process elements with this attribute on it or a parent
|  `htmx.config.timeout` | defaults to 0 in milliseconds

</div>

You can set them directly in javascript, or you can use a `meta` tag:

```html
    <meta name="htmx-config" content='{"defaultSwapStyle":"outerHTML"}'>
```

### Conclusion

And that's it!  Have fun with htmx: you can accomplish [quite a bit](/examples) without a lot of code.

</div>
</div>
