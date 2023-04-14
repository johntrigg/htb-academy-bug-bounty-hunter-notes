## Intro To JavaScript Deobfuscation

Style references can be referened externally, or within ```<style>``` tag. Same with Javascript, in ```<script>``` tags.

Code obfuscation is when a piece of code is made more difficult to read from a human perspective. This can be for security (hide code from threat actors) or insecurity (obfucating code so it doesn't trigger alarms)

Minifying is a type of obfuscation that packs everything into one line, and packing moves around variable names to make things more difficult to understand.

https://obfuscator.io/

Javascript itself can be encoded in different types, such as base64, or JSfuck

## Deobfuscation

Some browsers, like firefox browser dev tools, provide means of deobfuscation, such as de-minimizing code. Online tools like 

https://prettier.io/playground/

https://beautifier.io/

http://jsnice.org/

Multiple steps may be required (sending it from prettier/beautifier to jsnice for example)

## Code Analysis

```
'use strict';
/**
 * @return {undefined}
 */
function generateSerial() {
  /** @type {string} */
  var flag = "HTB{1_4m_7h3_53r14l_g3n3r470r!}";
  /** @type {!XMLHttpRequest} */
  var xhr = new XMLHttpRequest;
  /** @type {string} */
  var url = "/serial.php";
  xhr.open("POST", url, true);
  xhr.send(null);
}
;
```

Let's inspect the following code for a moment. There's one function (generateSerial). XMLHTTPRequest requires some research, but it's a javascript function that handles web requests. We also see a url /serial.php. 

We can conclude that it's sending a POST request to the url.

Other than that, it doesn't do anything. It could be uncomplete or unfinished, but we don't know.

So, let's use curl to send a post request to that url.

```curl -s http://139.59.181.223:32007/serial.php -X POST```

If we want to actually send data, we can use

```curl -s http://139.59.181.223:32007/serial.php -X POST -d "param1=sample"```

We can also navigate to the page, capture a request in Burp, and change the method to POST.

## Decoding

The most important thing to do is to identify the type of encoding, and then find tools to decode.

### Base64

3 of the most common text encoding methods are base64, hex, and rot13. 

Base64 is used to reduce special characters, since it's character space is alphanumberic, along with "+", "/", and "=". Since base64 encoded strings in multiples of 4, it uses the = for padding, usually seen at the end of base64 encoding. If 1 space is needed, 1  is present, if two are needed, two = are present, and so on, like so: a===, ab==, abc=, abcd



```echo https://www.hackthebox.eu/ | base64```

```echo aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K | base64 -d```

### Hex

Hex encodes characters into its hex order in ASCII table (a is 61, b is 62, etc). Any hex encoded string will only have hex characters (0-9 and a-f)

```https://www.hackthebox.eu/ | xxd -p```

```68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r```

### Rot13

Caesar cipher, where everything is shifted by 13. We can use cyberchef to decode/encode.

## Skills Exercise

Go to website, deobfuscate the .js file (prettify and jsnice), run it, then put the flag together, and make the API calls as instructed.


