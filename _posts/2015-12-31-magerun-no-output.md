---
title:  "Magerun: no output on remote server"
date:   2015-12-31 05:36:00
categories: Magento
---

Yesterday I had the case that on a remote machine ```n98-magerun``` was not working.
It gave me no output at all. It seemed execution did not even start. Readme of
magerun documents this issue like this:

> On some Debian systems with compiled in suhosin the phar extension must be added to a whitelist.

Instead of adding this to ```php.ini```, in my case it was sufficient to pass the whitelist as an argument.
With this command I could use magerun as usual.

{% highlight bash %}
$ php -d suhosin.executor.include.whitelist="phar" shell/n98-magerun.phar dev:console
{% endhighlight %}

