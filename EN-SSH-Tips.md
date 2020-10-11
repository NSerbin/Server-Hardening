# ¿What is SSH?

***SSH*** or ***Secure Shell***, is a network protocol that gives users, particularly system administrators, a secure way to access a computer over an unsecured network. In addition to providing strong encryption, SSH is widely used by network administrators for managing systems and applications remotely, enabling them to log in to another computer over a network, execute commands and move files from one computer to another.

The most basic use of SSH is for connecting to a remote host for a terminal session. The form of that command is the following:

 `ssh {user}@{host} `

`{user}` The user you ll use to connect.
`{host}` The IP or Hostname you wanna connect.

Once you understand this...

# How we can improve the security of the SSH protocol on our servers?

### Create users in the server
To avoid using the root user, the idea is create the neccesary numbers of users.
For this, we need connect to the server with SSH and create the users with *useradd* command

#### This is an example:

`useradd -m -s /bin/bash usuario`

`-m` Setting the home directory for that user.

`-s` Setting the Shell

### Use SSH public key based login
SSH suppots various authentication. It's recommended that you use public key based authentication. There re a few different algorithms, but the most recommended public-key algorithm available today is *ed25519*. 
For that reason, we will create a public-key using *ed25519*.

#### To make the public-key, open the terminal and write:
`ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "user@ejemplo.com"`

`-o` Save the private-key using the new OpenSSH format rather than the PEM format.

`-a` It’s the numbers of KDF (Key Derivation Function) rounds. Higher numbers result in slower passphrase verification, increasing the resistance to brute-force password cracking should the private-key be stolen.

`-t` Specifies the type of key to create, in our case the Ed25519.

`-f` Specify the filename of the generated key file. If you want it to be discovered automatically by the SSH agent, it must be stored in the default `.ssh` directory within your home directory.

`-c` An option to specify a comment. It’s purely informational and can be anything. But it’s usually filled with <login>@<hostname> who generated the key.

Once the key is made, we must copy it to the server.

#### We'll use *ssh-copy-id.* command to do that:

`ssh-copy-id {user}@{host}`

### ALL THE MODIFICATIONS WE WILL MAKE BELOW, WILL BE ON THE *SSHD_CONFIG* FILE 

To achieve this, we need connect to the server with SSH using our user, then we need execute `sudo nano /etc/ssh/sshd_config` . This ll open the SSH configuration with the nano editor.

Inside *sshd_config* file, all the lines with the # re comments. To make it work, we need delete the *#*.

### 1° Change SSH Port
One of the benefits of change SSH Port is avoid the typical port scan and the script kiddies. Most hackers search Port 22 for SSH. For that reason its a good practice change it.

To change the port, we need change it in the *sshd_config* file, find the line *Port 22* or *#Port 22* and change it to another port.
The ideal is avoid 222 or 2222 ports...

#### This is an example:

`Port 5758`

From now, to login we need execute `ssh {user}@{host}:{nuevo puerto}`


### 2° Disable root user login and reduce Max Auth Tries
As we all know, the root user can do a lot of damage to our server if its used wrong.
For that reason, we ll change it in the *sshd_config* file, find the line *#PermitRootLogin* and add *no*. Also, find the line *#MaxAuthTries 6* and *UsePAM yes* and modify.

#### This is an example:

`PermitRootLogin no`

`MaxAuthTries 3`

`UsePAM no`


### 3° Limit Users SSH Access
Another good practice, its limit users to connect with SSH.
For that reason, we ll change it in the *sshd_config* file, buscamos donde dice *#AllowUsers*, en caso de que no aparezca, agregar *AllowUsers* y poner los usuarios creados anteriormente.

#### This is an example:

`AllowUsers user1 user2 user3`

	
### 4° Disable password based login and Empty Password Login.
Once we enable the Public-key login, its a good practice disable password login.
For that reason, we ll change it in the *sshd_config* file and add *AuthenticationMethods* and *PubkeyAuthentication*. Also, we can avoid use empty passwords, for that we go to the line *#PermitEmptyPasswords* and add *no*

#### This is an example:

`PermitEmptyPasswords no`

`AuthenticationMethods publickey`

`PubkeyAuthentication yes`


### 5° Configure idle log out timeout interval
A user can log in to the server with ssh, and you can set an idle timeout interval to avoid unattended ssh session.
For that reason, we ll change it in the *sshd_config* file, find the line *ClientAliveInterval 300* and modify reducing the number.

#### This is an example:

`ClientAliveInterval 120`

			
### 6° Disable .rhosts files			
SSH can emulate the behavior of the obsolete rsh command, just disable insecure access with RSH.
For that reason, we ll change it in the *sshd_config* file, find the line *#IgnoreRhosts yes* and modify.

#### This is an example:

`IgnoreRhosts yes`			

							
### 7° Disable host-based authentication
Host Based Auth is a non-interactive authentication, most common use is for automation but not secure because you dont know who use it.
For that reason, we ll change it in the *sshd_config* file, find the line *#HostbasedAuthentication no* and modify.

#### This is an example:

`HostbasedAuthentication no`

			
### 8° Change Protocol

SSH have 2 Protocols to use. The *Protocol 1* is older and less secure than *Protocol 2*.
For that reason, we ll change it in the *sshd_config* file, find the line *#Protocol 1* and modify. In case you cant find it, we must add it.

#### This is an example:

`Protocol 2`


#### After change all the settings inside *sshd_config* file, its neccesary save the changes and restart the SSH Service.
For that reason, we need execute `sudo service sshd restart`
