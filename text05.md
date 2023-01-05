# サーバー構築実習（CentOS7）

## リモートアクセス


### 

```shell
[root@localhost ~]# yum list installed | grep ssh
libssh2.x86_64                             1.4.3-10.el7_2.1            @anaconda
openssh.x86_64                             7.4p1-11.el7                @anaconda
openssh-clients.x86_64                     7.4p1-11.el7                @anaconda
openssh-server.x86_64                      7.4p1-11.el7                @anaconda
```

```shell
[root@localhost ~]# yum list updates
```

```shell
[root@localhost ~]# yum -y update openssh openssh-server openssh-clients
```

```shell
[root@localhost ~]# vim /etc/ssh/sshd_config 
```

```conf
# Logging
#SyslogFacility AUTH
SyslogFacility AUTHPRIV
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin yes
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

```shell
[root@localhost ~]# systemctl start sshd.service
```

```shell
[root@localhost ~]# systemctl enable sshd.service
```

```shell
[root@localhost ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since 木 2023-01-05 13:19:32 JST; 5min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 14345 (sshd)
   CGroup: /system.slice/sshd.service
           └─14345 /usr/sbin/sshd -D

 1月 05 13:19:32 localhost.localdomain systemd[1]: Starting OpenSSH server d...
 1月 05 13:19:32 localhost.localdomain sshd[14345]: Server listening on 0.0....
 1月 05 13:19:32 localhost.localdomain sshd[14345]: Server listening on :: p...
 1月 05 13:19:32 localhost.localdomain systemd[1]: Started OpenSSH server da...
Hint: Some lines were ellipsized, use -l to show in full.
```

```shell
[root@localhost ~]# ssh 10.45.48.14
The authenticity of host '10.45.48.14 (10.45.48.14)' can't be established.
ECDSA key fingerprint is SHA256:lWlMJ4ML43rKAcA8xseHtVtlF8za5Z3Va0Z+CMPNoMM.
ECDSA key fingerprint is MD5:92:2b:92:10:ba:13:bc:88:5c:59:45:c0:7d:c8:d0:b8.
Are you sure you want to continue connecting (yes/no)? yes
```

```shell
root@10.45.48.14's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```

```shell
[user@localhost ~]$ mkdir .ssh
```

```shell
[user@localhost ~]$ chmod 600 .ssh
```

```shell
[user@localhost ~]$ mv id_rsa.pub .ssh/authorized_keys
```

