---
layout: post
title: Provisioning multiple Linux servers with Ansible
categories: Linux
permalink: /:categories/:title
post_description: Automate all the things. This is where Ansible comes into play. Deploy to multiple hosts at once.
cover: "/assets/images/post-thumbnails/deploying-servers-with-ansible.svg"
author: dan_hand
---

Provisioning multiple Linux servers with Ansible
I like to automate all the things. I enjoy actually designing and creating scripts and I love how much time they can save once they’re set up and working as they should. Yes, you have the initial timely matter of creating the script and debugging it, but if it saves you around an hour each time you run it, then it’s an hour longer you can be doing something else!

I am a huge fan of scripting with bash. I find it very easy to make a relatively complicated script and I like that it’s super fast. One thing that I did find convoluted is running a bash script from an external source. By convoluted, I mean, I never quite got it working how I wanted. I didn’t like having to ssh into a box and run a script. I wanted to have a central point where my scripts are stored and I wanted an easy way to painlessly run a script on one or more servers with as little effort as possible. This is where I discovered Ansible.

Initially, I was quite slow to adopt this until a friend recommended I have a go. All I can say is, wow. Within about 30 minutes of an initial learning curve, I attempted to mimic an existing bash script by creating my first Ansible Playbook. An hour later, I was able to provision Nginx, PHP7 and MariaDB on a remote server.

Here’s what the file structure looks like:

{% highlight bash %}
.
├── group_vars
│   └── all
├── roles
│   ├── common
│   │   └── tasks
│   │   └── main.yml
│   ├── mysql
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │   └── my.cnf
│   ├── nginx
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │   ├── default.conf
│   │   └── nginx.conf
│   └── php-fpm
│   └── tasks
│   └── main.yml
└── site.yml
{% endhighlight %}

It’s quite simple once you get your head around the file structure. Once your playbook is ready, you can specify the hosts you want to deploy to and then deploy using `ansible-playbook site.yml`. Providing you have SSH keys configured to access the remove server without passwords, you’re pretty much set.

With the above playbook, I can deploy Nginx, PHP7 and MariaDB in around a minute with all my custom configurations and tweaks. If you want to deploy to more than one host then this is where Ansible really comes into it’s own. I have tested the above playbook by deploying to 10 remote hosts. I was able to deploy a working LEMP stack in under 3 minutes to 10 hosts. Imagine how much time that would take you to do this manually, on every host, one at a time!

I will write another post soon which will go into more detail about Ansible playbooks and how to create one.
