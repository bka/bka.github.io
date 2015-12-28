---
title:  "Setting values into core_config_data with an install script"
date:   2015-12-28 08:36:00
categories: Magento2
---

**Setting values into core_config_data with an install script**

It took me a while to figure out how to write configuration values within an
install script. My first approach to use ```\Magento\Config\Model\Config``` was
a dead end, because it requires some AuthenticationInterface and wants to
initialize a session which does not play very well with command line tasks
like ```bin/magento setup:upgrade```. In the end it was the famous ```Area code
not set``` Exception.

{% highlight bash %}
[Magento\Framework\Exception\LocalizedException]
Area code is not set
{% endhighlight %}

Instead of trying to use OOP and DI practice, just write them with a core db
connection, this reduces a lot of pain. By the way, this is how other core modules
are writing their values.

{% highlight bash %}
protected function saveConfigValue($path, $value){
  $data = [
      'scope' => 'default',
      'scope_id' => 0,
      'path' => $path,
      'value' => $value,
  ];
  $this->setup->getConnection()
      ->insertOnDuplicate($this->setup->getTable('core_config_data'), $data, ['value']);
}
{% endhighlight %}


{% highlight text %}
$this->saveConfigValue("design/header/welcome ", "Welcome")
{% endhighlight %}

