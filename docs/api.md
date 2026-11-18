# bPopup API

bPopup turns an existing element into a modal popup. It does not create or style the popup: you provide the markup and CSS, and bPopup handles centering, the modal overlay, transitions, content loading and close events.

## Requirements

- jQuery >= 1.7 (the plugin uses `.on`/`.off`). Tested with jQuery 1.7, 1.12, 2.2, 3.7 and 4.0; it uses no deprecated jQuery APIs, so jQuery Migrate is not needed. The slim build is not supported: it lacks ajax and effects.
- The popup element should be hidden by your CSS (for example `display: none`). bPopup shows it on open and hides it on close.

## Basic usage

```html
<div id="popup">
    <span class="b-close">X</span>
    Popup content
</div>
```

```javascript
$('#popup').bPopup();
```

## Signature

```javascript
var popup = $(selector).bPopup([options], [callback]);
```

| Argument | Type | Description |
|----------|------|-------------|
| `options` | object | Optional. Overrides `$.fn.bPopup.defaults` for this call. If you pass a function here, it is used as `callback`. |
| `callback` | function | Optional. Called when the open transition completes; `this` is the popup jQuery object. |

The call returns the jQuery object with two extra methods: `close()` and `reposition()` (see [Public methods](#public-methods)).

If the element is already open (it has `data('bPopup')`), calling `bPopup()` on it again does nothing. Keep the object returned by the first call and use it to control the popup.

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `amsl` | `50` | Vertical offset in px above the true center ("above mean sea level"). The top position never goes below 0. |
| `appending` | `true` | Move the popup element into `appendTo` on open. |
| `appendTo` | `'body'` | Selector or element that receives the popup and the modal overlay. |
| `autoClose` | `false` | Close automatically after this many ms. `false` disables it. The timer is cleared if the popup is closed earlier. |
| `closeClass` | `'b-close'` | Clicking any element with this class inside the popup closes it. The legacy class `bClose` also works. |
| `content` | `'ajax'` | How `loadUrl` is loaded: `'ajax'`, `'iframe'` or `'image'`. |
| `contentContainer` | `false` | Selector or element that receives loaded content. Defaults to the popup itself. |
| `easing` | `'swing'` | Easing for the slide transitions. |
| `escClose` | `true` | Close on the Esc key. The Esc `keydown` is stopped at `document` (`stopPropagation`), so `window` listeners do not receive it; other `document` handlers still run, and its default action is not prevented. With several popups open, Esc closes all of them. |
| `follow` | `[true, true]` | `[x, y]`: keep the popup centered on that axis while the page scrolls or resizes. Disabled on iOS 6 because of a scroll event bug there. |
| `followEasing` | `'swing'` | Easing for follow animations. |
| `followSpeed` | `500` | Duration in ms of follow and reposition animations. |
| `iframeAttr` | `'scrolling="no" frameborder="0"'` | Raw attribute string added to the iframe when `content: 'iframe'`. |
| `loadCallback` | `false` | Called when the content loads; `this` is the popup. See [Loading content](#loading-content). |
| `loadData` | `false` | Data sent with the ajax request (passed to jQuery `.load`; an object makes it a POST). |
| `loadUrl` | `false` | URL to load into the popup. If set, bPopup loads content as described by `content`. |
| `modal` | `true` | Show the modal overlay. |
| `modalClose` | `true` | Close when the overlay is clicked. |
| `modalColor` | `'#000'` | Overlay background color. |
| `onClose` | `false` | Called after the close transition completes and the popup is hidden; `this` is the popup. |
| `onOpen` | `false` | Called at the very start of opening, before anything is positioned or shown; `this` is the popup. |
| `opacity` | `0.7` | Overlay opacity. |
| `position` | `['auto', 'auto']` | `[x, y]` in px. A number fixes that axis instead of centering it. |
| `positionStyle` | `'absolute'` | CSS `position` of the popup: `'absolute'` or `'fixed'`. |
| `scrollBar` | `true` | If `false`, sets `overflow: hidden` on `html` while the popup is open, and `overflow: auto` on close. |
| `speed` | `250` | Duration in ms of the open/close transition and of the overlay fade. |
| `transition` | `'fadeIn'` | Open transition: `'fadeIn'`, `'slideDown'`, `'slideUp'`, `'slideIn'` (from the left) or `'slideBack'` (from the right). |
| `transitionClose` | `false` | Close transition, same values as `transition`. `false` means the same transition as on open. |
| `zIndex` | `9997` | Base z-index. With `n` popups open, the overlay gets `zIndex + n` and the popup gets `zIndex + n + 1`. |

### Changing the defaults globally

```javascript
$.fn.bPopup.defaults.speed = 400;
$.fn.bPopup.defaults.modalColor = '#fff';
```

## Public methods

### `close()`

Closes the popup: fades out and removes the overlay, unbinds events, clears the `autoClose` timer and runs the close transition.

```javascript
var popup = $('#popup').bPopup();
popup.close();
```

### `reposition([animateSpeed])`

Recalculates the center position, for example after the content size changes. `animateSpeed` defaults to `followSpeed`. The call is debounced by 50 ms and does nothing on an axis where the popup is larger than the window. With `positionStyle: 'fixed'` the position is set without animation.

```javascript
popup.reposition(100);
```

## Loading content

Set `loadUrl` to load content when the popup opens. The content goes into `contentContainer` (the popup by default), and is removed again on close.

| `content` | What happens | `loadCallback` arguments |
|-----------|--------------|--------------------------|
| `'ajax'` | The URL is loaded with jQuery `.load` into a `div.b-ajax-wrapper`, and the popup is re-centered to fit it. | `status`: `'success'` or `'error'` |
| `'iframe'` | An `iframe.b-iframe` with `iframeAttr` is added, the popup opens, then the iframe `src` is set. | none; called right after `src` is set, not when the page finishes loading |
| `'image'` | An `img` is loaded, and the popup is re-centered to fit it when it loads. | none |

```javascript
$('#popup').bPopup({
    content: 'iframe',
    contentContainer: '.content',
    loadUrl: 'http://example.com',
    iframeAttr: 'scrolling="yes" frameborder="0"'
});

$('#popup').bPopup({
    loadUrl: '/fragment.html',
    loadCallback: function(status) {
        if (status === 'error') { /* handle failure */ }
    }
});
```

## Elements and classes created by bPopup

| Selector | Element |
|----------|---------|
| `.b-modal` | Modal overlay. It also gets a per-popup class `__b-popupN__`. |
| `.b-iframe` | Iframe created for `content: 'iframe'`. |
| `.b-ajax-wrapper` | Wrapper div for `content: 'ajax'`. |

Events are bound in the `.__b-popupN__` namespace and are removed on close.

## Examples

```javascript
// Callback only
$('#popup').bPopup(function() {
    console.log('opened');
});

// Slide down on open, fade on close, close after 5 s
$('#popup').bPopup({
    transition: 'slideDown',
    transitionClose: 'fadeIn',
    speed: 450,
    autoClose: 5000
});

// Fixed position, no overlay, no follow
$('#popup').bPopup({
    modal: false,
    follow: [false, false],
    position: [150, 400],
    positionStyle: 'fixed'
});
```
