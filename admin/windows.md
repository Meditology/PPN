# Windows




## Processes

Kill process from cmd:

```
Cmd > taskkill /IM:calc.exe /F
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
secpol.msc  -- Local Security Policy          -- «Локальная политика безопасности»
gpedit.msc  -- Local Group Policy Editor      -- «Редактор локальной групповой политики»
lusrmgr.msc -- Local Users and Groups (Local) -- «Локальные пользователи и группы (локально)»
certmgr.msc -- Certificates - Current User    -- «Сертификаты - текущий пользователь»
```



### Administrative Tools

```
Cmd > mmc.exe %SystemRoot%\system32\dsa.msc      -- Active Directory Users and Computers
Cmd > mmc.exe %SystemRoot%\system32\dnsmgmt.msc  -- DNS
Cmd > mmc.exe %SystemRoot%\system32\gpmc.msc     -- Group Policy Management
Cmd > mmc.exe %SystemRoot%\system32\adsiedit.msc -- ADSI Edit
```




## KRShowKeyMgr

Run:

```
Cmd > rundll32.exe keymgr.dll, KRShowKeyMgr
```




## Permissions

Take own of a directory and remove it (run cmd.exe as admin):

```
Cmd > takeown /F C:\$Windows.~BT\* /R /A 
Cmd > icacls C:\$Windows.~BT\*.* /T /grant administrators:F 
Cmd > rmdir /S /Q C:\$Windows.~BT\
```

Change ownership of a file:

```
PS > $Acl = Get-ACL $filename
PS > $AccessRule= New-Object System.Security.AccessControl.FileSystemAccessRule("snovvcrash", "FullControl", "none", "none", "Allow")
PS > $Acl.AddAccessRule($AccessRule)
PS > Set-Acl $filename $Acl
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
