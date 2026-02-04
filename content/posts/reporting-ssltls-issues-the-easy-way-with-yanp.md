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
---
<p>What’s YANP I hear you ask? <a href="https://github.com/adipinto/yet-another-nessus-parser">YANP</a> stands for “Yet Another Nessus Parser” written by <a href="https://twitter.com/adipinto">Alessandro Di Pinto</a> and I’m over the moon that I found it. I’ll tell you why.</p>



<p>Getting all the IP addresses and ports together from Nessus to stick them into other tools such as TestSSL.sh or SSLScan can take away valuable time on large engagements when the time could be spent looking into more harder to detect vulnerabilities. Ultimately we have a duty to our clients to report all our findings and it’s just another thing that needs to be done.</p>



<p>But.. Just because it needs to be done doesn’t mean we can’t get it done quicker and that’s where <a href="https://github.com/adipinto/yet-another-nessus-parser">YANP</a> comes into play.</p>



<h2 class="wp-block-heading">What is YANP?</h2>



<p>The below is taken from the projects GitHub page.</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow"><p>Yet Another Nessus Parser (<a href="https://github.com/adipinto/yet-another-nessus-parser">YANP</a>) is a parser able to extract information from Tenable Nessus’s .nessus file format. The main tool’s objective is to export vulnerability assessment reports in a parsable way. The user is able to choose an appropriate output format in order to save the Nessus’ reports following various advanced needs.</p></blockquote>



<p>Here I will show you my flow of getting the SSL/TLS issues pulled out of Nessus easily and quickly , I’m writing this to help anyone who finds it time-consuming too. By all means if anyone out there reading this has a better way of doing things please let me know, your comments will be much appreciated.</p>



<h2 class="wp-block-heading">Installing YANP</h2>



<p>Any tools I get from GitHub I stick in my /home/james/tools directory. Let’s git clone YANP to our local system please change the path to a directory of your choosing.</p>



<h6 class="wp-block-heading">These steps were taken on a kali-linux virtual machine and assumes you have both git and python installed.</h6>



<pre class="wp-block-code lang:default decode:true"><code>cd /home/james/tools</code></pre>



<pre class="wp-block-code lang:default decode:true"><code>git clone https://github.com/adipinto/yet-another-nessus-parser</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17.png"><img decoding="async" width="902" height="149" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17.png" alt="" class="wp-image-1474" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17.png 902w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17-300x50.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-14-50-17-768x127.png 768w" sizes="(max-width: 902px) 100vw, 902px"></a></figure>



<pre class="wp-block-code lang:default decode:true"><code>cd yet-another-nessus-parser</code></pre>



<p>OK so we’ve cloned the repository and we’re now sitting in the YANP directory.</p>



<h2 class="wp-block-heading">Parsing for Use with Other Tools</h2>



<p>Now it’s time to parse your .nessus file to gain the information we need. As you know, this tutorial is aimed at SSL/TLS issues but you can parse the file for any issues of your choosing.</p>



<p>Let’s have a look the options first.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52.png"><img loading="lazy" decoding="async" width="715" height="128" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52.png" alt="" class="wp-image-1481" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52.png 715w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-32-52-300x54.png 300w" sizes="auto, (max-width: 715px) 100vw, 715px"></a></figure>



<p>As you can see there are a few options that we can use, we can search using the specific PluginID or PluginName for example. In this instance I’m going to search for “SSL” using the PluginName option using the “-d” switch. We’re also going to tell YANP where our .nessus file is using the “-i” switch.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus -d SSL</code></pre>



<p>You should see some output similar to mine below.</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59.png"><img loading="lazy" decoding="async" width="1048" height="838" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59.png" alt="" class="wp-image-1475" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59.png 1048w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59-300x240.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59-1024x819.png 1024w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-43-59-768x614.png 768w" sizes="auto, (max-width: 1048px) 100vw, 1048px"></a></figure>



<p>That’s all good and well but we can’t use that for anything except viewing it really. Let’s clean it up a little by piping it to “cut”. You will see I have set the delimiter to a space (-d ” “) and chosen the first field (-f 1) to grab just the list of IP addresses and ports.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus -d SSL | cut -d " " -f 1</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01.png"><img loading="lazy" decoding="async" width="564" height="640" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01.png" alt="" class="wp-image-1476" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01.png 564w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-15-48-01-264x300.png 264w" sizes="auto, (max-width: 564px) 100vw, 564px"></a></figure>



<p>That’s a little better but I see there are duplicates in there too. Let’s use “sort” and “uniq” to remove the duplicated entries.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus -d SSL |cut -d " " -f 1| sort | uniq</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37.png"><img loading="lazy" decoding="async" width="572" height="927" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37.png" alt="" class="wp-image-1478" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37.png 572w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-38-37-185x300.png 185w" sizes="auto, (max-width: 572px) 100vw, 572px"></a></figure>



<p>That looks a lot better and we now have usable list that we can pass to other tools for analysis but first let’s transfer this data into a file.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus -d SSL | cut -d " " -f 1 | sort | uniq &gt; ssl-issues.txt</code></pre>



<p>You can now use your newly created ssl-issues.txt file as the target list for other tools such as SSLScan. Again, you can search for anything via the PluginName or PluginID switches and output whatever you need.</p>



<h2 class="wp-block-heading">Parsing for Specific Issues</h2>



<p>You should now get the gist of what YANP can do and you’re probably coming up with your own ideas of how to use it. For this next example we will search for a specific SSL issue such as “BEAST” , we will get the affected host details for later use in our report.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus -d BEAST</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02.png"><img loading="lazy" decoding="async" width="1163" height="476" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02.png" alt="" class="wp-image-1480" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02.png 1163w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02-300x123.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02-1024x419.png 1024w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-39-02-768x314.png 768w" sizes="auto, (max-width: 1163px) 100vw, 1163px"></a></figure>



<p>You now see a list that we can use for our report which tells the client the IP, Port and Issue.</p>



<h2 class="wp-block-heading"> </h2>



<h2 class="wp-block-heading">Parsing to CSV</h2>



<p>We can also parse the .nessus file and create a CSV for later manipulation using the following command which will create the file “issues.csv”.</p>



<pre class="wp-block-code lang:default decode:true"><code>python yanp.py -i /home/james/Downloads/sample.nessus --csv issues</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10.png"><img loading="lazy" decoding="async" width="536" height="189" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10.png" alt="" class="wp-image-1479" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10.png 536w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-12-16-25-10-300x106.png 300w" sizes="auto, (max-width: 536px) 100vw, 536px"></a></figure>



<p>Well, that’s it for this post. Learning this saved me a heap of time on client sites. There is more than likely better ways to do this but this suited me at the time and thought it could help somebody else facing the same issues.</p>



<p>As always please feel free to contact me I’d love to hear better ways of doing things. We’re all here to learn right?</p>



<p>Special thanks to <a href="https://twitter.com/adipinto?lang=en">Allesandro Di Pinto</a> for <a href="https://github.com/adipinto/yet-another-nessus-parser">YANP</a>.</p>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/reporting-ssltls-issues-the-easy-way-with-yanp/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/reporting-ssltls-issues-the-easy-way-with-yanp/&amp;text=Reporting%20SSL%2FTLS%20Issues%20the%20Easy%20Way%20with%20YANP" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Reporting%20SSL/TLS%20Issues%20the%20Easy%20Way%20with%20YANP&amp;body=/reporting-ssltls-issues-the-easy-way-with-yanp/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/reporting-ssltls-issues-the-easy-way-with-yanp/&amp;title=Reporting%20SSL%2FTLS%20Issues%20the%20Easy%20Way%20with%20YANP" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/reporting-ssltls-issues-the-easy-way-with-yanp/&amp;title=Reporting%20SSL%2FTLS%20Issues%20the%20Easy%20Way%20with%20YANP" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/reporting-ssltls-issues-the-easy-way-with-yanp/&amp;title=Reporting%20SSL%2FTLS%20Issues%20the%20Easy%20Way%20with%20YANP" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/reporting-ssltls-issues-the-easy-way-with-yanp/&amp;title=Reporting%20SSL%2FTLS%20Issues%20the%20Easy%20Way%20with%20YANP" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/quick-guide-to-installing-bloodhound-in-kali-rolling/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Quick Guide to Installing Bloodhound in Kali-Rolling</span></a></div><div class="nav-next"><a href="/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-1-of-2/" rel="next"><span>Executing Metasploit &amp; Empire Payloads from MS Office Document Properties (part 1 of 2) </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
