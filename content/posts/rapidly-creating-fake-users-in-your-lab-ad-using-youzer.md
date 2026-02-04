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
---
<h2 class="wp-block-heading">Penetration Testing Lab</h2>



<p>Whether you have a fully virtual organisation consisting of several different machines or the odd virtualised box you’re using to explore or freshen up on certain skills. They’re great fun and an asset to any security tester.</p>



<div class="wp-block-media-text alignfull is-stacked-on-mobile" style="grid-template-columns:57% auto"><figure class="wp-block-media-text__media"><img decoding="async" width="1024" height="767" src="/wp-content/uploads/2019/06/IMG_2874-1024x767.jpg" alt="" class="wp-image-1953 size-full" srcset="/wp-content/uploads/2019/06/IMG_2874-1024x767.jpg 1024w, /wp-content/uploads/2019/06/IMG_2874-300x225.jpg 300w, /wp-content/uploads/2019/06/IMG_2874-768x575.jpg 768w, /wp-content/uploads/2019/06/IMG_2874-1536x1151.jpg 1536w, /wp-content/uploads/2019/06/IMG_2874-2048x1534.jpg 2048w, /wp-content/uploads/2019/06/IMG_2874-1568x1175.jpg 1568w" sizes="(max-width: 1024px) 100vw, 1024px"></figure><div class="wp-block-media-text__content">
<p class="has-large-font-size">Having your own lab is a great way to perform security testing techniques in a controlled environment.</p>
</div></div>



<p>If you’re attempting to build out a lab that replicates a real organisation it’s always good to do things properly. Let’s assume for this post that you’ve already built a Windows Domain Controller for your penetration testing lab.</p>



<p>You now need to create those virtual employees within Active Directory. Creating a few different accounts here and there is a relatively easy task I agree, but what if you want your virtual organisation to consist of hundreds of different users in different departments or organisational units, especially with real<g class="gr_ gr_6 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del multiReplace gr-progress" id="6" data-gr-id="6">–</g>world passwords? Having an Active Directory full of users can be useful for a number of activities one being the extraction of NTDS.dit and practising cracking techniques on any hashes that it may contain.</p>



<h2 class="wp-block-heading">Enter Youzer</h2>



<figure class="wp-block-gallery columns-1 is-cropped wp-block-gallery-1 is-layout-flex wp-block-gallery-is-layout-flex"><ul class="blocks-gallery-grid"><li class="blocks-gallery-item"><figure><img loading="lazy" decoding="async" width="696" height="558" src="/wp-content/uploads/2019/06/youzer.png" alt="" data-id="1939" data-link="https://stealingthe.network/?attachment_id=1939" class="wp-image-1939" srcset="/wp-content/uploads/2019/06/youzer.png 696w, /wp-content/uploads/2019/06/youzer-300x241.png 300w" sizes="auto, (max-width: 696px) 100vw, 696px"></figure></li></ul></figure>



<p>Creating hundreds or even thousands of users is now achievable quickly and simply thanks to a tool called <a href="https://github.com/SpiderLabs/youzer">Youzer</a>.</p>



<p>Youzer was written by  <a href="https://www.twitter.com/lorentzenman">Matt Lorentzen</a> an ex-colleague of mine and an absolute brain on legs he describes Youzer’s goal on its GitHub page –</p>



<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow"><p> The goal of Youzer is to create information-rich Active Directory environments. This uses the python3 library ‘faker’ to generate random accounts. </p><p> You can either supply a wordlist or have the passwords generated. <g class="gr_ gr_18 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="18" data-gr-id="18">The  generated</g> option is great for testing things like <g class="gr_ gr_14 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling" id="14" data-gr-id="14">hashcat</g> rule masks.  Wordlist option is useful when wanting to supply a specific password <g class="gr_ gr_20 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="20" data-gr-id="20">l</g>ist seeded into an <g class="gr_ gr_16 gr-alert gr_gramm gr_inline_cards gr_run_anim Punctuation only-del replaceWithoutSep" id="16" data-gr-id="16">environment,</g> or to practice dictionary attacks.<br> The output is a CSV and a PowerShell script where both can be <g class="gr_ gr_19 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="19" data-gr-id="19">copied  to</g> the target. When executed, the PowerShell script binds over LDAP <g class="gr_ gr_21 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="21" data-gr-id="21">so  doesn’t</g> rely on the newer Active Directory modules and creates each <g class="gr_ gr_22 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="22" data-gr-id="22">user  object</g>. Currently the OU’s need to exist, but this tool is a  sub-project of ‘Labseed’ where the Active Directory structure will <g class="gr_ gr_23 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="23" data-gr-id="23">be  created</g>. </p><cite>https://github.com/SpiderLabs/youzer</cite></blockquote>



<h2 class="wp-block-heading">Prerequisites</h2>



<p>Ok, so you want to give Youzer a try on your newly created Domain Controller for your lab? There are a few pre-requisites that we need to install before we can proceed.</p>



<p>For our environment, I used Microsoft Windows 2012 for reasons. We also need to install the following.</p>



<p>The first being Python 3 – <a href="https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe">https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe</a></p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="703" height="440" src="/wp-content/uploads/2019/06/python3-variables.png" alt="" class="wp-image-1935" srcset="/wp-content/uploads/2019/06/python3-variables.png 703w, /wp-content/uploads/2019/06/python3-variables-300x188.png 300w" sizes="auto, (max-width: 703px) 100vw, 703px"><figcaption> Figure 1: When installing Python ensure you tick the box which says “Add Python to environment variables” </figcaption></figure></div>



<p>Once Python3 has been successfully installed we need to install the “faker” python library by issuing the following command from a command shell/powershell instance.</p>



<pre class="wp-block-code"><code>PS C:\Users\Administrator\&gt; pip3 install faker</code></pre>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="771" height="470" src="/wp-content/uploads/2019/06/pip-install-faker.png" alt="" class="wp-image-1938" srcset="/wp-content/uploads/2019/06/pip-install-faker.png 771w, /wp-content/uploads/2019/06/pip-install-faker-300x183.png 300w, /wp-content/uploads/2019/06/pip-install-faker-768x468.png 768w" sizes="auto, (max-width: 771px) 100vw, 771px"><figcaption>Figure 2: Installation of the Faker Python library</figcaption></figure></div>



<p>Now the faker library is installed we can move on to grabbing a password list for Youzer to utilise when generating the users passwords.</p>



<p>A good place to start is <a href="https://twitter.com/danielmiessler">Daniel Miessler’s</a> github which has a great selection of <a href="https://github.com/danielmiessler/SecLists/tree/master/Passwords/Common-Credentials">Common Passwords</a>. When using Youzer for the first time I grabbed the <a href="https://github.com/danielmiessler/SecLists/blob/master/Passwords/probable-v2-top12000.txt">Probable Top 12000</a>.</p>



<h2 class="wp-block-heading">Generating Users</h2>



<p>We’re now ready to start generating Youzers (see what I did there?), hopefully, by now you have created some organisational units within Active Directory. I created IT, Sales and Management <g class="gr_ gr_6 gr-alert gr_gramm gr_inline_cards gr_run_anim Grammar multiReplace" id="6" data-gr-id="6">OU’s</g> for our company’s training envir<g class="gr_ gr_4 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del multiReplace" id="4" data-gr-id="4">o</g>nment.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="628" height="402" src="/wp-content/uploads/2019/06/evilcorp-ou.png" alt="" class="wp-image-1936" srcset="/wp-content/uploads/2019/06/evilcorp-ou.png 628w, /wp-content/uploads/2019/06/evilcorp-ou-300x192.png 300w" sizes="auto, (max-width: 628px) 100vw, 628px"><figcaption>Figure 3: The newly created Sales OU</figcaption></figure></div>



<p>Let’s fire up Youzer and give it some parameters which I will explain…</p>



<pre class="wp-block-code"><code>PS C:\Users\Administrator\Downloads\youzer-master\&gt; python youzer.py --wordlist probable-v2-top12000.txt --ou "ou=Sales,dc=EVILCORP,dc=local" --domain EVILCORP --users 500 --output sales-users.csv</code></pre>



<p>Above we’ve run the Youzer script telling it the following:</p>



<p><strong>–wordlist</strong> – Where our password list is located</p>



<p><strong>–ou</strong> – The path to our Active Directory Organisational Unit</p>



<p><strong>–domain</strong> – Our Domain</p>



<p><strong>–users</strong> – How many users we’d like to generate</p>



<p><strong>–output</strong> – The name of the CSV file we want to dump the data into, Youzer will then create a PowerShell script of the same name for you to run.</p>



<p>Youzer should have now generated your fake users.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="987" height="342" src="/wp-content/uploads/2019/06/user-gen.png" alt="" class="wp-image-1937" srcset="/wp-content/uploads/2019/06/user-gen.png 987w, /wp-content/uploads/2019/06/user-gen-300x104.png 300w, /wp-content/uploads/2019/06/user-gen-768x266.png 768w" sizes="auto, (max-width: 987px) 100vw, 987px"><figcaption>Figure 4: Youzer being run from our PowerShell prompt.</figcaption></figure></div>



<p>Our output file should have also been populated with all of our newly generated users, Youzer would have also generated a PowerShell script to automate the task of taking these users and populating Active Directory.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="746" height="639" src="/wp-content/uploads/2019/06/users-csv.png" alt="" class="wp-image-1942" srcset="/wp-content/uploads/2019/06/users-csv.png 746w, /wp-content/uploads/2019/06/users-csv-300x257.png 300w" sizes="auto, (max-width: 746px) 100vw, 746px"><figcaption>Figure 6: Our newly created sales-users.csv</figcaption></figure></div>



<h2 class="wp-block-heading">Populating Active Directory</h2>



<p>Now our users have been generated and the needed files created we can go ahead and launch the PowerShell script which Youzer created for us in order to populate our Active Directory.</p>



<pre class="wp-block-code"><code>PS C:\Users\Administrator\Downloads\youzer-master&gt; .\sales-users.ps1</code></pre>



<figure class="wp-block-image"><img loading="lazy" decoding="async" width="693" height="644" src="/wp-content/uploads/2019/06/user-gen2-1.png" alt="" class="wp-image-1944" srcset="/wp-content/uploads/2019/06/user-gen2-1.png 693w, /wp-content/uploads/2019/06/user-gen2-1-300x279.png 300w" sizes="auto, (max-width: 693px) 100vw, 693px"><figcaption>Figure 7: Youzer populating Active Directory</figcaption></figure>



<p>Voila, 500 users created with passwords supplied via our wordlist in a matter of minutes. </p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="670" height="667" src="/wp-content/uploads/2019/06/populated-ad.png" alt="" class="wp-image-1945" srcset="/wp-content/uploads/2019/06/populated-ad.png 670w, /wp-content/uploads/2019/06/populated-ad-300x300.png 300w, /wp-content/uploads/2019/06/populated-ad-150x150.png 150w" sizes="auto, (max-width: 670px) 100vw, 670px"><figcaption>Figure 8: Populated Active Directory</figcaption></figure></div>



<p>That brings us to the end of this post, I hope you found the information valuable the tool really does save time and has great potential. Having spoken to Matt Lorentzen he has some great plans coming in the near future so make sure to star the project in GitHub and keep up to date with any new developments.</p>



<p>Until next time.</p>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&amp;text=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer&amp;body=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&amp;title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&amp;title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&amp;title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/&amp;title=Rapidly%20Creating%20Fake%20Users%20in%20your%20Lab%20AD%20using%20Youzer" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/wardriving-with-kismet-gps-and-google-earth/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Wardriving with Kismet, GPS and Google Earth.</span></a></div><div class="nav-next"><a href="/posts/summary-of-the-cups-vulnerability/" rel="next"><span>Summary of the CUPS Vulnerability </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
