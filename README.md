*This project has been created as part of the 42 curriculum by eturini*

# Born2beroot

## Description
Born2beroot is a project that introuce to the virtualization and server administration.

### Hypervisor

![Hypervisor Image](https://docs.aws.amazon.com/images/whitepapers/latest/security-design-of-aws-nitro-system/images/virtualization-architecture.png)

To create a Virtual Machine we need a **Hypervisor**, a software that takes the processing, memory and storage from the host machine and reallocates to be used by the virtual machine.
Thanks to this technology is possible run multiple virtual machines in a single physical machine.
The hypervisor, use some, defined resources, such as: *memory*, *process*, *process scheduler*, *input/output stack*, *device drivers*, *network stack* and more. All of this resoures are defined by the hypervisor, and this create an isolated environment.
Because they are isolated, even if a *VM* is compromised, the entire system shoudln't be impacted.
However if the hypervisor itself is compromised, all the instances of *VMs* can be affected.
There are two tipes of **Hypervisors**:
- *Type 1 (Bare-Metal)*
The *Bare-Metal* type is installed directly on the hosing machine, bypassing the OS, in some case is even incorporated in the firmware of the machine.
It can use resources flexibly with multiple VMs based on the requests of the ones.
This type is more common on data centers or other server-based environments.

- *Type 2 (Hosted)*
This type of **Hypervisor** runs on an application, and from there interacts with the hardware. It is installed on the physical machine and then executed by an application, that have to negotiate with the hosting OS, to gain the resources that tey are set to have, despite of that, the hosting os have the priority on the process over the virtual OSs.

In this project we will use Virtualbox, an open source type2 hypervisor.

### Linux Distribution
For this project we have to use **Rocky Linux** or **Debian**.
**Rocky Linux** is a free and open source enterprise-oriented and aims to provide a solid community support, providing a robust option for servers. \
**Debian** on the other hand, is an open source that supports a lot of architectures, the Debian's slogan is "*universal operating system*", and is one of the totally non-profit linux distro.

In my project I personally choosen **Debian** distro, because is easier to new in system administration.

### Partitioning
Partitioning means that we don't put all the files in a single and uniform part, we create the partition to store, specific values, that will not interfere each other stabilizing the system.
If we don't partition and a program fills up the disk with log files, the OS will crash. Partitioning we create a folder called */var* that if filled up won't cause the OS crash.
We need to implement some partition:
- `/home` that contain the files the user is allowed to modify, they don't comprehend the system files, that makes the system will not be touched by a reckless user.
- `/var/log` this partition is for log files, that as said before, if a programm fill the folder with log files, will fill only this partition without make the machine crash.
- `/swap` this partition is an extra space for volatile memory on the disk, if the RAM is full the machine will not crash, but starts to fill the swap memory.

Also I set the **encryption** on two partition, so if we don't put the password we set, those area of memory will not be reachable.

I also set, on the encrypted partitions, the **LVM** (Logical Volume Manager) that make the partition resizable, without formatting the whole disk.
LVM works on three levels:
- **PV** (Physical Volume) that is the whole disk, where the other partition will be.
- **VG** (Volume Group) is a group of partition, that stands with the same prefix: `wil--vg`.
- **LV** (Logical Volume) is the partition that is resizable. Like: `wil--vg-home` and `wil--vg-root`.

### SSH
If we would like to connect to our machine remotely I had to setup the **ssh** (Secure SHell) creating an encrypted tunnel where te information can pass securely. Also I have changed the port 22, the default one, to add a security layer. Also another layer is added adding that the SSH connection fail to log into the root user, so remotely can't be modified the root files; only knowing the user, and his password permit to access the root folder.
Also adding a **firewall** permit the incoming and outgoing traffic only by the port selected.

### Sudo and Groups
Sudo is a fondamental program to use another user's priviliges, by default the **superuser**, the super user is the default user in linux that have the permission to modify every file on the disk.
A **group** of users is a set of users that shares the same permission on the system.
I also modified the sudo rules, that are:
- `Default secure_path=".../path/..."` that limits the paths that can be used by sudo to run commands.
- `Default requiretty` that requires **TTY** to use sudo, TTY is a command that check if the output medium is a terminal. The command prints the file name of the terminal connected to the standard input.
- `Default badpass_message="*message*"` that display a custom message when using a wrong password with sudo.
- `Default logfile="/var/log/sudo/sudo.log"` that sets the path of the sudo log.
- `Default log_input` that sets to register the log input on the log file.
- `Default log_output` that sets to register the log output on the log file.
- `Default iolog_dir="/var/log/sudo"` that sets the directory to save additional log files.
- `Default passwd_tries=3` that sets the limit of retry to writing the password.

### Password Policy
In the `etc/login.defs` config file I saved the policy of the password to make the password difficult to bruteforce, they are:
- The password will expire in 30 days.
- The modified password cannot be modified not before 2 days.
- A warning will be displayed 7 days before the password will expire.
Another set of rules is handled by `pwquality`:
- Only three retry are allowed.
- The minimum size of the password has to be 10.
- Compared to the previous password the new one have to be 7 character different. This rule doesn't affect the root user.
- The max of consecutive character repeated is 3.
- The minimum number of lowercase letter is 1.
- The minimum number of uppercase letter is 1.
- The minimum number of digit is 1.
- The password cannot include the username.
- All this rules, with an ecception, are valid also to the root.

### Monitoring Script
I wrote a script that every 10 minutes, is displayed and contains machine information like the usage of RAM and the percentage of free space on disk, etc.
This scheduling is handled by the crontab, that in his config file execute the `monitoring.sh` script every 10 minutes, the config file is stored in `/etc/cron.d`.
It is print with a programm called **Wall**.

## Instructions
### Passwords
Here are listed the password and the users in this VM:
- **root**: Aglione4basta
- **disk encryption**: Aglione4basta
- **eturini**: PerForzissima42 -> provvisoria = "ciaone"

### Command that I used
- SSH commands
	- Check the ssh service status: `systemctl status ssh`.
	- To restart ssh: `systemctl restart ssh`.
- User and Groups commands
	- To login as root: `su`.


## Resources

- **Hypervisor content**:
	- https://www.redhat.com/en/topics/virtualization/what-is-a-hypervisor
	- https://aws.amazon.com/it/compare/the-difference-between-type-1-and-type-2-hypervisors/
- **Distros content**:
	- https://it.wikipedia.org/wiki/Rocky_Linux
	- https://it.wikipedia.org/wiki/Debian
- **Sudo and Groups content**:
	- https://en.wikipedia.org/wiki/Sudo
	- https://wiki.archlinux.org/title/Users_and_groups
	- https://en.wikipedia.org/wiki/Tty_(Unix)