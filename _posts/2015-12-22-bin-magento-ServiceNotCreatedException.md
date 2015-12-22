---
title:  "bin/magento: ServiceNotCreatedException"
date:   2015-12-21 21:33:00
categories: Magento2
---

**bin/magento ServiceNotCreatedException**

I stumpeled upon a strange error message when running an upgrade task in magento2.

{% highlight bash %}
bin/magento setup:upgrade
{% endhighlight %}


{% highlight text %}
We are sorry, an error occurred. Try clearing the cache and code generation
directories. By default, they are: var/cache, var/di, var/generation, and
var/page_cache.

[Zend\ServiceManager\Exception\ServiceNotCreatedException]
An abstract factory could not create an instance of
magentosetupconsolecommandmoduleuninstallcommand(alias:
Magento\Setup\Console\Command\ModuleUninstallCommand).

[Zend\ServiceManager\Exception\ServiceNotCreatedException]
An exception was raised while creating "Magento\Setup\Console\Command\ModuleUninstallCommand"; no instance returned

[Magento\Framework\Exception\FileSystemException]
The file "/composer.json" doesn't exist
{% endhighlight %}


Even running ```bin/magento``` without any argument showed the same behavior.

{% highlight bash %}
bin/magento
{% endhighlight %}

The solution is simple and stupid somehow. It turned out that I had a copy-paste
error in my ```module.xml```. Note the additional ' at the end of the module name
which was the cause for this frightening error in my case.

{% highlight xml %}
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../lib/internal/Magento/Framework/Module/etc/module.xsd">
  <module name="Foo_Bar'" setup_version="0.0.1" data_version="0.0.1"/>
</config>
{% endhighlight %}
