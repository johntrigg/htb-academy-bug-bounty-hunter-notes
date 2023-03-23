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

```curl -A "custom-user-agent" -v https://inlanefreight.com```
# HTTP Methods

# Methods and Codes

Methods are used to specify what a request does: Grabbing a webpage, posting data, etc. Response codes are used to specify how the request was handled: satisfied, wrong request, redirect, etc.

![web-requests-4](web-requests-4.png)

## GET 

GET requests grab a webpage. Sometimes, they can require authentication.

```curl -u admin:admin amazon.com:7070```

```curl admin:admin@amazon.com:7070```

Tries to grab the webpage with basic authentication admin:admin. We can use -u or specify it in the url. These are stored in the authorization header, which is a base64 encoded value of the credentials (admin:admin).

You can grab requests in devtools, copy them ot a curl command, and execute the curl command in your terminal. This is convenient.

```curl -v "http://admin:admin@167.172.52.160:32532/search.php?search=flag"```

Covers basic auth, post, and most of what we've covered.

## POST

POST requests are usually used to move significant amounts of data to a server, such as files. They require less logging, less encoding, and less overhead.

To solve the challenge here, we login with admin:admin to grab a valid cookie ID. We then specify a few other options: it's a post request, with a content type json header, and a json in {"search":"flag"},

```curl -X POST -d '{"search":"flag"}' -b 'PHPSESSID=p62bamllihv2kard82pv3qntmr' -H 'Content-Type: application/json' http://209.97.134.177:30734/search.php -v```

## API/CRUD API

An API, or application programming interface, APIs are primarily uses to interact with a database. 

The CRUD API can perform various operations: Create, Read, Update, Delete, and so on.

We can use curl to submit API requests.

```curl http://<SERVER_IP>:<PORT>/api.php/city/london```

Will tell api.php to look in the "database" for cities, and return the "entry" for London. We can pipe this command to jq, such that the data it returns is loaded properly

```curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq```

```
[
  {
    "city_name": "London",
    "country_name": "(UK)"
  }
]
```

We can also mess with the last part of the request, such as changing "london" to "lon", which will return every city with "lon" in the name.

Most importantly, passing an empty string will return the entire table.

```curl -s http://<SERVER_IP>:<PORT>/api.php/city/ | jq```

In the same vein, we can use this to upload data to the table like so:

```curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ -d '{"city_name":"HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'```

We can also update and delete entries.

The goal here is to change a city name to flag, then delete a city, then read a city with name flag.

```curl -X PUT "http://188.166.154.164:31042/api.php/city/derby" -d '{"city_name":"flag"}' -H 'Content-Type: application/json'```

```curl -X DELETE "http://188.166.154.164:31042/api.php/city/Manchester"```

```curl -s "http://188.166.154.164:31042/api.php/city/" -v | grep flag```