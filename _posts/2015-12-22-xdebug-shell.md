---
title:  "XDebug for shell tasks"
date:   2015-12-22 19:00:00
categories: Magento2
---

**XDebug for shell tasks**

Because I needed this some days ago: triggering XDEBUG when executing a shell
command like ```bin/magento```. This also works from inside a docker container
on a vagrant machine. In the following example 192.168.56.1 actually is my
host machine which serves as a gateway for the vagrant machine. For local debugging
remote_host needs to be set localhost.

{% highlight shell %}

XDEBUG_CONFIG="idekey=vim" php -dxdebug.remote_host=192.168.56.1 -dxdebug.remote_enable=on -f bin/magento setup:upgrade

{% endhighlight %}

