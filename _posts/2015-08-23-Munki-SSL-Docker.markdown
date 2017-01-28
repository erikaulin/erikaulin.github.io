---
title:  "Munki with SSL using Docker"
date:   2015-08-28 19:49:00
description: Munki with SSL client Authentication using docker .
keywords: erik,aulin,blog,osx,docker,munki,vmware,ustwo,development,ssl
---

In this tutorial I will cover how to setup an [Munki](https://www.munki.org/munki) environment with client SSL authentication. Hosting the munki data and web server in docker and using a osx machine for populating data using [Autopkgr](https://github.com/lindegroup/autopkgr). Most guides out there use [Chef](https://www.chef.io) or [Puppet](https://puppetlabs.com) to push client configuration but I wanted to focus on docker and have the possibility to move between solutions depending on situation and current infrastructure.

***

### Prerequisites
* [Mac Computer](http://www.apple.com/mac/).
* [VMWare Fusion](http://www.vmware.com/products/fusion) or [Virtualbox](https://www.virtualbox.org).
* [Docker Toolbox](https://www.docker.com/toolbox).
* [Virtual Docker](https://docs.docker.com/machine).
* [Virtual OSX](http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2082109#) Client and Server.

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

***

### What about data in your repo?
It's high time to fill your repo with data, in my lab I used [smb share](https://hub.docker.com/r/nmcspadden/smb-munki/) to share the munki-data container then I used [Autopkgr](http://www.lindegroup.com/autopkgr) and [MunkiAdmin](http://hjuutilainen.github.io/munkiadmin/) to fill it.
It will not be covered in this guide but [google](google.com) will help your out.

***

### Munki Client setup

Transfer *client-munki.crt.pem* and *client-munki.key.pem* to your client.
{% highlight bash %}
scp client-munki.* admin@client.example.com:/tmp
{% endhighlight %}
The ssh to your client machine and continue the setup.

###### Place certs in Managed Install folder
{% highlight bash %}
sudo mkdir -p /Library/Managed\ Installs/certs
sudo chmod 0700 /Library/Managed\ Installs/certs
sudo cp /tmp/client-munki.crt.pem /Library/Managed\ Installs/certs/client-munki.crt.pem
sudo cp /tmp/client-munki.key.pem /Library/Managed\ Installs/certs/client-munki.key.pem
sudo chmod 0600 /Library/Managed\ Installs/certs/client-munki*
sudo chown root:wheel /Library/Managed\ Installs/certs/client-munki*
{% endhighlight %}

######  Change the ManagedInstalls.plist defaults:
{% highlight bash %}
sudo defaults write /Library/Preferences/ManagedInstalls SoftwareRepoURL "https://munki.example.com/repo"
sudo defaults write /Library/Preferences/ManagedInstalls ClientCertificatePath "/Library/Managed Installs/certs/client-munki.crt.pem"
sudo defaults write /Library/Preferences/ManagedInstalls ClientKeyPath "/Library/Managed Installs/certs/client-munki.key.pem"
sudo defaults write /Library/Preferences/ManagedInstalls UseClientCertificate -bool TRUE
{% endhighlight %}

###### Test out the client:
{% highlight bash %}
sudo /usr/local/munki/managedsoftwareupdate -vvv --checkonly
{% endhighlight %}

***

###### Sources

* [afp548](https://www.afp548.com/2015/01/22/building-munki-with-docker)
* [mtigas](https://gist.github.com/mtigas/952344)
* [pravka](https://pravka.net/nginx-mutual-auth)
* [nginx](http://wiki.nginx.org/FullExample)
