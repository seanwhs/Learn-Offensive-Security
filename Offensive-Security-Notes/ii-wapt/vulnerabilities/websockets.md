# WebSockets

WebSocket connections are long-lived HTTP initiations allowing bidirectional messaging. The connection stays open until a message is sent by the client or server. WebSocket is ideal for low-latency and server-triggered messages, such as real-time financial data feeds.

<details>

<summary>How are WebSocket connections established?</summary>

WebSocket connections are normally created using client-side JavaScript like the following:

```javascript
var ws = new WebSocket("wss://normal-website.com/chat");
```

The `wss` protocol establishes a WebSocket over an encrypted TLS connection, while the `ws` protocol uses an unencrypted connection.

To establish the connection, the browser and server perform a WebSocket handshake via HTTP. The browser sends a WebSocket handshake request like this:

```http
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

```http
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
```

</details>

<details>

<summary>Headers</summary>

* The `Connection` and `Upgrade` headers in the request and response indicate that this is a WebSocket handshake.

<!---->

* The `Sec-WebSocket-Version` request header specifies the WebSocket protocol version that the client wishes to use. This is typically 13.

<!---->

* The `Sec-WebSocket-Key` request header contains a Base64-encoded random value, which should be randomly generated in each handshake request.

<!---->

* The `Sec-WebSocket-Accept` response header contains a hash of the value submitted in the Sec-WebSocket-Key request header, concatenated with a specific string defined in the protocol specification. This is done to prevent misleading responses resulting from misconfigured servers or caching proxies.

</details>

<details>

<summary>What do WebSocket messages look like?</summary>

* WebSocket messages can contain any content or data format

```javascript
ws.send("Peter Wiener");
```

* It is common to use json

```json
{"user":"Hal Pline","content":"Hello"}
```

</details>

## <mark style="color:yellow;">Manipulating WebSocket connections</mark>

To do ...

## <mark style="color:yellow;">WebSockets vulnerabilities</mark>

* If inputs are transmitted and processed server-side
  * Server-side attacks (SQLi, XXE, etc.)
* If attacker-controlled data is transmitted via WebSockets to other application users
  * Client-side attacks (XSS, etc.)
    * Example if the content of a message is transmitted to another user (via chat...)
    * `{"message":"<img src=1 onerror='alert(1)'>"}`
* Also blind vulnerabilities

## <mark style="color:yellow;">Cross-site WebSocket hijacking</mark>

An attacker can craft a malicious webpage on their domain, initiating a cross-site WebSocket connection to the susceptible application.

* Perform unauthorized actions masquerading as the victim user (like CSRF)
* Retrieve sensitive data that the user can access.
  * Cross-site WebSocket hijacking grants the attacker bidirectional access to the vulnerable application via the hijacked WebSocket. If the application utilizes server-generated WebSocket messages to send sensitive user data, the attacker can intercept these messages and capture the victim user's data.
* Waiting for incoming messages to arrive containing sensitive data.



