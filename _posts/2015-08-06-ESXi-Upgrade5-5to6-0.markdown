---
title:  "Upgrade ESXi from 5.5 to 6"
date:   2015-08-06 11:59:00
description: Macmini's need an upgrade from ESXi 5.5 to 6 using SSH and esxcli
keywords: erik,aulin,blog,osx,ustwo,development,esxi,ESXi-6,esxcli
---

It was time to upgrade my macmini's running vmware ESXi 5.5 to 6.
Since I didn't want to hazzel with ISO's and do it from the comfort of my desk I chose to do it via SSH and esxcli.

Shut down all VMs on your ESXi host machine.

Connect via SSH and enter maintenance mode.
{% highlight bash %}
vim-cmd /hostsvc/maintenance_mode_enter
{% endhighlight %}

To be able to download the updates you need a firewall rule for httpClient.
{% highlight bash %}
esxcli network firewall ruleset set -e true -r httpClient
{% endhighlight %}

Next, you need to list the ESXi 6.x files available. It's listed with Software, version, release number and build. In my case as of 2015-08 it's ESXi-6.0.0-2494585-standard.
{% highlight bash %}
esxcli software sources profile list -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml | grep ESXi-6
{% endhighlight %}

Now you are ready to upgrade your system.
{% highlight bash %}
esxcli software profile update -d https://hostupdate.vmware.com/software/VUM/PRODUCTION/main/vmw-depot-index.xml -p ESXi-6.0.0-2494585-standard
{% endhighlight %}

Now you are ready to restart the machine and enjoy ESXi-6 and its features.
Remember if you are upgrading a macmini you might need to reselect EFI Boot after restart.
