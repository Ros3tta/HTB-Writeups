<b>

# Information Gathering
First i ping the target 4 times to ensure good connection:<br>
`ping -c 4 10.10.10.75`
```
PING 10.10.10.75 (10.10.10.75) 56(84) bytes of data.
64 bytes from 10.10.10.75: icmp_seq=1 ttl=63 time=109 ms
64 bytes from 10.10.10.75: icmp_seq=2 ttl=63 time=108 ms
64 bytes from 10.10.10.75: icmp_seq=3 ttl=63 time=108 ms
64 bytes from 10.10.10.75: icmp_seq=4 ttl=63 time=108 ms
```
After that i run nmap with `-Pn` to not check if the host is alive and `-sVC` for version and script enumeration:<br>
`nmap -sVC -Pn 10.10.10.75`
```
Nmap scan report for 10.10.10.75
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Then i ran `whatweb` to gather additional information about the website:<br>
`whatweb -a 3 10.10.10.75`
```
http://10.10.10.75 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.75]
```
After checking the source code for the page i found this:
![image](https://github.com/user-attachments/assets/cea217ff-0f73-4330-9765-f02383d391aa)

Browsing to the directory revealed blog page<br>
I ran feroxbuster which showed that the directories were readable:
![image](https://github.com/user-attachments/assets/b6297995-cf2b-42fd-96a0-f940124852d8)

The important one was this:<br>
`http://10.10.10.75/nibbleblog/content/private/`

Two of them revealed login information:
```
http://10.10.10.75/nibbleblog/content/private/users.xml
http://10.10.10.75/nibbleblog/content/private/config.xml
```
And one revealed the version of `Nibbleblog`:<br>
`http://10.10.10.75/nibbleblog/README`
</b>
