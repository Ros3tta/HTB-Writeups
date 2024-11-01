![image](https://github.com/user-attachments/assets/2baeb2b0-9373-4b54-a138-9b63c5a8e113)<b>

# Information Gathering
I start by pinging the target 4 times:<br>
`ping -c 4 10.10.10.37`
```
PING 10.10.10.37 (10.10.10.37) 56(84) bytes of data.
64 bytes from 10.10.10.37: icmp_seq=1 ttl=63 time=109 ms
64 bytes from 10.10.10.37: icmp_seq=2 ttl=63 time=108 ms
64 bytes from 10.10.10.37: icmp_seq=3 ttl=63 time=108 ms
64 bytes from 10.10.10.37: icmp_seq=4 ttl=63 time=108 ms
```
Followed by `nmap` scan:<br>
`nmap -sVC -Pn 10.10.10.37`
```
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
|_http-generator: WordPress 4.8
```
There is website on port 80 that we need to add the domain for:<br>
`echo "10.10.10.37 blocky.htb" | sudo tee -a /etc/hosts`<br>
![image](https://github.com/user-attachments/assets/190a46ff-ec03-47bf-919e-9c51fcb9b0f2)

After that i also ran `whatweb` scan to get additional information:<br>
`whatweb -a 3 blocky.htb`
```
http://blocky.htb [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.37], JQuery[1.12.4], MetaGenerator[WordPress 4.8], PoweredBy[WordPress,WordPress,], Script[text/javascript], Title[BlockyCraft &#8211; Under Construction!], UncommonHeaders[link], WordPress[4.7.8,4.7.9,4.8]
```
Since there is a domain, i tried fuzzing for subdomains but it didn't work:<br>
`ffuf -w STW_20k.txt:FUZZ -u http://10.10.10.37/ -H 'Host: FUZZ.blocky.htb' -ac`<br>
Back to the website, i started dirbusting with `feroxbuster`:<br>
`feroxbuster -r -k -E -g -C 400,403,404,500,502,503 --auto-tune -u http://blocky.htb/`
And found that directory listing was enabled
Most of the files were part of WordPress so they didnt really matter<br>
![image](https://github.com/user-attachments/assets/5f91d95a-a592-4ad4-ba97-dea1741d28a1)

2 files however were unusual:
- `BlockyCore.jar`
- `griefprevention-1.11.2-3.1.1.298.jar`

The second one didnt really matter
Extracting the `BlockyCore.jar` and looking inside, there is a file called `BlockyCore.class` located at `/com/myfirstplugin/`<br>
Running `strings BlockyCore.class` outputs this:<br>
![image](https://github.com/user-attachments/assets/d7b41f42-a89c-4ae4-b5d2-d843e74baeb7)




</b>
