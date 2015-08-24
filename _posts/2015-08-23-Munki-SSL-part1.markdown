---
title:  "Munki with SSL Part1"
date:   2015-08-23 12:20:00
description: Setup a Munki environment with docker .
---

### Introduction
In this tutorial I will cover how to setup an [Munki](https://www.munki.org/munki) environment with client SSL authentication. Hosting the munki data and web server in docker and using a osx machine for populating data using [Autopkgr](https://github.com/lindegroup/autopkgr). Most guides out there use [Chef](https://www.chef.io) or [Puppet](https://puppetlabs.com) to push client configuration but I wanted to focus on docker and have the possibility to move between solutions depending on situation and current infrastructure.

***

### Prerequisites
* [Mac Computer](http://www.apple.com/mac/).
* [VMWare Fusion](http://www.vmware.com/products/fusion) or [Virtualbox](https://www.virtualbox.org).
* [Docker Toolbox](https://www.docker.com/toolbox).
* [Virtual Docker](https://docs.docker.com/machine).
* [Virtual OSX](http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2082109#) Client and Server.

***

### Create our lab environment
VMware Fusion will be used in this tutorial so there will different steps for Virtualbox. It's high time to get started so lets create a Docker Machine where we will be hosting three containers: Data-only, Nginx with SSL certs and finally a SMB share used with the osx server.

##### Docker Machine
{% highlight bash %}
docker-machine create -d vmwarefusion munki
eval "$(docker-machine env munki)"
{% endhighlight %}

##### Network configuration
In my lab environment I chose to use the NAT (vmware8) network.


I want my VM's to have static IP's then I can create records in /etc/hosts on my machine and osx vm's.

*Edit the dhcpd file with your favorit edit and add your hosts after the line ####### VMNET DHCP Configuration. End of "DO NOT MODIFY SECTION" ######.*

{% highlight bash %}
sudo vim /Library/Preferences/VMware\ Fusion/vmnet8/dhcpd.conf
{% endhighlight %}

*Want to use a CLI to check hosts MAC Address but during this lab lets use VMware Fusion App and check each hosts Network > Settings > Advanced Options and write down the MAC Address.*

Added three host in my case, docker(munki) vm, osx client vm and a osx server vm.

{% highlight bash %}
host munki {
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

*Edit the hosts file with your favorit edit and add your hosts after the last line*

{% highlight bash %}
sudo vim /etc/hosts
{% endhighlight %}

{% highlight bash %}
192.168.125.140 munki.example.com
192.168.125.141	client.example.com
192.168.125.142 server.example.com
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

### Create certificates and Docker containers.

In this lab I'm using self-signed certificates but when you plan for a production solution you should go with certificates from an [Provider](https://en.wikipedia.org/wiki/Certificate_authority#Providers).


During the signing proccess you need to fill in *County Code, State, City, Organization, Common Name, Department and e-mail* just remember the **password** as it will be used in the convert process.

###### Create a lab catalog and clone docker-munki-ssl repo.
{% highlight bash %}
mkdir -p ~/munki-lab
cd ~/munki-lab
git clone git@github.com:ustwo/docker-munki-ssl.git
{% endhighlight %}

###### Create a Certificate Authority root
{% highlight bash %}
openssl genrsa -des3 -out ca.key 4096
openssl req -new -x509 -days 365 -key ca.key -out ca.crt
{% endhighlight %}

###### Create the Client Key and CSR
{% highlight bash %}
openssl genrsa -des3 -out client.key 4096
openssl req -new -key client.key -out client.csr
{% endhighlight %}

###### Self-sign Client crt
{% highlight bash %}
openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
{% endhighlight %}

###### Convert Client Key and crt to PEM
{% highlight bash %}
openssl x509 -in client.crt -out client-munki.crt.pem -outform PEM
openssl rsa -in client.key -out client-munki.key.pem -outform PEM
{% endhighlight %}

###### Create the Server Key and CRT
{% highlight bash %}
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
{% endhighlight %}

###### Build the munki container
{% highlight bash %}
docker build -t munki-ssl .
{% endhighlight %}

###### Create a Data Container:
{% highlight bash %}
docker run -d --name munki-data --entrypoint /bin/echo munki-ssl Data-only container for munki-ssl
{% endhighlight %}

###### Start the munki-ssl container
{% highlight bash %}
docker run -d --name munki-ssl --volumes-from munki-data -p 443:443 -h munki-ssl munki-ssl
{% endhighlight %}

###### Download and start the munki-smb container
*Remember to only run this container when needed as it is a public share and require no password.*
{% highlight bash %}
docker run -d -p 445:445 --volumes-from munki-data --name munki-smb ustwo/munki-smb /munki_repo
{% endhighlight %}
*You may have to change permissions on /munki_repo to allow for read-write privileges by a guest.*
{% highlight bash %}
docker exec munki-smb chown -R nobody:nogroup /munki_repo/
docker exec minki-smb chmod -R ugo+rwx /munki_repo/
{% endhighlight %}

### End of Part1

Next part will cover the osx server and client configuration.

###### Sources

* [afp548](https://www.afp548.com/2015/01/22/building-munki-with-docker)
* [mtigas](https://gist.github.com/mtigas/952344)
* [pravka](https://pravka.net/nginx-mutual-auth)
* [nginx](http://wiki.nginx.org/FullExample)
