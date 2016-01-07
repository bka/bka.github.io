---
title:  "Changing Templates in Magento2"
date:   2016-01-05 19:30:00
categories: Magento2
---

This is a short explanation on how templates of Magento2 can be customized. Making changes to the original
template files is bad practise and should be avoided. So there are basically two ways to change a template.

# Custom Theme

Inside a custom theme any template can be changed, following the folder hierachy of Magento2.
E.g. changing the ```login.phtml``` (coming from the module ```Magento_Customer```) filepath would look like this:

    app/design/frontend/${VENDORNAME}/${THEMENAME}/Magento_Customer/templates/form/login.phtml

The original source file is living in:


    app/code/Magento/Customer/view/frontend/templates/form/login.phtml


I hope this helps to understand the naming scheme here.

# Custom Module

Inside a custom module it is a little bit harder to change templates. Here is an example of changing ```topmenu.html```
coming from the module ```Magento_Theme```. In the module the template needs to be placed here inside ```view/frontend/templates```:

    app/code/${VENDORNAME}/${MODULENAME}/view/frontend/templates/html/topmenu.phtml

Additionally, a layout definition is required to change to template for the corresponding block.

    app/code/${VENDORNAME}/${MODULENAME}/view/frontend/layout/default.xml

{% highlight xml %}
<?xml version="1.0"?>
<page layout="1columns" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
  <body>
    <referenceBlock class="Magento\Theme\Block\Html\Topmenu" name="catalog.topnav" template="${VENDORNAME}_${MODULENAME}::html/topmenu.phtml" ttl="false"/>
  </body>
</page>
{% endhighlight %}

