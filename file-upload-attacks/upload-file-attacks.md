# Intro to File Upload Attacks

Occurs when a web application doesn't have proper restrictions on what types of files can be uploaded, allowing the user to upload malicious files, that can lead to PHP code execution.

# Absent Validation

Occurs when there is absolutely no validation on uploaded files. We can just upload malicious files. It's important to identify the framework (IE PHP, nodeJS), so we know what to target.

We can use 

```echo '<?php echo "Hello HTB";?>' > test.php``` 

and upload test.php, to do a trivial test and do a POC.

# Upload Exploitation

```echo "<?php system($_REQUEST['cmd']); ?>" > backdoor.php"```

We can upload a simple 1 liner after testing for the previous POC to see if we can get command execution. If this works, we can also try uploading a straight up reverse shell.

We can use pentestmonkey's reverse shell, or use msfvenom to generate a reverse shell.

# Client Side Verification

If our file does not even make it to the server, there might be some kind of client side verification. The two ways of bypassing this are modifying the upload request, or manipulating the front end cod.

For example, we take our backdoor.php, rename it to backdoor.png, and upload it and intercept the request. Once we intercept the request, we can simply change the extension in the request from .png to .php.

Alternatively, if we can find the code for the input field (in the source code) that restricts malicious uploads, we can simply disable or delete it, and then proceed.

# Blacklist Filters

One type of validation: blacklist (don't allow certain extensions)

Blacklists are weak, and may not be all inclusive. For example, they might not block .phtml, .pHP, .PHP, or other extensions.

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst

Or we can use seclists extensions, and fuzz extensions to see what's accepted.

# Whitelist filters

Another type of validation is whitelist (only allow certain extensions).

A vulnerability exists here

```if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {```

This uses regex to validate the extension: however, it does not verify that it ends with the extension. Therefore, a double extension ```.jpg.php``` bypasses the filter. Strict regex checking might not allow this to work, if there are vulnerability misconfigurations. IE, ```.php.jpg```, a reverse double extension, might allow us to grab the flag.

Special characters can also bypass validation in PHP servers with version 5.x or earlier.

```shell.php%00.jpg``` saves the file as shell.php

```getFlag.phar.jpg``` is our exercise solution, reverse double extension.

# Type Filters

Two headers validate the file content:

```
Content-Type Header or File Content
'image/jpg', 'image/jpeg', 'image/png', 'image/gif'
```

Are some common valid content type headers. This doesn't actually affect the uploaded file in most cases, so it's free to bypass.

However, more complicated filters will check the MIME file type, which is done by checking the first few bytes. An easy example is gif, the first few bytes of a gif file should be ```GIF8```, which we can put before our payload in the burpsuite request. 