---
title: Installing Windows 10 in Qemu with KVM
tags: 'windows10, ubuntu, ubuntu18.04, kvm, qemu, virtio, virt-install, virsh, vm'
date: 2018-11-04 20:34:27
---


These are the steps I arrived at in order to install a Windows 10 Guest on an Ubuntu 18.04 host using virt-manager to install a Qemu VM with KVM acceleration and VirtIO drivers.

## Prerequisites

Create a directory to work in and install the tools we will need.  The _virtio-win_ ISO image contains the drivers we will need in order to make Windows bootable.

```
mkdir windows
cd windows/
sudo apt-get install virt-manager
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.160-1/virtio-win-0.1.160.iso
```

You will also need to copy your Windows 10 installer ISO into your `windows/` directory.

## Disk Setup

This creates a blank image that will be attached as a virtual hard drive to the Windows instance.  We use qcow2 because it supports some nice extensions above those on the _raw_ format, like thin provisioning of storage.

```
qemu-img create -f qcow2 windows_10_x64.qcow2 75G
```

## Creating the VM

Call `virt-install` to create the VM.

```
virt-install \
    --name=windows10 \
    --ram=2048 \
    --cpu=host \
    --vcpus=2 \
    --os-type=windows \
    --os-variant=win10 \
    --disk path=windows_10_x64.qcow2,format=qcow2,bus=virtio \
    --disk en_windows_10_enterprise_x64_dvd_6851151.iso,device=cdrom,bus=ide \
    --disk virtio-win-0.1.160.iso,device=cdrom,bus=ide \
    --network network=default,model=virtio \
    --graphics vnc,password=trustworthypass,listen=0.0.0.0

```

Verify the running VMs with the `virsh` command:

```
~/windows $ virsh list
 Id    Name                           State
----------------------------------------------------
 4     windows10                      running

```

## Connecting to the VM

Now that the VM is running, you need to connect to it and install Windows.  VNC to the host machine on port 5900 using the password you specified in the `virt-install` command. _You did change that password, didn't you?_

On Mac OS you can use `open` to call the built in VNC client.

```
open vnc://<host>:5900
```

## Windows Install

There will be a few extra steps to install Windows above and beyond a normal install.  Because the accelerated VirtIO drivers required to interface with the virtual disk controller are not bundled with Windows, we need to load them into the installer before it will be able to talk to the virtual hard drive.

When asked to do an _Upgrade_ or a _Custom install_, select _Custom Install_.
{% asset_img "Windows 10 Install - 1 - Custom Install.png" %}

Select _Load Driver_ to point the installer to your driver file.
{% asset_img "Windows 10 Install - 2 - Load Driver.png" %}

Navigate to `E:\viostor\w10\amd64`
{% asset_img "Windows 10 Install - 3 - viostor driver.png" %}

There should only be one option, the VirtIO SCSI Controller.
{% asset_img "Windows 10 Install - 4 - select driver.png" %}

The installer should now see your virtual disk.  Hit next to let Windows automatically partition it.
{% asset_img "Windows 10 Install - 5 - Partition.png" %}

## Next steps

Once the installer finishes and you get into Windows you may want to do a few more things

* Install the other drivers on the virtio-win ISO (_network, etc..._)
* Apply Updates
* Enable RDP
