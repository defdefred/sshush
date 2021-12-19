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
<John Doe>AAAAC3NzaC1lZDI1NTE5AAAAILNNuqT+MXwIyGXopB0Fj6TBXtpqUe8PnyafFqPLK8aA
<Jane Doe>AAAAC3NzaC1lZDI1NTE5AAAAIHx1fwSGUGmO3n2FqKnWAm0ErbQ26A37rglryJuPTnPs
```

# Server side
Two mecanisms:
- Primary contact
- Regular contact

## Primary contact
When you are not in relation 
# Client side
