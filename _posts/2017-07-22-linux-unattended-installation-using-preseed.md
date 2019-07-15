---
layout: post
title: Linux unattended installation using preseed
categories: Linux
permalink: /:categories/:title
post_description: Using preseed to completely automate the installation of Ubuntu 16.04.
cover: "/assets/images/post-thumbnails/ubuntu-16-04-preseed-unattended-install.svg"
author: dan_hand
---

Linux unattended installation using preseed
For a while now, I have been looking for a way to automate UEFI Linux installations. I use Hyper-V on a daily basis and have tried various methods from PXE, System Center Virtual Machine Manager, as well as other FAI systems like Foreman. PXE and System Center Virtual Machine Manager both work perfectly with Legacy or Generation 1 virtual machines with Hyper-V, but not for Generation 2 (UEFI). System Center Virtual Machine Manager is supposed to be compatable with Generation 2 virtual machines, but since 2012 R2, when Linux support was announced, there has been a bug where by a virtual machine template will not boot once created. I’ve scoured the internet to try and find a fix for this, but it doesn’t look like I’m the only one.

So, I couldn’t get Generation 2 virtual machines to boot without user interaction, so I decided to try and automate things post install. This is where I wanted to use Ansible. I wrote a post about provisioning multiple Linux servers with Ansible, explaining how to automate the installation of packages and other items. I figured, I could create the new virtual machine, then SSH in afterwards with Ansible. However, I then came across the issue of SSH keys, as well as needing Python to properly use Ansible. Yes, there are ways to enter a password in order to dial in and install the SSH keys and Python, but these methods were dirty and required extra flags.

I called out to Reddit. Another user suggested using a preseed to install the SSH keys and packages needed for Ansible in a post install script. I had heard of using preseeds in the past but this never actually crossed my mind as a viable solution. Not one that I thought would work with UEFI installations anyway. I got reading. This is when I discovered the power of using a preseed. My hands raised to the air with joy. I had spent many, many hours gritting my teeth at solutions that didn’t work (at least for me).

I started trying to make my own ISO with by editing the default built in preseed with my own. I got a basic preseed working, although a bit rusty. However, I figured, I may want to edit the preseed on the fly on a per VM basis but I didn’t want to have to make new install media each time. Then I came across how I could load the preseed file over the network. This meant, I could edit various parameters of the preseed, easily and without creating new install media.

Here’s how I did it. You can also see a scaled down guide on how to complete this by visiting the Github repository here: [https://github.com/dsgnr/Ubuntu-16.04-Unattended-Install](https://github.com/dsgnr/Ubuntu-16.04-Unattended-Install).

Follow the steps below to create your own Ubuntu 16.04 preseed ISO.

### Get Ubuntu
First, we need to start by grabbing the original install media.

{% highlight bash %}
cd /root
wget http://releases.ubuntu.com/16.04.2/ubuntu-16.04.2-server-amd64.iso
{% endhighlight %}

### Extract the ISO
We then need to extract our newly downloaded Ubuntu image.

{% highlight bash %}
xorriso -osirrox on -indev ubuntu-16.04.2-server-amd64.iso -extract / custom-iso
{% endhighlight %}

The above command will extract the ISO you have just downloaded and place the files into a new directory called `custom-iso`.

### Edit /boot/grub/grub.cfg
This is for UEFI booting. This specifies the timeout on the grub menu to 1 second, tells grub what network adapter to use and grabs the preseed file.

{% highlight bash %}
cd /root/custom-iso
nano boot/grub/grub.cfg
{% endhighlight %}

Replace the contents with grub.cfg with the following. **Make sure you alter the HTTP location of your preseed file!** My preseed file is currently stored in http://preseed.handsoff.local/ubuntu-16-04/preseed.cfg

{% highlight bash %}
if loadfont /boot/grub/font.pf2 ; then
set gfxmode=auto
insmod efi_gop
insmod efi_uga
insmod gfxterm
terminal_output gfxterm
fi
set default=0
set timeout=1
set menu_color_normal=white/black
set menu_color_highlight=black/light-gray

menuentry "Install Ubuntu Server" {
 set gfxpayload=keep
 linux /install/vmlinuz gfxpayload=800x600x16,800x600 hostname=ubuntu16-04 --- auto=true url=http://preseed.handsoff.local/ubuntu-16-04/preseed.cfg quiet
 initrd /install/initrd.gz
}
{% endhighlight %}

### Edit /isolinux/txt.cfg
This will allow you to install a legacy or generation 1 virtual machine. If you do not want to enable preseeding on a legacy device, this step isn’t necessary.

{% highlight bash %}
nano isolinux/txt.cfg
{% endhighlight %}

Replace the contents with the following. **Make sure you alter the HTTP location of your preseed file!**

{% highlight bash %}
GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=1

default install
label install
menu label ^Install Ubuntu Server
kernel /install/vmlinuz
append preseed/url=http://preseed.handsoff.local/ubuntu-16-04/preseed.cfg bga=788 netcfg/choose_interface=auto initrd=/install/initrd.gz priority=critical --
{% endhighlight %}

### Obtain isohdpfx.bin
This file will allow you to install a legacy or generation 1 virtual machine. If you do not wish to have this functionality then skip this step.

{% highlight bash %}
cd ../
sudo dd if=ubuntu-16.04.2-server-amd64.iso bs=512 count=1 of=custom-iso/isolinux/isohdpfx.bin
{% endhighlight %}

### Create new ISO
We need to create the new ISO. This command will specify the different parameters of the ISO.

{% highlight bash %}
cd custom-iso
sudo xorriso -as mkisofs -isohybrid-mbr isolinux/isohdpfx.bin -c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -isohybrid-gpt-basdat -o ../custom-ubuntu-http.iso .
{% endhighlight %}

You will now have a custom built Ubuntu 16.04 install ISO in `/root/custom-ubuntu-http.iso`.

### Confirming partitions
As we have created a hybrid ISO, meaning it can be used in both UEFI and Legacy modes, it’s good to make sure EFI is specified in the partition table. You can check this by using the following command:

{% highlight bash %}
fdisk -l custom-ubuntu-http.iso
{% endhighlight %}

If the partitions have been created correctly, you should see something similar to the following:

{% highlight bash %}
Disk custom-ubuntu-http.iso: 841 MiB, 881852416 bytes, 1722368 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x19a15b31

Device Boot Start End Sectors Size Id Type
custom-ubuntu-http.iso1 * 0 1722367 1722368 841M 0 Empty
custom-ubuntu-http.iso2 4036 8899 4864 2.4M ef EFI (FAT-12/16/32)
{% endhighlight %}

### The Preseed file
You will need to upload your preseed .cfg file to location specified in grub.cfg and txt.cfg. As we have specified networking within the Grub menu, we are now able to pull the preseed file over network using DHCP.

### User passwords
You can either encrypt your password, or pass it over plain text. If you want to use plain text, then comment the line `d-i passwd/user-password-crypted` and uncomment the two lines above containing `passwd/user-password` and `passwd/user-password-again`. **Don’t forget to update your password**. If you would like to hash the password, enter the following line. This will prompt for a password and return a hash for you to use. You must then add your hash to the preseed.cfg file.

{% highlight bash %}
mkpasswd -m sha-512
{% endhighlight %}

### The post install script
The preseed uses wget to download your post-install.sh file to the /tmp folder just before the machine reboots after installation. My script does the following:

* apt-get update && upgrade -y
* apt-get dist-upgrade -y
* Installs basic packages defined at the head of the script.
* Installs SSH keys for both the default user, and root. You must edit the post-install.sh file to add your keys.
* Edits /etc/sshd_config to allow root ssh access, as well as password authentication.
* Configures networking specified at the top of the script.
* Changes the hostname which is defined at the top of the script.

Unfortunately, there seems to be a bug in Xenial which ignores the hostname set in the preseed file. Therefore, a default hostname is set within grub to remove the prompt during installation. If you know a better way around this, please commit a PR on the Github repository.
