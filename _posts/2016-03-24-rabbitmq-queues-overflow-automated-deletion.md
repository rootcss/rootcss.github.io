---
layout: post
title:  "RabbitMQ: Automated deletion of 1000s of queues"
subtitle: ""
date:   2016-03-24 18:34:01
categories: [DevOps, Hacker, Programming]
comments: true
---

Recently, I was using [sneakers](https://github.com/jondot/sneakers){:target="_blank"} for rails, which is a small framework for Ruby and RabbitMQ. One issue with sneakers is that, if you have faulty configuration for a queue or you do not provide a queue name, it leaves it upto rabbitmq to define it. So, for some reason (which I don't want to focus on), we had more than 1600 queues created on that particular exchange, and unfortunately they were not Auto-delete and we didn't want other exchanges and queues to get hurt because of this ;) 

Anyway, Now the challenge was to delete them, because for that you have to manually click on each queue, select 'Delete' on the new page and finally, confirm it on pop-up. 

I was like, :O

Anyways, Thanks to handy APIs of rabbitmq, these are the following few commands I used in order to delete them quickly. (rabbitmq generally creates default queues with names like `amq.gen--*`)

First let's list all the queues in the form of bash arrays:
{% highlight bash %}
rabbitmqadmin --host=<mqserver.hostname.com> --port=443 --ssl --vhost=<your_vhost> --username=<your_username> --password=<your_password> list queues | awk '{print $2}' | grep amq.gen  | xargs | sed -e 's/ /" "/g'
{% endhighlight %}

Now copy the output of it, declare as an array and run a loop to delete them all.

{% highlight bash %}
declare -a arr=("amq.gen--PxKpFBHkIxxebJEwbmV6g" "amq.gen--Q6BeLdfGHsXY6RgVmu8Ig" "amq.gen--WI0hRAHCOkPIrEULYc1vQ" "amq.gen--XufS0RrnfZUXyf0Rt1tAg" "amq.gen--_NXdwlSHYDJwGDiuX8_XA" ......)

for i in "${arr[@]}"
do
   echo "$i"
   rabbitmqadmin --host=<mqserver.hostname.com> --port=443 --ssl --vhost=<your_vhost> --username=<your_username> --password=<your_password> delete queue name="$i"
done
{% endhighlight %}

That's it. These small hacks makes me fall in love with programming everyday <3

Thanks!

<style type="text/css">
pre {
    white-space: pre-wrap;
}
</style>