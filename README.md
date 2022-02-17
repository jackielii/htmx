# See [HTMX base repo](https://github.com/bigskysoftware/htmx)

# Changes in this fork
## Fixes
- Fix outerHTML swap if parent is null (i.e. too fast clicking for example) : an error would be fired trying to access the parent's properties and block further interactions with the element => now ignored when parent is null
- The [`checked`](https://developer.mozilla.org/en/docs/Web/HTML/Element/Input/checkbox#checked) attribute is used instead of the [`value`](https://developer.mozilla.org/en/docs/Web/HTML/Element/Input/checkbox#value) attribute for checkbox inputs, when building the request payload, which can then correctly be parsed as a boolean by the backend
- Unchecked checkboxes aren't excluded anymore from request payload _(I needed this for PATCH requests for example)_
- Fix target and source elements sent to `selectAndSwap` during the **SSE** swapping process that were inverted in the original code, now allowing to properly use `hx-target` for example to specify which element to swap with SSE
- [SSE swap listeners](https://htmx.org/attributes/hx-sse/) are now removed from the event sources when the associated element gets removed from the page
## Changes
- If the target specified to [`htmx.ajax()`](https://htmx.org/api/#ajax) (and so the `targetOverride` passed to `issueAjaxRequest()`) can't be found, an error is thrown (instead of the HTMX default implementation that crawls the hierarchy to find the first `hx-target` specified on a parent element as a fallback)
- In the default HTMX implementation, elements that have an ID have a special treatment during swapping ; if an element with an ID is about to be inserted, and to replace another element with the same ID, for a reason I ignore, HTMX clones the old elements "settling attributes" _(that are, by default, `class`, `style`, `width` & `height`)_ into the new element, then restores the new element initial attributes after the settling. My problem with this ; using a [MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver) to apply style to newly added nodes _(for example a line that should be positionned at current local time with a line of JS in a calendar)_ results in those modifications being reverted by HTMX after the settling. Since this cloning step appears useless to me (maybe am I wrong though ?), I got rid of it here, as well as the `attributesToSettle` property that was only used for that purpose.
- When an HTMX error is logged, the `detail` context object passed along in the internal functions is now included in the log, for an easier debugging. This way, a `hx-targetError` for example, will also log the actual target that HTMX was trying to retrieve.
- The attribute `hx-push-url` is no longer inherited by default by children elements. You can specify `hx-push-url="inherit"` on children elements though, to inherit the value from the closest parent that defines a `hx-push-url`.
- HTMX uses `writeLayout` and `readLayout` _(more details below)_ internally to postpone the elements clean up & settling [to the next animation frame](https://developer.mozilla.org/fr/docs/Web/API/Window/requestAnimationFrame), so the swap function returns faster _(now takes roughly half the time)_ and the browser can update the DOM _(and the page layout display)_ sooner, providing a better visual experience.
- When using the [`hx-swap`](https://htmx.org/attributes/hx-swap/) attribute set to `"none"`, it's no longer required to have a value defined for the [`hx-target`](https://htmx.org/attributes/hx-target/) attribute _(in this case, simply no element will trigger the [htmx:beforeSwap](https://htmx.org/events/#htmx:beforeSwap) event, nor being added/removed the [swapping class](https://htmx.org/reference/#classes))_
## Additions
- `hx-error-target` and `hx-error-swap` attributes to allow swapping server's reponse on error (i.e request with a `status >= 300`). Those attributes behave exactly like, respectively, [hx-target](https://htmx.org/attributes/hx-target/) and [hx-swap](https://htmx.org/attributes/hx-swap/)
- `hx-sse-swap` attribute to override `hx-swap` attribute's value for SSE swapping, ie to allow an element to have different swapping behaviours for requests responses & for SSE. Please note that `hx-swap` is used as a fallback in case `hx-sse-swap` is not defined, so this attribute should only be used when necessary, ie to split swapping behaviours apart. 
- The [HX-Trigger](https://htmx.org/headers/hx-trigger/) header in server's response supports a comma-and-space-separated event names list, to send multiple events to trigger without data, and without having to use a JSON format. Sending for example the header `HX-Trigger: myEvent, myOtherEvent` would trigger both the events `myEvent` and `myOtherEvent` on the client
- The event sources array ([SSE](https://htmx.org/attributes/hx-sse/)) is now exposed in the htmx API, so client can add eventListeners to the same sources created by htmx with the [hx-sse](https://htmx.org/attributes/hx-sse/) attribute
- [`htmx.ajax()`](https://htmx.org/api/#ajax) supports an additional property `swap` in the `context` argument, to override the swapping behaviour. This argument must be a string and follows the syntax of the attribute [`hx-swap`](https://htmx.org/attributes/hx-swap/)
- The [`htmx:beforeOnLoad`](https://htmx.org/events/#htmx:beforeOnLoad), [`htmx:beforeSwap`](https://htmx.org/events/#htmx:beforeSwap), [`htmx:afterSwap`](https://htmx.org/events/#htmx:afterSwap), [`htmx:afterSettle`](https://htmx.org/events/#htmx:afterSettle), [`htmx:swapError`](https://htmx.org/events/#htmx:swapError), [`htmx:afterRequest`](https://htmx.org/events/#htmx:afterRequest), [`htmx:afterOnLoad`](https://htmx.org/events/#htmx:afterOnLoad), [`htmx:onLoadError`](https://htmx.org/events/#htmx:onLoadError) events are now passed an additional property `isError` in the event's `detail` property, which indicates if the request's response has an error status _(ie a code >= 400)_
- The `hx-target` attribute accepts 2 new values : `next-sibling` (which targets the [next sibling element](https://developer.mozilla.org/en-US/docs/Web/API/Element/nextElementSibling)) and `previous-sibling` (which targets the [previous sibling element](https://developer.mozilla.org/en-US/docs/Web/API/Element/previousElementSibling))
- The HTMX API now provides the methods `readLayout` and `writeLayout`, to execute layout read/write operations while avoiding [layout thrashing](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing#avoid_layout_thrashing)
- The HTMX API now provides the method `resizeSelect`, which takes a [select HTML element](https://developer.mozilla.org/fr/docs/Web/HTML/Element/select) as parameter, and resizes it so it fits its selected option
