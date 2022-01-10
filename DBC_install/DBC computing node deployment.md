#                        DBC computing node deployment



## Pre-installation preparation (based on the configured fixed public network ip address), deploy the KVM installation environment

### Note: Please uninstall the installed graphics driver before starting, this operation cannot have graphics driver

```shell
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get  install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager ovmf cpu-checker vim -y
```



## create and mount the XFS file system

### 1. Check the hard disk partition

`lsblk`

### 2. Create a data disk folder, format the hard disk, and mount the hard disk (the data disk mounting directory must be /data)

```shell
sudo mkdir /data
sudo apt-get install xfsprogs -y
sudo mkfs.xfs -n ftype=1 -f /dev/sdb  （Whether it is sdb here depends on the situation of lsblk）
sudo mount  -o pquota /dev/sdb /data
sudo chmod 777 /data
sudo echo "/dev/sdb /data     xfs pquota 0 1" >> /etc/fstab
sudo mount -a
```



## Determine whether the machine supports virtualization

### 1. Turn on hardware support

> BIOS open VT-d (search according to the motherboard type browser)
> VT (VT-x) and VT-d support, you need to set related support to enable, which is enabled by default
>
> Path under normal circumstances: Processor—IIO Configuration—Intel@ VT for Directed I/O(VT-d)

### 2. Environment dependence, check whether the CPU supports virtualization and whether KVM is available

`egrep -c '(svm|vm)' /proc/cpuinfo`

> CPU detection, if it is displayed as 0, virtualization is not supported

`kvm-ok`

> Check if kvm is available
>
> display INFO: /dev/kvm exists  
> KVM acceleration can be used
> Indicates that subsequent operations can be performed. If the display does not match it, please check whether VT-d is turned on correctly

## If you are a ubuntu20.04 system, please do the following

+ Set up a blacklist to prevent the card from being occupied
```shell
sudo vim /etc/modprobe.d/blacklist.conf
#Finally add content:
blacklist snd_hda_intel
blacklist amd76x_edac
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist nvidiafb
blacklist rivatv
```
+ Set up graphics pass-through

```shell
#Query graphics card ID
lspci -nnv | grep NVIDIA
Copy the ID of the graphics card, such as 10de:2231 10de:1aef, and keep the duplicate content only once

#Modify the core file
sudo vim /etc/default/grub
#Add in GRUB_CMDLINE_LINUX_DEFAULT field (if it is AMD platform, intel_iommu=on is changed to amd_iommu=on)
quiet splash intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=<graphics card id, separated by commas>

#Update kernel
sudo update-grub

#Restart the machine
#Query graphics card occupancy
lspci -vv -s <graphics card PCI interface> | grep driver
```
> If vfio-pci is displayed, it is normal. For non-vfio-pci, please go back and check whether the grub file is written correctly or ***Follow step 6 and 2 for manual binding***
***This is the end of the 20.04LTS system graphics isolation step, please go to step 7 to continue the operation***


## If you are a ubuntu18.04 system, please continue to operate
## enable system grouping

### 1. Configure intel_iommu

```shell
sudo vim /etc/default/grub

#Add in the GRUB_CMDLINE_LINUX_DEFAULT field
intel_iommu=on iommu=pt rd.driver.pre=vfio-pci
#Add in GRUB_CMDLINE_LINUX field
intel_iommu=on iommu=pt rd.driver.pre=vfio-pci
```

### 2. Configure the module file

```shell
sudo vim  /etc/modules
#Add the following content:
pci_stub
vfio
vfio_iommu_type1
vfio_pci
kvm
kvm_intel

#Update grub.cfg file
sudo update-grub

#Restart the machine, check whether iommu is correctly enabled (or restart and check after subsequent operations)
dmesg | grep -i iommu

#Display is similar to [3.887539] pci 0000:83:00.1: Adding to iommu group 46 means successful activation
```



## isolate GPU resources

### 1. Set up a blacklist to prevent the card from being occupied

```shell
sudo vim /etc/modprobe.d/blacklist.conf  
#Finally add content:
blacklist snd_hda_intel
blacklist amd76x_edac
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist nvidiafb
blacklist rivatv
```

### 2. Collect PCI device information

```shell
lspci -nnv | grep NVIDIA
#Display similar to
17:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080] [10de:1e82] (rev a1) (prog-if 00 [VGA controller])
17:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
17:00.2 USB controller [0c03]: NVIDIA Corporation TU104 USB 3.1 Host Controller [10de:1ad8] (rev a1) (prog-if 30 [XHCI])
17:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller [10de:1ad9] (rev a1)
65:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080] [10de:1e82] (rev a1) (prog-if 00 [VGA controller])
65:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
65:00.2 USB controller [0c03]: NVIDIA Corporation TU104 USB 3.1 Host Controller [10de:1ad8] (rev a1) (prog-if 30 [XHCI])
65:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller [10de:1ad9] (rev a1)

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
#Record all device codes and PCI id (repeated codes are only taken once)
#E.g:
#equipment number:
10de:1e82,10de:10f8,10de:1ad8,10de:1ad9    （Repeat only once）
#PCI interface id (The PCI interface of each machine is different, please note the record)
17:00.0
17:00.1
17:00.2
17:00.3
65:00.0
65:00.1
65:00.2
65:00.3
```

### 3. Set up vfio and isolate the GPU for pass-through

```shell
sudo vim /etc/modprobe.d/vfio.conf
#Write the device code information collected above (if repeated, just write it once):
options vfio-pci ids=10de:1e82,10de:10f8,10de:1ad8,10de:1ad9

sudo vim /etc/modules-load.d/vfio-pci.conf
#Write the following
vfio-pci kvmgt vfio-iommu-type1 vfio-mdev

#Restart the machine
sudo reboot
```

### 4. Check the GPU status (all interfaces must be queried to prevent it from being occupied by vfio-pci)

```shell
#Please pay attention to the replacement of PCI interface content!
lspci -vv -s <PCI interface> | grep driver
#E.g:
lspci -vv -s 17:00.0 | grep driver
lspci -vv -s 17:00.1 | grep driver
lspci -vv -s 17:00.2 | grep driver
lspci -vv -s 17:00.3 | grep driver

#No output means there is no driver.
#If Kernel driver in use: vfio-pci is displayed, the isolation is successful
#If the display is similar to Kernel driver in user: snd_hda_intel indicates that the device is occupied by other drivers
```

> If there is a PCI that is not occupied by vfio-pci, please continue to execute, if it has been successfully occupied by vfio-pci, you can skip the next step



## If the driver query is Kernel driver in use: vfio-pci, there is no need to operate the following content, please continue to execute if the binding is not successful

### 1. Unbind the device

> If the driver query shows a non-Kernel driver in user: vfio-pci, unbind the device (each group of IDs must be unbound, the following is only an example, please modify it according to your own query pci interface)

```shell
#Please pay attention to the replacement of the content, the following command is only for demonstration (need to unbind all occupied graphics card pci interfaces)
sudo -i
sudo echo 0000:17:00.0 > /sys/bus/pci/devices/0000\:17\:00.0/driver/unbind
sudo echo 0000:83:00.0 > /sys/bus/pci/devices/0000\:83\:00.0/driver/unbind


sudo modprobe vfio
sudo modprobe vfio-pci
sudo reboot

#Restart the host and check if the GPU is isolated in a different IOMMU group and the vfio driver is being used
#Execute the command to check whether the GPU is isolated in different IOMMU groups
find /sys/kernel/iommu_groups/*/devices/*
#Display grouping is normal

#Re-query PCI (pay attention to replace), if vfio-pci is still not queried or other content is displayed, please perform a next step
lspci -vv -s 17:00.0 | grep driver
```

### 2. Manually bind the GPU

```shell
#Execute the command to bind (Note: the content after echo is the ID of the graphics card that the machine queried) The already occupied PCI does not need to be manually bound
sudo -i
sudo echo 10de 1e82 > /sys/bus/pci/drivers/vfio-pci/new_id
sudo echo 10de 2206 >> /sys/bus/pci/drivers/vfio-pci/new_id
…………


#Check again after binding (check all items of each card)
lspci -vv -s 17:00.0 | grep driver
#If Kernel driver in use: vfio-pci appears, the binding is successful. If still unsuccessful, please go back and check
```



## After confirming that the graphics card of the machine is occupied by vfio-pci, start the libvirtd service and set the boot to start automatically

### 1. Turn on the virt tcp monitoring service:

```shell
sudo vim /etc/libvirt/libvirtd.conf
#After the arrow is the modified content: remove the # in front of these three lines, and change sasl to none

#listen_tls = 0	=======>	listen_tls = 0
#listen_tcp = 1	=======>	listen_tcp = 1
#auth_tcp = "sasl"	======>	auth_tcp = "none"

sudo vim /etc/default/libvirtd
#Corresponding modification to the following configuration
libvirtd_opts="-l"
```

### 2. Start libvirtd and set it to start at boot

sudo systemctl start libvirtd.service
sudo systemctl enable libvirtd.service



## Create a dbc user

```shell
sudo wget http://116.85.24.172:20444/static/add_dbc_user.sh
sudo chmod +x add_dbc_user.sh
sudo ./add_dbc_user.sh dbc
#dbcUser password is set by yourself
```



## Install the DBC node program

+ Note: need to switch to dbc user installation （github：https://github.com/DeepBrainChain/DBC-AIComputingNet/releases）
+ Please refer to the installation/upgrade DBC program https://github.com/DeepBrainChain/DBC-AIComputingNet/releases



## Download the mirror template

+ http://36.102.233.175:20027/index.html/ubuntu-img/

Download: ubuntu.qcow2 and ubuntu-2004.qcow2 these two mirrors


## Back up the machine id and private key (very important,if this private key is lost, 50% of the pledged coins will be lost, please pay attention to multiple backups)

Back up the contents of the following file: ` <installation path>/dat/node.dat`, put it in a safe place, and use it later If you reinstall the system or reinstall DBC later, you need to use the original id and private key, otherwise the pledged coins will be deducted


## Test to create a virtual machine with graphics card pass-through to check whether the previous configuration is correct
+ Test program download address: https://github.com/DeepBrainChain/DBC-AIComputingNet/releases
+ Binary file, add execute permission and execute directly: chmod 777 chec_env ; ./check_env
+ If the green check 'vm domain_test successful' appears, it means success. If it does not appear, please check whether the previous configurations are correct.

## Check whether the various hardware parameters of the machine are normal
+ If the previous step is successful, a virtual machine will be successfully created, and log in to the virtual machine through ssh, where: vm_local_ip is the virtual machine's intranet ip address, the user name is dbc, and pwd is the login password
+ ![image](https://user-images.githubusercontent.com/32829693/129731433-3e01b669-f274-419e-9ea0-d7891705a12e.png)
+ Then cd to the test script directory and run:【pytest .】，
     + cd /test/dbc_gpu_server_test/
     + sudo -i (Switch to root user)
     + pytest .
+ A total of 18 tests;
     + 10 unit tests, testing CPU, memory, hard disk, graphics card, video memory, cuda usability, etc.;
     + 7 integration tests to test whether the actual usage conditions are normal (such as pytorch calculation, training and inference), and eliminate potential hardware failures;
     + 1 benchmark speed test, testing the training and inference of dozens of CNN networks, lasting about ten minutes;
     + If there is no red error, it will pass. If there is a red F/error, the test item corresponding to the error will be displayed, which can be checked according to the information;
     + The full test process of 4 cards 2080ti is about 10 minutes. If the test time is too long, such as more than half an hour, there may be a problem with the machine, and the test can be aborted in advance.
     + Short test summary info in the test result: If all are passed, it means the test passed, as long as one item is failed, it means the test failed and the fault needs to be checked;
     + After the end, the 'result' folder is generated to export the performance report;
+ Back to the host, shut down and delete the tested virtual machine: ./check_env --localip x.x.x.x (x.x.x.x is the internal network ip address of the virtual machine. If you do not operate this step, the dbc program will not be able to start the new virtual machine. Passed on-chain verification)


## Check whether the machine is correctly added to the computing power network
+ Use the official client node to view
+ Mine pool build client node
+ For the above two points, please see: https://github.com/DeepBrainChain/DBC-DOC/blob/master/DBC_install/DBC_client_node_construction_steps_english.md
+ About client nodes: It is recommended that each mining pool set up 2 or more client nodes to ensure that the network can still be normal when the official nodes or other mining pools provide nodes are offline. If there are too few client nodes in the network or hang Too much drop will affect the rental situation of the machine. The client node construction can start a container to deploy on other servers without taking up too much resources.


## Machine on the chain

https://github.com/DeepBrainChain/DBC-DOC/blob/master/chain_ops/machine_online_en.md

