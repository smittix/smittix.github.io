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
---
<h2 class="wp-block-heading">Intro</h2>



<p>I have had a few people over the last couple of months asking me how to get <a href="https://github.com/BloodHoundAD/Bloodhound/wiki">Bloodhound</a> up and running after I had sung its praise since seeing the “Six Degrees to Domain Admin” video from BSIDES Las Vegas. If you still haven’t seen the video I am referring to I suggest you take a peek before proceeding.</p>


<p>[su_youtube url=”https://www.youtube.com/watch?v=lxd2rerVsLo”]</p>



<p>It really is such an awesome tool and I highly recommend it to not only info-sec professionals but to anyone who administrates an Active Directory environment.</p>



<p>The awesome news is that <a href="https://github.com/BloodHoundAD/Bloodhound/wiki">Bloodhound</a> is now in the Kali Linux repository’s and is super easy to install and get up and running and I will show you how.</p>



<h2 class="wp-block-heading">Ensure an up-to-date system.</h2>



<p>Firstly, please ensure you’re running the latest and greatest by performing a dist-upgrade like so.</p>



<pre class="wp-block-code lang:default decode:true"><code>apt-get update</code></pre>



<p>and then</p>



<pre class="wp-block-code lang:default decode:true"><code>apt-get dist-upgrade</code></pre>



<h2 class="wp-block-heading">Installing Bloodhound</h2>



<p>You guessed it, simply run the following. Bloodhound depends on neo4j so that will be installed as well.</p>



<pre class="wp-block-code lang:default decode:true"><code>apt-get install bloodhound</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14.png"><img decoding="async" width="862" height="616" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14.png" alt="" class="wp-image-1451" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14.png 862w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14-300x214.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-11-38-14-768x549.png 768w" sizes="(max-width: 862px) 100vw, 862px"></a></figure>



<h2 class="wp-block-heading">Change the Default Password for Neo4j</h2>



<p>We really should change the default password for Neo4j, you know.. For reasons.</p>



<p>Let’s launch Neo4j</p>



<pre class="wp-block-code lang:default decode:true"><code>neo4j console</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41.png"><img loading="lazy" decoding="async" width="912" height="390" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41.png" alt="" class="wp-image-1454" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41.png 912w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41-300x128.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-24-41-768x328.png 768w" sizes="auto, (max-width: 912px) 100vw, 912px"></a></figure>



<p>We now have a remote interface available at <strong>http://localhost:7474</strong>. Let’s head over there via a browser and change that default password. You will also see that it enabled Bolt on the localhost, we need this for later.</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13.png"><img loading="lazy" decoding="async" width="709" height="298" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13.png" alt="" class="wp-image-1455" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13.png 709w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-13-300x126.png 300w" sizes="auto, (max-width: 709px) 100vw, 709px"></a></figure>



<p>Login with the default credentials (below) you will then be asked to change the password :-</p>



<ul class="wp-block-list"><li>Username: <strong>neo4j</strong></li><li>Password: <strong>neo4j</strong></li></ul>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27.png"><img loading="lazy" decoding="async" width="656" height="215" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27.png" alt="" class="wp-image-1456" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27.png 656w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-25-27-300x98.png 300w" sizes="auto, (max-width: 656px) 100vw, 656px"></a></figure>



<p>Go ahead and complete the password change and close the browser window.</p>



<h2 class="wp-block-heading">Let the Hound See The Blood</h2>



<p>Pop a new terminal window open and run the following command to launch Bloodhound, leave the Neo4j console running for obvious reasons.</p>



<pre class="wp-block-code lang:default decode:true"><code>bloodhound</code></pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43.png"><img loading="lazy" decoding="async" width="415" height="414" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43.png" alt="" class="wp-image-1457" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43.png 415w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43-300x300.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-26-43-150x150.png 150w" sizes="auto, (max-width: 415px) 100vw, 415px"></a></figure>



<p>As you can see, Bloodhound is now running and waiting for some user input. Earlier when launching Neo4j it also enabled Bolt on bolt://127.0.0.1:7687. You need to use this as your Database URL.</p>



<ul class="wp-block-list"><li>Database URL – <strong>bolt://127.0.0.1:7687</strong></li><li>Username – <strong>neo4j</strong></li><li>Password – <strong>your newly changed password</strong></li></ul>



<p>Hit login and you should be presented with the Bloodhound tool minus any data. You can now import your data and get analyzing.</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17.png"><img loading="lazy" decoding="async" width="806" height="696" src="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17.png" alt="" class="wp-image-1458" srcset="/wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17.png 806w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17-300x259.png 300w, /wp-content/uploads/2017/10/Screenshot-from-2017-10-11-12-27-17-768x663.png 768w" sizes="auto, (max-width: 806px) 100vw, 806px"></a></figure>



<p>Hopefully this was a nice and quick guide to help anyone out there having any issues getting up and running with the awesome tool that is Bloodhound.</p>



<p>I also want to take a moment to thank <a href="https://www.twitter.com/_wald0">@_wald0</a>, <a href="https://twitter.com/CptJesus">@CptJesus</a>, and <a href="https://twitter.com/harmj0y">@harmj0y </a>for their continued hard work on this amazing project.</p>



<p>Cheers Guys!</p>



<figure class="wp-block-image"><a href="https://github.com/BloodHoundAD/Bloodhound/wiki"><img loading="lazy" decoding="async" width="300" height="169" src="/wp-content/uploads/2017/10/Bloodhound-Dogs-300x169.jpg" alt="" class="wp-image-1460" srcset="/wp-content/uploads/2017/10/Bloodhound-Dogs-300x169.jpg 300w, /wp-content/uploads/2017/10/Bloodhound-Dogs.jpg 625w" sizes="auto, (max-width: 300px) 100vw, 300px"></a></figure>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/quick-guide-to-installing-bloodhound-in-kali-rolling/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&amp;text=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling&amp;body=/quick-guide-to-installing-bloodhound-in-kali-rolling/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&amp;title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&amp;title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/quick-guide-to-installing-bloodhound-in-kali-rolling/&amp;title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/quick-guide-to-installing-bloodhound-in-kali-rolling/&amp;title=Quick%20Guide%20to%20Installing%20Bloodhound%20in%20Kali-Rolling" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/exploiting-ms17-010-using-eternalblue-and-doublepulsar-to-gain-a-remote-meterpreter-shell/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Exploiting MS17-010 – Using EternalBlue and DoublePulsar to gain a remote Meterpreter shell</span></a></div><div class="nav-next"><a href="/posts/reporting-ssltls-issues-the-easy-way-with-yanp/" rel="next"><span>Reporting SSL/TLS Issues the Easy Way with YANP </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
