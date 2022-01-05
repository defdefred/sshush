# SSHush
Beyond smtp mail.

# Why? Really?
It's time to move on. All very old protocol like FTP or SMTP are so fucked...

You can't send binary data with SMTP, so all is bloated.

Metadata are accessible to all.

SPAM must be eradicated.

Should be easily home hosted or shoudn't need excessive trust to the hosting server.

# SSHush Address
1 user = 1 public ssh-key and one or many server where you can receive mail.

Two servers are enought for most of people to be always online.

Example of ed25519 public ssh-keys:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA John Doe (id_ed25519_john)
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs Jane Doe (id_ed25519_jane)
```
Proposal for SSHush email presentation:
```
<John Doe>@111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU[server1,server2:2222]
<Jane Doe>@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV[W.X.Y.Z:4444,server4]
```
The sshush-key is a base58 representation of the ssh-key with `@` prefix for filename and username compatibility (and fun).
```
$ echo -n @ ; echo AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA | base64 -d | base58
@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV
$ echo -n @ ; echo AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs | base64 -d | base58
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
```
The protocol part of the ssh-key is kept for future compatibily with future key type (quantum proof).
```
$ echo "AAAAC3NzaC1lZDI1NTE5AAAA" | base64 -d
ssh-ed25519
```
QRcode are welcome to exchange email address!

# Server side operation
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact
 
If you are not known by the receiver, you need to ask for agreement before sending the first email. Agreement is easily revocable. Yes, SPAM is dead.

Every server is accepting sftp connexion with a dedicated user, named `@`, using a ssh-key known by everybody. Allowed commands are a dramaticaly limited subset of sftp working in a chrooted folder.

Request for agreement is uploaded to a dedicated folder (the receiver sshush-key). You need to know the receiver email, folders are not browsable.

[tar gzip ou pas?]
The filename of the request for agreement is the requester sshush-key and the max file size is 1KB. The content is a tar file with:
- "RFA": the request for agreement which is utf-8 text encryted with the receiver public ssh-key using the `age` or `rage` tool for privacy.
- "RFA.sig": the signing of the request for agreement with the requester private ssh-key. 

The server is capable to make signature validation, but not to access the RFA content.

The server is regulary checking the folders to:
- validating/transmetting the request to the dedicated folder in the receiver storage.
- suppress false request with bad signature.

## Regular contact

When the request for agreement is validated by the receiver, your public ssh-key is configured on all server used by the receiver and your can connect with sftp using the receiver sshush key as login name. Allowed commands are a dramaticaly limited subset of sftp working in a chrooted folder.

[tar gzip ou pas?]
You are only authorized to upload files to a dedicated folder. The file format is again a tar file with encrytion and signature but without the 1KB limitation. Filenames must be different, because you can't overwrite files. A timestap could help.

The encrypted file content is the real message and the format is TBD. Maybe pure text with url detection and pure data with file extension is enougth.
Example:
```
date -u '+%s'
1641250382

1641250382.txt = "Hello John, look at this"
1641250383.jpg
```

## Anonymous contact management

The server need to manager the receiver `authorized_keys` with all validated public ssh-key of senders.
The server is regulary building the `authorized_keys` using a `allowed_signers` anonymous standard ssh file managed by the receiver.
```
cat > allowed_signers
@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
^D
```


# Client side operation

## Sender asking for Primary contact

You need to create a `RAF` file with private information only for the receiver (ex: your real name):
```
$ echo "<Jane Doe>@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV[W.X.Y.Z:4444,server4]" | age -R ./id_ed25519_john.pub > RAF
#-rw-r--r-- 1 root root      314 Jan  4 00:33 RAF
$ ssh-keygen -Y sign -f ./id_ed25519_jane -n sshush RAF
Signing file RAF
Write signature to RAF.sig
#-rw-r--r-- 1 root root      314 Jan  4 00:33 RAF
#-rw-r--r-- 1 root root      298 Jan  4 00:35 RAF.sig
$ tar zcf @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV RAF RAF.sig
#-rw-r--r-- 1 root root      713 Jan  4 00:37 @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV
```
Be careful the size is 1KB max.

Sending the request:
```
$ (echo put @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV @111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU ) | sftp -i ~/.ssh/id_sshush @@server1
Connected to server1.
sftp> put @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV @111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU
Uploading @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV to /@111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU/@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV
@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV                                                                               100%  594    26.1KB/s   00:00
```
if `server1` is offline, just use `server2`.

## Receiver validating primary contact
```
$ tar zxf @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV' | cut -c 2- | base58 -d | base64)
$ echo $PUBKEY | egrep -q '^AAAAC3NzaC1lZDI1NTE5AAAA' && echo ssh-ed25519 $PUBKEY
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA

cat > RAF_signers
RAF ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

$ ssh-keygen -Y verify -f RAF_signers -I RAF -n sshush -s RAF.sig < RAF
Good "sshush" signature for RAF with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
Signature is correct.
```
$ age -d -f ./id_ed25519_john RAF > RAF.txt
$ cat RAF.txt
<Jane Doe>@111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV[W.X.Y.Z:4444,server4]
```
Ok I know Jane, she is a friend, I will accept email from her:
```
echo @111RN3t1cWCcecTLM26gmqhchjRURTAu5E4U6HGDXURNL51LuvXEy9PDb1nxMNuy4wKV ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs >> allowed_signers
```
Update all my sshush server with my new `allowed_signers` file:
```
$ for i in server1 server2
do
  (echo put allowed_signers  ) | sftp -i ./id_ed25519_john @111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU@$i
done
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
```
root@minipc1:/etc# getconf LOGIN_NAME_MAX
256
groupadd sshush
root@minipc1:/chroot# /sbin/useradd --badnames -d /chroot/_111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU -g sshush -s /bin/false __
root@minipc1:/chroot# mkdir /chroot/_111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU
root@minipc1:/chroot# cat /root/age/id_ed25519_2.pub > /chroot/_111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU_authorized_keys

`vi /etc/passwd` and `vi /etc/shadow` to change `__` by `_111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU`
root@minipc1:/chroot# sftp -i /root/age/id_ed25519_2 _111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU@localhost
Connected to localhost.
sftp> pwd
Remote working directory: /
sftp> ls
Couldn't read directory: Permission denied

root@minipc1:/chroot# /sbin/useradd --badnames -d /chroot/_111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU -g sshush -s /bin/false  __
root@minipc1:/chroot# ln -s -- _111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU -111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU
root@minipc1:/chroot# cat /root/age/id_ed25519.pub > -111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU_authorized_keys
`vi /etc/passwd` and `vi /etc/shadow` to change `__` by `-111RN3t1cWCcecTLM26gmqhceEPJtchvH98AuNGEBhAfHzTsmK8vJjTgUwnkoytVrXoU`
```


