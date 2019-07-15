---
layout: post
title: Secure multiple sites on Nginx using separate PHP pools
categories: Linux
permalink: /:categories/:title
post_description: Separate your websites on the same server and protect them against XSS
cover: "/assets/images/post-thumbnails/multiple-php-pools.svg"
cover_color: "#6682ba"
author: dan_hand
---

Secure multiple sites on Nginx using separate PHP pools
I have extreme OCD about security. I have only been hacked once, when I first started out. As a noob, I had no backup of the database. In those days, I was a cowboy coder. I basically had to rebuild the site from scratch. This has only happened once and it was painful. Unfortunately, as I had a few websites on the same account in a shared environment, they were all affected. I learnt from this and decided to learn how to lock down everything as much as I could. Here’s my post talking about securing your WordPress website.

I recently moved my production server to Digital Ocean (get $10 credit when you sign up by clicking here) and after being on a server which used WHM/cPanel, I wanted to replicate the way each cPanel account was separated from each other. The idea was, if one site were compromised, my other sites should be fine.

I thought that this would be something that was widely spoken about online, but information about doing this on Nginx was quite scarce. I thought this would have been a done thing to separate accounts on a single box.

What needs separating?
I am using a Digital Ocean box running Ubuntu 16.04. I have also installed Nginx, PHP7-FPM and MariaDB. I have decided on this particular box to host multiple WordPress websites.

Create a new user and group on a per site basis
Give the user it’s own home folder and lock the user to it’s home
Create individual PHP pools on a per site basis
Create a new Nginx vhost and point it to the new home folder
Essentially, this is pretty simple, but if you are setting up a few sites then doing this can be quite time consuming. Fortunately, I came across a bash script that was created a while back for PHP 5.6. I have simply updated this in order to work with PHP7. The bash script basically asks you a few questions and does the rest for you. Here is the bash script: https://github.com/dsgnr/create-php-host.
The script creates a very basic WordPress ready Nginx host file so if you need something more complex for your project then simply edit the nginx.vhost.conf.template file but just make sure you keep the variables as this talks to the script itself.
To run the script, all you need to do is import these files to your server, let’s say it’s in the /root folder and run:

bash /root/create_php_site.sh domain.com
Don’t forget to replace domain.com with your respective domain. Do NOT include www. at this stage.

What does it do?
The bash script above will ask you a few questions. If you answer those questions correctly, everything should be set up correctly. Obviously, if there are few things that you specifically need to add to your Nginx vhost then you can do this afterwards. If you need to change the location of where the script puts your files then you must change this before running the script by editing the lines 91-93 in the create_php_site.sh file. This script also assumes a default set up.

The script asks the following:

1. Please specify the username for this site?
This can be anything but typically, I’ll call this something related to the site so it’s easily remembered. However, as with anything, try to make it unique!

2. Would you like to change to web root directory (y/n)?
I said above that you can edit the location of where the public_html files are positioned. However, if you want to just change the location for one site then you can specify this here, otherwise say no to use the default location specified in the script.

3. How many FPM servers would you like by default:

4. Min number of FPM servers would you like:

5. Max number of FPM servers would you like:

Conclusion

