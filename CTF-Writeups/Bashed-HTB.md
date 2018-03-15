[Back to Main Page](../index.html) 

# Bashed on HackTheBox





## Introduction

Bashed is a beginner's box which was the first CTF I ever attempted. When I tried it, I had booted up Kali and knew that a couple tools existed, but did not have any strategies, context or experience. 

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





![a2-nikto](https://i.imgur.com/XHOA47W.png)

My nikto scan didn't actually finish until after my dirsearch, and ended up not telling me anything dirsearch didn't tell me already. This will not always be the case, I always run both.


`dirsearch` is a web directory bruteforcer implemented in python3. If you don't want to use it, you can use `dirb` or `dirbuster`, which come with Kali, or another program such as `gobuster`.

I cloned it using

`git clone https://github.com/maurosoria/dirsearch /opt/dirsearch`

and ran it with

`python3 /opt/dirsearch/dirsearch -u "http://10.10.10.37" -e sh,txt,php,html,htm,zip,tar.gz,tar --plain-text-report=dirsearch_europa_http_quick -t 25`

* `-u` points to our URL starting point
* `-e` lists some common file extensions to look for
* `--plain-text-report=` specifies that I want a log file generated
* `-t 25` uses more CPU threads


![a3](https://i.imgur.com/HReNAk5.png)


We find a whole bunch of paths that look interesting. The fact that we have directories that begin with `wp-...` is a good place to start. This means our site runs Wordpress, which can have a whole host of vulnerabilities. Let's use the WPScan tool.


wpscan -u http://10.10.10.37 -e uvp


![A4](https://i.imgur.com/XDfpv8b.png)

(Today, I would append ` | tee wpscan_blocky.txt` to save the output.)


WPScan finds many, many vulnerabilities we can dive into, but lets keep enumerating to make sure we get the full lay of the land.

## 2. Using our Reconnoitered Information to Dig Deeper

Let's review what we have so far. We find the username *notch*, and 2 places to log in, *phpmyadmin* and *wp-login*. Well, 3 places if we count SSH. The strategy here is to try to use this username to log in somewhere for further access

If we get phpmyadmin database access, we might be able to find some credentials to login somewhere else, or drop in some kind of reverse shell. If we get Wordpress admin access, we can also spawn a reverse shell pretty easily.

Maybe the site has some more useful information. It looks like a simple Wordpress blog with only one post. Let's read it.


![A5-eyeballsite](https://i.imgur.com/ORSRzbV.png)


*"...developing a wiki system... and a core plugin..."*  Maybe some of that is already available. Let's revisit our dirsearch results.

We have both the `/wp-content/plugins` folder (where wpscan found the akismet plugin) and a separate `/plugins` directory. This is atypical for a Wordpress site - why does it have both?



![A6-plugins](https://i.imgur.com/cpRuBMY.png)



Two .jars for us to download. The *Core* plugin was mentioned in the blog post, so let's start there - download it after copying the link location.


* `wget http://10.10.10.37/plugins/files/BlockyCore.jar`

* `unzip BlockyCore.jar -d blockycore`


![A7-DLblockycore](https://i.imgur.com/OPSss30.png)


* `cd blockycore`

* `ls -laR *`

![A8-blockycorecomps](https://i.imgur.com/HvhHJVx.png)

Now I don't know too much about Java, but Blockycore.class seems appealing to me. Can we just read it and find something interesting?


`cat com/myfirstplugin/BlockyCore.class`


![A9 - cat](https://i.imgur.com/WtGUamo.png)


This file appears to have been compiled and has some lines we can't read. We can isolate the human-readable parts with the `strings` command.


`strings com/myfirstplugin/BlockyCore.class`


![A10-strings](https://i.imgur.com/sIfty8D.png)


There are some strings that begin to look useful, but maybe we can find even more context. I did some online searching and come across the `javap` command, which can be use to disassemble Java classes. I may have had to install it using `apt-get`


`javap -c com/myfirstplugin/BlockyCore.class | tee  ~/CTF/blocky/blockycore_disassembled.txt`

The ` | tee ...` command copies the output to a a file we can easily read later.


![A11-javap](https://i.imgur.com/FKLeNA1.png)


That's the kind of detail I like to see! `sqluser` and `sqlpass` are VERY promising. These credentials are likely to  logged us in to the phpmyadmin database, and might even be re-used elsewhere.


## 3. Gaining Webserver Access


When I first attempted this box, I got stuck here for a few hours. I knew I was missing something very simple, but couldn't find my way through all the information I had at my disposal. What I should have done was get organized  and take another inventory of all that had been found.
So far we have:


Usernames:
* *notch* - from the WP site
* *root* - exists by default for SSH and phpmyadmin, seen in plugin code
* *admin* - exists by default on WP


Passwords:
* *8YsqfCTnvxAUeduzjNSXe22* - from the plugin code

and

Login Areas:
* Wordpress
* phpmyadmin
* ssh


Let's try `root:8YsqfCTnvxAUeduzjNSXe22` as phpmyadmin credentials.


![A12- phpmyadmin log in](https://i.imgur.com/VrqfIZt.png)


Great, we're in the database. Odds are we are going to find hashes here that will lead to the Wordpress password. Before we go trying to finding and cracking them, however, let's keep it simple. We should first try logging in with our credentials somewhere else.


I'm feeling optimistic; I say we go right for the big fish. Root access via SSH.




* `ssh root@10.10.10.37`


and give the password when asked.


![A13-rootsshfail](https://i.imgur.com/PGuwdxH.png)


Didn't hurt to try, as unlikely as it was. Since we only have a few usernames, and only one password candidate, it is feasible to simply try them all. Since SSH access is easier to work with than a webshell, let's try to get in here before trying wp-admin access.


* `ssh notch@10.10.10.37`


![A14-notchsshwin](https://i.imgur.com/wDjcZOo.png)


And we're in!


## 4. Capturing the User Flag


Let's grab user

* `ls /home`
* `ls -la /home/notch`
* `cat /home/notch/user.txt`


A little prior knowledge tells me that Notch is the inventor and boss of the Minecraft universe - Could this mean he is the admin of this system? Let's find out what we can run as superuser.


![A15-sudo_L](https://i.imgur.com/CQOQ3YB.png)


Yup, he's an admin. This could be all we need to know to gain root access.



* `sudo su`

![A16 - rootshell](https://i.imgur.com/vc1DDda.png)

We have a root shell, and it feels good


## 5. Call Me Markus, Superuser.


From here, it is as trying to read the root flag.

`ls /root`
`cat /root/root.txt`

Success! Grab the root flag and let's clean up.


__NOTE:__ These are some pretty simple cleanup commands meant to cover our tracks a little bit, but only a little bit. A trained admin would notice that these files have been altered, so look at these commands as the beginning of your tracks-covering career and not as a MIB-Style mind wipe.


* `echo '' > /home/notch/.bash_history`
* `echo '' > /var/log/auth.log `
* `echo '' > /var/www/admin/logs/access.log`
* `echo '' > /root/.bash_history`


Exit the root shell with


`history -c  && kill -9 $$`


then repeat this command as notch, and you're all done.


Thanks to HackTheBox and Arrexel!

## Thanks for reading!

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>
