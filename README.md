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
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA John Doe (id_ed25519)
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs Jane Doe (id_ed25519_2)
```
Proposal for SSHush email presentation:
```
<John Doe>AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA[server1,server2]
<Jane Doe>AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs[server3,server4]
```
QRcode welcome!

# Server side operation
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact
 
If you are not known by the receiver, you need to ask for agreement before sending the first email. Agreement is easily revocable. Yes, SPAM is dead.

Every server is accepting sftp connexion with a dedicated user, named `_`, using a ssh key known by everybody and with very limited command allowed.

Request for agreement is uploaded to a dedicated folder per receiver email. You need to know the receiver email, folder are not browsable.

The server is regulary checking the folders and transmetting the request to the dedicated folder in the receiver storage.



## Regular contact

# Client side operation

## Asking for Primary contact

You need to create a `identity` file signed with your public key: 
```
$ cat > identity
test freD
$ ssh-keygen -Y sign -f ./id_ed25519_2 -n email identity
Signing file identity
Write signature to identity.sig
$ tar zcf identity.tgz identity identity.sig
```
This identity file could always be the same and be careful the size is 1KB max.

To send it to the new contact, you need to encrypt it using the receiver SSH public key and make the final filename your SSH public key:
```
$ age -R ./id_ed25519.pub identity.tgz > AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
```
Sending the request:
```
$ (echo put AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA ) | sftp -i ~/.ssh/id_sshush _@server1
Connected to server1.
sftp> put AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
Uploading AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs to /AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA/AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs                             100%  594   363.7KB/s   00:00
```
if `server1` is offline, just use `server2`.

You can't overwrite files:
```
$ (echo put AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA ) | sftp -i ~/.ssh/id_sshush _@server1
Connected to server1.
sftp> put AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
Uploading AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs to /AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA/AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
dest open("/AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA/AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs"): Permission denied
```

## Validating Primary contact
```
root@minipc1:~/age# cat > allowed_signers
test2 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

root@minipc1:~/age# rm test*
root@minipc1:~/age# age -d -i ./id_ed25519 Hx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs | tar zxf -
root@minipc1:~/age# ls -l
total 44
-rw-r--r-- 1 root root 778 Dec  9 01:41 Hx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
-rw-r--r-- 1 root root  87 Dec  9 01:55 allowed_signers
-rw------- 1 root root 399 Dec  9 00:43 id_ed25519
-rw-r--r-- 1 root root  94 Dec  9 00:43 id_ed25519.pub
-rw------- 1 root root 399 Dec  9 00:45 id_ed25519_2
-rw-r--r-- 1 root root  94 Dec  9 00:45 id_ed25519_2.pub
-rw-r--r-- 1 root root 628 Dec  9 01:23 test.txt
-rw-r--r-- 1 root root 294 Dec  9 01:31 test.txt.sig

root@minipc1:~/age# fgrep Hx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs allowed_signers
test2 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

root@minipc1:~/age# ssh-keygen -Y verify -f allowed_signers -I test2 -n mage -s test.txt.sig < test.txt
Good "mage" signature for test2 with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
# Server configuration
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact

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
(maybe more limit need to be set, and fail2ban configured)
```
$ cat /etc/security/limits.d/_.conf
_       hard    fsize   1
```
Folder creation for email address
```
$ mkdir /chroot/_/AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ chown _._ /chroot/_/AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ chmod 700 /chroot/_/AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
$ ls -lR /chroot/
/chroot/:
total 8
drwxr-xr-x 3 root root 4096 Dec 17 00:11 _
-r--r--r-- 1 _    _      94 Dec 16 23:55 authorized_keys

/chroot/_:
total 4
drwx------ 2 _ _ 4096 Dec 17 00:35 AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA

/chroot/_/AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA:
total 16
---------- 1 _ _  594 Dec 17 00:29 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
```
## Regular contact

