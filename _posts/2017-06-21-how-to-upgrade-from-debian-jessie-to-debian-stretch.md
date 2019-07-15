---
layout: post
title: How To Upgrade from Debian Jessie to Debian Stretch
categories: Linux
permalink: /:categories/:title
post_description: Find out how to easily upgrade Debian 8 ‘Jessie’ to Debian 9 ‘Stretch’ with these quick and simple steps.
cover: "/assets/images/post-thumbnails/speak-wp-update-debian-8-jessie-debian-9-stretch.svg"
author: dan_hand
---

How To Upgrade from Debian Jessie to Debian Stretch
I like to keep up to date with the latest software. I am usually an early adopter of operating systems, particularly beta releases. Now that Debian 9 ‘Stretch’ has officially been released, you may be interested in upgrading your Debian operating system to the latest version. In this guide, I will explain the few steps needed to upgrade from Debian 8 ‘Jessie’ to Debian 9 ‘Stretch’.

The first step is ensuring that everything is fully up-to-date with your current operating system. This could help make the process run a little smoother, especially if you are upgrading from an out-of-date package.

Run the following commands:

{% highlight bash %}
apt-get update
apt-get upgrade
apt-get dist-upgrade
{% endhighlight %}

You may also want to see if any packages have errors, are obsolete or missing. To do this, you can run the following:

{% highlight bash %}
dpkg -C
{% endhighlight %}

If the previous command comes back clean, then check to make sure there are no held packages:

{% highlight bash %}
apt-mark showhold
{% endhighlight %}

Once you are happy with upgrading your current packages, you will upgrade your package sources to look for Stretch repositories instead of Jessie. To do this, simply edit the `/etc/apt/sources.list` file and replace **Jessie** with **Stretch**.

Original
{% highlight bash %}
deb http://deb.debian.org/debian/ jessie main contrib non-free
deb-src http://deb.debian.org/debian/ jessie main contrib non-free
{% endhighlight %}

Edited

{% highlight bash %}
deb http://deb.debian.org/debian/ stretch main contrib non-free
deb-src http://deb.debian.org/debian/ stretch main contrib non-free
{% endhighlight %}

You may want to look through the other files in your /etc/apt folder (if you have any), and update those sources too.

Now you have updated the source lists, you can now perform the upgrade to ‘Stretch’.

We’ll do the following. Update the databases to the latest sources and upgrade installed applications. Then we’ll perform a distribution upgrade to get the latest kernel modules and other useful upgrades.

{% highlight bash %}
apt-get update
apt-get upgrade
apt-get dist-upgrade
{% endhighlight %}

You may be asked various questions throughout. Most likely, you will be asked what to do with configuration files. The default is to keep existing configurations. However, it is recommended to back up your existing configurations and install the new configuration. This will install the new configuration file for the specific version of the package you are installing, thus reducing risks of completely breaking the package.

Once the dist-upgrade has completed, you must reboot in order to use the latest kernel.

You may wish to clean out redundant caches and kernels after this upgrade by running the following:

{% highlight bash %}
apt-get clean
apt-get autoremove
{% endhighlight %}

You can double check which version of Linux you are using by typing `lsb_release -a`.

Well done. You’ve now upgraded from Debian 8 ‘Jessie’ to Debian 9 ‘Stretch’.

