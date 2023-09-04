express-sse
============

## Fork Differences

This project was forked from [dpskvn/express-sse](https://github.com/dpskvn/express-sse/tree/master) and introduces the following changes:

### Heartbeat/ping event

Adds support for a `ping` event that can be used to keep idle connections alive through some networking middleware like haproxy.

Specify `options.pingInterval` with the desired heartbeat interval in ms.

The heartbeat event name will default to `ping` unless overridden with `options.pingEvent`.

```
new SSE([], { pingInterval: 20_000, pingEvent: 'thump' })
```

This approach can be utilized from the frontend to also detect stale connections (expect a ping event every N seconds.)

An alternative approach to this would be to emit a [comment](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event_stream_format), which is not currently supported by this library.

### Change initial event behavior

The original initial event implementation had the following unwanted behavior: When a new client connected, the initial event would be re-broadcast to all connected clients.

This has been changed to the expected approach of broadcasting the initial event only to new clients.

### Make initial event optional

The initial event defaulted to `[]` when not defined. This implementation changes the default to be `undefined` and not being broadcast unless explicitly specified.

### Retry support

SSE supports a `retry` field which instructs the client to change the default reconnection timeout.

Set `options.retry` to a value in milliseconds and the first event emitted will include a retry header field.

```
new SSE([], { retry: 5_000 });
```

### Omit IDs option

The constructor options support `.noID` to skip assigning IDs to events that do not explicitly have one.

An Express middleware for quick'n'easy server-sent events.

### Support for express without compression

The `res.flush` call is made optional, which enabled support for express without the `compression` middleware.

### Error handling while setting headers

The header setting parts that can throw have been wrapped in a try/catch

---

The original README follows with the installation links replaced.

---

## About
`express-sse` is meant to keep things simple. You need to send server-sent events without too many complications and fallbacks? This is the library to do so.

## Installation:
`npm install --save @takadenoshi/express-sse`

or

`yarn add @takadenoshi/express-sse`

## Usage example:
### Options:
You can pass an optional options object to the constructor. Currently it only supports changing the way initial data is treated. If you set `isSerialized` to `false`, the initial data is sent as a single event. The default value is `true`.

```js
var sse = new SSE(["array", "containing", "initial", "content", "(optional)"], { isSerialized: false, initialEvent: 'optional initial event name' });
```

### Server:
```js
var SSE = require('@takadenoshi/express-sse');
var sse = new SSE(["array", "containing", "initial", "content", "(optional)"]);

...

app.get('/stream', sse.init);

...

sse.send(content);
sse.send(content, eventName);
sse.send(content, eventName, customID);
sse.updateInit(["array", "containing", "new", "content"]);
sse.serialize(["array", "to", "be", "sent", "as", "serialized", "events"]);
```

### Client:
```js
var es = new EventSource('/stream');

es.onmessage = function (event) {
  ...
};

es.addEventListener(eventName, function (event) {
  ...
});
```
