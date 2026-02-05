---
title: "CentOS 7 Server Hardening Guide"
date: 2016-02-03
draft: false
toc: false
images:
  - "/wp-content/uploads/2016/02/kissclipart-linux-centos-clipart-centos-red-hat-enterprise-lin-b2486c5cdad38a37.png"
categories:
  - Fedora
  - Linux
  - Security
tags:
  - apache
  - attackers
  - centos
  - fail2ban
  - fedora
  - hackers
  - hard
  - hardening
  - how
  - httpd
  - redhat
  - rhel
  - secure
  - securing
  - server
  - ssh
  - to
aliases:
  - "/posts/2016/02/centos-7-server-hardening-guide/"
---

So… you’ve just setup a shiny new server and you want to take measures to keep the bad guys out? Well, here I will give you a few tips on how to do just that.



This guide was written with [CentOS 7.1](https://www.centos.org/) in mind but other up-to-date variants such as [Fedora](https://getfedora.org/) and [RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) should be pretty similar if not the same.


# Hardening SSH (Secure Shell)

 [![OpenSSH](/wp-content/uploads/2016/02/OpenSSH-150x150.png)](/wp-content/uploads/2016/02/OpenSSH.png)

Most of you will be using this protocol as a means to remotely administrate your Linux server and your right to. Using SSH is by far the best method to administer your server due to its use of encrypted communications unlike it’s older cousins rlogin and telnet which provide no secure methods of communication.


## Create a standard user



Use the ‘useradd’ command to add a username of your choice.


```
useradd YOURUSER
```



Set a password for your newly created user.


```
passwd YOURUSER
```



Add your user to the WHEEL group to enable that user to use the sudo command.


```
usermod -aG wheel YOURUSER
```


## 


## Create an authentication key

 [![RSA](/wp-content/uploads/2016/02/RSA-300x210.png)](/wp-content/uploads/2016/02/RSA.png)

This method of authenticating with your server is much more secure that using a standard password, part of this process will require you to create the key on your local machine which you will be connecting from.



You will be asked if you would like to protect the key with a password, I advise you to do this but it’s not mandatory.


### Creating the key on Mac/Linux


```
ssh-keygen -b 4096
```



Press Enter to use the default names **id_rsa** and **id_rsa.pub** in **/home/your_username/.ssh** before entering your passphrase.


### Upload your public key to your server


#### For **Linux** ```
ssh-copy-id YOURUSER@YOURSERVER
```


#### For **Mac** On your server do.


```
sudo mkdir -p ~/.ssh && sudo chmod -R 700 ~/.ssh
```



From your Mac do the following making sure to substitute ‘youruser’ and ‘yourserver’.


```
scp ~/.ssh/id_rsa.pub YOURUSER@YOURSERVER.0:~/.ssh/authorized_keys

```



Now on to the configuration changes.


### Open up the SSH config file for editing



In this section we will be performing the following actions


- Disallowing root logins
- Setting allowed users
- Changing the default port
- Disabling password authentication
- Force protocol 2



You can replace nano with your favourite text editor such as vi.


```
sudo nano /etc/ssh/sshd_config
```


#### Disallow root logins.



Find the line that says


```
#PermitRootLogin yes
```



and change it to


```
PermitRootLogin no
```


#### Setting your user as an allowed user.



Add the following line to the bottom of your sshd_config file substituting ‘YOURUSER’ with your newly created account.


```
AllowUsers YOURUSER
```


#### Change the default service port.



Find the line that says


```
Port 22
```



Change to something other than 22 such as 22000


```
Port 22000
```


#### Disabling password authentication



We can disable password authentication because we will now be using our newly created key pair to authenticate to the server.



Look for the line that has


```
#PasswordAuthentication yes
```



and replace it with the below line.


```
PasswordAuthentication no
```


#### Only use SSH protocol 2



SSH Protocol 1 is generally considered obsolete as it’s vulnerable and old so lets go ahead and only use SSH Protocol 2. Protocol 2 should be enforced by default but it’s worth checking.



Look for the line that says.


```
#Protocol 2
```



Uncomment the line so it looks like this.


```
Protocol 2
```



There are many more options that we could set but this should be suffice in securing your SSH Service.



Save the file and restart the SSH service by doing the following.


```
sudo systemctl reload sshd.service
```



You should now be able to login on your chosen port with your authorised keys by connecting like this.


```
ssh YOURUSER@YOURSERVER -p 22000
```


# 


# Fail2ban



[Fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) is a handy tool/service that monitors system log files to detect potential intrusion attempts and places bans using a variety of methods.



To install on [CentOS](https://www.centos.org/) we need to enable the EPEL repository by doing the following.


```
sudo yum install epel-release
```



Once the installation has completed we need to then go ahead and install Fail2ban


```
sudo yum install fail2ban fail2ban-systemd
```



Fail to ban comes with a wealth of options that would deserve a post all to itself so in this instance we will create a basic configuration file that will help secure your server, especially the SSH service.



Using SELinux? Then you will want to update your policy by doing the following.


```
yum update -y selinux-policy*
```


#### Configuration



We will be configuring fail2ban for use with Firewalld as it is implemented by default in CentOS 7.



Create a sshd.local file ready for editing.


```
sudo nano /etc/fail2ban/jail.d/sshd.local
```



Add the following lines.


```
[sshd]
enabled = true
port = 22000
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400
```



Save the file and go ahead and start fail2ban.


```
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

```



You should now have a working fail2ban installation which will automatically ban IP addresses after 3 failed attempts at logging in to your system via SSH.


# Apache Hardening



The default [Apache](http://www.apache.org/) configuration just works but there’s a few tweaks we can do here and there that makes the bad guys job a little harder. One of the things we can do is try and prevent information leakage.



By default [Apache](http://www.apache.org/) gives out server version information on error pages. We can prevent this by adding a couple of lines to our httpd.conf file.


### Version banner



Open up the httpd config file ready for editing.


```
sudo nano /etc/httpd/conf/httpd.conf
```



add the following lines to the bottom of the file


```
ServerTokens Prod
ServerSignature Off
```


### Cookies


#### Trace Requests



To protect yourself from [Cross Site Tracing](https://www.owasp.org/index.php/Cross_Site_Tracing) attacks append the following line to the end of your configuration file.


```
TraceEnable off
```


#### Set the HttpOnly and Secure flag



To mitigate against most of the common [Cross Site Scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) attacks you can set the following directive, again add the following line at the end of your configuration file.


```
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
```


#### X-Frame Options



Adding this option to your configuration file will indicate whether or not a browser should be allowed to open a webpage in a frame or iframe. This will prevent site content embedded into other sites. See – [https://www.owasp.org/index.php/Clickjacking](https://www.owasp.org/index.php/Clickjacking).



Append the following line in your configuration file.


```
Header always append X-Frame-Options SAMEORIGIN

```


#### XSS Protection Again



To ensure and enforce web browser [Cross Site Scripting](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) protection append the following to your configuration file.


```
Header set X-XSS-Protection “1; mode=block”

```



Now with those options set we need to restart the Apache daemon.


```
sudo systemctl reload httpd.service
```


# Logwatch



Now to keep tabs on all of those logs, Logwatch is a great tool to monitor your servers logs and email the administrator a digest on a daily basis.


#### Installation


```
sudo yum install logwatch sendmail
```



Now start sendmail.


```
sudo systemctl start sendmail
```


#### Configuration



The default configuration file for Logwatch is located at the below path.


```
/usr/share/logwatch/default.conf/logwatch.conf.
```



This file contains information on which directories for Logwatch to track, how the digest is sent and where the digest is sent to.



By default Logwatch keeps track of everything in /var/log but if you have other log files that you wish to add you can do this by adding the below to your logwatch.conf under the heading ‘Default Log Directory’.


```
LogDir = /some/path/to/your/logs
```


#### Email your daily digest



let’s go ahead and edit the logwatch.conf file.


```
sudo nano /usr/share/logwatch/default.conf/logwatch.conf
```



We need to change add your email into the configuration file so that the digest gets delivered to your inbox.



Look for the following section.


```
# Default person to mail reports to.  Can be a local account or a
# complete email address.  Variable Output should be set to mail, or
# --output mail should be passed on command line to enable mail feature.
MailTo = root
```



change ‘root’ to your own personal email address or wherever you want the digest sending to.


#### Adding Logwatch to Cron



Open up the crontab.


```
crontab -e
```



Now add the following line to the end of the file. This line will make logwatch run at midnight each day.


```
00 00  * * *          /usr/sbin/logwatch
```



This guide was a little quick and dirty so you have any additions to this guide I would love to hear them, also if you think something is wrong or could have been done more efficiently please get in contact.


### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/centos-7-server-hardening-guide/) [**](http://twitter.com/share?url=/centos-7-server-hardening-guide/&text=CentOS%207%20Server%20Hardening%20Guide) [**](mailto:?subject=CentOS%207%20Server%20Hardening%20Guide&body=/centos-7-server-hardening-guide/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/centos-7-server-hardening-guide/&title=CentOS%207%20Server%20Hardening%20Guide) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/centos-7-server-hardening-guide/&title=CentOS%207%20Server%20Hardening%20Guide) [**](https://api.whatsapp.com/send?text=/centos-7-server-hardening-guide/&title=CentOS%207%20Server%20Hardening%20Guide) [**](https://www.reddit.com/submit?url=/centos-7-server-hardening-guide/&title=CentOS%207%20Server%20Hardening%20Guide) [**](javascript:window.print())    
## Post navigation

 [Exploiting the OpenNMS/Jenkins RMI Java Deserialization Vulnerability](/posts/exploiting-the-opennmsjenkins-rmi-java-deserialization-vulnerability/)
