# サーバー構築実習（CentOS7）

## メール


```shell
[root@localhost ~]# yum -y install bind bind-chroot bind-utils
```

```shell
$TTL    86400
@       IN      SOA     ns1.j00sangidai.com. postmaster.j00.sangidai.com. (
                2022111702      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum


        IN      NS      ns1.j00.sangidai.com.
        IN      MX      10      mail.j00.sangidai.com.
        IN      A       10.46.48.16

ns1     IN      A       10.45.48.16
mail    IN      A       10.45.48.16
```

```shell
$TTL    86400
@       IN      SOA     ns1.j00.sangidai.com. postmaster.j00.sangidai.com. (
                2022111701      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum

        IN      NS      ns1.j00.sangidai.com.
16      IN      PTR     ns1.j00.sangidai.com.
16      IN      PTR     mail.j00.sangidai.com.
```

```shell
[root@localhost ~]# yum -y install postfix
```

```shell
[root@localhost ~]# yum -y install dovecot
```

```shell
[root@localhost ~]# firewall-cmd --add-service=smtp --zone=public
success
[root@localhost ~]# firewall-cmd --add-service=smtp --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --add-port=110/tcp --zone=public
success
[root@localhost ~]# firewall-cmd --add-port=110/tcp --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --add-port=143/tcp --zone=public
success
[root@localhost ~]# firewall-cmd --add-port=143/tcp --zone=public --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
```

```shell
[root@localhost ~]# vim /etc/postfix/main.cf 
```

```conf

/* 省略 */

# INTERNET HOST AND DOMAIN NAMES
# 
# The myhostname parameter specifies the internet hostname of this
# mail system. The default is to use the fully-qualified domain name
# from gethostname(). $myhostname is used as a default value for many
# other configuration parameters.
#
#myhostname = host.domain.tld
#myhostname = virtual.domain.tld
myhostname = mail.j00.sangidai.com
# The mydomain parameter specifies the local internet domain name.
# The default is to use $myhostname minus the first component.
# $mydomain is used as a default value for many other configuration
# parameters.
#
#mydomain = domain.tld
mydomain = j00.sangidai.com

# SENDING MAIL
# 
# The myorigin parameter specifies the domain that locally-posted
# mail appears to come from. The default is to append $myhostname,
# which is fine for small sites.  If you run a domain with multiple
# machines, you should (1) change this to $mydomain and (2) set up
# a domain-wide alias database that aliases each user to
# user@that.users.mailhost.
#
# For the sake of consistency between sender and recipient addresses,
# myorigin also specifies the default domain name that is appended
# to recipient addresses that have no @domain part.
#
#myorigin = $myhostname
myorigin = $mydomain

/* 省略  */

# Specify a list of host or domain names, /file/name or type:table
# patterns, separated by commas and/or whitespace. A /file/name
# pattern is replaced by its contents; a type:table is matched when
# a name matches a lookup key (the right-hand side is ignored).
# Continue long lines by starting the next line with whitespace.
#
# See also below, section "REJECTING MAIL FOR UNKNOWN LOCAL USERS".
#
#mydestination = $myhostname, localhost.$mydomain, localhost
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
#       mail.$mydomain, www.$mydomain, ftp.$mydomain

/* 省略 */

# The right-hand side of the lookup tables is conveniently ignored.
# In the left-hand side, specify a bare username, an @domain.tld
# wild-card, or specify a user@domain.tld address.
# 
local_recipient_maps = unix:passwd.byname $alias_maps
#local_recipient_maps = proxy:unix:passwd.byname $alias_maps
#local_recipient_maps =

# The unknown_local_recipient_reject_code specifies the SMTP server
# response code when a recipient domain matches $mydestination or
# ${proxy,inet}_interfaces, while $local_recipient_maps is non-empty
# and the recipient address or address local-part is not found.
#
# The default setting is 550 (reject mail) but it is safer to start
# with 450 (try again later) until you are certain that your
# local_recipient_maps settings are OK.
#
unknown_local_recipient_reject_code = 550

/* 省略 */

# You can also specify the absolute pathname of a pattern file instead
# of listing the patterns here. Specify type:table for table-based lookups
# (the value on the table right-hand side is not used).
#
#mynetworks = 168.100.189.0/28, 127.0.0.0/8
mynetworks = 127.0.0.1, 10.45.48.0/24
#mynetworks = $config_directory/mynetworks
#mynetworks = hash:/etc/postfix/network_table

# The relay_domains parameter restricts what destinations this system will
# relay mail to.  See the smtpd_recipient_restrictions description in
# postconf(5) for detailed information.

/* 省略 */

# DELIVERY TO MAILBOX
#
# The home_mailbox parameter specifies the optional pathname of a
# mailbox file relative to a user's home directory. The default
# mailbox file is /var/spool/mail/user or /var/mail/user.  Specify
# "Maildir/" for qmail-style delivery (the / is required).
#
#home_mailbox = Mailbox
home_mailbox = Maildir/

# The mail_spool_directory parameter specifies the directory where
# UNIX-style mailboxes are kept. The default setting depends on the
# system type.
#
#mail_spool_directory = /var/mail
#mail_spool_directory = /var/spool/mail
```

```shell
[root@localhost ~]# postfix check
```

```shell
[root@localhost ~]# vim /etc/dovecot/dovecot.conf 
```

```conf

/* 省略 */

# Default values are shown for each setting, it's not required to uncomment
# those. These are exceptions to this though: No sections (e.g. namespace {})
# or plugin settings are added by default, they're listed only as examples.
# Paths are also just examples with the real defaults being based on configure
# options. The paths listed here are for configure --prefix=/usr
# --sysconfdir=/etc --localstatedir=/var

# Protocols we want to be serving.
protocols = imap pop3 lmtp

# A comma separated list of IPs or hosts where to listen in for connections. 
# "*" listens in all IPv4 interfaces, "::" listens in all IPv6 interfaces.
# If you want to specify non-default ports or anything more complex,
# edit conf.d/master.conf.
#listen = *, ::
```

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-mail.conf 
```

```conf

/* 省略 */

# See doc/wiki/Variables.txt for full list. Some examples:
#
#   mail_location = maildir:~/Maildir
#   mail_location = mbox:~/mail:INBOX=/var/mail/%u
#   mail_location = mbox:/var/mail/%d/%1n/%n:INDEX=/var/indexes/%d/%1n/%n
#
# <doc/wiki/MailLocation.txt>
#
mail_location = maildir:~/Maildir

/* 省略 */

# ':' separated list of directories under which chrooting is allowed for mail
# processes (ie. /var/mail will allow chrooting to /var/mail/foo/bar too).
# This setting doesn't affect login_chroot, mail_chroot or auth chroot
# settings. If this setting is empty, "/./" in home dirs are ignored.
# WARNING: Never add directories here which local users can modify, that
# may lead to root exploit. Usually this should be done only if you don't
# allow shell access for users. <doc/wiki/Chrooting.txt>
valid_chroot_dirs = /home
```

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-auth.conf 
```

```conf

/* 省略 */

## Authentication processes
##

# Disable LOGIN command and all other plaintext authentications unless
# SSL/TLS is used (LOGINDISABLED capability). Note that if the remote IP
# matches the local IP (ie. you're connecting from the same computer), the
# connection is considered secure and plaintext authentication is allowed.
# See also ssl=required setting.
disable_plaintext_auth = no

/* 省略 */

# Take the username from client's SSL certificate, using 
# X509_NAME_get_text_by_NID() which returns the subject's DN's
# CommonName. 
#auth_ssl_username_from_cert = no

# Space separated list of wanted authentication mechanisms:
#   plain login digest-md5 cram-md5 ntlm rpa apop anonymous gssapi otp skey
#   gss-spnego
# NOTE: See also disable_plaintext_auth setting.
auth_mechanisms = plain login

##
## Password and user databases
##
```

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-master.conf 
```

```conf

/* 省略 */

# Internal user is used by unprivileged processes. It should be separate from
# login user, so that login processes can't disturb other processes.
#default_internal_user = dovecot

service imap-login {
  inet_listener imap {
    port = 143
  }
  inet_listener imaps {
    #port = 993
    #ssl = yes
  }

/* 省略 */

service pop3-login {
  inet_listener pop3 {
    port = 110
  }
  inet_listener pop3s {
    #port = 995
    #ssl = yes
  }
}

service lmtp {
  unix_listener lmtp {
    #mode = 0666
  }
```

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-ssl.conf 
```

```conf
##
## SSL settings
##

# SSL/TLS support: yes, no, required. <doc/wiki/SSL.txt>
# disable plain pop3 and imap, allowed are only pop3+TLS, pop3s, imap+TLS and imaps
# plain imap and pop3 are still allowed for local connections
ssl = no

# PEM encoded X.509 SSL/TLS certificate and private key. They're opened before
# dropping root privileges, so keep the key file unreadable by anyone but
# root. Included doc/mkcert.sh can be used to easily generate self-signed
# certificate, just make sure to update the domains in dovecot-openssl.cnf
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
```

