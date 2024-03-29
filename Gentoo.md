# Gentoo

Gentoo is a Linux distribution which is compiled from source code and the user gets to decide which configure and compile options are used for each and every package. 

Unlike others, Gentoo has the most powerful package manager out of the bunch, Portage. Portage is a very strong and customizable package manager but it is much slower than it's peers. But it still is much faster than packet managers used by Linux From Scratch (LTS) where you build everything manually. The downside of Gentoo is that it is not user friendly which makes it harder for users to get accustomed to Gentoo.

# CONFIGURING AND INSTALLING GENTOO

Steps down below are taken and changed from:

https://wiki.gentoo.org/wiki/Handbook:Main_Page

In order for us to choose an installation medium we need to know our surroundings, for this instance, we will be using https://wiki.gentoo.org/wiki/Handbook:AMD64  since my PC is x86_64 (64 bit) and I happen to be using Intel Core i7 for my processor.

Once you boot the iso you downloaded from

https://www.gentoo.org/downloads/

you will see a screen which has boot: in it. You need to type "gentoo" and hit enter to log in to the system.

Once you get in, you should check your network settings using "ifconfig" to see if you have the network adapter configured. 

Then proceed to ping a location to see internet connectivity. Pinging Google dns server's is plenty.

```ping 8.8.8.8```

If you get replies, this means you are good to go.

# Preparing the Disks

Taken from: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

A block device is a computer data storage device that supports reading and writing data in fixed-size blocks or sectors. 

Example block devices are floppy disks, hard drives and optical drives.

On amd64, disk block devices are split up into smaller, managable block devices, called partitions. Though it is possible to use disk as a whole for RAID(redundant array of independent disks.) for storing data on multiple hard disks.

There are two partitioning standards, MBR(Master boot record) and GPT(GUID Partition Table).

MBR uses 32-bit identifiers for partitions which compared to, GPT which uses 64-bit partitioning is very small. We prefer GPT for this case.

Our partition layout will be like this:

/dev/sda1

BIOS boot partition

/dev/sda2

Boot partition

/dev/sda3

Swap partition

/dev/sda4

 Root partition

BIOS boot partition is a small (1-2MB) partition in which boot loaders can put additional data that doesn't fit allocated storage, also in case of crashes this can help us.

Swap partition allows kernel to move memory pages that will not be accessed soon to disk.

Parted offers us an interface to partition our disks.

Get into the disk using the code:

```parted /dev/sda```

Most x86 and amd64 disks use ms-dos(MBR) label as default, change it to GPT inside parted using:

```mklabel gpt```
 
GPT allows us to have more than 4 partitions, which was unavailable on MBR due to limited memory.
To create a partition we need to define type, start location of memory partition and end point of it.
 
Next, tell parted to use megabytes for sizing.
 
```unit mib```
 
Now start creating the partitions:
```
mkpart primary 1 3
name 1 grub
set 1 bios_grub on
What this does is it creates partition grub with memory being 2MB between 1 and 3, and enable this to record that the selected partition is a GRUB BIOS partition.
mkpart primary 3 131
name 2 boot
set 2 boot on
mkpart primary 131 643
name 3 swap
mkpart primary 643 -1
name 4 rootfs
```

-1 here means we want the remaining memory to be assigned to the root partition. We can check the partitions using "print".
Hit 'quit' to leave parted.
 
# Creating filesystems

A filesystem is the way in which files are named and where they are placed logically for storage and retrieval. after we create the partitions we need to place filesystems on them.

There are many filesystems available on amd64 those interested in them can check https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

mkfs calculates how many inodes a filesystem should have.
```
mkfs.ext2 /dev/sda2
mkfs.ext4 /dev/sda4
```

The reason we gave ext2 to boot partition and ext4 to root partition is because ext4 is a much more advanced filesystem that boot partition doesn't need to use for saving time.
 
A BIOS boot partition doesn't contain a filesystem. It's just a place to put some GRUB code on MBR.

On a GPT disk, that area is used by the larger partition table and isn't available for bootloader code, so the bootloader code goes in a small partition instead.

After preparing the filesystems for the partitions it's time to mount them in order to use them.

At first all we need to do is mounting the root partition and swap partition.

```mount /dev/sda4 /mnt/gentoo```
 
Swap partitions can't be mounted the same way as other partitions.
 
```mkswap /dev/sda3 swapon /dev/sda3```

temporary might need a seperate partition:

```chmod 1777 /var/tmp```

# Stage 3

A Stage tarball is an archive of the basic files used for the installation of Gentoo Linux. Multilib stage 3 is preferred since no-multilib is 64-bit only and for seasoned users.

Go to where your root partition is mounted:

```cd /mnt/gentoo```

Next go into the downloads site and download stage3 for amd64:

links https://www.gentoo.org/downloads/mirrors/

After you download it, unpack it using:

```tar xpvf stage3-*.tar.bz2 --xattrs-include='*.*' --numeric-owner```

this might give and error depending on if you used bz2 or xz, change accordingly.

Portage, the package manager of Gentoo can be configured to operate faster if you have x2 or x4 CPUs.

```nano -w /mnt/gentoo/etc/portage/make.conf```

go to the file and add

```MAKEOPTS="-j2"```

j2 or j4 depending on the number of CPUs you use.

# Installing Base System

We need a new root for us to securely install the system into. We can't have both our root and new environment under the same tree since it might lead to problems between one another. So we create a new environment inside the system to be able to work in there.

Before chrooting in our new environment so we can start installing Gentoo, there are few things left to do. First is setting a mirror for software downloads in make.conf.

```mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf```

select the server closest to you.

We want networking in our new environment. This is done so we can still have network even after entering the new environment by copying the contents of our main resolv.conf file.

```cp --dereference /etc/resolv.conf /mnt/gentoo/etc/```

In order for us to enter the new environment, and that it works properly we need some essential files in the environment.
/proc is very special in that it is also a virtual filesystem. 

It's sometimes referred to as a process information pseudo-file system. It doesn't contain 'real' files but runtime system information (system memory, devices mounted, hardware configuration, etc). For this reason it can be regarded as a control and information centre for the kernel.

/sys is a more structured filesystem than /proc.

The bind mounts are necessary if you want to have access to things like block devices (which are at /dev), an internet connection, and other services managed/started by the kernel on boot.
```
 mount --types proc /proc /mnt/gentoo/proc
 mount --rbind /sys /mnt/gentoo/sys
 mount --make-rslave /mnt/gentoo/sys
 mount --rbind /dev /mnt/gentoo/dev
 mount --make-rslave /mnt/gentoo/dev
 ```
The --make-rslave operations are needed for systemd support later in the installation.

In order to enter the new environment, change root into the new environment root, get important files settings from profile and (optional)show that we are in chroot.
```
chroot /mnt/gentoo /bin/bash
 source /etc/profile
 export PS1="(chroot) ${PS1}"
 ```
# Mounting the boot partition

Now that we are on the new environment, we now need to mount the boot partition.

```mount /dev/sda2 /boot```

We need to inform portage about available software.

```emerge-webrsync```

When behind a firewall that does not permit rsync traffic through port 873, the emerge-webrsync command can be used to fetch and install a Portage snapshot through regular HTTP.

# Choosing a profile

A profile holds important data about USE and CFLAGS, it also determines what will be our interface between computer and us.

```eselect profile list```
```
Available profile symlink targets:
  [1]   default/linux/amd64/17.0 *
  [2]   default/linux/amd64/17.0/desktop
  [3]   default/linux/amd64/17.0/desktop/gnome
  [4]   default/linux/amd64/17.0/desktop/kde
```

For this tutorial, we will be using the bare minimum without GUI and only shell since we are system administrators.

```eselect profile set 1```

After we set the profile we need to update the world set.

A world set holds data about Linux(system set and selected set) to run properly, namely software packages.

```emerge --ask --verbose --update --deep --newuse @world```

Note: Selecting profile without desktop will require less packages which means less install time. Desktop has many packages which almost takes 4 times longer than others.

# Configuring the USE variable

USE is the most important aspect of Gentoo. It allows users to allow or block use of services such as a camera support, bluetooth support, KDE support etc.. We can customize everything we need or don't need in here. A list of USE flags is located here.

```emerge --info | grep ^USE```
```
USE="X acl alsa amd64 berkdb bindist bzip2 cli cracklib crypt cxx dri ..."
```

For example acl here is to support acl( Access control user) which lets us see the permissions for users or groups.
Let's say we want dvd support on our system what we want to do is go to the make.conf file and paste the following:
USE="dvd"

This can be configured towards our needs.

Update the timezone for system now.

```ls /usr/share/zoneinfo echo "Europe/Istanbul" > /etc/timezone emerge --config sys-libs/timezone-data```

emerge here updates localtime file based on timezone entry.

```nano -w /etc/locale.gen```

here uncheck
```
en_US ISO-8859-1
en_US.UTF-8 UTF-8
#de_DE ISO-8859-1
#de_DE.UTF-8 UTF-8
```
next generate locales specified in locale file.

```locale-gen```

```eselect locale list```

set US utf-8, whichever number that is.

```eselect locale set 9```

reload the environment.

```env-update && source /etc/profile && export PS1="(chroot) ${PS1}"```

# Configuring the Linux kernel

Gentoo provides users with kernel sources which we need in order to set for communicating between user and hardware.

```emerge --ask sys-kernel/gentoo-sources```

To see it installed run:

```ls -l /usr/src/linux```

Now it is time to configure and compile the kernel sources. There are two approaches for this:

- The kernel is manually configured and built.

- A tool called genkernel is used to automatically build and install the Linux kernel.

We will use manual configuration for educational purposes.

# Manual configuration

To know where kernel is configured we need

```emerge --ask sys-apps/pciutils```

```cd /usr/src/linux```

 ```make menuconfig```
 
This will open a screen which can be configured from this page.
 
After you are done exit and start compiling.

```make && make modules_install```

copy the kernel image to /boot

```make install```

Filesystem information
Fstab is our operating system’s file system table.

Once a filesystem is created it gets a unique UUID which it holds untill it's removed. This ensures even if the filesystem changes disks, it's still locatable without errors. Those UUID's can be found using 'blkid' command.
```
nano -w /etc/fstab
/dev/sda2   /boot        ext2    defaults,noatime     0 2
/dev/sda3   none         swap    sw                   0 0
/dev/sda4   /            ext4    noatime              0 1
 
/dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0
```
here you can change the filseystem location with the UUID=xxxx.... for safety reasons. Since we want boot partition to be checked during boot we put defaults and cdrom needs to be mounted manually every time so noauto. noatime is for improved performance.

# Networking information

we might want to define a hostname for our PC.

```nano -w /etc/conf.d/hostname```

# Set the hostname variable to the selected host name

```hostname="isken"```

add name to hosts file

```nano -w /etc/hosts```

# This defines the current system and must be set

```127.0.0.1     isken isken localhost```

if you don't have a domain name configured you should go to /etc/issue file and delete \o to bypass domain name from needing to show. For the contents of issue file click here.

Now give a password to the root using passwd.

Stage 3 doesn't come with all the essential tools like syslogger.

```emerge --ask app-admin/sysklogd rc-update add sysklogd default```

this will enable logs on boot.

Then install DHCP Client.

```emerge --ask net-misc/dhcpcd```
# Configuring the bootloader

Bootloader loads kernel module when system is booted, without it, the system would not know how to proceed when the power button has been pressed.

For amd64,  GRUB2 or LILO for BIOS based systems, and GRUB2 or efibootmgr for UEFI systems is used.
GRUB2
GRUB2 is the advanced version of GRUB, Grand Unified Bootloader.

```emerge --ask --verbose sys-boot/grub:2```

```grub-install /dev/sda```

Then tell GRUB about the kernel to load.

```grub-mkconfig -o /boot/grub/grub.cfg```

# Rebooting the system

After we configure GRUB we can exit the environment for final touches.
```
exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot
```
For reboot to load the new system, we need to boot it from the Hard Disk and not the CD.

After we get into the new environment we go into root using
```Login:root Password: (Enter the root password)```

then add a new user since we don't want to be working with root at all.
useradd -m -G users,wheel,audio -s /bin/bash isken passwd isken
Giving sudo privileges to the user will be discussed on the next blog.
 
 
 
 
 
 
 
