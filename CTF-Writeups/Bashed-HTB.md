[Back to Main Page](../index.html) 

# Bashed on HackTheBox





## Introduction

Bashed is a beginner's box which was the second CTF I ever attempted. When I tried it, I had booted up Kali and knew that a couple tools existed, but did not have any strategies, context or experience. 

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.

## 1. Initial Scanning

All we do have is an IP so we start to run a nmap scan.

`nmap 10.10.10.68`

![1nmap](https://i.imgur.com/fyklCXk.png)

nmap did find TCP port 80.

Kali catogorizes most of the HTTP tools under "Web Application Analysis," so I took a peek and tried to get one of them working. I found the most useful starting commands for a website CTF is `nikto`

`nikto -h http://10.10.10.68:80`

Our `-h` flag tells nikto the location of our target. 

![1nikto](https://i.imgur.com/h1r2hqi.png)

Nikto shows us many intersting directories - one of them being /dev/. So I connected on it and found those .php scripts.

![2nikto](https://i.imgur.com/tupNPIp.png)


## 2. Using the found scripts to get access



