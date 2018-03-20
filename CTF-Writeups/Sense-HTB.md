
First I started as usual an nmap scan `nmap 10.10.10.60`.

1

Port 80 and 443 tcp are open, so I started enumerating the whole server with dirbuster.   
Wordlist: directory-list-lowercase-2.3-medium.txt   
File extension: php, html,txt,sh,xml

2

After over an hour I found something interesting.

3

/installer/system-users.txt   
And there is the password and user. Rohit:pfsense as pfsense is the standard password. 

4

AAAAND we're in already.





