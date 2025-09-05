# WebSocket

In principle, practically any web security vulnerability might arise in relation to WebSockets:

* User-supplied input transmitted to the server might be processed in unsafe ways, leading to vulnerabilities such as SQL injection or XML external entity injection.
* Some blind vulnerabilities reached via WebSockets might only be detectable using [out-of-band (OAST) techniques](https://portswigger.net/blog/oast-out-of-band-application-security-testing).
* If attacker-controlled data is transmitted via WebSockets to other application users, then it might lead to XSS or other client-side vulnerabilities.

#### Manipulating WebSocket messages to exploit vulnerabilities <a href="#manipulating-websocket-messages-to-exploit-vulnerabilities" id="manipulating-websocket-messages-to-exploit-vulnerabilities"></a>

For example, suppose a chat application uses WebSockets to send chat messages between the browser and the server. When a user types a chat message, a WebSocket message like the following is sent to the server:

`{"message":"Hello Carlos"}`

The contents of the message are transmitted (again via WebSockets) to another chat user, and rendered in the user's browser as follows:

`<td>Hello Carlos</td>`

In this situation, provided no other input processing or defenses are in play, an attacker can perform a proof-of-concept XSS attack by submitting the following WebSocket message:

`{"message":"<img src=1 onerror='alert(1)'>"}`

#### Manipulating the WebSocket handshake to exploit vulnerabilities <a href="#manipulating-the-websocket-handshake-to-exploit-vulnerabilities" id="manipulating-the-websocket-handshake-to-exploit-vulnerabilities"></a>

Some WebSockets vulnerabilities can only be found and exploited by [manipulating the WebSocket handshake](https://portswigger.net/web-security/websockets#manipulating-websocket-connections). These vulnerabilities tend to involve design flaws, such as:

* Misplaced trust in HTTP headers to perform security decisions, such as the `X-Forwarded-For` header.
* Flaws in session handling mechanisms, since the session context in which WebSocket messages are processed is generally determined by the session context of the handshake message.
* Attack surface introduced by custom HTTP headers used by the application.
