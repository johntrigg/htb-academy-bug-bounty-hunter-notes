## Intro To Web Proxies

Proxies are often used to manipulate requests. They go inbetween two network nodes, such as taking traffic from a client, and sending it to a server from the proxy. An example of this is burpsuite, which is very useful for web application testing.

## OWASP ZAP

Is a free and open source alternative to Burpsuite, with theoretically more functionality.

## Setting up

Burpsuite comes pre-installed with most relevant distros, and can be installed with package managers. ZAP is similar, and uses the zaproxy command.

## Proxy Setup

Use foxyproxy extension to set up a proxy on firefox, and point burpsuite proxy to the relevant IP (usually loophack) and port.

In both cases, you need to install the Burpsuite certificate for the proxy to streamline the process, and make the browser more acceptable to the proxy's interference.

## Intercepting Web Requests

Boot up Burp, turn on foxyproxy proxy, ensure proper configurations on both ends, and turn on intercept in burpsuite proxy, or leave it off and go to burpsuite HTTP history.

For Zap, it's the green button in the top right to toggle intercept.

Manipulating web requests help with making a lot of pentesting easier.

## Intercepting responses

Can go to HTTP history in Burp, or use repeater.

In Zap, we can go step by step.

## Automatic Modification

We can use burp proxy>options>find and replace to easily modify headers like user agents.

ZAP replacer functions similarly

## Repeating Requests

Burpsuite repeaer is a good option if you want to repeatedly try modifying a request.

## Encoding/Decoding

URLs are very picky about what characters they want in a link, so they use encoding to help with input validation. Some of the types of encoding supported are HTML, unicode, base64, ASCII hex

## Proxying Tools

Proxychains - edit /etc/proxychains.conf to have proxies,

Metasploit - set proxies flag -

## Fuzzing with Burp

We use intruder for fuzzing, but it's a lot slower than ffuf. We specify payload location, content, etc

