## CrackMapExec Usage

##### The general format for using CrackMapExec is as follows:
```shell-session
lizergindoma@htb[/htb]$ crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

 Example WinRM:
```shell-session
lizergindoma@htb[/htb]$ crackmapexec winrm 10.129.42.197 -u user.list -p password.list
```

Hydra - SSH
```shell-session
lizergindoma@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197
```

Hydra - RDP
```shell-session
lizergindoma@htb[/htb]$ crackmapexec winrm 10.129.42.197 -u user.list -p password.list
```




