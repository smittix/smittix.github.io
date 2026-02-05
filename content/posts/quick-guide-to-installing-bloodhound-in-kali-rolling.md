---
title: "Quick Guide to Installing Bloodhound in Kali-Rolling"
date: 2017-10-11
draft: false
toc: false
images:
  - "/wp-content/uploads/2017/10/Bloodhound.png"
categories:
  - General
  - Linux
  - Security
tags:
  - admin
  - bloodhound
  - degrees
  - domain
  - guide
  - install
  - installing
  - kali
  - linux
  - neo4j
  - rolling
  - setup
  - six
aliases:
  - "/posts/2017/10/quick-guide-to-installing-bloodhound-in-kali-rolling/"
---

## Intro



I have had a few people over the last couple of months asking me how to get [Bloodhound](https://github.com/BloodHoundAD/Bloodhound/wiki) up and running after I had sung its praise since seeing the “Six Degrees to Domain Admin” video from BSIDES Las Vegas. If you still haven’t seen the video I am referring to I suggest you take a peek before proceeding.



[su_youtube url=”https://www.youtube.com/watch?v=lxd2rerVsLo”]



It really is such an awesome tool and I highly recommend it to not only info-sec professionals but to anyone who administrates an Active Directory environment.



The awesome news is that [Bloodhound](https://github.com/BloodHoundAD/Bloodhound/wiki) is now in the Kali Linux repository’s and is super easy to install and get up and running and I will show you how.


## Ensure an up-to-date system.



Firstly, please ensure you’re running the latest and greatest by performing a dist-upgrade like so.


```
apt-get update
```



and then


```
apt-get dist-upgrade
```


## Installing Bloodhound



You guessed it, simply run the following. Bloodhound depends on neo4j so that will be installed as well.


```
apt-get install bloodhound
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14.png)
## Change the Default Password for Neo4j



We really should change the default password for Neo4j, you know.. For reasons.



Let’s launch Neo4j


```
neo4j console
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41.png)

We now have a remote interface available at **http://localhost:7474**. Let’s head over there via a browser and change that default password. You will also see that it enabled Bolt on the localhost, we need this for later.

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13.png)

Login with the default credentials (below) you will then be asked to change the password :-


- Username: **neo4j** - Password: **neo4j** [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27.png)

Go ahead and complete the password change and close the browser window.


## Let the Hound See The Blood



Pop a new terminal window open and run the following command to launch Bloodhound, leave the Neo4j console running for obvious reasons.


```
bloodhound
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43.png)

As you can see, Bloodhound is now running and waiting for some user input. Earlier when launching Neo4j it also enabled Bolt on bolt://127.0.0.1:7687. You need to use this as your Database URL.


- Database URL – **bolt://127.0.0.1:7687** - Username – **neo4j** - Password – **your newly changed password** Hit login and you should be presented with the Bloodhound tool minus any data. You can now import your data and get analyzing.

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17.png)

Hopefully this was a nice and quick guide to help anyone out there having any issues getting up and running with the awesome tool that is Bloodhound.



I also want to take a moment to thank [@_wald0](https://www.twitter.com/_wald0), [@CptJesus](https://twitter.com/CptJesus), and [@harmj0y](https://twitter.com/harmj0y)for their continued hard work on this amazing project.



Cheers Guys!

 [![](/wp-content/uploads/2017/10/Bloodhound-Dogs-300x169.jpg)](https://github.com/BloodHoundAD/Bloodhound/wiki)
### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/quick-guide-to-installing-bloodhound-in-kali-rolling/) [**](http://twitter.com/share?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&text=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling) [**](mailto:?subject=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling&body=/quick-guide-to-installing-bloodhound-in-kali-rolling/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling) [**](https://api.whatsapp.com/send?text=/quick-guide-to-installing-bloodhound-in-kali-rolling/&title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling) [**](https://www.reddit.com/submit?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling) [**](javascript:window.print())    
## Post navigation

 [Exploiting MS17-010 – Using EternalBlue and DoublePulsar to gain a remote Meterpreter shell](/posts/exploiting-ms17-010-using-eternalblue-and-doublepulsar-to-gain-a-remote-meterpreter-shell/)[Reporting SSL/TLS Issues the Easy Way with YANP](/posts/reporting-ssltls-issues-the-easy-way-with-yanp/)
