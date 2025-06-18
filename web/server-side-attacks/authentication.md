# Authentication

## Password Based

### User Enumeration

While attempting to brute-force a login page, you should pay particular attention to any differences in:

* response status
* response error messages
* response time

### Bypass ip lock

In some implementations, the counter for the number of failed attempts resets if the IP owner logs in successfully.

Or use headers like `X-Forward` with a random number

## MFA

You can try to bypass it by simply changing the url when the mfa is prompted or change the cookie

#### example

request a code for the victim by changing the cookie and then brute force it

***

