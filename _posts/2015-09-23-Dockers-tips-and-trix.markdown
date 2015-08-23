---
title:  "Dockers"
date:   2015-08-23 10:27:00
description: Dockers in you day to day work.
---

### Stuff I learnd and use with dockers.
I'm using a Apple Mac with VMware Fusion and the latest [Docker Toolbox](https://www.docker.com/toolbox).
Docker default configuration is to use Virtualbox so I added some tips how to use VMware Fusion instead, also added some more cpu and memory if I need some more beef for full system images.

#### Create a docker environment using vmware fusion.

{% highlight bash %}
docker-machine create -d vmwarefusion \
  --vmwarefusion-cpu-count 2 \
  --vmwarefusion-memory-size 4096 \
docker
eval "$(docker-machine env docker)"
{% endhighlight %}

#### Set static dhcp rules for dockers environment
I want my VM's to have the same IP so the docker-machine environment is always the same.
If you want to use the NAT network you can edit the dhcpd file with your favorit edit and add your host after the line ####### VMNET DHCP Configuration. End of "DO NOT MODIFY SECTION" ######.

{% highlight bash %}
sudo vim /Library/Preferences/VMware\ Fusion/vmnet8/dhcpd.conf
{% endhighlight %}

Added three host in my case, dockers vm, osx client vm and a osx server vm.

{% highlight bash %}
host docker {
    hardware ethernet EE:EE:EE:EE:EE:EE;
    fixed-address 192.168.125.140;
}
host client {
    hardware ethernet EE:EE:EE:EE:EE:EE;
    fixed-address 192.168.125.141;
}
host server {
    hardware ethernet EE:EE:EE:EE:EE:EE;
    fixed-address 192.168.125.142;
}
{% endhighlight %}

I added some records to my hostfile for easy access.

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

{% highlight bash %}
192.168.125.140 docker.example.com
192.168.125.141	client.example.com
192.168.125.142 server.example.com
{% endhighlight %}

##### Restarting the vmware fusion network environment

{% highlight bash %}
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --configure
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start
{% endhighlight %}
