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
ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): ./id_ed25519_john
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./id_ed25519_john
Your public key has been saved in ./id_ed25519_john.pub
The key fingerprint is:
SHA256:qg4lXYkwnhjPZRn4cLmB8+y1Pcfomzb3Rr75SDLiwD8 root@minipc1
The key's randomart image is:
+--[ED25519 256]--+
|. oo++           |
| *=*=. .         |
|. =B.oo          |
|   .=..          |
|  ..o. oSo       |
|   o....+ o .    |
|  .   +..ooo.    |
|   . . +Eo.+oo   |
|   .o  .== o=o.  |
+----[SHA256]-----+
cat id_ed25519_john.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty John Doe (id_ed25519_john)
cat id_ed25519_jane.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs Jane Doe (id_ed25519_jane)
```
Dedicated ssh-key for SSHush minimal access to server when asking for contact aggrement:
```
$ cat id_sshush
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACC8q8l354zGaHMyyMsHLbBGTyzH+9AcEy5sRkDxOXCOwAAAAJBv1wBOb9cA
TgAAAAtzc2gtZWQyNTUxOQAAACC8q8l354zGaHMyyMsHLbBGTyzH+9AcEy5sRkDxOXCOwA
AAAEDRZmZyP3OmytYLo5MRPcvfFOjRoSr+rhpI3LOjmwOOQLyryXfnjMZoczLIywctsEZP
LMf70BwTLmxGQPE5cI7AAAAADHJvb3RAbWluaXBjMQE=
-----END OPENSSH PRIVATE KEY-----
$ cat id_sshush.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILyryXfnjMZoczLIywctsEZPLMf70BwTLmxGQPE5cI7A @everywhere
```
Proposal for SSHush email presentation:
```
<John Doe>@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm[server1,server2:2222]
<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[W.X.Y.Z:4444,server4]
```
The sshush-key is a base58 representation of the ssh-key with `@` prefix for filename and username compatibility (and fun).
```
$ echo -n @ ; echo AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty | base64 -d | base58
@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
$ echo -n @ ; echo AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs | base64 -d | base58
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
```
The protocol part of the ssh-key is kept for future compatibily with future key type (quantum proof).
```
$ echo "AAAAC3NzaC1lZDI1NTE5AAAA" | base64 -d
ssh-ed25519
```
The idea behind SSHush is that your sshush-key email address is also a username to connect to the server using the secure sftp protocol.
So `<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[server3,server4]` will be able to retreive email doing:
```
$ sftp -i id_ed25519_jane.pub @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@server3
$ sftp -i id_ed25519_jane.pub @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@server4
```
# Accepting contact
## Client side operations
Now imagin that Jane wants to send an email to John. She found his sshush email easily on his blog and the first step is to allow John to send email to her.

The `contact` file is usefull to know the sshush-key email of John Doe and the servers he used.

The `allowed_signers` file is an openssh standard file to allow signature verification and we add the sshush-key as an pseudo-anonymous reference.
```
$ echo "<John Doe>@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm[server1,server2]" >> contact
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm' | cut -c 2- | base58 -d | base64)
$ echo $PUBKEY | egrep -q '^AAAAC3NzaC1lZDI1NTE5AAAA' && \
echo @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm ssh-ed25519 $PUBKEY | tee -a allowed_signers
@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty
```
The updated `allowed_signers` file must be transfered to all sshush servers used by Jane to allow the administrateur to update the `authorized_keys` file related to Jane's sshush-key user.
```
$ for i in server3 server4
do
  (echo put allowed_signers  ) | sftp -i ./id_ed25519_jane @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@$i
done
Connected to server3.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                                                                                     100%   88    51.3KB/s   00:00
Connected to server4.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                                                                                     100%   88    51.3KB/s   00:00
```
That's all, in few  minutes, John will be able to upload email for Jane to `server3` or `server4` using ̀`@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym` as username.
## Server side operations
What's going on in `server3` or `server4` with the new `allowed_signers` file for Jane? Simply rebuild her `authorized_keys` file with the updated data. Maybe some new contact added and maybe some useless one deleted...

Mandatory limited access for Jane:
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym' | cut -c 2- | base58 -d | base64)
$ echo $PUBKEY | egrep -q '^AAAAC3NzaC1lZDI1NTE5AAAA' && \
echo "command=\"internal-sftp -P mkdir,rmdir,setstat,fsetstat -u 775\" ssh-ed25519 $PUBKEY" \
> @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys
```
Ultra-limited access for Jane authorized contact:
```
$ cat allowed_signers | while read SSHUSHKEY KEYTYPE SSHPUBKEY
do
   echo "command=\"internal-sftp -d /$SSHUSHKEY -P readdir,remove,mkdir,rmdir,chdir,setstat,fsetstat -u 775\" $KEYTYPE $SSHPUBKEY" \
   >> @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys
done
   
$ echo $PUBKEY | egrep -q '^AAAAC3NzaC1lZDI1NTE5AAAA' && \
echo @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm ssh-ed25519 $PUBKEY \
> @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys
```
John is not allowed to upload email to Jane folder.
# Server side operation
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact
 
If you are not known by the receiver, you need to ask for agreement before sending the first email. Agreement is easily revocable. Yes, SPAM is dead.

Every server is accepting sftp connexion with a dedicated user, named `@`, using a ssh-key known by everybody. Allowed commands are a dramaticaly limited subset of sftp working in a chrooted folder.

Request for agreement is uploaded to a dedicated folder (the receiver sshush-key). You need to know the receiver email, folders are not browsable.

The filename of the request for agreement is the requester sshush-key and the max file size is 1KB. 2 files are mandatory:
- "sshush-key.txt": the request for agreement which is utf-8 text encryted with the receiver public ssh-key using the `age` or `rage` tool for privacy.
- "sshush-key.sig": the signing of the request for agreement with the requester private ssh-key using ssh-keygen.

The server is capable to make signature validation, but not to access the `sshush-key.txt` content, nor to assure you that the identity is real. You need to ensure by yourself, who is asking for agreement!

The server is regulary checking the folders to:
- validating/transmetting the request to the dedicated folder in the receiver storage.
- suppress false request with bad signature.

## Regular contact

When the request for agreement is validated by the receiver, the requester public ssh-key is configured on all server used by the receiver and he can connect with sftp using the receiver sshush key as login name. Allowed commands are a dramaticaly limited subset of sftp working in a chrooted folder.

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
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
^D
```


# Client side operation

## Sender asking for Primary contact

You need to create 2 files with private information only for the receiver (ex: your real name and full sshush email address):
```
$ echo "<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[server3,server4]" | age -R ./id_ed25519_john.pub > @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
#-rw-r--r-- 1 root root      314 Jan  4 00:33 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
$ ssh-keygen -Y sign -f ./id_ed25519_jane -n sshush @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Signing file @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Write signature to @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
#-rw-r--r-- 1 root root      314 Jan  4 00:33 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
#-rw-r--r-- 1 root root      298 Jan  4 00:35 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
```
Be careful the size is 1KB max.

Sending the request:
```
$ (echo put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm; echo put  @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm ) | sftp -i ~/.ssh/id_sshush @server1
Connected to localhost.
sftp> put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
Uploading @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym                                                                               100%  314   186.5KB/s   00:00
sftp> put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
Uploading @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig                                                                           100%  298   210.4KB/s   00:00
```
if `server1` is offline, just use `server2`.

Don't forget to accept response from the receiver! You already know him and found his sshush email somewhere...


## Receiver validating primary contact
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym' | cut -c 2- | base58 -d | base64)
$ echo $PUBKEY | egrep -q '^AAAAC3NzaC1lZDI1NTE5AAAA' && echo @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym ssh-ed25519 $PUBKEY | tee RAF_signers
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym ssh-ed25519 AAAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

$ ssh-keygen -Y verify -f RAF_signers -I @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym -n sshush -s @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig < @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Good "sshush" signature for @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
Signature is correct.
```
$ age -d -i ./id_ed25519_john @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[W.X.Y.Z:4444,server4]
```
Ok I know Jane, she is a friend, I will accept email from her:
```
cat RAF_signers >> allowed_signers
```
Update all my sshush server with my new `allowed_signers` file:
```
$ for i in server1 server2
do
  (echo put allowed_signers  ) | sftp -i ./id_ed25519_john @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm@$i
done
Connected to server1.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                                                                                     100%   88    51.3KB/s   00:00
Connected to server2.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                                                                                     100%   88    51.3KB/s   00:00
```
Let's inform the requester 
```
$ echo "Welcome to my network Jane Doe, have a good day." | age -r "$(fgrep @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym allowed_signers | cut -d ' ' -f 2- )" > Welcome.txt
$ ssh-keygen -Y sign -n sshush -f id_ed25519_john Welcome.txt
Signing file Welcome.txt
Write signature to Welcome.txt.sig
$ ( echo put Welcome.txt ; echo put Welcome.txt.sig ) | sftp -i id_ed25519_john @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@server3
Connected to server3.
sftp> put Welcome.txt
Uploading Welcome.txt to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/Welcome.txt
Welcome.txt                                                                                                                                         100%  261    20.8KB/s   00:00
sftp> put Welcome.txt.sig
Uploading Welcome.txt.sig to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/Welcome.txt.sig
Welcome.txt.sig                                                                                                                                     100%  298   227.1KB/s   00:00

```
## Sender allowing the receiver to respond

## Receiver to respond to the sender

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

## create user jane
 737  /sbin/useradd --badnames -d /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym -g sshush -s /usr/sbin/nologin _
  738  vi /etc/passwd
  739  vi /etc/shadow
  740  cat /chroot/\@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys
  741  cat > /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym_authorized_keys
  742  mkdir /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
  743  touch /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/allowed_signers
  744  chown @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym:sshush /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/allowed_signers
  745  mkdir /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
  746  chmod 777 /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm


root@minipc1:~/age# cat /chroot/\@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym_authorized_keys
command="internal-sftp -P mkdir,rmdir,setstat,fsetstat -u 775" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
command="internal-sftp -d /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm -P readdir,remove,mkdir,rmdir,chdir,setstat,fsetstat -u 775" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty

## create user john

root@minipc1:~/age# cat /chroot/\@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys
command="internal-sftp -P mkdir,rmdir,setstat,fsetstat -u 775" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty
command="internal-sftp -d /@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym -P readdir,remove,mkdir,rmdir,chdir,setstat,fsetstat -u 775" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs


# Links
https://www.openssh.com/
https://github.com/FiloSottile/age


