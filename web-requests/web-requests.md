# HTTP Fundamentals


## Structure of a URL


http:// (identifies scheme and protocol)

user:password@ (optional authentication credentials, ends with the @ sign)

amazon.com (hostname and top-level-domain )

:80 (port)

/dashboard.html

?admin=true&authentication=1234 (query string, begins with a ?, consistents of one or more parameters)

#management (identifies fragment/secton of the page)

http://user:password@amazon.com:80/dashboard.html?admin=true&authentication=123#management

The only mandatory part is the protocol/scheme/hostname

http://amazon.com

## Flow of HTTP

Request for a domain checks /etc/hosts, then checks other DNS servers, which can recursively query other DNS servers. By fault, it will try to access the index.html of a website.

## CURL

CURL is a CLI library that supports interacting with web interfaces. Ie 

```curl google.com```, will output the raw contents of the response for google.com.

```curl -0 google.com/index.html``` Saves the contents to a local file index.html We can use -o in place of -0 if we want to specify an exact filename to save to.

```curl google.com/file.png > myFile.png``` Will take the contents of file.png from google's server, and save it to a local file myFile.png. 

## HTTPS

Version of HTTP that encrypts the data in transit, using the TLS/SSL security suite.    

Curl can run into some issues with HTTPS, it will attempt to authenticate, but it may not always do so successfully.

```curl -k https://inlanefreight.com```

Will skip the certificate check, which can cause problems sometimes.

## Responses and Requests

You send a request to a URL, with a particular method and parameters, and get a response back.

Sample request:
```
GET /hello.htm HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: www.tutorialspoint.com
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
```

Sample response:
```
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache/2.2.14 (Win32)
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
Content-Length: 88
Content-Type: text/html
Connection: Closed

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head>
   <title>404 Not Found</title>
</head>
<body>
   <h1>Not Found</h1>
   <p>The requested URL /t.html was not found on this server.</p>
</body>
</html>
```

We can add the -v (Verbose) flag to curl to view the response text.

```curl -k -0 -v https://inlanefreight.com```

## Browser Dev Tools

This allows you to have more significant interaction, reconnasaince, and information gathering on any webpage. You can view cookies, response codes, source code, manipulate elements locally, among other things.

## Headers

![web-requests-1](web-requests-1.png)
![web-requests-2](web-requests-2.png)
![web-requests-3](web-requests-3.png)
# HTTP Methods

