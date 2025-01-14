---
layout: default
title: Windows Privesc
tags:
    - administrator
    - escalation
    - microsoft
    - nt
    - privilege
    - system
---
# Windows Privesc
Setting up a lower priv user for authenticated assessments
```shell
net user [username] [password] /add     # Create new user
net localgroup [group] [username] /add  # Add to desired group, may need a localgroup to allow logon
runas /user:[username] powershell.exe   # Run powershell as user
net user [username] /delete             # Delete when finished
```
## Automated Checks
### PowerUp
PowerUp.ps1.b64
```powershell
powershell.exe -nop -exec bypass
IEX([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String((Get-Content -Path .\PowerUp.ps1.b64))))
Invoke-AllChecks -ErrorAction SilentlyContinue | Out-File -Encoding ASCII powerup.log
```
## Manual Checks
### OS

General details
```powershell
systeminfo
```

```powershell
hostname
```

Our user and their permissions
```powershell
echo %username%
```

```powershell
net users
```

```powershell
net user [USERNAME]
```

### Networking
Network interfaces
```powershell
ipconfig /all
```

Routing table
```powershell
route print
```

ARP cache for all interfaces
```powershell
arp -A
```

Network connections
```powershell
netstat -ano  # -anob if admin
```

Firewall rules
```powershell
netsh firewall show state
```

```powershell
netsh firewall show config
```

### Applications
Scheduled tasks
```powershell
schtasks /query /fo LIST /v
```

Running processes and services
```powershell
tasklist /SVC
```

Windows services
```powershell
net start
```

Drivers can have vulns
```powershell
DRIVERQUERY
```

### File system
Files modified in last _n_ days
```powershell
forfiles /P directory /S /D +(today'date - [DAYS] days)
```

Files modified since date
```powershell
forfiles /P directory /S /D +dd/mm/yyyy
```

```powershell
Dir C:\ -r | ? {! $_.PSIsContainer -AND $_.lastwritetime -ge 'dd/mm/yy'} 
```
- check date format on machine; you might be dealing with Americans

### WMIC
[FuzzySecurity .bat script for extracting relevant info using WMIC](http://www.fuzzysecurity.com/tutorials/files/wmic_info.rar)

System Patches
```powershell
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

### Helpful links
- <https://www.fuzzysecurity.com/tutorials/16.html>