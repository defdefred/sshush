# SSHush
Beyond smtp mail.

# Why? Really?
It's time to move on. All very old protocol like Telnet, FTP or SMTP are so fucked...

You can't send binary data with SMTP, all is so bloated.

Metadata are accessible to all.

SPAM must be eradicated. 

# Address
1 user = 1 ssh public keys and one or many server where you can receive mail.

Two servers are enougth for most of people to be always online.

Example of ed25519 ssh public keys:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA John Doe
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs Jane Doe
```
Proposal for SSHush email presentation:
```
<John Doe>AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA[server1,server2]
<Jane Doe>AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs[server3,server4]
```

# Server side
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact
If you are not known by the receiver, you need to ask for agreement before sending the first email. Yes, SPAM is dead.

Every server is accepting sftp connexion with a dedicated user, named `_`, using a ssh key known by everybody and with very limited command allowed (`cd` and `put`).

SSHd configuration (chroot, sftp restriction and umask to prevent overwriting files):
```
Match user _
    AuthorizedKeysFile /chroot/authorized_keys
    ChrootDirectory /chroot/_
    ForceCommand internal-sftp -P readdir,remove,mkdir,rmdir -u 775 (need to be changed with -p only allowed command)
    AllowTcpForwarding no
    X11Forwarding no
```
Folder creation for chrooted environnement:
```
$ mkdir /chroot
$ chmod 700 /chroot
$ mkdir /chroot/_
$ chown _._ /chroot/_
$ chmod 500 /chroot/_
```
`_` user creation
```
$ useradd -d /_ -s /bin/false -U _
$ passwd _
$ cp well_known_ssh_key.pub /chroot/authorized_keys
$ chmod 444 /chroot/authorized_keys
```
`_` user can't write more than 1KB per files.
```
$ cat /etc/security/limits.d/_.conf
_       hard    fsize   1

```
Folder creation for email address
```
$ mkdir /chroot/_/LNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ chown _._ /chroot/_/LNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ chmod 700 /chroot/_/LNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ ls -lR /chroot/
/chroot/:
total 8
drwxr-xr-x 3 root root 4096 Dec 17 00:11 _
-r--r--r-- 1 _    _      94 Dec 16 23:55 authorized_keys

/chroot/_:
total 4
drwx------ 2 _ _ 4096 Dec 17 00:35 LNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA

/chroot/_/LNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA:
total 16
---------- 1 _ _  595 Dec 17 00:29 Hx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
```

## Regular contact

# Client side
