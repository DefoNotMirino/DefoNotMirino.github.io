[Back to Main Page](../index.html) 

# Introduction

This penetration cheat sheet should be used as a quick reference overview for various and typical commands used when performing penetration tests.

**Table of Contents** 

- [Pre-engagement](#pre-engagement)
    - [Network Configuration](#network-configuration)
        - [Set IP Address](#set-ip-address)
        - [Subnetting](#subnetting)
- [nformation Gathering](#information-gathering)
    - [DNS Enumeration](#dns-enumeration)
    - [Active Information Gathering](#active-information-gathering)
        - [DNS Bruteforce](#dns-bruteforce)
        - [Port Scanning](#port-scanning)        


# Pre-engagement

## Network Configuration

### Set IP Address

`ifconfig eth0 xxx.xxx.xxx.xxx/24 `

### Subnetting

`ipcalc xxx.xxx.xxx.xxx/24`

`ipcalc xxx.xxx.xxx.xxx 255.255.255.0`

# Information Gathering

## DNS Enumeration 

`whois domain.com`   
Domain name lookup service to search the whois database for domain name registration information.

`dig a domain.com @nameserver`    
Dig dns lookup / nameserver query.

`dig mx domain.com @nameserver`   
Perform MX record Lookup.

`dig axfr domain-name-here.com @nameserver`   
Perform Zone Transfer.

##  Active Information Gathering

### DNS Bruteforce

`dnsrecon -d TARGET -D /usr/share/wordlists/dnsmap.txt -t std --xml ouput.xml`   
DNS Recon via Kali Tools.

### Port Scanning

`nmap -v -sS -A -T4 target`
Nmap verbose scan, runs syn stealth, T4 timing (should be ok on LAN), OS and service version info, traceroute and scripts against services.

`nmap -v -sS -p--A -T4 target`
As above but scans all TCP ports (takes a lot longer).

`nmap -v -sU -sS -p- -A -T4 target`
As above but scans all TCP ports and UDP scan (takes even longer).

`netdiscover -r 192.168.1.0/24`
Discovers IP, MAC Address and MAC vendor on the subnet from ARP, helpful for confirming you're on the right VLAN at $client site.

