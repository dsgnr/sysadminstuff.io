---
layout: post
title: Install phpMyAdmin with Nginx
categories: Linux
permalink: /:categories/:title
post_description: How to install phpMyAdmin on an Nginx web server running PHP7 without the need for Apache!
cover: "/assets/images/post-thumbnails/phpmyadmin-with-nginx.svg"
author: dan_hand
---


Install phpMyAdmin with Nginx
I tend to work mainly from the command line now. I never used to. I used to be scared of the command line but after forcing myself to use it, by removing any GUI tool I had, I have now overcome this fear and have embraced the ease and unlimited uses the command line brings.

I do however, still have a need for a few GUI tools. PhpMyAdmin is one of those tools I do tend to use from time to time. Sometimes, I don’t have access to the command line, whether I don’t have VPN access to my home network or don’t have my SSH keys on the device I am on.

My web servers now use Nginx along with a concoction of other services like Varnish and Elasticsearch. I also don’t like to have anything installed that I don’t need.

This being the case, I wanted to use phpMyAdmin without Nginx. I did not want to install Apache as I have a strong dislike for it and it’s unnecessary in my case.

The following guide explains how to install phpMyAdmin on a server running Nginx and PHP7.0-FPM on a Debian Jessie box.

**Prerequisites:**
* Nginx
* MySQL or MariaDB
* PHP7

First, make sure your server is up-to-date.
{% highlight bash %}
$ sudo apt-get update && apt-get ugprade -y
{% endhighlight %}
Next, install phpMyAdmin:
{% highlight bash %}
$ sudo apt-get install phpmyadmin
{% endhighlight %}

You’ll see the following screen. Press tab and then enter. This way you’re not installing any configuration files.

<figure><img src="/assets/images/install-phpmyadmin-without-apache-installer-screen.jpg" alt="Install PHPMyAdmin without Apache"></figure>

Installing phpMyAdmin will automatically install Apache, unfortunately. This is fine, but it will cause an issue in this case as both Apache and Nginx will be trying to listen to port 80 by default. So something is going to break!

So, let’s remove Apache:
{% highlight bash %}
$ sudo apt-get remove apache2 --purge
{% endhighlight %}

Lets create a symbolic link to our www-data folder so that we can access phpMyAdmin from our browser:

{% highlight bash %}
$ ln -s /usr/share/phpmyadmin /var/www/phpmyadmin
{% endhighlight %}

Next, we need to make a new Nginx host file to bring everything to life. Just make sure to change the server_name to the url you’d like your installation to be accessible from:
{% highlight bash %}
server {
	listen 80;
	server_name phpmyadmin.domain.com;
	root /var/www/phpmyadmin/;
	index index.php;

	access_log /var/log/nginx/phpmyadmin.access.log;
	error_log /var/log/nginx/phpmyadmin.error.log;

	location ~ \.php$ {
		include fastcgi_params;
		fastcgi_pass unix:/run/php/php7.0-fpm.sock;
		fastcgi_split_path_info ^(.+\.php)(.*)$;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
}
{% endhighlight %}
You should now be able to access phpMyAdmin by going to the domain you specified in the server_name field.
