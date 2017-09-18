# Cross Domain IFrame Communication Example With Origin Verification

This is a sample project to show communication from a child iFrame to the parent window. This sample performs origin verification to demonstrate the ability to restrict malicious third parties from retrieving data from the iframe by wrapping it from another domain.

## Setup

To run this sample, you'll need to:

1. Install [Caddy web server](https://caddyserver.com).
1. Add this line to your hosts file:

    ```
    127.0.0.1	parent.example.com evil-parent.example.com iframe.example.com
    ```

1. Clone this repo.
1. Run `caddy -port 1337` in the root of the repository.

## Use

While the Caddy server is running, visit the working example site: http://parent.example.com:1337/

From this page you can save a storage item, delete the item, and re-send the modification to the parent page. Whenever the page is reloaded or the "Send local storage to parent" button is clicked you should see the data fetched update.

To test a malicious party attempting to wrap your iframe you can perform the same actions by visiting: http://evil-parent.example.com:1337/

## How it Works

The parent window is listening to the `message` event, like so:

```js
window.addEventListener('message', function (event) {
  // Verify the message's origin before trusting it.
  if (event.origin !== 'http://iframe.example.com:1337') {
    // Handle message from an evil domain.
    alert('Message event received from bad origin');
  }
  else {
    // Handle message from a valid domain.
    document.getElementById('dataRetrieved').textContent = event.data;
    document.getElementById('messageOrigin').textContent = event.origin;
  }
});
```

Other windows (an iframe in our case), calls `postMessage()` on the parent `window` object. The [postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) call includes  the origin allowed to view the request for security, in our case http://parent.example.com:1337/. Example:

```js
parent.postMessage('Data to send here', 'http://parent.example.com:1337/');
```

## Code

The code for the iframe is in `iframe/index.html`.

The code for the parent is in `parent/index.html`. Whether the `postMessage()` succeeds or not is dictated by the domain the code is served from.

## Notes

This sample utilizes http, instead of https. The reason this is done is because we are creating domains with self-signed certificates. If you have not explicitly allowed requests to these domains in your browser during the session, the sample would break in unexpected ways. Thus https has been excluded for simplicity.
