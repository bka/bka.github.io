---
title:  "Logging SQL queries"
date:   2015-12-28 12:36:00
categories: Magento2
---

**Logging SQL queries in Magento2**

In order to enable logging of sql queries in Magento2 you need to edit ```app/etc/di.xml``` and
change the type for ```LoggerInterface``` from ```Magento\Framework\DB\Logger\Quiet``` to ```Magento\Framework\DB\Logger\File```.

{% highlight xml %}
<preference for="Magento\Framework\DB\LoggerInterface" type="Magento\Framework\DB\Logger\File"/>
{% endhighlight %}

Append this at the end of the file to pass required arguments:

{% highlight xml %}
<type name="Magento\Framework\DB\Logger\File">
    <arguments>
        <argument name="logAllQueries" xsi:type="boolean">true</argument>
        <argument name="debugFile" xsi:type="string">log/sql.log</argument>
    </arguments>
</type>
{% endhighlight %}

Because I'm paranoid, I also executed a ```bin/magento cache:clean``` afterwards and from now on all queries
should be logged into ```var/log/sql.log```.

