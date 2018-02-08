---
layout: post
title: Ovftool is a must have for ESXi user
date: 2018-02-08 17:02:42 +0800
categories: ovftool vmware ESXi vsphere
---

When downloaded a sample VM image
=================================

Today, to kick start my cloudra learning journey, I finally downloaded
a cloudra getting started image from Cloudra official site. Well this
image is not so huge, I still don't have much space to store it on my
local machine.

As I have a ESX host siting several cubes away from my seat. I decide
to upload the image to the host and run from there.

## import a VM to ESX ##

After play around the ESX client for some minutes I find that I can
only import a OVF or OVA image.

## upload VM to ESX storage directly ##

Alternatively one have to upload the
whole VM image by using the storage manager, then try add the vmx file
to inventory from there.


As said, and sad, the image is a little bit big, my VM image upload
process end up successful, but the VM added from storage manager
failed to start up. An error of "can not find vmdisk xxxx" with a
mysterious vmdk filename showing up.

## convert and then import, in one STEP ##

The last solution for me to upload the image to ESX is to convert the
vm image to OVF format and then import to ESX host after the
conversion.

But wait, the OVF tool is actually able to take one step further. It
is able to ESX host directly. It has this feature built-in, and the
conversion and uploading is really convenient according to OVF tool
document:

    Deploying an OVF Package Directly on an ESXi Host
    The following command deploys an OVF package on an ESXi host.
    > ovftool package.ovf vi://my.esx-machine.example.com/
    If your host has multiple data stores, use the -ds option:
    > ovftool package.ovf -ds=storage1 vi://my.esx-machine.example.com/
    See also "Special Consideration - Running the OVF Tool from ESXi instead of vCenter" on page 12.
	
My own command looks like following:

    ekaifan@CN00059491 C:\Users\ekaifan\Downloads\cqvm
    > "c:\Program Files\VMware\VMware OVF Tool\ovftool.exe" --skipManifestGeneration
    Opening VMX source: ..\cloudera-quickstart-vm-5.12.0-0-vmware\cloudera-quickstar
    Accept SSL fingerprint (45:32:08:70:60:0F:A8:92:A1:53:C4:3D:02:2E:E5:35:63:1B:C3
    Fingerprint will be added to the known host file
    Write 'yes' or 'no'
    yes
    Enter login information for target vi://100.98.40.225/
    Username: root
    Password: ********
    Opening VI target: vi://root@100.98.40.225:443/
    Deploying to VI: vi://root@100.98.40.225:443/
    Transfer Completed
    Completed successfully

My VM started up successfully without any problem. Happy ending.
