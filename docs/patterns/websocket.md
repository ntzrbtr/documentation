---
title: WebSocket Plugin - Elysia.js
head:
  - - meta
    - property: 'title'
      content: WebSocket Plugin - Elysia.js

  - - meta
    - name: 'description'
      content: Plugin for Elysia that add support for using WebSocket. Start by registering the plugin and declare WebSocket route with "ws".

  - - meta
    - name: 'og:description'
      content: Plugin for Elysia that add support for using WebSocket. Start by registering the plugin and declare WebSocket route with "ws".
---

# WebSocket
Elysia WebSocket is extending Bun WebSocket which use [uWebSocket](https://github.com/uNetworking/uWebSockets) under the hood.

To start using WebSocket, register the WebSocket plugin, and start declaring WebSocket route with `.ws`.
```typescript
import { Elysia, ws } from 'elysia'

new Elysia()
    .use(ws())
    .ws('/ws', {
        message(ws, message) {
            ws.send(message)
        }
    })
    .listen(8080)
```

WebSocket route is extending Bun WebSocket configuration, for additional configuration please refers to [Bun's WebSocket](https://bun.sh/docs/api/websockets).

Just like normal route, WebSocket also accepts **schema** to strictly type and validate requests.

## Configuration
This plugin extends config from [Bun WebSocket](https://github.com/oven-sh/bun#websockets-with-bunserve)

Below is a config extends from [Bun WebSocket](https://github.com/oven-sh/bun#websockets-with-bunserve)

### perMessageDeflate
@default `false`

Enable compression for clients that support it. 

By default, compression is disabled.

### maxPayloadLength
The maximum size of a message.

### idleTimeout
@default `120`

After a connection has not received a message for this many seconds, it will be closed.

### backpressureLimit
@default `16777216` (16MB)

The maximum number of bytes that can be buffered for a single connection.

### closeOnBackpressureLimit
@default `false`

Close the connection if the backpressure limit is reached.

## Method
Below are the new methods register by the plugin

## ws
Create websocket handler

Example:
```typescript
import { Elysia, ws } from 'elysia'

new Elysia()
    .use(ws())
    .ws('/ws', {
        message(ws, message) {
            ws.send(message)
        }
    })
    .listen(8080)
```

Type:
```typescript
.ws(endpoint: path, options: Partial<WebSocketHandler<Context>>): this
```

endpoint: A path to exposed as websocket handler 
options: Customize WebSocket handler behavior

## WebSocketHandler
WebSocketHandler extends config from [config](#config).

Below is a config which accepted by `ws`.

## schema
Validatation for an incoming WebSocket request.

- headers: validate headers before upgrade to WebSocket
- params: validate path paramters
- query: validate query parameters
- body: validate websocket message
- response: validate websocket response

::: tip
It's recommended to use query parameters instead of path parameters in WebSocket, as parsing path parameters is expensive and sometime unrealiable for multiple data with long value.
:::

## open
Callback function for new websocket connection.

Type:
```typescript
open(ws: ServerWebSocket<{
    // uid for each connection
    id: string
    data: Context
}>): this
```

## message
Callback function for incoming websocket message.

Type:
```typescript
message(
    ws: ServerWebSocket<{
        // uid for each connection
        id: string
        data: Context
    }>, 
    message: Message
): this
```

`Message` type based on `schema.message`. Default is `string`.

## close
Callback function for closing websocket connection.

Type:
```typescript
close(ws: ServerWebSocket<{
    // uid for each connection
    id: string
    data: Context
}>): this
```

## drain
Callback function for the server is ready to accept more data.

Type:
```typescript
drain(
    ws: ServerWebSocket<{
        // uid for each connection
        id: string
        data: Context
    }>, 
    code: number,
    reason: string
): this
```

## parse
`Parse` middleware to parse the request before upgrading the HTTP connection to WebSocket.

## beforeHandle
`Before Handle` middleware which execute before upgrading the HTTP connection to WebSocket.

Ideal place for validation.

## transform
`Transform` middleware which execute before validation.

## transformMessage
Like `transform`, but execute before validation of WebSocket message

## header
Additional headers to add before upgrade connection to WebSocket.

## Pattern
Below you can find the common patterns to use the plugin.

## WebSocket message validation:
Validate incoming WebSocket message.

By default Elysia will parse incoming stringified JSON message as Object for validation.

```typescript
import { Elysia, ws } from 'elysia'

new Elysia()
    .use(ws())
    .ws('/ws', {
        schema: {
            // validate incoming message
            body: t.Object({
                message: t.String()
            })
        },
        message(ws, { message }) {
            ws.send({
                message,
                time: Date.now()
            })
        }
    )
    .listen(8080)
```
