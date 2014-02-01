---
layout: post
title:  "Shared folder permissions between an OS X Host and an Ubuntu VM (501 dialout?!)"
date:   2014-02-01 00:13:59
categories: sysadmin
tags: ubuntu osx
permalink: vmware-shared-folders-permissions
---

We use VMWare Fusion to streamline development environments for projects and clients. In general each product, app or client has their own self contained virtual machine. On that VM is a the full stack, typically running on an Ubuntu linux OS. This allows us to do a number of things quickly and efficiently - sharing environments, onboarding, archiving, backups, etc. 

We use the VMWare Shared Directories function to share our host sourcecode directory to the guest OS. This allows us to work on the host using whatever editors, IDEs, tools we choose and the guest OS picks up changes instantly as if edits were done locally on the guest OS.

VMWare Fusion for Mac (we're using 6.0.2) has an issue with how it handles permissions for these shared directories, and apparently it has been around for a while. The issue is that the guest OS (Ubuntu) sees directory onwership as the uid and gid of your OSX account. The default uid for the first account on your Mac OS X is 501 and so you will most likely see the files and directories similar to below:

{% highlight sh %}
paulcowles@ubuntu:/$ ls -al /mnt/hgfs
drwxr-xr-x 1 501 dialout 1836 Jan 22 12:42 sourcecode
{% endhighlight %}

In most cases, 501 won't be a valid uid on the guest OS (it wasn't for us). This doesn't usually cause problems, but sometimes you need ownership by a valid user. For example, if you need Apache running as www-data to be able to write to the shared directory.

To get the host and guest working together better, we need to make sure there is an account on your linux VM with same uid and that this user is part of the www-data group that your Apache typically operates as.

For this scenario, our Mac OS X user is "paulcowles" and we login to the Ubuntu guest as "paulcowles" which is a user account with sudo access. We also have another account setup on the Ubuntu guest "admin" that has sudo access.

On our Ubuntu VM confirm the uid

{% highlight sh %}
paulcowles@ubuntu:/$ id
uid=1000(paulcowles) gid=1000(paulcowles) groups=1000(paulcowles)...
{% endhighlight %}

On our Mac confirm the uid (verify it is 501)

{% highlight sh %}
Pauls-MacBook-Pro:~ paulcowles$ id
uid=501(paulcowles) gid=20(staff) groups=20(staff)...
{% endhighlight %}

Login as the "secondary" admin account on your VM. Assuming our usual account for development on VM matches our Mac and is "paulcowles" then let's logout of paulcowles and login as admin

{% highlight sh %}
Pauls-MacBook-Pro:~ paulcowles$ ssh admin@192.168.2.85
{% endhighlight %}

Make sure our primary development account on linux is a member of the www-data group Apache is using

{% highlight sh %}
admin@ubuntu:/$ sudo adduser paulcowles www-data
{% endhighlight %}

Change the uid of the paulcowles account on our Ubuntu VM. It is a good idea to take a VM snapshot before mucking with uid changes just in case you need to rollback.

{% highlight sh %}
admin@ubuntu:/$ sudo usermod -u 501 paulcowles
admin@ubuntu:/$ sudo find / -user 1000 -exec chown -h 501 {} \;
{% endhighlight %}

*Note* 501 is the uid we want the account to have (to match our OS X host) and 1000 was the old uid

Reboot and check again

{% highlight sh %}
admin@ubuntu:/$ sudo reboot
{% endhighlight %}

Then

{% highlight sh %}
Pauls-MacBook-Pro:~ paulcowles$ ssh paulcowles@192.168.2.85
paulcowles@ubuntu:/$ ls -al /mnt/hgfs
total 11
dr-xr-xr-x 1 root       root    4192 Jan 31 14:47 .
drwxr-xr-x 4 root       root    4096 Jun 21  2013 ..
drwxr-xr-x 1 paulcowles dialout 1836 Jan 22 12:42 sourcecode
{% endhighlight %}

Notice 501 has changed to paulcowles. Now Apache and your local account should jive nicely and Apache should be able to write to your shared directory. paulcowles on OS X and paulcowles on Ubuntu both have the uid 501. paulcowles on Ubuntu is part of the www-data group, same as Apache, so everyone can read and write to the directory without error. 



