---
title:  "VMware-Fusion Docker-machine"
date:   2015-08-23 12:20:00
description: Setup a VMware-Fusion docker-machine lab environment.
keywords: erik aulin,blog,osx,docker,vmware,ustwo,development,server,lab,fusion
---

### Introduction
In this tutorial I will cover how I a docker-machine lab environment using VMWare Fusion.
***

### Prerequisites
* [Mac Computer](http://www.apple.com/mac/).
* [VMWare Fusion](http://www.vmware.com/products/fusion)
* [Docker Toolbox](https://www.docker.com/toolbox).

***

### Create our lab environment
If you followed all the prerequisites you should now have a standard vmware and docker toolbox environment setup.
Lets start with creating the docker-machine in VMWare Fusion.

##### Docker Machine
{% highlight bash %}
docker-machine create -d vmwarefusion docker
eval "$(docker-machine env docker)"
{% endhighlight %}

##### Network configuration
In my lab environment I chose to use the NAT (vmware8) network.
Since I have several other virtual machines that I want to keep in the same environment.

I want my VM's to have static IP's then I can create records in /etc/hosts on my machine and osx vm's.

Edit the dhcpd file with your favorit edit and add your hosts after the line *####### VMNET DHCP Configuration. End of "DO NOT MODIFY SECTION" ######.*

{% highlight bash %}
sudo vim /Library/Preferences/VMware\ Fusion/vmnet8/dhcpd.conf
{% endhighlight %}

*Want to use a CLI to check hosts MAC Address but during this lab lets use VMware Fusion App and check each hosts Network > Settings > Advanced Options and write down the MAC Address.*

Added three host in my case, docker(munki) vm, osx client vm and a osx server vm.

{% highlight bash %}
host docker {
    hardware ethernet EE:EE:EE:EE:EE:EE;
    fixed-address 192.168.125.140;
}
{% endhighlight %}

*Edit the hosts file with your favorit edit and add your hosts after the last line*

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

{% highlight bash %}
192.168.125.140 docker.example.com
{% endhighlight %}

Restart the vmnet, docker machine and your OSX VM's.

{% highlight bash %}
export PATH="$PATH:/Applications/VMware Fusion.app/Contents/Library/"
sudo vmnet-cli --configure
sudo vmnet-cli --stop
sudo vmnet-cli --start
docker-machine stop munki
docker-machine start munki
{% endhighlight %}

***

Thats it, you now have a docker-machine running in vmware and you are ready to start your journey into the fantastic world of docker.
