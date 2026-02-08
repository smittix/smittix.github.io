---
title: "Reporting SSL/TLS Issues the Easy Way with YANP"
date: 2017-10-12
draft: false
toc: false
images:
  - "/wp-content/uploads/2017/10/programming-code-abstract-technology-background.jpg"
categories:
  - Linux
  - Security
tags:
  - issues
  - nessus
  - parser
  - parsing
  - reporting
  - ssl
  - tls
  - vulnerabilities
aliases:
  - "/posts/2017/10/reporting-ssltls-issues-the-easy-way-with-yanp/"
---

What’s YANP I hear you ask? [YANP](https://github.com/adipinto/yet-another-nessus-parser) stands for “Yet Another Nessus Parser” written by [Alessandro Di Pinto](https://twitter.com/adipinto) and I’m over the moon that I found it. I’ll tell you why.



Getting all the IP addresses and ports together from Nessus to stick them into other tools such as TestSSL.sh or SSLScan can take away valuable time on large engagements when the time could be spent looking into more harder to detect vulnerabilities. Ultimately we have a duty to our clients to report all our findings and it’s just another thing that needs to be done.



But.. Just because it needs to be done doesn’t mean we can’t get it done quicker and that’s where [YANP](https://github.com/adipinto/yet-another-nessus-parser) comes into play.


## What is YANP?



The below is taken from the projects GitHub page.


>

Yet Another Nessus Parser ([YANP](https://github.com/adipinto/yet-another-nessus-parser)) is a parser able to extract information from Tenable Nessus’s .nessus file format. The main tool’s objective is to export vulnerability assessment reports in a parsable way. The user is able to choose an appropriate output format in order to save the Nessus’ reports following various advanced needs.



Here I will show you my flow of getting the SSL/TLS issues pulled out of Nessus easily and quickly , I’m writing this to help anyone who finds it time-consuming too. By all means if anyone out there reading this has a better way of doing things please let me know, your comments will be much appreciated.


## Installing YANP



Any tools I get from GitHub I stick in my /home/james/tools directory. Let’s git clone YANP to our local system please change the path to a directory of your choosing.


###### These steps were taken on a kali-linux virtual machine and assumes you have both git and python installed.


```
cd /home/james/tools
```


```
git clone https://github.com/adipinto/yet-another-nessus-parser
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17.png)
```
cd yet-another-nessus-parser
```



OK so we’ve cloned the repository and we’re now sitting in the YANP directory.


## Parsing for Use with Other Tools



Now it’s time to parse your .nessus file to gain the information we need. As you know, this tutorial is aimed at SSL/TLS issues but you can parse the file for any issues of your choosing.



Let’s have a look the options first.


```
python yanp.py
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52.png)

As you can see there are a few options that we can use, we can search using the specific PluginID or PluginName for example. In this instance I’m going to search for “SSL” using the PluginName option using the “-d” switch. We’re also going to tell YANP where our .nessus file is using the “-i” switch.


```
python yanp.py -i /home/james/Downloads/sample.nessus -d SSL
```



You should see some output similar to mine below.

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59.png)

That’s all good and well but we can’t use that for anything except viewing it really. Let’s clean it up a little by piping it to “cut”. You will see I have set the delimiter to a space (-d ” “) and chosen the first field (-f 1) to grab just the list of IP addresses and ports.


```
python yanp.py -i /home/james/Downloads/sample.nessus -d SSL | cut -d " " -f 1
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01.png)

That’s a little better but I see there are duplicates in there too. Let’s use “sort” and “uniq” to remove the duplicated entries.


```
python yanp.py -i /home/james/Downloads/sample.nessus -d SSL |cut -d " " -f 1| sort | uniq
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37.png)

That looks a lot better and we now have usable list that we can pass to other tools for analysis but first let’s transfer this data into a file.


```
python yanp.py -i /home/james/Downloads/sample.nessus -d SSL | cut -d " " -f 1 | sort | uniq > ssl-issues.txt
```



You can now use your newly created ssl-issues.txt file as the target list for other tools such as SSLScan. Again, you can search for anything via the PluginName or PluginID switches and output whatever you need.


## Parsing for Specific Issues



You should now get the gist of what YANP can do and you’re probably coming up with your own ideas of how to use it. For this next example we will search for a specific SSL issue such as “BEAST” , we will get the affected host details for later use in our report.


```
python yanp.py -i /home/james/Downloads/sample.nessus -d BEAST
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02.png)

You now see a list that we can use for our report which tells the client the IP, Port and Issue.




## Parsing to CSV



We can also parse the .nessus file and create a CSV for later manipulation using the following command which will create the file “issues.csv”.


```
python yanp.py -i /home/james/Downloads/sample.nessus --csv issues
```

 [![](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10.png)](/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10.png)

Well, that’s it for this post. Learning this saved me a heap of time on client sites. There is more than likely better ways to do this but this suited me at the time and thought it could help somebody else facing the same issues.



As always please feel free to contact me I’d love to hear better ways of doing things. We’re all here to learn right?



Special thanks to [Allesandro Di Pinto](https://twitter.com/adipinto?lang=en) for [YANP](https://github.com/adipinto/yet-another-nessus-parser).
