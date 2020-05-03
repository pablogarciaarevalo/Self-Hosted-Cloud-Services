# MyLab based on Raspberry

This repository includes information about:
* [Tasks done and things to do](README.md)
* [Architecture & Components](/architecture.md)
* [References](/references.md)

## Tasks done and things to do

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

* Current Raspberry nodes: 192.168.1.42, 192.168.1.43, 192.168.1.44 / user: pi
* Set hostname, change pi user password, set wifi SSID and enable SSH
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



- [x] NFS Storage

