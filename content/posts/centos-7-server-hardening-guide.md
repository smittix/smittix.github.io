---
title: "CentOS 7 Server Hardening Guide"
date: 2016-02-03
draft: false
toc: false
images:
  - "/wp-content/uploads/2016/02/kissclipart-linux-centos-clipart-centos-red-hat-enterprise-lin-b2486c5cdad38a37.png"
categories:
  - Fedora
  - Linux
  - Security
tags:
  - apache
  - attackers
  - centos
  - fail2ban
  - fedora
  - hackers
  - hard
  - hardening
  - how
  - httpd
  - redhat
  - rhel
  - secure
  - securing
  - server
  - ssh
  - to
---
<p>So… you’ve just setup a shiny new server and you want to take measures to keep the bad guys out? Well, here I will give you a few tips on how to do just that.</p>



<p>This guide was written with <a href="https://www.centos.org/" target="_blank" rel="noopener">CentOS 7.1</a> in mind but other up-to-date variants such as <a href="https://getfedora.org/" target="_blank" rel="noopener">Fedora</a> and <a href="https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux" target="_blank" rel="noopener">RHEL</a> should be pretty similar if not the same.</p>



<h1 class="wp-block-heading">Hardening SSH (Secure Shell)</h1>



<div class="wp-block-image"><figure class="aligncenter"><a href="/wp-content/uploads/2016/02/OpenSSH.png" rel="attachment wp-att-499"><img decoding="async" width="150" height="150" src="/wp-content/uploads/2016/02/OpenSSH-150x150.png" alt="OpenSSH" class="wp-image-499"></a></figure></div>



<p>Most of you will be using this protocol as a means to remotely administrate your Linux server and your right to. Using SSH is by far the best method to administer your server due to its use of encrypted communications unlike it’s older cousins rlogin and telnet which provide no secure methods of communication.</p>



<h2 class="wp-block-heading">Create a standard user</h2>



<p>Use the ‘useradd’ command to add a username of your choice.</p>



<pre class="wp-block-code lang:default decode:true"><code>useradd YOURUSER</code></pre>



<p>Set a password for your newly created user.</p>



<pre class="wp-block-code lang:default decode:true"><code>passwd YOURUSER</code></pre>



<p>Add your user to the WHEEL group to enable that user to use the sudo command.</p>



<pre class="wp-block-code lang:default decode:true"><code>usermod -aG wheel YOURUSER</code></pre>



<h2 class="wp-block-heading"> </h2>



<h2 class="wp-block-heading">Create an authentication key</h2>



<div class="wp-block-image"><figure class="aligncenter"><a href="/wp-content/uploads/2016/02/RSA.png" rel="attachment wp-att-505"><img loading="lazy" decoding="async" width="300" height="210" src="/wp-content/uploads/2016/02/RSA-300x210.png" alt="RSA" class="wp-image-505" srcset="/wp-content/uploads/2016/02/RSA-300x210.png 300w, /wp-content/uploads/2016/02/RSA.png 382w" sizes="auto, (max-width: 300px) 100vw, 300px"></a></figure></div>



<p>This method of authenticating with your server is much more secure that using a standard password, part of this process will require you to create the key on your local machine which you will be connecting from.</p>



<p>You will be asked if you would like to protect the key with a password, I advise you to do this but it’s not mandatory.</p>



<h3 class="wp-block-heading">Creating the key on Mac/Linux</h3>



<pre class="wp-block-code lang:default decode:true"><code>ssh-keygen -b 4096</code></pre>



<p>Press Enter to use the default names <strong>id_rsa</strong> and <strong>id_rsa.pub</strong> in <strong>/home/your_username/.ssh</strong> before entering your passphrase.</p>



<h3 class="wp-block-heading">Upload your public key to your server</h3>



<h4 class="wp-block-heading">For <strong>Linux</strong></h4>



<pre class="wp-block-code lang:default decode:true"><code>ssh-copy-id YOURUSER@YOURSERVER</code></pre>



<h4 class="wp-block-heading">For <strong>Mac</strong></h4>



<p>On your server do.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo mkdir -p ~/.ssh &amp;&amp; sudo chmod -R 700 ~/.ssh</code></pre>



<p>From your Mac do the following making sure to substitute ‘youruser’ and ‘yourserver’.</p>



<pre class="wp-block-code lang:default decode:true"><code>scp ~/.ssh/id_rsa.pub YOURUSER@YOURSERVER.0:~/.ssh/authorized_keys
</code></pre>



<p>Now on to the configuration changes.</p>



<h3 class="wp-block-heading">Open up the SSH config file for editing</h3>



<p>In this section we will be performing the following actions</p>



<ul class="wp-block-list"><li>Disallowing root logins</li><li>Setting allowed users</li><li>Changing the default port</li><li>Disabling password authentication</li><li>Force protocol 2</li></ul>



<p>You can replace nano with your favourite text editor such as vi.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo nano /etc/ssh/sshd_config</code></pre>



<h4 class="wp-block-heading">Disallow root logins.</h4>



<p>Find the line that says</p>



<pre class="wp-block-code lang:default decode:true"><code>#PermitRootLogin yes</code></pre>



<p>and change it to</p>



<pre class="wp-block-code lang:default decode:true"><code>PermitRootLogin no</code></pre>



<h4 class="wp-block-heading">Setting your user as an allowed user.</h4>



<p>Add the following line to the bottom of your sshd_config file substituting ‘YOURUSER’ with your newly created account.</p>



<pre class="wp-block-code lang:default decode:true"><code>AllowUsers YOURUSER</code></pre>



<h4 class="wp-block-heading">Change the default service port.</h4>



<p>Find the line that says</p>



<pre class="wp-block-code lang:default decode:true"><code>Port 22</code></pre>



<p>Change to something other than 22 such as 22000</p>



<pre class="wp-block-code lang:default decode:true"><code>Port 22000</code></pre>



<h4 class="wp-block-heading">Disabling password authentication</h4>



<p>We can disable password authentication because we will now be using our newly created key pair to authenticate to the server.</p>



<p>Look for the line that has</p>



<pre class="wp-block-code lang:default decode:true"><code>#PasswordAuthentication yes</code></pre>



<p>and replace it with the below line.</p>



<pre class="wp-block-code lang:default decode:true"><code>PasswordAuthentication no</code></pre>



<h4 class="wp-block-heading">Only use SSH protocol 2</h4>



<p>SSH Protocol 1 is generally considered obsolete as it’s vulnerable and old so lets go ahead and only use SSH Protocol 2. Protocol 2 should be enforced by default but it’s worth checking.</p>



<p>Look for the line that says.</p>



<pre class="wp-block-code lang:default decode:true"><code>#Protocol 2</code></pre>



<p>Uncomment the line so it looks like this.</p>



<pre class="wp-block-code lang:default decode:true"><code>Protocol 2</code></pre>



<p>There are many more options that we could set but this should be suffice in securing your SSH Service.</p>



<p>Save the file and restart the SSH service by doing the following.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo systemctl reload sshd.service</code></pre>



<p>You should now be able to login on your chosen port with your authorised keys by connecting like this.</p>



<pre class="wp-block-code lang:default decode:true"><code>ssh YOURUSER@YOURSERVER -p 22000</code></pre>



<h1 class="wp-block-heading"> </h1>



<h1 class="wp-block-heading">Fail2ban</h1>



<p><a href="http://www.fail2ban.org/wiki/index.php/Main_Page" target="_blank" rel="noopener">Fail2ban</a> is a handy tool/service that monitors system log files to detect potential intrusion attempts and places bans using a variety of methods.</p>



<p>To install on <a href="https://www.centos.org/" target="_blank" rel="noopener">CentOS</a> we need to enable the EPEL repository by doing the following.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo yum install epel-release</code></pre>



<p>Once the installation has completed we need to then go ahead and install Fail2ban</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo yum install fail2ban fail2ban-systemd</code></pre>



<p>Fail to ban comes with a wealth of options that would deserve a post all to itself so in this instance we will create a basic configuration file that will help secure your server, especially the SSH service.</p>



<p>Using SELinux? Then you will want to update your policy by doing the following.</p>



<pre class="wp-block-code lang:default decode:true"><code>yum update -y selinux-policy*</code></pre>



<h4 class="wp-block-heading">Configuration</h4>



<p>We will be configuring fail2ban for use with Firewalld as it is implemented by default in CentOS 7.</p>



<p>Create a sshd.local file ready for editing.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo nano /etc/fail2ban/jail.d/sshd.local</code></pre>



<p>Add the following lines.</p>



<pre class="wp-block-code lang:default decode:true"><code>[sshd]
enabled = true
port = 22000
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400</code></pre>



<p>Save the file and go ahead and start fail2ban.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo systemctl enable fail2ban
sudo systemctl start fail2ban
</code></pre>



<p>You should now have a working fail2ban installation which will automatically ban IP addresses after 3 failed attempts at logging in to your system via SSH.</p>



<h1 class="wp-block-heading">Apache Hardening</h1>



<p>The default <a href="http://www.apache.org/" target="_blank" rel="noopener">Apache</a> configuration just works but there’s a few tweaks we can do here and there that makes the bad guys job a little harder. One of the things we can do is try and prevent information leakage.</p>



<p>By default <a href="http://www.apache.org/" target="_blank" rel="noopener">Apache</a> gives out server version information on error pages. We can prevent this by adding a couple of lines to our httpd.conf file.</p>



<h3 class="wp-block-heading">Version banner</h3>



<p>Open up the httpd config file ready for editing.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo nano /etc/httpd/conf/httpd.conf</code></pre>



<p>add the following lines to the bottom of the file</p>



<pre class="wp-block-code lang:default decode:true"><code>ServerTokens Prod
ServerSignature Off</code></pre>



<h3 class="wp-block-heading">Cookies</h3>



<h4 class="wp-block-heading">Trace Requests</h4>



<p>To protect yourself from <a href="https://www.owasp.org/index.php/Cross_Site_Tracing" target="_blank" rel="noopener">Cross Site Tracing</a> attacks append the following line to the end of your configuration file.</p>



<pre class="wp-block-code"><code>TraceEnable off</code></pre>



<h4 class="wp-block-heading">Set the HttpOnly and Secure flag</h4>



<p>To mitigate against most of the common <a href="https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)" target="_blank" rel="noopener">Cross Site Scripting (XSS)</a> attacks you can set the following directive, again add the following line at the end of your configuration file.</p>



<pre class="wp-block-code"><code>Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure</code></pre>



<h4 class="wp-block-heading">X-Frame Options</h4>



<p>Adding this option to your configuration file will indicate whether or not a browser should be allowed to open a webpage in a frame or iframe. This will prevent site content embedded into other sites. See – <a href="https://www.owasp.org/index.php/Clickjacking" target="_blank" rel="noopener">https://www.owasp.org/index.php/Clickjacking</a>.</p>



<p>Append the following line in your configuration file.</p>



<pre class="wp-block-code"><code>Header always append X-Frame-Options SAMEORIGIN
</code></pre>



<h4 class="wp-block-heading">XSS Protection Again</h4>



<p>To ensure and enforce web browser <a href="https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)" target="_blank" rel="noopener">Cross Site Scripting</a> protection append the following to your configuration file.</p>



<pre class="wp-block-code"><code>Header set X-XSS-Protection “1; mode=block”
</code></pre>



<p>Now with those options set we need to restart the Apache daemon.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo systemctl reload httpd.service</code></pre>



<h1 class="wp-block-heading">Logwatch</h1>



<p>Now to keep tabs on all of those logs, Logwatch is a great tool to monitor your servers logs and email the administrator a digest on a daily basis.</p>



<h4 class="wp-block-heading">Installation</h4>



<pre class="wp-block-code lang:default decode:true"><code>sudo yum install logwatch sendmail</code></pre>



<p>Now start sendmail.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo systemctl start sendmail</code></pre>



<h4 class="wp-block-heading">Configuration</h4>



<p>The default configuration file for Logwatch is located at the below path.</p>



<pre class="wp-block-code lang:default decode:true"><code>/usr/share/logwatch/default.conf/logwatch.conf.</code></pre>



<p>This file contains information on which directories for Logwatch to track, how the digest is sent and where the digest is sent to.</p>



<p>By default Logwatch keeps track of everything in /var/log but if you have other log files that you wish to add you can do this by adding the below to your logwatch.conf under the heading ‘Default Log Directory’.</p>



<pre class="wp-block-code lang:default decode:true"><code>LogDir = /some/path/to/your/logs</code></pre>



<h4 class="wp-block-heading">Email your daily digest</h4>



<p>let’s go ahead and edit the logwatch.conf file.</p>



<pre class="wp-block-code lang:default decode:true"><code>sudo nano /usr/share/logwatch/default.conf/logwatch.conf</code></pre>



<p>We need to change add your email into the configuration file so that the digest gets delivered to your inbox.</p>



<p>Look for the following section.</p>



<pre class="wp-block-code lang:default decode:true"><code># Default person to mail reports to.  Can be a local account or a
# complete email address.  Variable Output should be set to mail, or
# --output mail should be passed on command line to enable mail feature.
MailTo = root</code></pre>



<p>change ‘root’ to your own personal email address or wherever you want the digest sending to.</p>



<h4 class="wp-block-heading">Adding Logwatch to Cron</h4>



<p>Open up the crontab.</p>



<pre class="wp-block-code lang:default decode:true"><code>crontab -e</code></pre>



<p>Now add the following line to the end of the file. This line will make logwatch run at midnight each day.</p>



<pre class="wp-block-code lang:default decode:true"><code>00 00  * * *          /usr/sbin/logwatch</code></pre>



<p>This guide was a little quick and dirty so you have any additions to this guide I would love to hear them, also if you think something is wrong or could have been done more efficiently please get in contact.</p>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/centos-7-server-hardening-guide/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/centos-7-server-hardening-guide/&amp;text=CentOS%207%20Server%20Hardening%20Guide" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=CentOS%207%20Server%20Hardening%20Guide&amp;body=/centos-7-server-hardening-guide/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/centos-7-server-hardening-guide/&amp;title=CentOS%207%20Server%20Hardening%20Guide" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/centos-7-server-hardening-guide/&amp;title=CentOS%207%20Server%20Hardening%20Guide" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/centos-7-server-hardening-guide/&amp;title=CentOS%207%20Server%20Hardening%20Guide" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/centos-7-server-hardening-guide/&amp;title=CentOS%207%20Server%20Hardening%20Guide" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-next"><a href="/posts/exploiting-the-opennmsjenkins-rmi-java-deserialization-vulnerability/" rel="next"><span>Exploiting the OpenNMS/Jenkins RMI Java Deserialization Vulnerability </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
