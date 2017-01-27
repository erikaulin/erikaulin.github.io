---
title:  "Unity Cache Server"
date:   2016-05-16 19:49:00
description: Unity Cache Server using Docker
keywords: erik,aulin,blog,unity3d,docker,development
---

### Introduction
This is a marvelous service to have running. Especially when you are switching a lot between different platforms and working in a team. Developers and testers working on both iOS and Android builds waste logs of time when doing platform change. Some of this pain can be avoided with Unity Cache Server for Unity3d.

Since Unity Cache Server supports multiple OS installations including linux I choose to build a docker container. Container is built to use standard ports and if you want to change default settings just [clone](https://github.com/erikaulin/docker-unitycacheserver) the repo check the [manual](http://docs.unity3d.com/Manual/CacheServer.html) and edit configurations files and rebuild your own image. For additional official information check [In-depth cache server](http://blogs.unity3d.com/2012/10/26/in-depth-cache-server/).


***

### Lets get started
There are many ways you use this service,  I chose to docker-compose to do it in this guide.

Unity Cache Server has a default cache roof set to 50gb of disk space. Keep this in mind when setting the the capacity on the docker engine host.

{% highlight bash %}
docker-compose.yml
unitycacheserver:
  image: erikaulin/unitycacheserver:latest
  container_name: unitycacheserver
  ports:
    - "8125:8125" # legacy port
    - "8126:8126"
  volumes_from:
    - volumes
volumes:
  image: busybox:latest
  container_name: unitycacheserver_data
  volumes:
    # Main Directory
    - /opt/cache
    - /opt/cache5.0
  command: /bin/echo
{% endhighlight %}

Moment of truth when you run docker-compose up -d it will pull the image from docker hub and create a two containers.

Unitycacheserver houses software and unitycacheserver_data the cached data that we want to be persistent. If all went well we should get log that looks like this in the end.

{% highlight bash %}
unitycacheserver      | Starting Services
unitycacheserver      | Cache Server directory /opt/cache
unitycacheserver      | Cache Server size 0
unitycacheserver      | Cache Server max cache size 53687091200
unitycacheserver      | Cache Server directory /opt/cache5.0
unitycacheserver      | Cache Server size 0
unitycacheserver      | Cache Server max cache size 53687091200
unitycacheserver      | Legacy Cache Server version 4.6
unitycacheserver      | Legacy Cache Server on port 8125
unitycacheserver      | Legacy Cache Server is ready
unitycacheserver      | Cache Server version 5.3
unitycacheserver      | Cache Server on port 8126
unitycacheserver      | Cache Server is ready
{% endhighlight %}

***

#### Unity3d

Setting up the Cache Server configuration couldn’t be easier. All you need to do is click Use Cache Server in the preferences and this can be found in Unity->Preferences on the Mac or Edit->Preferences on the PC.

![Folder Stucture](/images/unity3d/CacheServerEnabled.png)

Thats it!

Hope you have use for it or just inspire you.

Cheers Erik
