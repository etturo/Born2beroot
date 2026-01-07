*This project has been created as part of the 42 curriculum by eturini*

# Born2beroot

## Description
Born2beroot is a project that introduces virtualization and server administration.

### Hypervisor

![Hypervisor Image](https://docs.aws.amazon.com/images/whitepapers/latest/security-design-of-aws-nitro-system/images/virtualization-architecture.png)

To create a Virtual Machine, we need a **Hypervisor**, software that takes processing power, memory, and storage from the host machine and reallocates them for use by the virtual machine.
Thanks to this technology, it is possible to run multiple virtual machines on a single physical machine.
The hypervisor uses defined resources such as *memory*, *process(es)*, *process scheduler*, *input/output stack*, *device drivers*, *network stack*, and more. All of these resources are managed by the hypervisor, creating an isolated environment.
Because they are isolated, even if a *VM* is compromised, the entire system shouldn't be impacted.
However, if the hypervisor itself is compromised, all instances of *VMs* can be affected.

There are two types of **Hypervisors**:

- *Type 1 (Bare-Metal)*
The *Bare-Metal* type is installed directly on the hosting machine, bypassing the OS; in some cases, it is even incorporated into the firmware of the machine.
It can flexibly use resources with multiple VMs based on their demands.
This type is more common in data centers or other server-based environments.

- *Type 2 (Hosted)*
This type of **Hypervisor** runs as an application and interacts with the hardware from there. It is installed on the physical machine and executed by an application that negotiates with the hosting OS to gain the resources they are set to have. However, the hosting OS has priority over the virtual OSs.

In this project, we will use **VirtualBox**, an open-source Type 2 hypervisor that also supports cross-platform usage. On Apple Silicon Macs, performance could be slower.
An alternative could have been **UTM**, but it is exclusive to macOS, so working on a Linux machine, the choice was obvious.

### Linux Distribution
For this project, we have to use **Rocky Linux** or **Debian**.
**Rocky Linux** is a free and open-source enterprise-oriented OS that aims to receive solid community support, providing a robust option for servers.  
**Debian**, on the other hand, is an open-source OS that supports many architectures. Debian's slogan is "the *universal operating system*", and it is one of the strictly non-profit Linux distros.

In my project, I personally chose the **Debian** distro because it is easier for beginners in system administration.

It is mandatory to mention that **Debian** has a security module in the *kernel* called **AppArmor**, which allows the system administrator to restrict permissions. AppArmor is offered as an alternative to **SELinux**, which is based on applying labels to files. AppArmor is preferred over SELinux because it is considered simpler to administer and maintain.

Debian comes pre-installed with **apt**, a package manager needed to install programs. **apt** retrieves the program from a list stored in `/etc/apt/sources.list` and installs all dependencies required by the application.
**apt** uses a *CLI* to work. There is an alternative called **aptitude**, another package manager that also includes a visual interface. I stuck with **apt** because I do not have a *DE* (Desktop Environment) in my environment, so I didn't need a visual interface.

### Partitioning
Partitioning means we don't put all files in a single, uniform space; instead, we create partitions to store specific data so that they do not interfere with each other, stabilizing the system.
If we don't partition and a program fills up the disk with log files, the OS will crash. By partitioning, we create a folder called `/var`, which, if filled up, won't cause the OS to crash.
We need to implement some partitions:
- `/home` contains files the user is allowed to modify. It doesn't include system files, ensuring the system will not be affected by a reckless user.
- `/var/log` is for log files. As mentioned, if a program fills the folder with logs, it will only fill this partition without crashing the machine.
- `/swap` is an extra space for virtual memory on the disk. If the RAM is full, the machine will not crash but start filling the swap memory.

I also enabled **encryption** on two partitions, so if the set password is not entered, those memory areas will not be accessible.

Additionally, I set up **LVM** (Logical Volume Manager) on the encrypted partitions, which makes them resizable without formatting the whole disk.
LVM works on three levels:
- **PV** (Physical Volume): The whole disk where the other partitions will reside.
- **VG** (Volume Group): A group of partitions that share the same prefix, e.g., `wil--vg`.
- **LV** (Logical Volume): The resizable partition, such as `wil--vg-home` and `wil--vg-root`.

### SSH
To connect to our machine remotely, I had to set up **SSH** (Secure Shell), creating an encrypted tunnel where information can pass securely. I changed the default port (22) to add a security layer. Another security measure is disabling SSH login for the root user; this prevents remote modification of root files, as one must know a user's credentials to access the system and then elevate privileges.
To enter the virtual machine remotely, use the command:  
`ssh [user]@localhost -p [port]`

To enhance security, we also set up a **firewall**, a program that filters incoming and outgoing network traffic and establishes a barrier between a trusted network and an untrusted one.
For this project, I chose **UFW** (Uncomplicated Firewall). As the name suggests, it is designed to be easy to use. It utilizes a CLI (Command Line Interface) and **iptables** to handle configuration rules (check `man iptables`).  
There was another firewall I could have chosen, **firewalld** (its name comes from firewall + the *d* suffix commonly used for daemons). It also uses **iptables**, but its command line is more complicated than **UFW**.

### Sudo and Groups
**Sudo** is a fundamental program used to execute commands with another user's privileges, by default the **superuser**. The superuser is the default user in Linux that has permission to modify every file on the disk.
A **group** is a set of users that share the same system permissions.

I also modified the sudo rules as follows:
- `Default secure_path=".../path/..."`: Limits the paths that can be used by sudo to run commands.
- `Default requiretty`: Requires a **TTY** to use sudo. TTY is a command that checks if the output medium is a terminal.
- `Default badpass_message="*message*"`: Displays a custom message when a wrong password is used with sudo.
- `Default logfile="/var/log/sudo/sudo.log"`: Sets the path for the sudo log.
- `Default log_input`: Logs input commands to the log file.
- `Default log_output`: Logs output to the log file.
- `Default iolog_dir="/var/log/sudo"`: Sets the directory to save additional log files.
- `Default passwd_tries=3`: Sets the retry limit for password entry.

### Password Policy
In the `/etc/login.defs` config file, I defined the password policy to make passwords difficult to brute-force:
- The password will expire in 30 days.
- The password cannot be modified for at least 2 days.
- A warning will be displayed 7 days before the password expires.

Another set of rules is handled by `pwquality`:
- Only three retries are allowed.
- The minimum password length is 10 characters.
- Compared to the previous password, the new one must have at least 7 different characters. This rule doesn't affect the root user.
- The maximum number of consecutive repeated characters is 3.
- The minimum number of lowercase letters is 1.
- The minimum number of uppercase letters is 1.
- The minimum number of digits is 1.
- The password cannot include the username.
- All these rules, with one exception, apply to the root user as well.

### Monitoring Script
I wrote a script that displays machine information, such as RAM usage and free disk space, every 10 minutes.
This scheduling is handled by `cron`, which executes the `monitoring.sh` script every 10 minutes via a config file stored in `/etc/cron.d`.
The output is broadcast to all terminals using a program called **Wall**.

The crontab job definition works like this:
```bash
	# Example of job definition:
	# .---------------- minute (0 - 59)
	# |  .------------- hour (0 - 23)
	# |  |  .---------- day of month (1 - 31)
	# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
	# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)
	# |  |  |  |  |
	# *  *  *  *  *   command to be executed
```

## Instructions
### Passwords
Here are the users and passwords for this VM:
- **disk encryption** (not a user): Aglione4basta
- **root**: Aglione4basta
- **eturini**: Perforzissima42
- **gigio**: Madonnissima44

Here are listed the password of the database and Wordpress.
- **wordpress**: eturini -> ciaonewordpres

### Commands Used
- Partitioning
	- To list block devices (partitions): `lsblk`.
- Services
	- To `restart` or check the `status` of a process: `systemctl [status|restart] [process]`.
	- To see the status of UFW: `sudo ufw status`.
- SSH commands
	- Check the SSH service status: `systemctl status ssh`.
	- To restart SSH: `systemctl restart ssh`.
	- To see the ports used by SSH: `sudo grep Port /etc/ssh/sshd_config`.
- User and Groups commands
	- To login as root: `su -`.
	- To add a user: `sudo useradd [new_user]`.
	- To verify the groups of a user: `id [user]`.
	- To add a group (as root): `groupadd [name]`.
	- To add a user to multiple groups: `usermod -a -G [group1],[group2] [user]`.
	- To see which groups a user is in: `groups [user]`.
	- To update a user's password: `passwd [user]`.
	- To check the hostname: `hostname`.
	- To change the hostname (temporary, resets on restart): `hostname [new_host]`.
	- To change the hostname permanently: `hostnamectl set-hostname [new_host]`.

## Resources

- **Hypervisor content**:
	- [What is a Hypervisor](https://www.redhat.com/en/topics/virtualization/what-is-a-hypervisor)
	- [Difference of Hypervisor type](https://aws.amazon.com/it/compare/the-difference-between-type-1-and-type-2-hypervisors/)
	- [UTM vs VirtualBox](https://www.howtogeek.com/virtualbox-vs-utm-which-is-best-for-linux-vms-on-mac/)
- **Distros content**:
	- [Rocky Linux wiki](https://it.wikipedia.org/wiki/Rocky_Linux)
	- [Debian wiki](https://it.wikipedia.org/wiki/Debian)
	- [AppArmor wiki](https://en.wikipedia.org/wiki/AppArmor)
	- [SELinux wiki](https://en.wikipedia.org/wiki/Security-Enhanced_Linux)
	- [APT vs Aptitude](https://blog.packagecloud.io/know-the-difference-between-apt-and-aptitude/)
- **SSH**:
	- [UFW wiki](https://en.wikipedia.org/wiki/Uncomplicated_Firewall)
	- [firewalld wiki](https://en.wikipedia.org/wiki/Firewalld)
	- [Firewall wiki](https://en.wikipedia.org/wiki/Firewall_(computing))
	- [Iptables wiki](https://en.wikipedia.org/wiki/Iptables)
- **Sudo and Groups content**:
	- [sudo wiki](https://en.wikipedia.org/wiki/Sudo)
	- [user and group wiki](https://wiki.archlinux.org/title/Users_and_groups)
	- [TTY wiki](https://en.wikipedia.org/wiki/Tty_(Unix))
	- [Hostname Commands](https://www.redhat.com/en/blog/configure-hostname-linux)