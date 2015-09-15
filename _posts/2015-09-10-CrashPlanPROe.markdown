---
title:  "Crashplan PROe"
date:   2015-09-15 12:37:00
description: How to setup CrashPlan PROe server on a synology box using Docker,
keywords: erik aulin,blog,crashplan,code42,ustwo,development,synology,docker
---

### Introduction
In this tutorial I will cover how to setup a [CrashPlan](http://www.code42.com/products/crashplan) PROE server hosted in docker container running on a [Synology](https://www.synology.com/en-global).

***
### Prerequisites
* [Synology hardware](https://www.synology.com/en-us/products).

***

### Get started

First you need create the folder structure that will be used to expose the code42 data and make it accessible directly from synology shares.
In this example I want to expose all folder that contain CrashPlan data, logs and use a specific backup destination.

![Folder Stucture](/assets/images/code42/code42_folders.png)

Next you need to install the Docker Applications using Package Center.

![Folder Stucture](/assets/images/code42/code42_docker_install.png)

#### Download image and create container.

Start the Docker from menu and go to Registry search for *crashplanproe* select it and click on Download.

![Folder Stucture](/assets/images/code42/code42_docker_registry.png)

Next go to Image and Launch dropdown menu select *Launch with wizard*

![Folder Stucture](/assets/images/code42/code42_docker_image.png)

Choose a name for your Container and add [ports](http://support.code42.com/Administrator/3/Planning_And_Installing/TCP_And_UDP_Ports) needed.

![Folder Stucture](/assets/images/code42/code42_docker_wizard.png)

Select CPU Priority level and optional shortcut on desktop.

![Folder Stucture](/assets/images/code42/code42_docker_wizard2.png)

Before you hit Apply we need to set some Advanced Settings.

![Folder Stucture](/assets/images/code42/code42_docker_wizard3.png)

Now we get the option to specify what location should be exposed to the container.
In this case you can use the folders we created. Once done hit OK and then Apply.

![Folder Stucture](/assets/images/code42/code42_docker_volume.png)

### Using Code42 CrashPlanPROe Server

#### Starting the Container

Now you can click Container and you should see the code42 container.
If its not activated click the power on icon. This part will take a wile as it will download the binary's and setup a default environment.

![Folder Stucture](/assets/images/code42/code42_docker_container.png)

Once the service is up and running you can see that the folder's you created have been filled with data.

![Folder Stucture](/assets/images/code42/code42_logs.png)

#### Configure Crash Plan PROe services.

It's time to configure the service. Head to the same IP as the synology but with https and port 4285.
In my case `https://10.2.0.200:4285`. Use the MasterKey or sign up for a trial.

Now go to *Settings > Server* and change *Website protocol, host and port* to your synology IP.
You also need to configure *Primary network address* that should be the same IP but with port 4282.

![Folder Stucture](/assets/images/code42/code42_network.png)

Now got to *Destinations > Server* and click the default server > cog > *Add Store Point*.
Fill in a name and location /opt/backup_destination
![Folder Stucture](/assets/images/code42/code42_server_storepoints.png)

Next go to *Destianations > Server* and click the default storage > cog > *Pause Incomming Data* and *Reject New Archives*

![Folder Stucture](/assets/images/code42/code42_default_store.png)

Next go to *Destianations > Server* and click the default storage > cog > *Accept Incomming Data* and *Accept New Archives*

![Folder Stucture](/assets/images/code42/code42_new_store.png)

### Thats it!

Now the server is up and running and you can continue to configure the Code42 CrashPlanPROe server as it fits your needs.

Cheers!
