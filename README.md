# RHCSA Cheatsheet

## Archives

```bash
# Compress
tar cvf filename.tar /var/log

# Extract
tar xvf filename.tar

# Compress gzip
gzip file

# Extract gzip
gzip -d file.gz

# Compress bzip2
bzip2 file

# Extract bzip2
bzip2 -d file.bz2

tar czvf filename.tar.gz /var/log
tar xzvf filename.tar.gz /var/log

tar cjvf filename.tar.bz2 /var/log
tar xjvf filename.tar.bz2 /var/log

zip -r filename.zip /var/log
unzip filename.zip

```

## Permissions

```bash
chmod a+w file # Gives everyone write permissions

umask 0222 # Subtracts from actual permissions

# Persist umask
vi /etc/profile
vi /etc/bashrc
```

## Grub commands

```bash
ls
cat
search.file /grub2/grub.cfg
insmod lvm
```

## Reset root password

- Press e on boot menu
- Add `rd.break` at the end of linux16 line
- Press ctrl-x to boot

```bash
mount -o rw,remount /sysroot
chroot /sysroot
passwd root

touch .autorelabel
exit
exit
```

## Virtual Machines

```bash
yum install qemu-kvm python-virtinst libvirt libvirt-python virt-manager libguestfs-tools virt-install qemu-img libvirt-client

virt-manager # GUI

virt-install --name centos7 --ram 1024 --vpus 2 --disk path=/var/lib/libvirt/images/image.qcow2 --os-type linux --network bridge=virbr0 --location=/tmp/centos7.iso --extra-args 'console=ttyS0'

virsh list --all
virsh start <name>
virsh edit <name>
```

## LVM Basics

```bash
pvcreate /dev/sdb1
vgcreate <VGNAME> /dev/sdb1 -s <extent-size>
lvcreate -L 100M <VGNAME> -n <LVNAME>
lvcreate -n <LVNAME> -l <extents> <VGNAME>
```

## Manipulating partitions

#### Fdisk
```bash
fdisk -l <disk>
fdisk <disk>
# n -> new partition
# d -> delete partition
# p -> print current partition
# w -> write partition table
# 82 is Swap
# 83 is Linux
# 8e is Linux LVM
partprobe
```

#### Gdisk
```bash
gdisk -l <disk>
gdisk <disk>
# n -> new partition
# d -> delete partition
# p -> print current partition
# w -> write partition table
# 8200 is Swap
# 8300 is Linux
# 8e00 is Linux LVM
partprobe
```

```bash
mkfs.xfs /dev/device
partprobe
mkdir /mnt/mountpoint
mount /dev/device /mnt/mountpoint
lsblk
blkid
echo "UUID=ksjfslkfjsdaf /mnt/mountpoint xfs defaults 0 0" >> /etc/fstab
mount -a
```

## Managing Disks

```bash
xfs_admin -L "LADisk" /dev/device
xfs_admin -l /dev/device

# Reset UUID
xfs_admin -U nil /dev/device
xfs_admin -U restore /dev/device
xfs_Admin -U generate /dev/device


tune2fs -L "ext4Disk" /dev/device
tune2fs -l /dev/device
```

## Setting up NFS

```bash
yum install nfs-utils
mkdir /nfs
chmod 777 /nfs
echo "/nfs *(rw)" > /etc/exports
systemctl start {rpcbind,nfs-server,rpc-statd,nfs-idmapd}
showmount -e localhost

# On client
systemctl start rpcbind
showmount -e <ip-of-server>
mount -t nfs <ip>:/nfs /mnt/nfs
```

## Setting up samba

```bash
# On server
yum install samba
mkdir /smb
chmod 777 /smb

echo "[share]" >> /etc/samba/smb.conf
echo "browseable = yes" >> /etc/samba/smb.conf
echo "path = /smb" >> /etc/samba/smb.conf
echo "writable = yes" >> /etc/samba/smb.conf

useradd sambauser
smbpasswd -a sambauser

setenforce 0


# On client
mkdir /mnt/smb
yum install cifs-utils samba-client
mount -t cifs //<ip-of-server>/share /mnt/smb -ousername=sambauser,password=12345
```

## ACL File Permissions

```bash
getfacl file
# Set permissions
setfacl -m g:groupname:rwx file
# Remove permissions
setfacl -x u:username:rwx file
# Recursive permissions
setfacl -R -m g:groupname:rw folder/
# Default permissions
setfacl -d -m g:groupname:rwx folder/
# Remove default permissions
setfacl --remove-default folder/
# Copy permissions
getfacl file1 | setfacl --set-file=- file2
```

## Network Configuration

```bash
nmcli conn show

nmcli conn add con-name <name> autoconnect yes type ethernet ifname eth0 ip4 <ip> gw4 <ip>

nmcli conn up <name>

nmcli conn modify <name>

nmcil conn modify <name> +ipv4.dns 8.8.8.8

## Check dns first
vi /etc/nsswitch.conf
```

## NTP

```bash
timedatectl status

yum install chrony

chronyc tracking

chronyc sources -v

# Edit NTP servers
vi /etc/chrony.conf

systemctl restart chrony
```

## Scheduling Tasks

```bash
at now +1 minute
at midnight
atq # List pending jobs
atrm # Delete job by number

echo "username" >> /etc/at.deny # For denying users
```

```bash
crontab -e # Current User
crontab -e -u <username> # For username

echo "*/15 * * * * script.sh" >> /etc/cron/user # Run every 15 minutes

echo "username" >> /etc/cron.deny # Deny specific user
```

## Modifying bootloader

```bash
grubby --info=ALL
grub2-set-default <index>
grubby --set-default-index <index>
```

## Yum Repos

```bash
echo "[base]" >> /etc/yum.repos.d/base.repo
echo "name = Base" >> /etc/yum.repos.d/base.repo
echo "enabled = 1" >> /etc/yum.repos.d/base.repo
echo "baseurl = http://baseurl.com" >> /etc/yum.repos.d/base.repo
echo "gpgcheck = 0" >> /etc/yum.repos.d/base.repo

yum repolist
```

## Using LDAP for SSO
```bash
yum install authconfig-gtk nss-pam-ldapd pam_krb5 autofs nfs-utils openldap-clients

authconfig --enableldap --enableldapauth --enablemkhomedir --enableldaptls --ldaploadcacert=http://server.com/path/to/ca.pem --ldapserver=ldap.server.com --ldapbasedn="dc=example,dc=com" --update

# For GUI
authconfig-gtk

su - ldapuser1
```

## Configure AD using realmd

```bash
yum install realmd
realm discover <AD Server>
realm join <AD Server>

ssh -l test@example.com <IP-address>
```

## Manipulating Users

```bash
id <user>
getent shadow <user>

# Change display name
usermod -c "Name" <user>

usermod -aG <group> <user>

# Lock user
usermod -L <user>


# Skeleton directory
ll /etc/skel/

# useradd defaults
cat /etc/default/useradd

cat /etc/login.defs
```

## Password management

```bash
chage -l <user>
 
# Password validity
chage -M 30 <user>

# Inactivity after expiry
chage -I 1 <user>

# Set lockout date
chage -E 2020-11-11 <user>

# Days before expiry when user starts to receive warning
chage -W 10 <user>
```

## Group management

```bash
groupadd <group>

# Change primary group
usermod -g <group> <user>

# Change secondary group
usermod -G <group> <user>

# Add additional group
usermod -aG <group> <user>

groupmod -g <new-id> <group>
```

## Configure firewall

```bash
yum install firewalld firewall-config
firewall-cmd --get-zones
firewall-cmd --get-default-zone
firewall-cmd --zone=home list-all
firewall-cmd add-service=http
firewall-cmd add-service=http --permanent
firewall-cmd add-port=80/tcp
firewall-cmd --reload
```

## SELinux

```bash
getenforce
setenforce 1
getsebool httpd_enable_cgi
setsebool httpd_enable_cgi on
getsebool -a | grep httpd

semanage fcontext -l

ls -lZ /var/www/html

ps auxZ | grep httpd

semanage fcontext -a -t httpd_sys_content_t '/content(/.*)?'
restorecon -R /content

# Troubleshooting
yum install setroubleshoot-server
sealert -a /var/log/audit/audit.log
```


## Make journald persistent

```bash
mkdir -p /var/log/journal
systemctl restart systemd-journald
```

## Adjust process priority via nice

Highest priority = -20
Lowest priority = 19

```bash
nice -n 5 script.sh

renice -n <priority> -p <pid>
```

## Disk compression (VDO)

```bash
yum install vdo kmod-kvdo
systemctl start vdo.service

vdo create --name=MyVDO --device=/dev/nvme1n1 --vdoLogicalSize=60G --deduplication=disabled

mkfs.xfs -K /dev/mapper/MyVDO
udevadm settle
mount /dev/mapper/MYVDO /mnt/vdo
```

## Yum modules

```bash
yum module enable <module-name>:<stream>
yum module enable postgresql:9.6

yum install postgresql

yum module list

```

## Linux Schedulers

```bash
chrt -p <pid>

# Use FIFO
chrt -f -p <PRIO> <pid>

chrt -f <PRIO#> /path/to/command

chrt -d --sched-runtime 5000000 --sched-deadline 1000000 --sched-period 1666666 0 /your/command/here
```

## Extending LVM Disks

```bash
pvcreate /dev/sdc1
vgextend <VGNAME> /dev/sdc1
lvextend -r -L +100%FREE /dev/<VGNAME>/<LVNAME>
```

## Extending VDO Disks

```bash
vdo growPhysical --name=my_vdo
vdo growLogical --name=my_vdo -vdoLogicalSize=new_logical_size

vdostats --human-readable
```

## Sample exam questions

- Create users and groups etc

```bash
useradd tom
useradd kenny
useradd derek
groupadd instructors
usermod -aG instructors tom
usermod -aG instructors kenny
usermod -aG instructors derek
usermod -s /sbin/nologin tom
chage -E $(date -d "+10 days" %y-%m-%d) tom
```

- Configure apache

```bash
nmcli conn show
nmcli conn up eth0

yum repolist

yum install httpd

vi /etc/httpd/conf/httpd.conf # Change DocumentRoot to desired root

mkdir /var/web
echo "Hello world!" > /var/web/index.html
systemctl restart httpd

firewall-cmd list-services
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

ls -lZ /var/web/

yum whatprovides semanage
yum install policycoreutils-python

semanage fcontext -a -t httpd_sys_content_t '/var/web(/.*)?'
restorecon -R /var/web
systemctl enable httpd
```

- Modify umask for all users to 006

```bash
vi /etc/profile
vi /etc/bashrc
# Change umask value to desired
```

- Find files that are 720 days old

```bash
find /etc -maxdepth 1 -mtime +720 > /root/oldfiles
```

- Save logs containing ACPI to another file

```bash
cat /var/log/messages | grep ACPI > /root/logs

tar -czf /tmp/log_archive.tgz /var/log
```

- Reduce boot timout for grub to 2 seconds

```bash
vi /etc/default/grub # Change GRUB_TIMOUT to 2
grub2-mkconfig
```

- Schedule for user

```bash
crontab -e derek
# 27 16 * * * cat /etc/redhat-release >> /home/derek/release
```

- Change NTP server

```bash
vi /etc/chrony.conf
# Remove all other servers and add required one
systemctl restart chronyd
```

- Create swap partition

```bash
fdisk -l
# Create new partition with 82 as type for swap
mkswap /dev/vdb1
blkid
echo "UUID=<UUID> swap swap defaults 0 0" >> /etc/fstab
mount -a
free
swapon /dev/vdb1
partprobe
free
```

- Create new volume using LVM

```bash
yum install lvm2
pvcreate /dev/vdc
vgcreate myVG /dev/vdc -s 32m
lvcreate -n myLV -l 30 myVG
mkfs.xfs /dev/myVG/myLV
blkid
echo "/dev/mapper/myVG-myLV /mnt xfs defaults 0 0" >> /etc/fstab
partprobe
mount -a
df -h
```

- Change hostname

```bash
hostnamectl set-hostname <hostname>
```

- Configure LDAP properly

```bash
yum install -y authconfig-gtk nss-pam-ldapd pam_krb5 autofs nfs-utils openldap-clients
authconfig-gtk

echo "/home/guests /etc/auto.ldap" >> /etc/auto.master.d/ldap.autofs
echo "* -rw ldap.linuxacademy.com:/home/guests/&" >> /etc/auto.ldap
vi /etc/pam.d/sshd
# Add `auth sufficient pam_ldap.so` to the start of the file
systemctl restart ssdh
su - ldapuser3
```
