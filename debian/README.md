# Setting up a new Debian system from scratch

Assume you have access to a brand new Debian system. You have:
- root user with password
- ssh access with root user and IP of server

Here are the steps you should go trough to set up the server:

## Connect as root

Connect to the server with ssh. You will be promted for the root password.

```bash
ssh root@<server-IP>
```


## Add new user

Create a new user you will use in the future.
The system will prompt for a couple of things, most importantly the password.
We will give this user admin rights afterwards with `visudo`.

```bash
adduser my-new-user

visudo
```

`visudo` will get you into nano with a file opened that allows specifying user privileges.
You will find a special section for privileges.
Only root should be in there.
Add your new user and give it unconstrained rights.

```
# User privilege specification
root        ALL=(ALL:ALL) ALL
newuser    ALL=(ALL:ALL) ALL
```

Save the configuration with `ctrl + x` followed by `Y`.
Then log off by typing `exit`.

## Add ssh public key

You can now ssh into the machine with the new user.

```bash
ssh my-new-user@<server-IP>
```

Make sure you have your public key ready, for instance in the clipboard.
Then, create an `.ssh/authorized_keys` file and add the public key to grant access

```
mkdir ~/.ssh
chmod 700 ~/.ssh
echo "<public-key>" > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Test the configuration by logging off (`exit`) and loggin in again with `ssh my-new-user@<server-IP>`.
The machine should not ask for a password but log you in directly.

## Enforce key-based authentication

Now, we reconfigure the ssh daemon such that it
- does not allow root logins directly (`PermitRootLogin no`)
- does not allow logging in without private key (`PasswordAuthentication no`)

```bash
sudo vim /etc/ssh/sshd_config
```

Now make the necessary configurations

```
...
PermitRootLogin no
...
PasswordAuthentication no
```

Restart the ssh daemon to make sure the changes have effect.

```bash
sudo systemctl restart ssh
```

## Configure firewall

We want to make sure that port 22 is the only open port (we need it for ssh).
For firewall configuration, we use `ufw` since it is easy to use.
Install it if necessary.
By default, `ufw` will
- block any incoming traffic regardless of the port
- allow any outgoing traffic regardless of the port

We allow incoming ssh traffic by

```bash
sudo ufw allow ssh
```

We then enable the firewall with this command

```bash
sudo ufw enable
```

IMPORTANT: make sure you have successfully allowed ssh traffic before enabling `ufw`.
If you fail to do so, you will lock yourself out of the machine.

## Configure timezone

Debian has a simple tool to configure the timezone.

```bash
sudo dpkg-reconfigure tzdata
```

## Install NTP

It is recommended to install NTP to synchronise time with other NTP servers.

```bash
sudo apt-get update
sudo apt-get install ntp
```

## Enable email sending

TODO...

## Enable unattended updates

You want to get security updates automatically for installed packages.
Debian can be configured to update automatically [click](https://wiki.debian.org/UnattendedUpgrades).

```bash
sudo apt-get install unattended-upgrades apt-listchanges

sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
```

The command above creates the file `/etc/apt/apt.conf.d/50unattended-upgrades`, which contains the configuration for unattended upgrades.
Uncomment the line `Unattended-Upgrade::Mail "root";` to get emails about updates.
Next, you must enable upgrading with:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades

cat /etc/apt/apt.conf.d/20auto-upgrades
```

The command will create a file with the following content:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

You can check the logs with `cat /var/log/unattended-upgrades/unattended-upgrades.log`.
