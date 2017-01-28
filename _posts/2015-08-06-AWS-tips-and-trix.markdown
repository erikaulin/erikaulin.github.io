---
title:  "AWS day to day"
date:   2015-08-06 16:12:00
description: AWS in you day to day work
keywords: erik,aulin,blog,osx,aws,ustwo,development
---

I love working from the console and aws-cli could be your best friend.
Guide how to install this on you system can be found on [Getting Set Up with the AWS Command Line Interface] (http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html).

Once you have finished all the step and can execute aws ec2 describe-instances from your favorit console your are ready!
{% highlight bash %}
aws ec2 describe-instances
-------------------
|DescribeInstances|
+-----------------+
{% endhighlight %}

First step is to generate a SSH key, you can use multiple keys depending on needs.
{% highlight bash %}
ssh-keygen -t rsa -f ~/.ssh/aws-<NAME_OF_KEY>-ec2 -b 4096
{% endhighlight %}


Next step is to upload your SSH key to your AWS.
{% highlight bash %}
aws ec2 import-key-pair --key-name aws-<NAME_OF_KEY>-ec2-key --public-key-material "$(cat ~/.ssh/aws-<NAME_OF_KEY>-ec2.pub)”
{% endhighlight %}

Now its time to crete a security group and some basic rules.
In my case I only want to allow SSH traffic from a know WAN IP.
{% highlight bash %}
aws ec2 create-security-group --group-name SSHRule --description "Inbound SSH from know IP address"

aws ec2 authorize-security-group-ingress --group-name SSHRule --cidr <WAN_IP>/32 --protocol tcp --port 22
{% endhighlight %}

I use Frankfurt DataCenter and you can find images on [Amazon Machine Image (AMI)](https://eu-central-1.console.aws.amazon.com/ec2/v2/home?region=eu-central-1#LaunchInstanceWizard:).
In this example I'm using Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-accff2b1 on a micro hardware model.

{% highlight bash %}
aws ec2 run-instances --image-id ami-accff2b1 --key-name aws-<NAME_OF_KEY>-ec2-key --instance-type t2.micro --security-groups SSHRule
{% endhighlight %}

Machine will now be build and external IP address associated with the host.
You can find the PublicIp address greping the describe-instances.
{% highlight bash %}
aws ec2 describe-instances | grep PublicIp
|||  PublicIpAddress       |  52.XX.XX.83                                         |||
|||||  PublicIp      |  52.XX.XX.83                                             |||||
||||||  PublicIp      |  52.XX.XX.83
{% endhighlight %}

Now you are ready to SSH to your machine.
{% highlight bash %}
ssh -i ~/.ssh/aws-<NAME_OF_KEY>-ec2 -l ubuntu 52.XX.XX.185
{% endhighlight %}
