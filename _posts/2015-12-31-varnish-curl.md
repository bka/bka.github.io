---
title:  "Varnish debugging with curl"
date:   2015-12-31 05:36:00
categories: Magento
---

Trying to access an ESI block in a Magento shop may result in following error:
```Error 403 External ESI requests are not allowed```. Makes sense, but for some
debugging I needed to request the block manually. Following example using curl
applies for a setup where varnish is running on port 80 and apache on 8080.

{% highlight bash %}
curl -H "X-Turpentine-Secret-Handshake:1" -H "X-Varnish-Esi-Method: esi" --header 'Host: www.example.de' http://localhost:8080/index.php/turpentine/esi/getBlock/method/esi/access/private/hmac/cf292cebffc75d1521ebfb6784eff77c729418ff6fc0171037b9eabe9862c28d/data/6AryhgsBOYGddSdbsuY3tfaKeRenzd5PP8TyaLFSegp4SJA6wHRhBWmtOWz6HW0xguJUrVdg0he4DBYbnvWPo.W.y88zGfIkkq6KgkkTXOWLemU2j9ZevdGZCGmp6FppZe2LzbvEM5VaAfKunkS3Bo7MV1TL-dI3pqZwrUqToalc-Kboy8NDpTG-9Dc3c7RLISvPquR3eAvKFNCYhX6.ZiO1KH4vGi8nfO1zmFcD9LrFCwQeOIW5tWzYOJXS368fycQjW-Y9Tgk4oNJHZF4Gb59KJTSU6GQM79Vagscrtr2c8MwlQGdOzVcFm1LGsIEA6rMXMHGQBJgVFo5uxZZOVH1yC4zntgZv/
{% endhighlight %}

