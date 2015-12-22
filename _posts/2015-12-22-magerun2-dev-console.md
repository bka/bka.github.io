---
title:  "magerun2 dev:console"
date:   2015-12-21 21:30:00
categories: Magento2
---

**magerun2 dev:console**

Magento2 makes heavy use of Dependency Injection. There is no longer a need for
the God-Class Mage to create objects like ```Mage::getModel("...")```. The
DI concept is a reasonable one and I like it but I was wondering how I could
continue using [magerun](https://github.com/netz98/n98-magerun) for testing
code snippets. Use ```bin/n98-magerun2.phar dev:console``` in your shell which
is a great tool for this purpose. However, it took me some time to figure out
how to load Magento models without a Mage class. The solution is that the
ObjectManager can be accessed directly to create required instances. Here is an
example:

{% highlight php startinline=true %}
<?php
$objectManager = \Magento\Framework\App\ObjectManager::getInstance();
$productCollection = $objectManager->create("\Magento\Catalog\Model\ResourceModel\Product\Collection");
$product = $productCollection->getFirstItem();
$product->getData()
{% endhighlight %}
