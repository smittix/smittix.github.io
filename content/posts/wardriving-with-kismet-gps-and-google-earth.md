---
title: "Wardriving with Kismet, GPS and Google Earth."
date: 2019-03-05
draft: false
toc: false
images:
  - "/wp-content/uploads/2019/03/3d-modern-wifi-router-black-background.jpg"
categories:
  - Linux
  - Security
tags:
  - alfa
  - globalsat
  - gpsd
  - hacking
  - kali
  - linux
  - Penetration Testing
  - wardriving
  - wireless
---
<p>Wardriving was once a really popular sport, I myself loved mapping new areas with my trusty Orinco Gold Card. I’m not sure how popular it is these days but I thought I’d write this guide as I came across my GPS dongle and got set it up in Kali Rolling. I then processed the results and dumped them into a usable format which you can then import into Google Earth. As with all of my guides I hope at least one person finds it useful.</p>



<h2 class="wp-block-heading">The Hardware</h2>



<p>This was all setup on a Lenovo thinkpad with the Kali Rolling distribution, I used the following Wireless and GPS adapters which I’ve also included an Amazon link just in case you wanted to purchase them.</p>



<h2 class="wp-block-heading">Wireless Adapter</h2>



<p>I use an <strong>Alfa AWUS036NHA</strong> which has an Atheros AR9271 chipset and is well supported in Linux it also supports packet injection</p>



<div class="wp-block-image"><figure class="aligncenter is-resized"><img decoding="async" src="https://i1.wp.com/stealingthe.network/wp-content/uploads/2019/03/617wm7-itL._SL1500_-1.jpg?fit=750%2C750&amp;ssl=1" alt="" class="wp-image-1846" width="290" height="290" srcset="/wp-content/uploads/2019/03/617wm7-itL._SL1500_-1.jpg 1500w, /wp-content/uploads/2019/03/617wm7-itL._SL1500_-1-300x300.jpg 300w, /wp-content/uploads/2019/03/617wm7-itL._SL1500_-1-1024x1024.jpg 1024w, /wp-content/uploads/2019/03/617wm7-itL._SL1500_-1-150x150.jpg 150w, /wp-content/uploads/2019/03/617wm7-itL._SL1500_-1-768x768.jpg 768w" sizes="(max-width: 290px) 100vw, 290px"></figure></div>



<p class="has-text-align-center"><strong>Amazon</strong> – <a href="https://www.amazon.co.uk/Network-AWUS036NHA-Adapter-150-Mbps-802-11b/dp/B004Y6MIXS/ref=sr_1_3?ie=UTF8&amp;qid=1551723601&amp;sr=8-3&amp;keywords=alfa+awus036nh%EF%BB%BF">LINK</a><br></p>



<h2 class="wp-block-heading">GPS Receiver</h2>



<hr class="wp-block-separator">



<p>For the GPS side of things I chose to use the <strong>GlobalSat BU-353-S4 receiver</strong>. Purely because it’s well supported under Linux and Kismet.</p>



<div class="wp-block-image"><figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://i1.wp.com/stealingthe.network/wp-content/uploads/2019/03/GlobalSat-BU353-S4_-1.jpg?fit=750%2C784&amp;ssl=1" alt="" class="wp-image-1845" width="193" height="201" srcset="/wp-content/uploads/2019/03/GlobalSat-BU353-S4_-1.jpg 1436w, /wp-content/uploads/2019/03/GlobalSat-BU353-S4_-1-287x300.jpg 287w, /wp-content/uploads/2019/03/GlobalSat-BU353-S4_-1-980x1024.jpg 980w, /wp-content/uploads/2019/03/GlobalSat-BU353-S4_-1-768x802.jpg 768w" sizes="auto, (max-width: 193px) 100vw, 193px"></figure></div>



<p class="has-text-align-center"><strong>Amazon </strong>– <a href="https://www.amazon.co.uk/GlobalSat-BU-353-S4-Receiver-SiRF-Black/dp/B008200LHW/ref=sr_1_1?ie=UTF8&amp;qid=1551724582&amp;sr=8-1&amp;keywords=globalsat+bu-353-s4+usb+gps+receiver%EF%BB%BF">Link</a></p>



<div style="height:100px" aria-hidden="true" class="wp-block-spacer"></div>



<h2 class="wp-block-heading">Setting up the devices</h2>



<p>There are some prerequisites we will need to install to get the GPS working which are not installed by default. <br></p>



<pre class="wp-block-code"><code>root@Hunter~# apt install gpsd gpsd-clients<br></code></pre>



<p>Once we have those installed we’re pretty much good to go. Go ahead and plug in your GPS receiver and run the following. My device was located at <strong>/dev/ttyUSB0</strong> but yours maybe different so please check.</p>



<pre class="wp-block-code"><code>root@hunter~# gpsd -n -N -D 2 /dev/ttyUSB0</code></pre>



<ul class="wp-block-list"><li><strong>-n</strong>  Don’t wait for a client to connect before polling whatever GPS is associated with it. Some RS232 GPSes wait in a standby mode (drawing less power) when the host machine is not asserting DTR, and some cellphone and handheld embedded GPSes have similar behaviors. Accordingly, waiting for a watch request to open the device may save battery power. (This capability is rare in consumer-grade devices). </li><li><strong>-N</strong>  Don’t daemonize; run in foreground. This switch is mainly useful for debugging. </li><li><strong>-D 2 </strong> Set debug level. At debug levels 2 and above, gpsd reports incoming sentence and actions to standard error if gpsd is in the foreground (-N) or to syslog if in the background. </li></ul>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="641" height="186" src="/wp-content/uploads/2019/03/GPS-KISMET-3.png" alt="" class="wp-image-1851" srcset="/wp-content/uploads/2019/03/GPS-KISMET-3.png 641w, /wp-content/uploads/2019/03/GPS-KISMET-3-300x87.png 300w" sizes="auto, (max-width: 641px) 100vw, 641px"><figcaption>GPSD up and running with debugging level set to 2.</figcaption></figure></div>



<p>To check whether your GPS receiver has locked onto satellites we can use cpsg which is used to test clients for gpsd, run the following command in a new tab or terminal window</p>



<pre class="wp-block-code"><code>root@hunter~# cgps -s</code></pre>



<ul class="wp-block-list"><li><strong>-s</strong> Be silent (don’t print raw gpsd data)</li></ul>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="542" height="300" src="/wp-content/uploads/2019/03/GPS-KISMET-1.jpg" alt="" class="wp-image-1872" srcset="/wp-content/uploads/2019/03/GPS-KISMET-1.jpg 542w, /wp-content/uploads/2019/03/GPS-KISMET-1-300x166.jpg 300w" sizes="auto, (max-width: 542px) 100vw, 542px"></figure></div>



<p>Like above you should see some relavant details regarding your position, heading and speed. If you don’t see something like the above then something has gone wrong.<br></p>



<div style="height:100px" aria-hidden="true" class="wp-block-spacer"></div>



<h2 class="wp-block-heading">On to Kismet</h2>



<p>Ok, now that we know our GPS receiver is working fine from the above steps let’s launch Kismet and start collecting data.</p>



<pre class="wp-block-code"><code>root@hunter~# kismet<br></code></pre>



<h2 class="wp-block-heading">Initial Steps</h2>



<p>You will be asked a few questions when launching kismet which are pretty straight forward.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="513" height="200" src="/wp-content/uploads/2019/03/GPS-KISMET-5.png" alt="" class="wp-image-1853" srcset="/wp-content/uploads/2019/03/GPS-KISMET-5.png 513w, /wp-content/uploads/2019/03/GPS-KISMET-5-300x117.png 300w" sizes="auto, (max-width: 513px) 100vw, 513px"></figure></div>



<p class="has-text-align-center">I was running my kali instance as root so if you are too you can ignore this and hit OK.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="444" height="138" src="/wp-content/uploads/2019/03/GPS-KISMET-6.png" alt="" class="wp-image-1854" srcset="/wp-content/uploads/2019/03/GPS-KISMET-6.png 444w, /wp-content/uploads/2019/03/GPS-KISMET-6-300x93.png 300w" sizes="auto, (max-width: 444px) 100vw, 444px"></figure></div>



<p class="has-text-align-center">Select Yes here.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="356" height="173" src="/wp-content/uploads/2019/03/GPS-KISMET-7.png" alt="" class="wp-image-1855" srcset="/wp-content/uploads/2019/03/GPS-KISMET-7.png 356w, /wp-content/uploads/2019/03/GPS-KISMET-7-300x146.png 300w" sizes="auto, (max-width: 356px) 100vw, 356px"></figure></div>



<p class="has-text-align-center">You can either change these or leave them as default. Next you will see a console window which you can close. You will then be asked to add a source which will be the name of your Wireless Device in my case it was wlan1</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="431" height="154" src="/wp-content/uploads/2019/03/GPS-KISMET-8.png" alt="" class="wp-image-1856" srcset="/wp-content/uploads/2019/03/GPS-KISMET-8.png 431w, /wp-content/uploads/2019/03/GPS-KISMET-8-300x107.png 300w" sizes="auto, (max-width: 431px) 100vw, 431px"></figure></div>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="326" height="188" src="/wp-content/uploads/2019/03/GPS-KISMET-9.png" alt="" class="wp-image-1857" srcset="/wp-content/uploads/2019/03/GPS-KISMET-9.png 326w, /wp-content/uploads/2019/03/GPS-KISMET-9-300x173.png 300w" sizes="auto, (max-width: 326px) 100vw, 326px"></figure></div>



<p class="has-text-align-center">Once you have entered the correct device name select “Add”</p>



<div style="height:100px" aria-hidden="true" class="wp-block-spacer"></div>



<h2 class="wp-block-heading">Capturing</h2>



<p>After the above steps you should now start seeing Kismet being populated with any Wireless SSID’s that it’s detected. Similar to the screenshot below. If it is then well done you’re successfully capturing wireless data.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="312" height="646" src="/wp-content/uploads/2019/03/GPS-KISMET-10-1.png" alt="" class="wp-image-1859" srcset="/wp-content/uploads/2019/03/GPS-KISMET-10-1.png 312w, /wp-content/uploads/2019/03/GPS-KISMET-10-1-145x300.png 145w" sizes="auto, (max-width: 312px) 100vw, 312px"></figure></div>



<div style="height:100px" aria-hidden="true" class="wp-block-spacer"></div>



<h2 class="wp-block-heading">Handling the Data</h2>



<p>GISKismet is a wireless recon visualisation tool to represent the data gathered using Kismet, we can use this tool to import our captured data and then export into a format which is usable with GoogleEarth so we can visualise our Wardrive. Let’s go ahead and issue our command. </p>



<pre class="wp-block-code"><code>root@hunter~# giskismet -x Kismet-(YOURFILE HERE).netxml<br></code></pre>



<p>We use the <strong>-x</strong> switch to tell the tool we’re importing an XML file, ensure you enter your capture file with the <strong>.netxml extension</strong>.</p>



<div class="wp-block-image"><figure class="aligncenter"><img loading="lazy" decoding="async" width="434" height="629" src="/wp-content/uploads/2019/03/GPS-KISMET-11.png" alt="" class="wp-image-1860" srcset="/wp-content/uploads/2019/03/GPS-KISMET-11.png 434w, /wp-content/uploads/2019/03/GPS-KISMET-11-207x300.png 207w" sizes="auto, (max-width: 434px) 100vw, 434px"></figure></div>



<p>Now we’ve imported our captured data into GISKismet’s SQLite database we can now grab that data by performing a simple SQL query and exporting it into a kml file which is usable by GoogleEarth. Obviously name your ouput file anything you like.</p>



<pre class="wp-block-code"><code>root@hunter~# giskismet -q "SELECT * FROM wireless" -o YOURNEWFILE.kml</code></pre>



<ul class="wp-block-list"><li><strong>-q</strong> Query</li><li><strong>-o</strong> Output file</li></ul>



<p>So we now have our newly created .kml file which you can open using GoogleEarth and you should have similar results to the below screenshot.</p>



<div class="wp-block-image is-style-default"><figure class="aligncenter"><img loading="lazy" decoding="async" width="1024" height="822" src="/wp-content/uploads/2019/03/Wireless-Map-1024x822.png" alt="" class="wp-image-1874" srcset="/wp-content/uploads/2019/03/Wireless-Map-1024x822.png 1024w, /wp-content/uploads/2019/03/Wireless-Map-300x241.png 300w, /wp-content/uploads/2019/03/Wireless-Map-768x616.png 768w, /wp-content/uploads/2019/03/Wireless-Map.png 1054w" sizes="auto, (max-width: 1024px) 100vw, 1024px"></figure></div>



<p>I Hope you found this guide quick, to the point and most of all helpful.<br></p>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/wardriving-with-kismet-gps-and-google-earth/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/wardriving-with-kismet-gps-and-google-earth/&amp;text=Wardriving%20with%20Kismet%2C%20GPS%20and%20Google%20Earth." target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Wardriving%20with%20Kismet,%20GPS%20and%20Google%20Earth.&amp;body=/wardriving-with-kismet-gps-and-google-earth/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/wardriving-with-kismet-gps-and-google-earth/&amp;title=Wardriving%20with%20Kismet%2C%20GPS%20and%20Google%20Earth." target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/wardriving-with-kismet-gps-and-google-earth/&amp;title=Wardriving%20with%20Kismet%2C%20GPS%20and%20Google%20Earth." target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/wardriving-with-kismet-gps-and-google-earth/&amp;title=Wardriving%20with%20Kismet%2C%20GPS%20and%20Google%20Earth." target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/wardriving-with-kismet-gps-and-google-earth/&amp;title=Wardriving%20with%20Kismet%2C%20GPS%20and%20Google%20Earth." target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Efficient Time Based Blind SQL Injection using MySQL Bit Functions and Operators</span></a></div><div class="nav-next"><a href="/posts/rapidly-creating-fake-users-in-your-lab-ad-using-youzer/" rel="next"><span>Rapidly Creating Fake Users in your Lab AD using Youzer </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
