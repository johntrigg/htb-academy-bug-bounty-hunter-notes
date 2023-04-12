# Introduction and Web Fuzzing

This section will go over fuzzing for directories and files, hidden vhosts, PHP parameters, and parameter values, with the ffuf tool.

Fuzzing refers to sending various types of user input to some interface, to observe how it reacts. This can include various get requests with special parameters, or trying to perform a SQL injection.

# Basic Fuzzing

## Directory Fuzzing

ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u website.com:80/FUZZ

Here, we assign the FUZZ keyword to the end of the wordlist, and this indictates the content of what will be sent (so in this case, it'll use that wordlist as input), and we specify on where this input goes by typing FUZZ at the end of the url (so in this case, it will fuzz for directories).

//Author's note: I use diresearch for directory fuzzing, but it's preference

![ffuz fuzzing 1](ffuf-fuzzing-1.png)

## Page Fuzzing