---
date: '2011-08-11 17:32:46'
layout: post
slug: setting-up-a-manageable-kvm-host-part-1-using-lvm-for-storage
status: publish
title: Setting up a manageable KVM host - part 1 - using LVM for storage
wordpress_id: '23'
categories:
- Server Administration
- Virtualization
tags:
- KVM
- LVM
- server administration
- ubuntu
- Ubuntu 10.10
- virtualization
---

.. role:: code
   :class: inline-code

This is going to be a three part series (maybe more, maybe less) highlighting different aspects of KVM that I have found difficult to setup and how I managed to solve them.

What's KVM and why use it?
==========================

KVM stands for Kernel-based Virtual Machine, "a full virtualization solution for Linux on x86 hardware containing virtualization extensions" - `KVM home <http://www.linux-kvm.org/page/Main_Page>`_, and is now part of the Linux kernel. As you might know, most linux distros are choosing KVM as their virtualization technology. This makes it the best option if you are just getting into the virtualization domain.

.. more

Setting up the Host and Building Virtual Machines
=================================================


There is a pretty extensive tutorial about setting up a KVM host and creating Virtual Machines using vmbuilder. So I'm just gonna link it here. I have not run into any problems using the linked tutorial except for one problem which I will highlight afterwards. So here is the link: `Virtualization With KVM On Ubuntu 10.10 <http://www.howtoforge.com/virtualization-with-kvm-on-ubuntu-10.10>`_.

Using LVM for storage
=====================

If you followed the above tutorial, it also highlights how to use LVM logical volumes as the storage for your VMs, but to my surprise my VMs were using qcow2 images instead even if I specificed the :code:`-f /dev/vg1/lv1` as the hard disk image file. Vmbuilder did not use LVM storage!

vmbuilder doesn't use the :code:`-f /dev/vg1/lv1` directive and instead creates a .qcow2 format image and uses it as the storage.

So what you need to do is copy the qcow2 file into the Logical Volume you create and set the disk of the VM to that Logical Volume.

Changing the VM to use the Logical Volume
-----------------------------------------

Before you do this you will need to stop your VM.

Then you need to copy the contents of qcow2 file into the logical volume. Say you created your vm in /vm/vm1 then your qcow2 file will be something like :code:`/vm/vm1/ubuntu-kvm/tmp5Wg2ak.qcow2`. The file name is different for each instance but will start with 'tmp' and end with '.qcow2' and your logical volume is :code:`/dev/vg1/lv1`.

.. sourcecode:: console
   :caption: bash
    
    qemu-img convert -f qcow2 /vm/vm1/ubuntu-kvm/tmp5Wg2ak.qcow2 -O host_device /dev/vg1/lv1

The use of :code:`-O host_device` is mostly found as :code:`-O raw`, but :code:`-O raw` does not currently work due to a bug.

Now the domain definition file for the VM has to be changed to point to the Logical Volume instead of the qcow2 file. If you named your VM vm1 then you can find the definition file in :code:`/etc/libvirt/qemu/vm1.xml`. You need to edit it:

.. sourcecode:: console
   :caption: bash
    
    nano /etc/libvirt/qemu/vm1.xml

Find the following lines:
    
.. sourcecode:: console
   :caption: /etc/libvirt/qemu/vm1.xml

    ...
    <disk type='file' device='disk'>
        <driver name='qemu' type='qcow2'/>
        <source file='/vm/vm1/ubuntu-kvm/tmp5Wg2ak.qcow2'/>
        ....

And make the following changes:

.. sourcecode:: console    
   :caption: /etc/libvirt/qemu/vm1.xml

    ...
    <disk type='file' device='disk'>
        <driver name='qemu' type='raw'/>
        <source file='/dev/vg1/lv1'/>
        ....

I have changed the driver type to raw which was qcow2 before and the source file to '/dev/vg1/lv1' which was '/vm/vm1/ubuntu-kvm/tmp5Wg2ak.qcow2'.

Now you need to activate the new definition:

.. sourcecode:: console    
   :caption: bash

    virsh connect qemu:///system
    virsh define /etc/libvirt/qemu/vm1.xml

Now when you start up the VM it will be using the Logical Volume.

Conclusion
==========

This concludes the part 1 of this series.

In the next part of this series I will show you how to easily create new VMs from a previous/template guest. There are some involved changes that needs to be done to get the template system.
