# Access to Pegasus

The Pegasus cluster will disable the passphrase authentication system from July 5th, 2020 (11:59 am IST). We will only use ssh key based authentication in order to gain access to the cluster. Please follow the instructions below to create your SSH keys. We provide methods for different platforms here.

# Table of contents
1. [SSH key generation](#keygen)
    1. [Linux/MacOSX/Windows](#allsys)
2. [Using SSH key to access pegasus](#access)
    1. [Linux](#linux-access)
    2. [OSX](#osx-access)
    3. [Windows](#win-access)
3. [Accessing pegasus through IUCAA Gateway account](#gateway)
    1. [SSH tunnels for secure copy of files to and from pegasus](#scp)

## SSH key generation <a name="keygen"></a>

### Linux/Mac OSX/Windows: <a name="allsys"></a>

Install `openssh-client` package on your Ubuntu machine, or an equivalent package provided by your operating system. MS Windows 10 users also should enable the `openssh-client` on their machines. Then run the following commands on a terminal (command prompt on windows):
```
ssh-keygen -t rsa -b 4096 -C "your_email_address"
```
Please follow the instructions carefully, most importantly **setup a passphrase for your ssh key**. This provides very important protection in case a random person is able to gain access to your private SSH key. A typical session for the key generation will look like:

```
Enter file in which to save the key (/home/yourusername/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
```
If all goes well, there will be two ssh keys created in the `.ssh` directory. With the default options, you will get a private key `.ssh/id_rsa` and a public key `.ssh/id_rsa.pub` in your home directory.

Remember that **the private key is not supposed to be shared with anyone**, while the public key can be made accessible to the whole world. Think about the public key as follows: anyone can use your public key to lock (encrypt) a message to be sent to you. Once encrypted the message can only be decrypted by the you with the help of the private key and its corresponding passphrase. Hence it is important to protect the private key and its passphrase.

Change the permissions of your private key to be readable and writable only by you.

```
chmod 600 $HOME/.ssh/id_rsa
chmod 600 $HOME/.ssh/id_rsa.pub
```

Add this public key to the server now assuming you are on VPN or on the IUCAA network (those using Gateway please take a look [here](#gateway)) by running:

```
ssh-copy-id your_username@pegasus.ac.iucaa.in
```

This above command will work until we have passphrase access enabled (July 5th 11:59 am IST). 

If you have not finished the above task in the grace period when the passphrase and key logins are both allowed, you should email the public key as an attachment to sysadm@iucaa.in. The title of the email should be `SSH key for Pegasus cluster access for your_username`. This will help us to filter the messages appropriately. You will receive a message saying that your ssh key has been installed once the system admins enable the ssh key.

## Using SSH key to access pegasus <a name="access"></a>

### Linux <a name="linux-access"></a>

If you are inside the IUCAA network or accessing the cluster through IUCAA VPN, then you can ssh in the normal manner.
```
ssh your_username@pegasus.ac.iucaa.in
```
Once you enter this command, the `ssh` daemon will ask you for the passphrase of your key and then log you in to pegasus.

(Optional) For ease of access to pegasus, we recommend adding the following lines in your `$HOME/.ssh/config` file
```
host pegasus
    hostname pegasus.ac.iucaa.in
    identityfile ~/.ssh/id_rsa
    User your_username
```
Please change the identity file name to the file you created during the ssh key generation step. Also replace `your_username` with your username. This creates an alias such that the command `ssh pegasus` will connect to the correct server and use the SSH key based authentication.

(Optional) To avoid entering the key passphrase every time, you can use `ssh-agent`. Create a file `run-ssh-agent.sh` in `$HOME/bin` directory with the following contents:
```
#!/bin/bash
if [[ -S $XDG_RUNTIME_DIR/ssh-agent.socket ]]; then
   echo "ssh agent exists"
   echo $SSH_AUTH_SOCK
else
   echo "ssh agent does not exist"
   export SSH_AUTH_SOCK=$XDG_RUNTIME_DIR/ssh-agent.socket
   /usr/bin/ssh-agent -a $SSH_AUTH_SOCK -s >& $HOME/.ssh/testit
   echo $SSH_AUTH_SOCK
fi
```

Change the file to be executable using
```
chmod a+x ~/bin/run-ssh-agent.sh
```
Make sure that the `$HOME/bin` is in your PATH variable.

```
export PATH=$PATH:$HOME/bin
```

After every session login, when you want to access pegasus, please run the following commands:
```
run-ssh-agent.sh
ssh-add $HOME/.ssh/id_rsa
```
This will prompt for the SSH key passphrase. 

Once you enter this passphrase, it will enable direct login to pegasus without passphrase using the command:
```
ssh pegasus
```

To remove the key identities from the agent's memory, run
```
ssh-add -D
```

### On MacOSX <a name="osx-access"></a>

If you are inside the IUCAA network or accessing the cluster through IUCAA VPN, then you can ssh in the normal manner.
```
ssh your_pegasus_username@pegasus.ac.iucaaa.in
```
Once you enter this command, the `ssh` daemon will ask you for the passphrase of your key and then log you in to pegasus.

(Optional) Mac OSX also comes in with a `ssh-agent` which can simplify managing the ssh key credentials and can avoid you having to type in the key passphrase every time you login. After logging in to your own account, run this command on a terminal and enter your key passphrase.
```
ssh-add ~/.ssh/id_rsa
```

Once this is done, you will have seemless access via the first ssh command in this section.


### On Windows <a name="win-access"></a>

There are multiple clients such as `PuTTY` or `termius` which will enable a similar access to pegasus using ssh keys. Look for documentation for ssh key access for your Windows SSH client. Please ask questions in the comments section below in case of any trouble. 

## Accessing pegasus through IUCAA Gateway account <a name="gateway"></a>

First copy the ssh key identity to the gateway account by running the following command:
```
ssh-copy-id your_username@gw.iucaa.in
```
It will ask for the passphrase of the gw account. Once the key is installed, you can run:
```
ssh -A your_username@gw.iucaa.in
```
This should allow access to the pegasus account if you have installed your ssh key on pegasus using methods discussed in the previous section already. In case you would like to install the ssh key by using the gw account itself. Please see the following section on tunnels.

### SSH tunnels for secure copy of files to and from pegasus <a name="scp"></a>

SSH tunnels can also be created from the gateway account to access the pegasus cluster for file transfers. For example to set up a ssh tunnel for scp'ing a file.

```
ssh -N -L 12345:pegasus.ac.iucaa.in:22 your_username@gw.iucaa.in
```

After the authentication is complete (using your gw password), then press `Ctrl-Z` and type `bg` to put the job in the background. A ssh tunnel will be established between your port 12345 and port 22 on the pegasus login nodes. Any data sent on your port 12345 gets sent to port 22 on pegasus from now on. If you have not yet installed the SSH key on pegasus then run

```
ssh-copy-id -p 12345 your_pegasus_username@localhost
```

Now in order to scp a file
```
scp -P12345 filename localhost:path_to_copy/
```
will copy the local file `filename` to the existing directory `path_to_copy` on pegasus. Similarly you can copy files from pegasus to your local machine by doing:

```
scp -P12345 localhost:path_to_copy/filename .
```
This command will copy the file `filename` from `path_to_copy` directory on pegasus to your local machine.

