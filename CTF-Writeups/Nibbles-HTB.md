[Back to Main Page](../index.html) 

# Nibbles on HackTheBox





## Introduction

--

## 1. Initial Scanning

As usual we only have the IP so we start an nmap scan.

`nmap 10.10.10.68`

![1nmap](https://i.imgur.com/2rih64N.png)

nmap did find SSH and Port 80.
Due this we check out the webpage and look closer at the source code.

![1source](https://i.imgur.com/lYBc5oS.png)

Intersting. /nibbleblog/
Therefore we use nibble against /nibbleblog/. DIRB is a Web Content Scanner.
`dirb http://10.10.10.75/nibbleblog/`

![1source](https://i.imgur.com/lYBc5oS.png)

It shows admin.php, which is an login interface.

![1adm](https://i.imgur.com/undefined.png)


## 2. Using the basics to get access

Before trying to break into it, always try some defaults. admin:nibbles

![1break](https://i.imgur.com/17wiJUm.png)

We're in! After a while of searching I decided to check for exploits. We just searched for nibble. 
 
`searchsploit nibble`

![1search](https://i.imgur.com/rFxtQsi.png)

## 2. Exploit

![1splot](https://i.imgur.com/CISJNvpg.png)

Setup the exploit. Additional we set the Listerner LHOST to our IP and LPORT to 4000.
After that we fired it up.

![2splot](https://i.imgur.com/ZE648q2.png)









