---
layout: post
title: Comparing Linode and Digital Ocean $5 VPS hosting
categories: Linux
permalink: /:categories/:title
post_description: Linode recently announced the release of their own $5 instance. A Linode vs Digital Ocean show down.
cover: "/assets/images/post-thumbnails/linode-vs-digital-ocean-vps.svg"
author: dan_hand
---

Linode recently announced the release of their own $5 instance. This new $5 instance boasts twice the amount memory than the same priced instance from Digital Ocean. I have never personally used Linode for any of my projects as I’ve always used Digital Ocean. I wanted to do a comparison of the $5 droplet from Digital Ocean against the new $5 instance from Linode.

I wanted to do a number of tests on both droplets. I wanted to see exactly how fast the droplets were on both Apache and Nginx web servers. I also wanted to see how well they scale. In this post, I do a number of comparisons, both real world examples, as well as Sysbench tests. Why both types of tests? I usually take Sysbench tests with a pinch of salt. A huge amount of factors can alter the outcome of these, so I wanted to do a load test using [loader.io](loader.io), too.

Here are some specifications of the two virtual machines to compare:

|      Linode       |   Digital Ocean   |
| :---------------: | :---------------: |
|      1GB RAM      |     512MB RAM     |
|    1 CPU Core     |    1 CPU Core     |
|     20GB SSD      |     20GB SSD      |
|   1TB bandwidth   |   1TB bandwidth   |
| 40Gbps Network In | 40Gbps Network In |
| 1Gbps Network Out | 1Gbps Network Out |

From the table above, the two companies’ pretty much offer the same specifications on their virtual machines. Linode offering twice the memory for the same cost. Lets take a deeper look of a few important details.

### CPU Comparison
{% highlight bash %}
Digital Ocean - Intel(R) Xeon(R) CPU E5-2650L v3 @ 1.80GHz
{% endhighlight %}
{% highlight bash %}
Linode - Intel(R) Xeon(R) CPU E5-2680 v3 @ 2.50GHz
{% endhighlight %}

Here we can see that the Linode CPU’s are slightly more powerful. This should have a noticeable impact on how well the virtual machine copes on load. It definitely felt a little quicker when setting up the virtual machine.

## CPU Sysbench
After seeing that the Linode machine had a faster CPU, I wanted to do a basic Sysbench prime number test on the processor.

### Digital Ocean
{% highlight bash %}
sysbench --test=cpu --cpu-max-prime=20000 run

Test execution summary:
 total time: 35.7738s
 total number of events: 10000
 total time taken by event execution: 35.7687
 per-request statistics:
 min: 3.51ms
 avg: 3.58ms
 max: 7.39ms
 approx. 95 percentile: 3.68ms

Threads fairness:
 events (avg/stddev): 10000.0000/0.00
 execution time (avg/stddev): 35.7687/0.00
 {% endhighlight %}

### Linode
{% highlight bash %}
sysbench --test=cpu --cpu-max-prime=20000 run

Test execution summary:
 total time: 34.8751s
 total number of events: 10000
 total time taken by event execution: 34.8653
 per-request statistics:
 min: 3.20ms
 avg: 3.49ms
 max: 43.05ms
 approx. 95 percentile: 3.88ms

Threads fairness:
 events (avg/stddev): 10000.0000/0.00
 execution time (avg/stddev): 34.8653/0.00
 {% endhighlight %}

The Linode virtual machine was less than a second faster to complete this task. I thought the Linode virtual machine would have been noticeably quicker over the Digital Ocean droplet on this test. Nevertheless, Linode was better at this task.

## SSD Disk Speed
I wanted to test how fast the SSD’s were on both virtual machines. This can have a noticeable impact in day to day usage so this was quite important for me.

### Digital Ocean
Disk write speed
{% highlight bash %}
sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
  1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 2.52029 s, 426 MB/s
 {% endhighlight %}

### Disk read speed
{% highlight bash %}
dd if=tempfile of=/dev/null bs=1M count=1024
  1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 1.12766 s, 952 MB/s
 {% endhighlight %}

### Linode
### Disk write speed
{% highlight bash %}
sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
  1024+0 records in
  1024+0 records out
  1073741824 bytes (1.1 GB) copied, 1.9069 s, 563 MB/s
   {% endhighlight %}

### Disk read speed
{% highlight bash %}
dd if=tempfile of=/dev/null bs=1M count=1024
  1024+0 records in
  1024+0 records out
  1073741824 bytes (1.1 GB) copied, 1.06713 s, 1.0 GB/s
   {% endhighlight %}

On the disk SSD speed test, Linode came out on top. This time by a decent amount.

## File I/O
### Digital Ocean
{% highlight bash %}
sysbench --test=fileio --file-total-size=10G prepare
sysbench --test=fileio --file-total-size=10G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run

Operations performed: 387600 Read, 258400 Write, 826795 Other = 1472795 Total
Read 5.9143Gb Written 3.9429Gb Total transferred 9.8572Gb (33.646Mb/sec)
 2153.33 Requests/sec executed

Test execution summary: total time: 300.0047s total number of events: 922500 total time taken by event execution: 180.7100 per-request statistics: min: 0.00ms avg: 0.20ms max: 23.52ms approx. 95 percentile: 0.58ms
Threads fairness: events (avg/stddev): 922500.0000/0.00 execution time (avg/stddev): 180.7100/0.00
   {% endhighlight %}

### Linode
{% highlight bash %}
sysbench --test=fileio --file-total-size=10G prepare
sysbench --test=fileio --file-total-size=10G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run

Operations performed: 387600 Read, 258400 Write, 826795 Other = 1472795 Total
Read 5.9143Gb Written 3.9429Gb Total transferred 9.8572Gb (33.646Mb/sec)
 2153.33 Requests/sec executed

Test execution summary:
 total time: 300.0004s
 total number of events: 646000
 total time taken by event execution: 80.1445
 per-request statistics:
 min: 0.00ms
 avg: 0.12ms
 max: 36.46ms
 approx. 95 percentile: 0.25ms

Threads fairness:
 events (avg/stddev): 646000.0000/0.00
 execution time (avg/stddev): 80.1445/0.00
{% endhighlight %}

## MySQL
Testing MySQL performance can be tricky. In this instance, we’ll create a new database called test and add 1000000 rows of data. We’ll then perform a read test in the database.

### Digital Ocean
After we’ve created the test database, we’ll create 1000000 rows of data.
{% highlight bash %}
sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root --mysql-password=yourrootsqlpassword prepare
{% endhighlight %}

We’ll perform a read test on our database
{% highlight bash %}
sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root --mysql-password=yourrootsqlpassword --max-time=60 --oltp-read-only=on --max-requests=0 --num-threads=8 run

OLTP test statistics: queries performed: read: 364728 write: 0 other: 52104 total: 416832 transactions: 26052 (434.14 per sec.) deadlocks: 0 (0.00 per sec.) read/write requests: 364728 (6077.99 per sec.) other operations: 52104 (868.28 per sec.)
Test execution summary: total time: 60.0080s total number of events: 26052 total time taken by event execution: 479.8152 per-request statistics: min: 1.24ms avg: 18.42ms max: 73.61ms approx. 95 percentile: 25.06ms
Threads fairness: events (avg/stddev): 3256.5000/6.91 execution time (avg/stddev): 59.9769/0.01
{% endhighlight %}

### Linode
The same again with Linode
{% highlight bash %}
sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root --mysql-password=yourrootsqlpassword prepare
{% endhighlight %}

{% highlight bash %}
sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root --mysql-password=yourrootsqlpassword --max-time=60 --oltp-read-only=on --max-requests=0 --num-threads=8 run

OLTP test statistics:
 queries performed:
 read: 282828
 write: 0
 other: 40404
 total: 323232
 transactions: 20202 (336.64 per sec.)
 deadlocks: 0 (0.00 per sec.)
 read/write requests: 282828 (4713.00 per sec.)
 other operations: 40404 (673.29 per sec.)

Test execution summary:
 total time: 60.0101s
 total number of events: 20202
 total time taken by event execution: 479.7918
 per-request statistics:
 min: 1.19ms
 avg: 23.75ms
 max: 327.54ms
 approx. 95 percentile: 36.06ms

Threads fairness:
 events (avg/stddev): 2525.2500/8.26
 execution time (avg/stddev): 59.9740/0.01
 {% endhighlight %}

This one did surprise me. Seeing as the CPU and the SSD speed on Linode is faster, I would have expected Linode to come out on top with this one. Instead, Digital Ocean was able to perform quite a lot more read queries than the Linode virtual machine.

### Bench test conclusions
On Sysbench testing alone, the Linode virtual machine didn’t stand out at all. I was expecting more as in theory, it is of higher specification than the Digital Ocean droplet.

Doing bench tests and speed comparisons of hardware is all well and good, but we know that it doesn’t necessarily point toward a faster website. I decided to do some testing between a two different web server configurations. I wanted to test both the overall website speed according to various website speed testing services, as well as how many concurrent users the server can handle. To test the website speed, I will be testing [tools.pingdom.com](tools.pingdom.com)
 and webpagespeedtest, testing from the closest server possible to the virtual server. In order to test for concurrent users, I will be using loader.io. Here I’ll be testing for 1 minute at the highest sustained users possible without receiving errors.

Here are the web server configurations I’ll be testing:
* Apache, PHP7 and MariaDB
* Nginx, PHP7 and MariaDB

I must note that I will be using the latest version of WordPress (4.7.2 at the time of writing this), the default Twenty Seventeen theme and no plugins, not even Hello Dolly or Akismet. Both web server setups will be using default configurations, no tweaks, no server side caching like fast-cgi or Varnish. Just very basic LEMP and LAMP stacks.

## Pingdom Tools
Each test was performed three times. All virtual machines are running the same version of WordPress, no plugins installed, just basic installations.

### Linode Apache, PHP7.0, MariaDB
<figure><img src="/assets/images/linode-apache-pingdom-tools.jpg"></figure>

### Digital Ocean, Apache, PHP7.0, MariaDB
Image to come

### Linode Nginx, PHP7.0, MariaDB
<figure><img src="/assets/images/linode-nginx-pingdom-tools.jpg"></figure>

### Digital Ocean Nginx, PHP7.0, MariaDB
Image to come

## Under Load
I wanted to test each virtual machine under load using the different web servers. I am performing a 1 minute test on each setup. The test will have a sustained user count. I have tested to the highest amount of users before the server crashed.

### Digital Ocean Apache
As you can see from the animation below, the Apache server on the Digital Ocean droplet was able to handle 60 concurrent users with an average load time of 1733ms.
<video class="full-img" autoplay="autoplay" loop="loop" muted="" playsinline="">
  <source src="/assets/images/speakwp-digital-ocean-apache-php7-loader-test.mp4" type="video/mp4">
</video>

### Linode Apache
The Linode virtual machine was able to cope with twice the amount of concurrent visitors, 120, without falling over. This was with an average load time of 1946ms.
<video class="full-img" autoplay="autoplay" loop="loop" muted="" playsinline="">
  <source src="/assets/images/speakwp-linode-apache-php7-loader-test.mp4" type="video/mp4">
</video>
No surprises here really. I’d be worried if a virtual machine with twice the amount of memory lost this one. Twice the amount of memory on the Linode machine so twice the amount of visitors. Yes, the average time is slower, but as there are twice the amount of concurrent visitors, I think this is acceptable.

### Digital Ocean Nginx
Testing the Digital Ocean droplet with Nginx, I saw a 0.1% error rate where the backend failed to load. I was not able to see which error this was, but I suspect this is due to memory exhuastion on the small 512MB droplet causing the database to crash. I did have to manually restart the database service after this test. I was able to push the droplet to 135 clients with an average load time of 2273ms.
<video class="full-img" autoplay="autoplay" loop="loop" muted="" playsinline="">
  <source src="/assets/images/speakwp-digital-ocean-nginx-php7-loader-test.mp4" type="video/mp4">
</video>
### Linode Nginx
Again, I was surprised at the Linode virtual machine. The machine experienced a 0.3% error rate with 140 concurrent visitors. Other than that, the virtual machine was able to cope with 5 more concurrent visitors with a slightly quicker average load time of 2234ms.
<video class="full-img" autoplay="autoplay" loop="loop" muted="" playsinline="">
  <source src="/assets/images/speakwp-linode-nginx-php7-loader-test.mp4" type="video/mp4">
</video>

### Overall Conclusion
Well, for me, the results of these tests are a surprise. I was expecting the Linode virtual machine to do much better than the Digital Ocean droplet. I was expecting the Linode virtual machine to wipe the floor with the Digital Ocean droplet with its more powerful CPU and larger memory allocation. The Linode machine was only marginally better than the Digital Ocean droplet when comparing Nginx webservers. I think if you are just starting out with your virtual machine experience, Linode is probably the way you want to go. They’re both the same price so why not go for the more powerful machine? But really, for me, I have no real reason to migrate sites from Digital Ocean to Linode as there is no real improvement in performance for me, despite boasting the larger memory allocation.

Have I missed anything out in these tests? Or anything you want me to elaborate more on? Let me know by commenting or emailing [email@sysadminstuff.io](mailto:email@sysadminstuff.io)
