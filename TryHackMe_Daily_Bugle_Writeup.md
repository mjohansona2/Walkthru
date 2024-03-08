# TryHackMe Daily Bugle Write-Up

This document outlines my journey through the TryHackMe Daily Bugle machine. As my first attempt at a write-up, my goal is to detail the steps taken from initial reconnaissance to gaining root access, highlighting the importance of not leaving sensitive credentials in configuration files.

## Enumeration

### Initial Scans

I initiated the process with an `nmap` scan to identify open ports and running services:

```bash
nmap -sC -sV -v -A 10.10.231.192

PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open     http     Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open     mysql    MariaDB (unauthorized)
5859/tcp filtered wherehoo

NSE: Script Post-scanning.
Initiating NSE at 10:58
Completed NSE at 10:58, 0.00s elapsed
Initiating NSE at 10:58
Completed NSE at 10:58, 0.00s elapsed
Initiating NSE at 10:58
Completed NSE at 10:58, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.27 seconds
```

### Directory Enumeration

Next, I used `Gobuster` to discover hidden directories:

```bash
gobuster dir -u http://10.10.231.192 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.192
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://10.10.231.192/images/]
/media                (Status: 301) [Size: 235] [--> http://10.10.231.192/media/]
/templates            (Status: 301) [Size: 239] [--> http://10.10.231.192/templates/]
/modules              (Status: 301) [Size: 237] [--> http://10.10.231.192/modules/]
/bin                  (Status: 301) [Size: 233] [--> http://10.10.231.192/bin/]
/plugins              (Status: 301) [Size: 237] [--> http://10.10.231.192/plugins/]
/includes             (Status: 301) [Size: 238] [--> http://10.10.231.192/includes/]
/language             (Status: 301) [Size: 238] [--> http://10.10.231.192/language/]
/components           (Status: 301) [Size: 240] [--> http://10.10.231.192/components/]
/cache                (Status: 301) [Size: 235] [--> http://10.10.231.192/cache/]
/libraries            (Status: 301) [Size: 239] [--> http://10.10.231.192/libraries/]
/tmp                  (Status: 301) [Size: 233] [--> http://10.10.231.192/tmp/]
/layouts              (Status: 301) [Size: 237] [--> http://10.10.231.192/layouts/]
/administrator        (Status: 301) [Size: 243] [--> http://10.10.231.192/administrator/]
/cli                  (Status: 301) [Size: 233] [--> http://10.10.231.192/cli/]
```


### Web Technology Analysis

Using Wappalyzer, I identified that the website is powered by Joomla.



### robots.txt Discovery

The `robots.txt` file revealed the path to the administrator page:



## Exploitation

### Administrator Login Page

The robots.txt file shows the Joomla administrator login page, which led to the next phase of the attack.


### Vulnerability Research

With version information in hand, I searched Exploit-DB using `searchsploit`:

```bash
# Insert searchsploit command and relevant exploit details
# Example: searchsploit Joomla 3.7
```


### Joomblah Exploitation

The TryHackMe room suggested using a Python script over SQLMap. I found and utilized the Joomblah (https://github.com/XiphosResearch/exploits/tree/master/Joomblah) script:

```bash
# Replace with the actual command to run Joomblah and its output
# Example: python joomblah.py http://10.10.10.10
```


### Password Cracking

After obtaining credentials, I used `john` to crack the password:



### Gaining Initial Access

Logging into the admin panel led me to upload a reverse shell and gain initial access to the machine as the `apache` user.



### Upgrading to Interactive Shell

To improve my shell, I executed:

```bash
python -c "import pty; pty.spawn('/bin/bash')"
export TERM=xterm; export SHELL=/bin/bash
# Followed by backgrounding and configuring the terminal
```

### Privilege Escalation

Despite initial limitations (no direct switch to another user, no sudo privileges, no cron jobs to exploit), I persevered:



### Finding Sudo Options

Exploring the system further revealed sudo options for the current user:




### Crafting and Executing the Exploit

With unrestricted `yum` access, I created an RPM package with a reverse shell using `fpm`, transferred it to the target, and executed it to gain root access:




## Conclusion

Completing the Daily Bugle machine was both challenging and rewarding. It served as a practical reminder of the risks associated with leaving sensitive information in configuration files. This walkthrough aimed to provide a detailed account of each step, from enumeration to privilege escalation, highlighting critical security practices along the way.
