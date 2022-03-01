# SSHush
Beyond smtp mail.

# Why? Really?
It's time to move on. All very old protocol like FTP or SMTP are so fucked...

You can't send binary data with SMTP, so all is bloated.

Metadata are accessible to all.

You can't always keep the same email when changing domain.

SPAM must be eradicated.

Should be easily home hosted or shoudn't need excessive trust to the hosting server.

You are creating your own sshush email address.

# SSHush Address
1 user = 1 public ssh-key and one or many server where you can receive mail.

Two servers are enought for most of people to be always online.

Your ssh-key is uniq to you and you are creating it by youself.

Example of ed25519 public ssh-keys:
```
$ ssh-keygen -t ed25519
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
$ cat id_ed25519_john.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty John Doe (id_ed25519_john)
$ cat id_ed25519_jane.pub
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
The `allowed_signers` file is an openssh standard file to allow signature verification and we add the sshush-key as a pseudo-anonymous reference.
```
$ JOHNKEY='@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm'
$ PUBKEY=$(echo $JOHNKEY | cut -c 2- | base58 -d | base64)
$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])
echo $JOHNKEY $KEYTYPE $PUBKEY >> allowed_signers

$ fold -s allowed_signers
@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty
```
The updated `allowed_signers` file must be transfered to all sshush servers used by Jane to allow the administrator to update the `authorized_keys` file related to Jane's sshush-key user. Example:
```
$ JANEKEY='@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym'
root@minipc1:~/age# sftp -qi ./id_ed25519_jane $JANEKEY@server3
sftp> ls -1
ask-UVatAmQs9VTvYBJTfkGAZ1EKWqAZoJuqyHLRcjQ6JV6idfCuxUJeCXRmGT24uBRKWvxkgY2S62QSkBZiotQcnuv
cfg-n9ohTb45UjUuQGKtoVXNTQUFcoBpVhivJ8LdgVt2h82yRtJospkPZRMrGjTumMy6L73Pr5GzzpWB2CnSdo9Ktfd
new
sftp> rm cfg-n9ohTb45UjUuQGKtoVXNTQUFcoBpVhivJ8LdgVt2h82yRtJospkPZRMrGjTumMy6L73Pr5GzzpWB2CnSdo9Ktfd/allowed_signers
sftp> put allowed_signers cfg-n9ohTb45UjUuQGKtoVXNTQUFcoBpVhivJ8LdgVt2h82yRtJospkPZRMrGjTumMy6L73Pr5GzzpWB2CnSdo9Ktfd
sftp> exit
```
That's all, in few  minutes, John will be able to upload email for Jane to `server3` or `server4` using ̀`@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym` as username.

## Server side operations
What's going on in `server3` or `server4` with the new `allowed_signers` file for Jane? Simply rebuild her `authorized_keys` file with the updated data. Maybe some new contact added and maybe some useless one deleted...

Find newer `allowed_signers`:
```
root@minipc1:/chroot/authorized_keys# while read U ; do A=$(ls -1 /chroot/$U/cfg-*/allowed_signers 2>/dev/null) && [ $A -nt $U ] && echo $A ; done < <(ls -1)
/chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/cfg-n9ohTb45UjUuQGKtoVXNTQUFcoBpVhivJ8LdgVt2h82yRtJospkPZRMrGjTumMy6L73Pr5GzzpWB2CnSdo9Ktfd/allowed_signers
```
Mandatory limited access for Jane:
```
$ JANEKEY='@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym'
$ PUBKEY=$(echo $JANEKEY | cut -c 2- | base58 -d | base64)
$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])

$ echo "restrict,command=\"internal-sftp -p open,close,write,opendir,realpath,stat,remove,rename,setstat,read,lstat -u 337" $KEYTYPE $PUBKEY" > \
> /chroot/authorized_keys/${JANEKEY}
```
Ultra-limited access for Jane authorized contact.
```
$ cat allowed_signers | while read SSHUSHKEY KEYTYPE SSHPUBKEY
do
   echo "restrict,command=\"internal-sftp -d /new -p open,close,write,realpath,stat -u 377" \
> $KEYTYPE $SSHPUBKEY" >> /chroot/authorized_keys/$JANEKEY
done
```
John is now allowed to upload email to Jane folder.

# Primary contact

## Jane side operation

Jane is now ready to ask John for a primary contact agreement. She need to create 2 files with private information only accessible to John (ex: your real name and full sshush email address, encrypted with the `age` tool and John public ssh-key).

The filename is Jane sshush-key:
```
$ JANEKEY='@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym'
$ echo "<Jane Doe>$JANEKEY[server3,server4]" | age -R ./id_ed25519_john.pub > $JANEKEY

$ ssh-keygen -Y sign -f ./id_ed25519_jane -n sshush $JANEKEY
Signing file @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
Write signature to @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig

$ls -l ${JANEKEY}*
-rw-r--r-- 1 root root  314 Jan  4 00:33 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
-rw-r--r-- 1 root root  298 Jan  4 00:35 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
```
Be careful the size is 1KB max per file depending on server side configuration.

Sending the request to any server used by John, using the user `@` and the sshush (not) private ssh-key.

The destination folder is the John sshush-key:
```
$ JOHNKEY='@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm'
$ sftp -qi ~/.ssh/id_sshush @@server1 << EOT
@put ${JANEKEY} $JOHNKEY
@put ${JANEKEY}.sig $JOHNKEY
EOT
```
if `server1` is offline, just use `server2`.

## John side operaions
John is finding the new request when pooling his server for new request. Example:
```
# sftp -qi ./id_ed25519_john $JOHNKEY@server1
sftp> cd ask-pWDu1A26qtDBN12ohpPEj9mULAPwUanGZcWaKZUYMix1sSrwwFk4uAmgDTpS2U6Axcs2K7EyBQuRUxjsZNjftjb/
sftp> ls -l
----r-----    1 1005     1002          314 Feb 26 22:51 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
----r-----    1 1005     1002          298 Feb 26 22:51 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
sftp> get @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
sftp> rm @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
sftp> get @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
sftp> rm @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym.sig
sftp> exit
```
He check the signature.
```
$ JANEKEY='@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym'
$ JOHNKEY='@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm'
$ PUBKEY=$(echo $JANEKEY | cut -c 2- | base58 -d | base64)
$ KEYTYPE=$(echo $PUBKEY | sed 's/AAAA/ /g' | cut -d ' ' -f 2 | base64 -d | tr -dc [a-z-0-9])
$ echo $JANEKEY $KEYTYPE $PUBKEY > signer
$ fold -s signer
@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
ssh-ed25519 AAAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs

$ ssh-keygen -Y verify -f signer -I $JANEKEY -n sshush -s ${JANEKEY}.sig < $JANEKEY
Good "sshush" signature for @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
   with ED25519 key SHA256:yWuAxYn/r52czjTeZySHdVIHY82w6ChyQ2LFNXhj3WY
```
Signature is correct. John will decrypt the message with his private ssh-key.
```
$ age -d -i ./id_ed25519_john $JANEKEY
<Jane Doe>@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym[server3,server4]
```
Ok I know Jane, she is a friend, I will accept email from her:
```
cat signer >> allowed_signers
```
Update all my sshush server with my new `allowed_signers`. Example:
```
# sftp -qi ./id_ed25519_john $JOHNKEY@server1
sftp> ls -1
ask-pWDu1A26qtDBN12ohpPEj9mULAPwUanGZcWaKZUYMix1sSrwwFk4uAmgDTpS2U6Axcs2K7EyBQuRUxjsZNjftjb
cfg-3wjDAdWfg2RgWWvzMSaikv67ZCi2k1pNrnitdJvv2djZQ975zGBgJ52BJLpSGaNHqcmxU1u5oSTUme8m8ceSsMam
new
sftp> rm cfg-3wjDAdWfg2RgWWvzMSaikv67ZCi2k1pNrnitdJvv2djZQ975zGBgJ52BJLpSGaNHqcmxU1u5oSTUme8m8ceSsMam/allowed_signers
sftp> put allowed_signers cfg-3wjDAdWfg2RgWWvzMSaikv67ZCi2k1pNrnitdJvv2djZQ975zGBgJ52BJLpSGaNHqcmxU1u5oSTUme8m8ceSsMam
sftp> exit
```
## Regular contact
Let's inform Jane with a sshush email. Using date from epoch is a practical way to avoid duplicate filename:
```
$ JANEKEY='@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym'
$ JOHNKEY='@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm'

$ MSG=$(date -u '+%s')-${JOHNKEY}.txt
$ echo $MSG
1641250382-@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JW.txt
$ echo "Welcome to my network Jane Doe, have a good day." | age -r "$(fgrep $JANEKEY allowed_signers \
> | cut -d ' ' -f 2- )" > $MSG
$ ssh-keygen -Y sign -n sshush -f id_ed25519_john $MSG
Signing file 1641250382-@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm.txt
Write signature to 1641250382-@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm.txt.sig
$ sftp -i id_ed25519_john $JANEKEY@server3 << EOT
cd new
@put 1641250382-@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm.txt
@put 1641250382-@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm.txt.sig
EOT
```
# Server configuration
## Folders
### First level
```
root@minipc1:/chroot# ls -la
total 24
drwxr-x---  6 root sshush 4096 Feb  5 23:59 .
drwxr-xr-x 19 root root   4096 Dec 26 23:38 ..
drwxr-x---  4 root sshush 4096 Jan 31 23:52 @
drwxr-x---  4 root sshush 4096 Feb  5 23:59 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
drwxr-x---  4 root sshush 4096 Feb  6 00:06 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
drwxr-x---  2 root sshush 4096 Feb  6 01:02 authorized_keys
```
### `@` user folder
```
root@minipc1:/chroot/@# ls -la
total 16
drwxr-x--- 4 root                                                                  sshush 4096 Jan 31 23:52 .
drwxr-x--- 6 root                                                                  sshush 4096 Feb  5 23:59 ..
drwxrwx--- 2 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush 4096 Feb  6 01:06 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
drwxrwx--- 2 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush 4096 Feb  6 00:41 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
```
One folder per sshush user with owner (rwx) = the sshush user and group (rwx) = the sshush group.

### sshush users folder
```
root@minipc1:/chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym# ls -l
total 12
drwxrwx--- 2 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush 4096 Feb 24 23:47 ask-UVatAmQs9VTvYBJTfkGAZ1EKWqAZoJuqyHLRcjQ6JV6idfCuxUJeCXRmGT24uBRKWvxkgY2S62QSkBZiotQcnuv
drwx------ 2 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush 4096 Feb 23 01:03 cfg-n9ohTb45UjUuQGKtoVXNTQUFcoBpVhivJ8LdgVt2h82yRtJospkPZRMrGjTumMy6L73Pr5GzzpWB2CnSdo9Ktfd
drwx------ 2 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush 4096 Feb 23 01:22 new
```
Second example:
```
root@minipc1:/chroot/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm# ls -l
total 16
drwxrwx--- 2 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush 4096 Feb 11 00:34 ask-pWDu1A26qtDBN12ohpPEj9mULAPwUanGZcWaKZUYMix1sSrwwFk4uAmgDTpS2U6Axcs2K7EyBQuRUxjsZNjftjb
drwx------ 2 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush 4096 Feb 24 23:36 cfg-3wjDAdWfg2RgWWvzMSaikv67ZCi2k1pNrnitdJvv2djZQ975zGBgJ52BJLpSGaNHqcmxU1u5oSTUme8m8ceSsMam
drwx------ 2 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush 4096 Feb 24 23:36 new
```
Random number for the `àsk-` and `cfg-` folder are generated using:
``` 
root@minipc1:/# SECRET=$(openssl rand 64|base58)
root@minipc1:/# mkdir xxx-$SECRET
``` 
Inside each sshush-user folder, the `ask-` folder is a mount --bind to the sshush-user freely accessible folder, while the `cfg-` folder is used to upload the allowed external sshush-user list.
```
root@minipc1:~# tail -2 /etc/fstab
/chroot/@/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym /chroot/@111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym/ask-UVatAmQs9VTvYBJTfkGAZ1EKWqAZoJuqyHLRcjQ6JV6idfCuxUJeCXRmGT24uBRKWvxkgY2S62QSkBZiotQcnuv none defaults,bind 0 0
/chroot/@/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm /chroot/@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm/ask-pWDu1A26qtDBN12ohpPEj9mULAPwUanGZcWaKZUYMix1sSrwwFk4uAmgDTpS2U6Axcs2K7EyBQuRUxjsZNjftjb none defaults,bind 0 0
```
### ssh authorized_keys folder
```
root@minipc1:/chroot/authorized_keys# ls -la
total 20
drwxr-x--- 2 root                                                                  sshush 4096 Feb  6 01:02 .
drwxr-x--- 6 root                                                                  sshush 4096 Feb  5 23:59 ..
-r-------- 1 @                                                                     sshush  759 Feb  6 01:02 @
-r-------- 1 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym sshush  419 Feb  6 00:58 @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
-r-------- 1 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm sshush  384 Jan  6 01:14 @111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm
```
## Ssh authorized_keys files
# `@` user
```
root@minipc1:/chroot/authorized_keys# cat @
restrict,command="internal-sftp -p open,close,write,opendir,realpath,stat -u 777" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILyryXfnjMZoczLIywctsEZPLMf70BwTLmxGQPE5cI7A
```
Only allowed to change directory to the target sshush user and to write a file asking for permission to send sshush mail.

# for sshush users
```
root@minipc1:/chroot/authorized_keys# cat @111RN3t1cWCcecTLM26gmqhce3LDjoBkpaBgq1jjKSUb6juugbvf3pBB768Rn6pU3Vym
restrict,command="internal-sftp -p open,close,write,opendir,readdir,realpath,stat,remove,setstat,read,lstat" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
restrict,command="internal-sftp -d /@111RN3t1cWCcecTLM26gmqhcmA6wJMHu1JFuDL83JAxwc9e5XRJKVtYaG8mVkci49JWm -p open,close,write,realpath,stat -u 777" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOYzWcb+bZKh1lKsSC+G/hICMdVNthuUwJzUHwANlcty
```
```

```

## Sshush admin user
```
$ /sbin/useradd -d /chroot/ -g sshush -s /usr/sbin/nologin sshush
```

## Primary contact

SSHd configuration (chroot, sftp restriction and umask to prevent overwriting files):
```
$ cat /etc/ssh/sshd_config.d/sshush.conf
Match user @*
    AuthorizedKeysFile /chroot/authorized_keys/%u
    ChrootDirectory %h
```
Folder creation for chrooted environnement:
```
$ mkdir /chroot
$ chmod 755 /chroot
$ mkdir /chroot/authorized_keys
$ chmod 755 /chroot/authorized_keys
$ mkdir /chroot/@
$ chmod 755 /chroot/@
$ echo 'restrict,command="internal-sftp -p open,close,write,opendir,realpath,stat -u 775" ssh-ed25519 \
AAAAC3NzaC1lZDI1NTE5AAAAILyryXfnjMZoczLIywctsEZPLMf70BwTLmxGQPE5cI7A' > /chroot/authorized_keys/@
$ chmod 644 /chroot/authorized_keys/@
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
TODO
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


