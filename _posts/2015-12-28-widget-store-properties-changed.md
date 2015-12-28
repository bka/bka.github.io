---
title:  "Error: Storefront properties changed"
date:   2015-12-28 18:36:00
categories: Magento2
---

**Error when creating custom widget**

When you are creating a custom widget for Magento2 you may encounter following error:

{% highlight text %}
Storefront Properties Changes have been made to this section that have not been saved. This tab contains invalid data. Please resolve this before saving.
Widget Options Changes have been made to this section that have not been saved. This tab contains invalid data. Please resolve this before saving.
{% endhighlight %}

I highly recomment checking your declared namespace. In may case the namespace
did not match, after I moved the file, resulting in this speaking error above.

{% highlight php %}
<?php

namespace Vendor\Modulename\Block\Adminhtml\Model\Widget;
{% endhighlight %}

