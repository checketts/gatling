---
title: "WebSocket"
description: "WebSocket support is an extension to the HTTP DSL"
lead: "Learn the possible WebSocket operations with Gatling: connect, close, send"
date: 2021-04-20T18:30:56+02:00
lastmod: 2021-04-20T18:30:56+02:00
weight: 005080
---

WebSocket support was initially contributed by [Andrew Duffy](https://github.com/amjjd).

WebSocket support is an extension to the HTTP DSL, whose entry point is the `ws(requestName: Expression[String])` method.

WebSocket protocol is very different from the HTTP one as the communication is 2 ways: both client-to-server and server-to-client, so the model is different from the HTTP request/response pair.

As a consequence, the main HTTP branch and a WebSocket branch can exist in a Gatling scenario in a dissociated way, in parallel.
When doing so, each flow branch has its own state, so a user might have to reconcile them, for example when capturing data from a WebSocket check and wanting this data to be available to the HTTP branch.

## Common operations

If you want to deal with several WebSockets per virtual users, you have to give them a name and pass this name on each ws operation:

`wsName(name: String)`

For example:

{{< include-code "WsSample.scala#wsName" scala >}}

If you set an explicit name for the WebSocket, you'll have to make it explicit for every other WebSocket actions you'll define later in the scenario.

Of course, this step is not required if you deal with one single WebSocket per virtual user.

### Connect

The first thing is to connect a WebSocket:

`connect(url: Expression[String])`

For example:

{{< include-code "WsSample.scala#wsConnect" scala >}}

You can specify a subprotocol:

{{< include-code "WsSample.scala#subprotocol" scala >}}

You can define a chain of actions to be performed after (re-)connecting with `onConnected`:

{{< include-code "WsSample.scala#onConnected" scala >}}

### Close

When you're done with a WebSocket, you can close it:

`close`

For example:

{{< include-code "WsSample.scala#close" scala >}}

### Send a Message

You may send text or binary messages:

* `sendText(text: Expression[String])`
* `sendBytes(bytes: Expression[Array[Byte]])`

For example:

{{< include-code "WsSample.scala#sendText" scala >}}

Note that:

* `ElFileBody`, `PebbleStringBody` and `PebbleFileBody` implement `Expression[String]` so they can be passed to `sendText`
* `RawFileBody` and `ByteArrayBody` implement `Expression[Array[Byte]]` so they can be passed to `sendBytes`.

See [HTTP request body]({{< ref "../request#request-body" >}}) for more information.

## Server Messages: Checks

Gatling currently only supports blocking checks that will waiting until receiving expected message or timing out.

### Set a Check

You can set a check right after connecting:

{{< include-code "WsSample.scala#check-from-connect" scala >}}

Or you can set a check right after sending a message to the server:

{{< include-code "WsSample.scala#check-from-message" scala >}}

You can set multiple checks sequentially. Each one will expect one single frame.

You can configure multiple checks in a single sequence:

{{< include-code "WsSample.scala#check-single-sequence" scala >}}

You can also configure multiple check sequences with different timeouts:

{{< include-code "WsSample.scala#check-multiple-sequence" scala >}}

### Create a check

You can create checks for text and binary frames with `checkTextMessage` and `checkBinaryMessage`.
You can use almost all the same check criteria as for HTTP requests.

{{< include-code "WsSample.scala#create-single-check" scala >}}

You can have multiple criteria for a given message:

{{< include-code "WsSample.scala#create-multiple-checks" scala >}}

checks can be marked as `silent`.
Silent checks won't be reported whatever their outcome.

{{< include-code "WsSample.scala#silent-check" scala >}}

### Matching messages

You can define `matching` criteria to filter messages you want to check.
Matching criterion is a standard check, except it doesn't take `saveAs`.
Non-matching messages will be ignored.

{{< include-code "WsSample.scala#matching" scala >}}

## Configuration

Websocket support introduces new HttpProtocol parameters:

`wsBaseUrl(url: String)`: similar to standard `baseUrl` for HTTP, serves as root that will be prepended to all relative WebSocket urls

`wsBaseUrls(urls: String*)`: similar to standard `baseUrls` for HTTP, serves as round-robin roots that will be prepended to all relative WebSocket urls

`wsReconnect`: automatically reconnect a WebSocket that would have been closed by someone else than the client.

`wsMaxReconnects(max: Int)`: set a limit on the number of times a WebSocket will be automatically reconnected

`wsAutoReplyTextFrame(f: PartialFunction[String, String])`: configure auto reply for specific WebSocket text messages. Example: `wsAutoReplyTextFrame({ case "ping" => "pong" })` will automatically reply with message `"pong"` when message `"ping"` is received. Those messages won't be visible in any reports or statistics.

`wsAutoReplySocketIo4`: enable partial support for Engine.IO v4 - Gatling will automatically respond to server ping messages (`2`) with pong (`3`). Cannot be used together with `wsAutoReplyTextFrame`.

## Debugging

In your logback configuration, lower logging level to `DEBUG` on logger `io.gatling.http.action.ws.fsm`:

```xml
<logger name="io.gatling.http.action.ws.fsm" level="DEBUG" />
```

## Example

Here's an example that runs against [Play 2.2](https://www.playframework.com/download#older-versions)'s chatroom sample (beware that this sample is missing from Play 2.3 and above):

{{< include-code "WsSample.scala#chatroom-example" scala >}}
