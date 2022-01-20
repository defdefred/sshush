# SSHush
Beyond smtp mail.

# Why? Really?
It's time to move on. All very old protocol like FTP or SMTP are so fucked...

You can't send binary data with SMTP, so all is bloated.

Metadata are accessible to all.

You can't always keep the same name when changing domain.

SPAM must be eradicated.

Should be easily home hosted or shoudn't need excessive trust to the hosting server.

# SSHush Address
1 user = 1 public ssh-key and one or many server where you can receive mail.

Two servers are enought for most of people to be always online.

Your ssh-key is uniq to you and you are creating it by youself.

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
Dedicated ssh-key for SSHush minimal access to server when asking for contact agreement:
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
$ echo "AAAAC3NzaC1lZDI1NTE5AAAA" | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9]
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

The `contact` file is usefull to know the sshush-key email of John Doe and the servers he use.
```
$ echo "<John Doe>@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm[server1,server2]" >> contact
```
The `allowed_signers` file is an openssh standard file to allow signature verification and we add the sshush-key as an pseudo-anonymous reference.
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm' | cut -c 2- \
| base58 -d | base64)

$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])
echo @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm $KEYTYPE $PUBKEY >> allowed_signers

$ fold -s allowed_signers
@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty
```
The updated `allowed_signers` file must be transfered to all sshush servers used by Jane to allow the administrator to update the `authorized_keys` file related to Jane's sshush-key user.
```
$ for i in server3 server4
do
  (echo put allowed_signers  ) | sftp -i ./id_ed25519_jane \
  @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@$i
done
Connected to server3.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                        100%   88    51.3KB/s   00:00
Connected to server4.
sftp> put allowed_signers
Uploading allowed_signers to /allowed_signers
allowed_signers                                                                        100%   88    51.3KB/s   00:00
```
That's all, in few  minutes, John will be able to upload email for Jane to `server3` or `server4` using ̀`@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym` as username.

## Server side operations
What's going on in `server3` or `server4` with the new `allowed_signers` file for Jane? Simply rebuild her `authorized_keys` file with the updated data. Maybe some new contact added and maybe some useless one deleted...

Mandatory limited access for Jane:
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym' | cut -c 2- \
| base58 -d | base64)

$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])

$ echo "command=\"internal-sftp -P mkdir,rmdir,setstat,fsetstat -u 775\" $KEYTYPE $PUBKEY" \
> /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym_authorized_keys
```
Ultra-limited access for Jane authorized contact:
```
$ cat allowed_signers | while read SSHUSHKEY KEYTYPE SSHPUBKEY
do
   echo "command=\"internal-sftp -d /$SSHUSHKEY -P readdir,remove,mkdir,rmdir,chdir,setstat,fsetstat -u 775\" \
   $KEYTYPE $SSHPUBKEY" >> /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym_authorized_keys
done
```
A dedicated folder is needed for each allowed sshush-key
```
mkdir -p /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
chown @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym:sshush /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm 
```
John is now allowed to upload email to Jane folder.

# Primary contact

## Client side operation

Jane is now ready to ask John for a primary contact agreement. She need to create 2 files with private information only accessible to John (ex: your real name and full sshush email address, encrypted with the `age` tool and John public ssh-key).

The filename is Jane sshush-key:
```
$ echo "<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[server3,server4]" \
| age -R ./id_ed25519_john.pub > @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
#-rw-r--r-- 1 root root  314 Jan  4 00:33 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
$ ssh-keygen -Y sign -f ./id_ed25519_jane -n sshush \
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Signing file @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Write signature to @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
#-rw-r--r-- 1 root root  314 Jan  4 00:33 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
#-rw-r--r-- 1 root root  298 Jan  4 00:35 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
```
Be careful the size is 1KB max per file depending on server side configuration.

Sending the request to any server used by John, using the user `@` and the sshush (not) private ssh-key.

The destination folder is the John sshush-key:
```
$ (echo put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
echo put  @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm ) \
| sftp -i ~/.ssh/id_sshush @@server1
Connected to server1.
sftp> put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
Uploading @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym                                   100%  314   186.5KB/s   00:00
sftp> put @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
Uploading @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig                               100%  298   210.4KB/s   00:00
```
if `server1` is offline, just use `server2`.

## Server side operations

To fight spamming, the server is able to validate the request for agreement ssh signature, but not to break the metadata privacy.
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym' | cut -c 2- \
| base58 -d | base64)
$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])
$ echo @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym $KEYTYPE $PUBKEY > RAF_signers
$ fold -s RAF_signers
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
ssh-ed25519 AAAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

$ ssh-keygen -Y verify -f RAF_signers -I @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym \
-n sshush -s @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig \
< @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Good "sshush" signature for @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
Signature is correct. Move the request to John folder:
```
$ mv @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vymv @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig \
/chroot/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/requests/
```


## Client side operaions
John is finding the new request when pooling his server for new email. He re-check the signature because he is not trusting the server.
```
$ PUBKEY=$(echo '@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym' | cut -c 2-\
| base58 -d | base64)
$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])
$ echo @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym \
$KEYTYPE $PUBKEY > RAF_signers
$ fold -s RAF_signers
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym \
ssh-ed25519 AAAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

$ ssh-keygen -Y verify -f RAF_signers -I @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym \
-n sshush -s @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig \
< @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Good "sshush" signature for @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
Signature is correct. John will decrypt the message with his private ssh-key.
```
$ age -d -i ./id_ed25519_john @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[server3,server4]
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
## Regular contact

Let's inform Jane with a regular email. Using date from epoch is a practical way to avoid duplicate filename:
```
date -u '+%s'
1641250382
$ echo "Welcome to my network Jane Doe, have a good day." | age -r "$(fgrep @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym allowed_signers | cut -d ' ' -f 2- )" > 1641250382.txt
$ ssh-keygen -Y sign -n sshush -f id_ed25519_john 1641250382.txt
Signing file 1641250382.txt
Write signature to 1641250382.txt.sig
$ ( echo put 1641250382.txt ; echo put 1641250382.txt.sig ) | sftp -i id_ed25519_john @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym@server3
Connected to server3.
sftp> put 1641250382.txt
Uploading 1641250382.txt to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/1641250382.txt
1641250382.txt                                                                          100%  261    20.8KB/s   00:00
sftp> put 1641250382.txt.sig
Uploading 1641250382.txt.sig to /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/Welcome.txt.sig
1641250382.txt.sig                                                                      100%  298   227.1KB/s   00:00
```
# Server configuration
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact

SSHd configuration (chroot, sftp restriction and umask to prevent overwriting files):
```
$ cat /etc/ssh/sshd_config.d/sshush.conf
Match user @*
    AuthorizedKeysFile /chroot/%u_authorized_keys
    ChrootDirectory %h
```
Folder creation for chrooted environnement:
```
$ mkdir /chroot
$ chmod 755 /chroot
$ mkdir /chroot/@
$ chmod 755 /chroot/@
$ echo 'restrict,command="internal-sftp -p open,close,write,opendir,realpath,stat -u 775" ssh-ed25519 \
AAAAC3NzaC1lZDI1NTE5AAAAILyryXfnjMZoczLIywctsEZPLMf70BwTLmxGQPE5cI7A' > /chroot/@_authorized_keys
$ chmod 644 /chroot/@_authorized_keys
```
`@` user creation
```
$ groupadd sshush
$ useradd --badnames -d /chroot/@ -g sshush -s /usr/sbin/nologin @
```
`@` user can't write more than 1KB per files.
```
$ cat /etc/security/limits.d/@.conf
@       hard    fsize   1
```
Folder creation for email address
```
```
## Regular contact
```
drwxr-xr-x 3 root                                                                  root   4096 Jan  6 02:13 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
-rw-r--r-- 1 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush  383 Jan  6 02:11 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym_authorized_keys
drwxr-xr-x 3 root                                                                  root   4096 Jan  6 00:57 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
-rw-r--r-- 1 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush  384 Jan  6 01:14 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm_authorized_keys


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

# What Next?
Maybe, the ancien tcp port 25 could be used...

Manual Proof Of Concept is fine, but who will developpe a real client ?


# Links
https://www.openssh.com/

https://github.com/FiloSottile/age


