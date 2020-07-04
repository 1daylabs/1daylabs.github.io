---
title: "Hackthebox Forwardslash Writeup"
date: 2020-07-04 12:00:00 +0530
categories: [HACKTHEBOX, Retired]
tags: [htb]     # TAG names should always be lowercase
---
![Intro]({{ "/assets/img/Posts/HTBforwardslash/1.PNG" | relative_url }})

>We would be exploiting the forwardslash box from hackthebox. The box comprises some enumeration for subdomain and then we would exploit it by Local File Inclusion vulnerability and then by getting shell we would go for post exploitation to gain the root shell. 

## Enumeration
By using nmapAutomator for enumeration. 

```console
$ ./nampautomator 10.10.10.183 All

Nmap scan report for 10.10.10.183
Host is up (0.25s latency).
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
| 2048 3c:3b:eb:54:96:81:1d:da:d7:96:c7:0f:b4:7e:e1:cf (RSA)
| 256 f6:b3:5f:a2:59:e3:1e:57:35:36:c3:fe:5e:3d:1f:66 (ECDSA)
|_ 256 1b:de:b8:07:35:e8:18:2c:19:d8:cc:dd:77:9c:f2:5e (ED25519)
80/tcp open http Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Did not follow redirect to http://forwardslash.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.57 seconds

```
There are 2 ports open 22 and 80. Before going for exploitation lets get some overview of the ports. The 22 port is used for SSH connection which is of OpenSSH version 7.6p1. The 80 port is much familiar as it is used for hosting a web server. 

![img2]({{ "/assets/img/Posts/HTBforwardslash/2.PNG" | relative_url }})

After running some common enumeration tools such as “nikto”, “gobuster”, “dirbuster”. But didn’t get any lead. So, I went for subdomain enumeration by running “ wfuzz”.

```console
$ wfuzz — hh 0 -H ‘Host: FUZZ.forwardslash.htb’ -u http://10.10.10.183/ 
— hc 400 -w /usr/share/wordlists/wfuzz/general/common.txt -c

–hh = for hiding the result which has 
–hc = for hiding result which have response 400
-c = for making the result 
```
![img3]({{ "/assets/img/Posts/HTBforwardslash/3.PNG" | relative_url }})

After getting a new subdomain add it to /etc/hosts file “backup.forwardslash.htb”. Now enumerating the new domain we get a login page. 

![img4]({{ "/assets/img/Posts/HTBforwardslash/4.PNG" | relative_url }})

By signing in as a new user we could get the dashboard. After observing the functionality we saw that we could upload an image by url. It gave me a strike of LFI (Local File Inclusion) vulnerability. But we need to enable the tabs for entering the data into the url text box and also enable the submit button.

![img5]({{ "/assets/img/Posts/HTBforwardslash/5.PNG" | relative_url }})

Trying with some common payloads extracting the “file:///etc/passwd” . 

![img6]({{ "/assets/img/Posts/HTBforwardslash/6.PNG" | relative_url }})

As enumerating the directories we saw /dev/ directory. But we got permission denied while extracting the content from the file.

![img7]({{ "/assets/img/Posts/HTBforwardslash/7.PNG" | relative_url }})

## Exploitation

Now we could exploit the vulnerability by including a PHP file through PHP wrapper . You can read about it from “PayloadAllThings.”

>payload = “php://filter/convert.base64-encode/resource=file:///var/www/backup.forwardslash.htb/dev/index.php”

We get the index.PHP file in base64 encoded. After decoding the data we get the credentials of FTP login. But since there is no Ftp-port open lets try logging into SSH. 

![img8]({{ "/assets/img/Posts/HTBforwardslash/8.PNG" | relative_url }})

```console
$ ssh chiv@10.10.10.183
```
![img9]({{ "/assets/img/Posts/HTBforwardslash/9.PNG" | relative_url }})
![img10]({{ "/assets/img/Posts/HTBforwardslash/10.PNG" | relative_url }})
![img11]({{ "/assets/img/Posts/HTBforwardslash/11.PNG" | relative_url }})


After getting the shell let's execute the LinuxPrivEsc commands and scripts for post-exploitation. After running the LinEnum script and looking at the results I found a SUID binary owned by pain user and one more interesting thing there is a file called config.PHP.bak which is also owned by user pain. But don’t have permission to r/w/x.

![img12]({{ "/assets/img/Posts/HTBforwardslash/12.PNG" | relative_url }})
![img13]({{ "/assets/img/Posts/HTBforwardslash/13.PNG" | relative_url }})

After observing the output of the backup file we got to know that it is generating a different hash every time. Observing the text in the output as it says “time based backup viewer” it may be the md5sum that is being generated it’s of the current timestamp including seconds that's why it's changing whenever you run it again. Here I made a bash script just for checking the md5 value if it will be the same.

![img14]({{ "/assets/img/Posts/HTBforwardslash/14.PNG" | relative_url }})

This will create a timestamp of the current time and convert it to md5 . And then it will link the config.PHP.bak file with the time variable and then run the backup binary.This will show us the content of the config.PHP.bak file.

![img15]({{ "/assets/img/Posts/HTBforwardslash/15.PNG" | relative_url }})

Great, we got the password of a pain user. Now let's move on to post exploitation.

## Post Exploitation

Firstly lets see what all are our rights and could perform as a root.
![img16]({{ "/assets/img/Posts/HTBforwardslash/16.PNG" | relative_url }})

Cryptsetup is used to map the images generally of backup images.And then we can mount the mapped images to any directory and access the files in it.

As we saw that there is an encryptorinator directory which encrypts the cipher text given . After analyzing the code I made some changes in it , here. To look for common words in the decrypted message and if we have then that’s our key to get the message.

![img17]({{ "/assets/img/Posts/HTBforwardslash/17.PNG" | relative_url }})

After running the decrypt.py file it gave me the key and the password.

![img18]({{ "/assets/img/Posts/HTBforwardslash/18.PNG" | relative_url }})

Now we could map the image in /var/backups/recovery. 
Entering the password “cB!6%sdH8Lj^@Y*$C2cf”

```console
$ sudo /sbin/cryptsetup luksOpen /var/backups/recovery/encrypted_backup.img backup
```
![img19]({{ "/assets/img/Posts/HTBforwardslash/19.PNG" | relative_url }})

After checking into /dev/mapper for the mapped images. Now that it has been created we can mount the images into the /mnt directory.That gives us the private key for the root.
![img20]({{ "/assets/img/Posts/HTBforwardslash/20.PNG" | relative_url }})

Let’s login as root and fetch the root.txt file.
![img21]({{ "/assets/img/Posts/HTBforwardslash/21.PNG" | relative_url }})


### References

[swisskyrepo/PayloadsAllTheThings
The File Inclusion vulnerability allows an attacker to include a file, usually exploiting a "dynamic file inclusion"…github.com](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#wrapper-phpfilter)

[Cryptsetup Vulnerability Grants Root Shell Access on Some Linux Systems
A vulnerability in cryptsetup, a utility used to set up encrypted filesystems on Linux distributions, could allow an…threatpost.com](https://threatpost.com/cryptsetup-vulnerability-grants-root-shell-access-on-some-linux-systems/121963/)

[Script to run migrate export backup
hi, How can we schedule the migrate export backup everyday and push it to another server with the backup file name with…community.checkpoint.com](https://community.checkpoint.com/t5/General-Management-Topics/Script-to-run-migrate-export-backup/td-p/23512)

[Cryptography with Python - Quick Guide
Cryptography is the art of communication between two users via coded messages. The science of cryptography emerged with…www.tutorialspoint.com](https://www.tutorialspoint.com/cryptography_with_python/cryptography_with_python_quick_guide.htm)


**By Swar Shah**