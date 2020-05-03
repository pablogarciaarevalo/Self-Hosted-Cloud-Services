# Self Hosted Cloud Services

This repository includes information about my own Self Hosted Cloud Services based on Kubernetes on Raspberry PI devices:
* [Cloud Services](README.md)
* [Tasks done and things to do](ToDo.md)
* [Architecture & Physical components](/architecture.md)
* [References](/references.md)

## Tasks done and things to do

The project current status can be check in the [Kanban board](https://github.com/pablogarciaarevalo/Self-Hosted-Cloud-Services/projects/1)

The below information includes the configuration details.

### Things to Do:

- [ ] K3S
- [ ] MetalLB
- [ ] PiHole
- [ ] Minio
- [ ] OpenFaas
- [ ] OpenVPN
- [ ] NextCloud
- [ ] Selfhosted apps
- [ ] TOR Gateway
- [ ] OpenHub

### Tasks Done:

- [x] Router configuration
* DNS1 = 192.168.1.100 / DNS2 = 8.8.8.8

- [x] Install Raspbian Lite
* Download the last [Raspbian Linux OS](https://www.raspberrypi.org/downloads/raspbian/) version
* Upgrade the OS
```shell
sudo apt-get update && sudo apt-get upgrade -y
```
* Configure a management and service static IP addresses in the /etc/dhcpcd.conf file
```shell
interface eth0
static ip_address=192.168.42.42/24
interface wlan0
static ip_address=192.168.1.42/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8
```
* Current configuration

| Node Name | Management IP |  Service IP  |
| --------- |:-------------:| ------------:|
|   node0   | 192.168.42.42 | 192.168.1.42 |
|   node1   | 192.168.42.43 | 192.168.1.43 |
|   node2   | 192.168.42.44 | 192.168.1.44 |

* Set hostnames, change pi user password, set wifi SSID and enable SSH
```shell
raspi-config
```
* Disable password login 
```shell
ssh-keygen
ssh-copy-id pi@192.168.1.xx
```
* Enable container features add the following to the end of the file /boot/cmdline.txt
```shell
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```
* Set local DNS resolution in /etc/hosts
```shell
192.168.1.42	node0
192.168.1.43	node1
192.168.1.44	node2
```

- [x] NFS Storage

* Find the disks name in the PI node0
```shell
fdisk -l
```
* Create the mirror raid in the PI node0
```shell
mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sda1 /dev/sdb1
```
* Check the raid status and run a config backup in the PI node0
```shell
cat /proc/mdstat
mdadm --detail /dev/md0
mdadm --readwrite /dev/md0
mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

cat /etc/mdadm/mdadm.conf
ARRAY /dev/md/0 metadata=1.2 name=node0:0 UUID=8d257fb4:26471052:6f05906b:3beff53f
```

* Create the partitions in the PI node0
```shell
mkfs -t ntfs --fast /dev/md0
mkfs -t ntfs --fast /dev/sdc1
```
* Automatically mount the disk on the boot process in the PI node0
```shell
blkid
/dev/md0: UUID="09719E230BFFEE00" TYPE="ntfs" PTTYPE="dos"
/dev/sdc1: UUID="5C36A84D6703C10E" TYPE="ntfs" PTTYPE="dos" PARTUUID="a6cd6b56-01"

vi /etc/fstab
UUID=09719E230BFFEE00 /mnt/raid1/ ntfs defaults,noatime,nofail 0 1
UUID=5C36A84D6703C10E /mnt/raid0/ ntfs defaults,noatime,nofail 0 1
```
* Mount and check in the PI node0
```shell
mkdir /mnt/raid0
mkdir /mnt/raid1
mount -a

df -Th
...
/dev/md0       fuseblk   7.7G   44M  7.6G   1% /mnt/raid1
/dev/sdc1      fuseblk   466G   84M  466G   1% /mnt/raid0
...
```
* Configure the NFS Server in the PI node0
```shell
apt-get install nfs-kernel-server -y

vi /etc/exports
/mnt/raid0 *(rw,no_root_squash,insecure,async,no_subtree_check,anonuid=1000,anongid=1000)
/mnt/raid1 *(rw,no_root_squash,insecure,async,no_subtree_check,anonuid=1000,anongid=1000)

exportfs -ra
```
* Configure the NFS Client in the PI node1 and node2
```shell
apt-get install nfs-common -y

mkdir /mnt/raid0
mkdir /mnt/raid1

vi /etc/fstab
192.168.1.42:/mnt/raid0  /mnt/raid0   nfs    rw  0  0
192.168.1.42:/mnt/raid1  /mnt/raid1   nfs    rw  0  0
mount -a

mount -Th
192.168.1.42:/mnt/raid0 nfs4      466G   84M  466G   1% /mnt/raid0
192.168.1.42:/mnt/raid1 nfs4      7.7G   44M  7.6G   1% /mnt/raid1
```
