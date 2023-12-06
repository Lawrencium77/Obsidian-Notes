These are a few notes on about SSH works. They don't go into tonnes of detail - we just cover the key concepts. The resources used were:

* [This Archlinux page](https://wiki.archlinux.org/title/SSH_keys)
* [This RobEdwards YouTube video](https://www.youtube.com/watch?v=dPAw4opzN9g)

```toc
```

## RobEdwards Video

> [!INFO]
> The info in this video is a simplification. It ignores many things, including the fact that the server has its *own* private-public key pair. 


When accessing a computer, it's common to use a password. But **SSH Keys** are more secure, and perhaps simpler to use.

There are two types of SSH key:

* **Private Key** - super secret
* **Public key** - public. We can share this anywhere

These two keys are related: the public key can be calculated from the private key, but not vice versa.

We have two different computer: your local machine, and a server we wish to connect to:

![](_attachments/Screenshot%202023-05-10%20at%2017.02.51.png)

We do this using keys. The **private key** is on the **local machine**; the **public key** is on the **server**:

![](_attachments/Screenshot%202023-05-10%20at%2017.03.49.png)

When we run `ssh`, the following stages happen:
* Read the private key. 
* Communicates to the server that it wants to use the public key equivalent of the private key (which `ssh` can calculate).
* Server generates a random string, and uses an algorithm to **encrypt** that string using the **public key**. The **only** way to decrypt the string is with the **private key**. Even the public key can't decrypt it; it's a one-way encryption.
* The string is then decrypted on the local machine, using the private key. 
* The local server does a calculation on that string to prove that it did decrypt it, and sends the calculation back to the server. 

At this point, `ssh` can tell that the keys match, and we are granted access to the remote machine.

Overall, this looks something like:

![](_attachments/Screenshot%202023-05-10%20at%2017.08.59.png)

Private keys often end with the `.pem` suffix.

## Archlinux 
SSH keys serve as a means of identifying yourself to an SSH server. It has two main benefits over password authentication:

* Not prone to **brute-force attacks**.
* Can be more **convenient** - you can connect to multiple servers without needing to enter a password for each one.

### Background
If an SSH server has your public key on file and sees you requesting a connection, it uses the public key to construct and send a challenge. This challenge is an encrypted message, and it must be met with the appropriate response before the server grants you access.

What makes the coded message secure is that it can only be understood by the private key holder. While the public key can encrypt the message, it cannot decrypt the message.

The private key is typically stored in `~/.ssh/`.

### Generating an SSH Key Pair
An SSH key pair can be generated with the `ssh-keygen` command, e.g:

```
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/home/<username>/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/<username>/.ssh/id_rsa.
Your public key has been saved in /home/<username>/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:gGJtSsV8BM+7w018d39Ji57F8iO6c0N2GZq3/RY2NhI username@hostname
The key's randomart image is:
+---[RSA 3072]----+
|   ooo.          |
|   oo+.          |
|  + +.+          |
| o +   +     E . |
|  .   . S . . =.o|
|     . + . . B+@o|
|      + .   oo*=O|
|       .   ..+=o+|
|           o=ooo+|
+----[SHA256]-----+
```

The private key can be stored in disk in an encrypted form. When the encrypted private key is required, a passphrase must be entered in order to decrypt it. This is why the `ssh-keygen` command asks for a passphrase.
While this might superficially appear as though you're providing a login password to the SSH server, the passphrase is only used to decrypt the private key on the local system.

### Copying the public key to the remote server
If your key file is `~/ssh/id_rsa.pub` you can simply do the following command:

```bash
ssh-copy-id remote_server.org
```

To do this, you'll typically need to provide a password for the remote user account.[^fn1] The `ssh-copy-id` file is a script that automates adding the public key to `~/.ssh/authorized_keys` on the remote server (see [below](#Some%20Other%20Bits) for more about this file).

### `scp`
`scp` is a tool that allows you to copy file between local and remote machines over SSH. This uses your private key, just as when you run `ssh`.

## Some Other Bits
There are a few other `ssh`-related files that are useful to understand. Specifically:

* `~/.ssh/authorized_keys` (on the server)
* `~/.ssh/known_hosts` and `~/.ssh/config` (on the client)

The `authorized_keys` file stores a list of **public keys** that are allowed to authenticate access to the server. Each line in the file represents a separate public key. The idea is that your private key has to match **one** of the public keys in this file in order for access to be granted.

The `known_hosts` file keeps a record of the public keys of the servers that the client has connected to via SSH. When connecting to a remote host for the first time, the user is prompted to verify the host's public key fingerprint. Once verified, the host's public key is stored in the `known_hosts` file.
Upon subsequent connections, the SSH client checks this file to ensure the public key of the remote host matches the stored key, preventing [man-in-the-middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) attacks.

This only works since the client can use the server's *public* key, to propose a challenge that the server must solve using its *private* key. So during an SSH connection, both the server and client propose challenges for one another. See [this StackExchange page](https://superuser.com/questions/1657387/how-does-host-key-checking-prevent-man-in-the-middle-attack) for more information.

The `config` file is an SSH client configuration file. This allows the user to define SSH-specific settings, simplifying the connection process. The settings in the `config` file are applied in the order they appear, with later settings potentially overriding earlier ones.

SSH automatically picks up the `config` file whenever it runs.
An example entry to this file is as follows:

```
Host lawrencea-vm
  HostName lawrencea.dev-vms.speechmatics.io
  User lawrencea
  IdentityFile ~/.ssh/lawrencea_key.pem
```

This aliases `lawrencea.dev-vms.speechmatics.io` to `lawrencea-vm`, and specifies the private key that SSH should use.

> [!INFO]
> #### Running Commands Through SSH
> It's handy to know how to run commands via SSH, without having to setup a connection every time you wish to do something on the sever.
> The syntax to do this is:
> ```bash
> ssh username@remote_host 'command'

> [!INFO]
> There is certainly a lot of detail in how SSH works. You could go as deep as understanding the networking behind it. I still have a bit of confusion surrounding the ordering of client & server verification - which happens first? This is related to the fact that both the client *and* server posses their own private-public key pairs.

[^fn1]: But you'll never need to provide a password again! Which is why SSH is good in the first place.