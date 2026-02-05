---
title: "Rapidly Creating Fake Users in your Lab AD using Youzer"
date: 2019-06-07
draft: false
toc: false
images:
  - "/wp-content/uploads/2019/06/programming-code-abstract-technology-background-software-developer-computer-script.jpg"
categories:
  - General
  - Security
tags:
  - active
  - cracking
  - directory
  - hacking
  - lab
  - Penetration Testing
  - pentest
  - security
  - youzer
aliases:
  - "/posts/2019/06/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/"
---

## Penetration Testing Lab



Whether you have a fully virtual organisation consisting of several different machines or the odd virtualised box you’re using to explore or freshen up on certain skills. They’re great fun and an asset to any security tester.

 ![](/wp-content/uploads/2019/06/IMG_2874-1024x767.jpg)

Having your own lab is a great way to perform security testing techniques in a controlled environment.

 

If you’re attempting to build out a lab that replicates a real organisation it’s always good to do things properly. Let’s assume for this post that you’ve already built a Windows Domain Controller for your penetration testing lab.



You now need to create those virtual employees within Active Directory. Creating a few different accounts here and there is a relatively easy task I agree, but what if you want your virtual organisation to consist of hundreds of different users in different departments or organisational units, especially with real–world passwords? Having an Active Directory full of users can be useful for a number of activities one being the extraction of NTDS.dit and practising cracking techniques on any hashes that it may contain.


## Enter Youzer


- ![](/wp-content/uploads/2019/06/youzer.png)



Creating hundreds or even thousands of users is now achievable quickly and simply thanks to a tool called [Youzer](https://github.com/SpiderLabs/youzer).



Youzer was written by [Matt Lorentzen](https://www.twitter.com/lorentzenman) an ex-colleague of mine and an absolute brain on legs he describes Youzer’s goal on its GitHub page –


>

 The goal of Youzer is to create information-rich Active Directory environments. This uses the python3 library ‘faker’ to generate random accounts.

 You can either supply a wordlist or have the passwords generated. The generated option is great for testing things like hashcat rule masks. Wordlist option is useful when wanting to supply a specific password list seeded into an environment, or to practice dictionary attacks.
 The output is a CSV and a PowerShell script where both can be copied to the target. When executed, the PowerShell script binds over LDAP so doesn’t rely on the newer Active Directory modules and creates each user object. Currently the OU’s need to exist, but this tool is a sub-project of ‘Labseed’ where the Active Directory structure will be created.

https://github.com/SpiderLabs/youzer


## Prerequisites



Ok, so you want to give Youzer a try on your newly created Domain Controller for your lab? There are a few pre-requisites that we need to install before we can proceed.



For our environment, I used Microsoft Windows 2012 for reasons. We also need to install the following.



The first being Python 3 – [https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe](https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe)

 ![](/wp-content/uploads/2019/06/python3-variables.png) Figure 1: When installing Python ensure you tick the box which says “Add Python to environment variables” 

Once Python3 has been successfully installed we need to install the “faker” python library by issuing the following command from a command shell/powershell instance.


```
PS C:\Users\Administrator\> pip3 install faker
```

 ![](/wp-content/uploads/2019/06/pip-install-faker.png)Figure 2: Installation of the Faker Python library

Now the faker library is installed we can move on to grabbing a password list for Youzer to utilise when generating the users passwords.



A good place to start is [Daniel Miessler’s](https://twitter.com/danielmiessler) github which has a great selection of [Common Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Common-Credentials). When using Youzer for the first time I grabbed the [Probable Top 12000](https://github.com/danielmiessler/SecLists/blob/master/Passwords/probable-v2-top12000.txt).


## Generating Users



We’re now ready to start generating Youzers (see what I did there?), hopefully, by now you have created some organisational units within Active Directory. I created IT, Sales and Management OU’s for our company’s training environment.

 ![](/wp-content/uploads/2019/06/evilcorp-ou.png)Figure 3: The newly created Sales OU

Let’s fire up Youzer and give it some parameters which I will explain…


```
PS C:\Users\Administrator\Downloads\youzer-master\> python youzer.py --wordlist probable-v2-top12000.txt --ou "ou=Sales,dc=EVILCORP,dc=local" --domain EVILCORP --users 500 --output sales-users.csv
```



Above we’ve run the Youzer script telling it the following:



**–wordlist** – Where our password list is located



**–ou** – The path to our Active Directory Organisational Unit



**–domain** – Our Domain



**–users** – How many users we’d like to generate



**–output** – The name of the CSV file we want to dump the data into, Youzer will then create a PowerShell script of the same name for you to run.



Youzer should have now generated your fake users.

 ![](/wp-content/uploads/2019/06/user-gen.png)Figure 4: Youzer being run from our PowerShell prompt.

Our output file should have also been populated with all of our newly generated users, Youzer would have also generated a PowerShell script to automate the task of taking these users and populating Active Directory.

 ![](/wp-content/uploads/2019/06/users-csv.png)Figure 6: Our newly created sales-users.csv
## Populating Active Directory



Now our users have been generated and the needed files created we can go ahead and launch the PowerShell script which Youzer created for us in order to populate our Active Directory.


```
PS C:\Users\Administrator\Downloads\youzer-master> .\sales-users.ps1
```

 ![](/wp-content/uploads/2019/06/user-gen2-1.png)Figure 7: Youzer populating Active Directory

Voila, 500 users created with passwords supplied via our wordlist in a matter of minutes.

 ![](/wp-content/uploads/2019/06/populated-ad.png)Figure 8: Populated Active Directory

That brings us to the end of this post, I hope you found the information valuable the tool really does save time and has great potential. Having spoken to Matt Lorentzen he has some great plans coming in the near future so make sure to star the project in GitHub and keep up to date with any new developments.



Until next time.


### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/) [**](http://twitter.com/share?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&text=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer) [**](mailto:?subject=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer&body=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer) [**](https://api.whatsapp.com/send?text=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer) [**](https://www.reddit.com/submit?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer) [**](javascript:window.print())    
## Post navigation

 [Wardriving with Kismet, GPS and Google Earth.](/posts/wardriving-with-kismet-gps-and-google-earth/)[Summary of the CUPS Vulnerability](/posts/summary-of-the-cups-vulnerability/)
