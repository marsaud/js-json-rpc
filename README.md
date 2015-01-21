# json-rpc

> Easy to use JSON-RPC library

## Installation

Installation of the [npm package](https://npmjs.org/package/json-rpc):

```
> npm install --save json-rpc
```

Then require the package:

```javascript
var jsonRpc = require('json-rpc');
```

## Usage

1. [Errors](#errors)
2. [Server](#server)
3. [Parsing](#parsing)
4. [Formatting](#formatting)

### Errors

This is the base error for all JSON-RPC errors:

```javascript
var JsonRpcError = require('./errors').JsonRpcError;
```

These are the specific errors:

```javascript
var InvalidJson = require('./errors').InvalidJson;
var InvalidRequest = require('./errors').InvalidRequest;
var MethodNotFound = require('./errors').MethodNotFound;
var InvalidParameters = require('./errors').InvalidParameters;
var UnknownError = require('./errors').UnknownError;
```

Custom errors can of course be created, they just have to inherit
`JsonRpcError`:

```javascript
function MyError() {
  MyError.super_.call(this, 'my error', 1);
}
require('util').inherits(MyError, JsonRpcError);
```

### Server

This library provides a high-level server implementation which should
be flexible enough to use in any environments.

#### Construction

```javascript
var server = jsonRpc.createServer(
  function onMessage(message) {
    // Here is the main handler where every incoming
    // notification/request message goes.
    //
    // For a request, this function just has to throw an exception or
    // return a value to send the related response.
    //
    // If the response is asynchronous, just return a promise.
  },
  function onSend(message) {
    // Here is the function where message to emit to the client goes.
    //
    // If the emission is asynchronous, it should return a promise.

    // Example:
    client.send(JSON.stringify(message));
  }
);

// When a message is received from the client, it must be injected in
// the server through this method.
client.on('message', function (message) {
  // Example:
  server.exec(message);
});
```

#### Notification

```javascript
server.notify('foo', ['bar']);
```

#### Request

The `request()` method returns a promise which will be resolved or
rejected when the response will be received.

```javascript
server.request('add', [1, 2]).then(function (result) {
  console.log(result);
}).catch(function (error) {
  console.error(error.message);
});
```

### Parsing

The `parse()` function parse JSON-RPC messages. These message can be
either JS objects or JSON strings (they will be parsed automatically).

This function may throws:

- `InvalidJson`: if the string cannot be parsed as a JSON;
- `InvalidRequest`: if the message is not a valid JSON-RPC message.

```javascript
jsonRpc.parse('{"jsonrpc":"2.0", "method": "foo", "params": ["bar"]}');
// → {
//   type: 'notification',
//   jsonrpc: '2.0',
//   method: 'foo',
//   params: ['bar']
// }

jsonRpc.parse('{"jsonrpc":"2.0", "id": 0, "method": "add", "params": [1, 2]}');
// → {
//   type: 'request',
//   jsonrpc: '2.0',
//   id: 0,
//   method: 'add',
//   params: [1, 2]
// }

jsonRpc.parse('{"jsonrpc":"2.0", "id": 0, "result": 3}');
// → {
//   type: 'response',
//   jsonrpc: '2.0',
//   id: 0,
//   result: 3
// }
```

### Formatting

The `format*()` functions can be used to create valid JSON-RPC message
in the form of JS objects. It is up to you to format them in JSON if
necessary.

#### Notification

```javascript
jsonRpc.formatNotification('foo', ['bars']);
// → {
//   jsonrpc: '2.0',
//   method: 'foo',
//   params: ['bar']
// }
```

The last argument, the parameters of the notification is optional and
defaults to `[]`.

#### Request

The second argument, the parameters of the notification is optional and
defaults to `[]`.

The last argument, the identifier of the request is optional and is
generated if missing via an increment.

```javascript
jsonRpc.formatRequest('add', [1, 2], 0);
// → {
//   jsonrpc: '2.0',
//   id: 0,
//   method: 'add',
//   params: [1, 2]
// }
```

#### Response

A successful response:

```javascript
jsonRpc.formatResponse(0, 3);
// → {
//   jsonrpc: '2.0',
//   id: 0,
//   result: 3
// }
```

A failed response:

```javascript
var MethodNotFound = require('json-rpc/errors').MethodNotFound;

jsonRpc.formatError(0, new MethodNotFound('add'));
// → {
//   jsonrpc: '2.0',
//   id: 0,
//   error: {
//     code: -3601,
//     message: 'method not found: add',
//     data: 'add'
//   }
// }
```

Note: the error to format must be an instance of `JsonRpcError` or it
will be automatically replaced by an instance of `UnknownError` for
security reasons.

## Contributions

Contributions are *very* welcomed, either on the documentation or on
the code.

You may:

- report any [issue](https://github.com/julien-f/js-json-rpc/issues)
  you've encountered;
- fork and create a pull request.

## License

ISC © [Julien Fontanet](http://julien.isonoe.net)

