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

