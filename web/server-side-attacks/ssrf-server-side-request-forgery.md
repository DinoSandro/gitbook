---
description: https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet
---

# SSRF - Server Side Request Forgery

#### SSRF with blacklist-based input filters <a href="#ssrf-with-blacklist-based-input-filters" id="ssrf-with-blacklist-based-input-filters"></a>

Some applications block input containing hostnames like `127.0.0.1` and `localhost`, or sensitive URLs like `/admin`. In this situation, you can often circumvent the filter using the following techniques:

* Use an alternative IP representation of `127.0.0.1`, such as `2130706433`, `017700000001`, or `127.1`.
* Register your own domain name that resolves to `127.0.0.1`. You can use `spoofed.burpcollaborator.net` for this purpose.
* Obfuscate blocked strings using URL encoding or case variation.
* Provide a URL that you control, which redirects to the target URL. Try using different redirect codes, as well as different protocols for the target URL. For example, switching from an `http:` to `https:` URL during the redirect has been shown to bypass some anti-SSRF filters.
*   You can embed credentials in a URL before the hostname, using the `@` character. For example:

    `https://expected-host:fakepassword@evil-host`
*   You can use the `#` character to indicate a URL fragment. For example:

    `https://evil-host#expected-host`
*   You can leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control. For example:

    `https://expected-host.evil-host`
* You can URL-encode characters to confuse the URL-parsing code. This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request. You can also try [double-encoding](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings#obfuscation-via-double-url-encoding) characters; some servers recursively URL-decode the input they receive, which can lead to further discrepancies.
* You can use combinations of these techniques together.

```
stockApi=http%3a//127.1:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

#### Bypassing SSRF filters via open redirection <a href="#bypassing-ssrf-filters-via-open-redirection" id="bypassing-ssrf-filters-via-open-redirection"></a>

For example, the application contains an open redirection vulnerability in which the following URL:

`/product/nextProduct?currentProductId=6&path=http://evil-user.net`

```
/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

## Blind SSRF vulnerabilities

Use collaborator
