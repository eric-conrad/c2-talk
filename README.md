# Demo

**Attacker**:

Password spray: `hydra -L users.txt -P seasons-2023.txt 192.168.37.237 smb -u`


**Defender**:

Count successful (4624) and failed (4625) logins:

`Get-WinEvent -Path C:\labs\valkyrie-security-logons.evtx | Group-Object id -NoElement | sort count`

**Long tail analysis**: `Get-WinEvent -Path C:\labs\valkyrie-security.evtx | Group-Object id -NoElement | sort count`

**Service creation events**: `Get-WinEvent @{Path="C:\labs\valkyrie-system.evtx"; ID=7045} | fl | more`

**Search for 'powershell -nop'**: `Get-WinEvent @{Path="C:\labs\valkyrie-security.evtx";id=4688}| Where {$_.Message -like "*powershell.exe -nop*"} | fl | more`

**Search for ADMIN\$**: `Get-WinEvent @{Path="C:\labs\valkyrie-security.evtx";id=4688}| Where {$_.Message -like "*ADMIN$*"} | fl`

**Search for 'plan.exe'**: `Get-WinEvent @{Path="C:\labs\valkyrie-sysmon.evtx";} | Where {$_.Message -like "*plan.exe*"} | fl`

**Search for 'Remote Desktop'**: `Get-WinEvent @{Path="C:\labs\valkyrie-system.evtx";id=7040} | Where {$_.Message -like "*Remote Desktop*"} | fl`

**Search for self-signed certificates** `Get-WinEvent @{Path="C:\labs\valkyrie-system.evtx";id=1056} | fl`

**Account creation (on the domain controller)**: `Get-WinEvent @{Path="\labs\pegasus-security.evtx"; id=4720} | fl`

**Account added to the domain admin group**: `Get-WinEvent @{Path="\labs\\pegasus-security.evtx"; id=4737} | fl`

**Windows Defender**: `Get-WinEvent @{Path="C:\labs\valkyrie-defender.evtx";id=1117} | fl | more`
