# The Code Caper - Write-Up by unguest
This write-up may be intended only for those who already finished this box. However, you can have a quick look at it if you're totally lost.

## I - Host enumeration
For this part, I'm using this command : 
```
nmap -p 1-1000 -sV -sS <ip>
```

-p 1-1000 => Scan only the 1000 first ports

-sV => Get the services' version (as we're blind for now, I expected to find a vulnerable service with a CVE)

-sS => Perform a SYN-STEALTH scan

Here is the output of the scan : 

```
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

```
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

I stopped the script as soon as I saw the administrator.php file : 
