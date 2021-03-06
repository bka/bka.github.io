---
title:  "Purging Assets in Varnish when Deploying Magento2"
date:   2017-01-14 08:00:00
categories: Magento2
---

For a Magento2 shop in production mode, a Varnish cache is essential. I don't want to dive into details on how to set up your environment with Varnish. This post is about an issue with static assets like stylesheets, javascripts or images. Having a deployment tool like [Magallanes](http://magephp.com/) or [Capistrano](http://capistranorb.com/) in place to deploy your shop, you usually want the cache the be cleared to deliver all changes instantly. One way is to automatically execute `bin/magento cache:clean` on your server. However, this will not affect static assets. Read on to see why.


# Configure Magento2 to Purge Varnish

To purge Varnish when clearing the Magento2 cache, you need to configure the address of your Varnish instance in
`app/etc/env.php`. Usually value for host is localhost if Varnish is installed in the same machine.

{% highlight php %}
<?php
return array (
  'http_cache_hosts' =>
    array (
      0 =>
      array (
        'host' => '172.17.0.1',
      ),
  )
);
{% endhighlight %}


When executing `bin/magento cache:clean`, Magento2 will use this setting to send an http request with method PURGE to Varnish. You can do the same thing using curl:

    curl -H "X-Magento-Tags-Pattern: .*" -X PURGE localhost


# Activating Debug Mode for Testing

According to [Magento2 Docs](http://devdocs.magento.com/guides/v2.0/config-guide/varnish/config-varnish-final.html), you may switch to developer mode to see `X-Magento-Cache-Debug` in your response header. This is very useful for debugging cache issues. It can be achieved by setting `MAGE_MODE` in your
`.htaccess`.

    SetEnv MAGE_MODE developer

However, for testing this issue, I recommend to modify your `/etc/varnish/default.vcl` as well. Developer mode instructs Magento2 to send an additional reponse header called `X-Magento-Debug`. The `default.vcl` generated by Magento2 reacts on this header setting and puts another header to indicate if result is coming from Varnish.

{% highlight c %}
    sub vcl_deliver {
        if (resp.http.X-Magento-Debug) {
            if (resp.http.x-varnish ~ " ") {
                set resp.http.X-Magento-Cache-Debug = "HIT";
            } else {
                set resp.http.X-Magento-Cache-Debug = "MISS";
            }
        } else {
            unset resp.http.Age;
        }
{% endhighlight %}

And here is the catch: this debug header only applies for content delivered by Magento2. You won't see this header on other assets served by your webserver. Hence, I suggest removing this condition for testing like this.

{% highlight c %}
    sub vcl_deliver {
        # if (resp.http.X-Magento-Debug) {
            if (resp.http.x-varnish ~ " ") {
                set resp.http.X-Magento-Cache-Debug = "HIT";
            } else {
                set resp.http.X-Magento-Cache-Debug = "MISS";
            }
        # } else {
        #    unset resp.http.Age;
        # }
{% endhighlight %}


# Issue with assets

With this modified `default.vcl` in place, we can check for `X-Magento-Cache-Debug` header. On first request, it will give a `MISS`.

    $ curl -I http://magento2.local/pub/static/frontend/Magento/luma/en_US/mage/calendar.css
    HTTP/1.1 200 OK
    Date: Sat, 14 Jan 2017 13:03:47 GMT
    Last-Modified: Fri, 02 Dec 2016 21:36:03 GMT
    Vary: Accept-Encoding
    Content-Type: text/css
    ETag: W/"1885-542b3b997c96a-gzip"
    Age: 0
    X-Magento-Cache-Debug: MISS
    Connection: keep-alive

Subsequent calls will give a `HIT` indicating, this file was cached by Varnish:

    $ curl -I http://magento2.local/pub/static/frontend/Magento/luma/en_US/mage/calendar.css
    HTTP/1.1 200 OK
    Date: Sat, 14 Jan 2017 13:03:47 GMT
    Last-Modified: Fri, 02 Dec 2016 21:36:03 GMT
    Vary: Accept-Encoding
    Content-Type: text/css
    ETag: W/"1885-542b3b997c96a-gzip"
    Age: 2
    X-Magento-Cache-Debug: HIT
    Connection: keep-alive


Now, the problem is that this cached asset will not be purged by clearing the Magento2 cache. After executing `bin/magento cache:clean` it will still give a a `HIT` for this asset. Changed scripts or styles will not be active after deployment.

One way is to purge Varnish manually by e.g. restarting it or wait for the TTL, but there seems to be no built-in way to do this with Magento2 out of the box.

# Better Varnish Configuration for Purging Assets

Now, why doesn't purging Varnish affect static assets? A closer look into the `default.vcl` explains why. Purge only affects objects having an `X-Magento-Tags` header. This header is generated by Magento2 and will not be present for static assets, because they are served by the webserver directly.

{% highlight c %}
if (req.method == "PURGE") {
    if (client.ip !~ purge) {
        return (synth(405, "Method not allowed"));
    }
    if (!req.http.X-Magento-Tags-Pattern) {
        return (synth(400, "X-Magento-Tags-Pattern header required"));
    }
    ban("obj.http.X-Magento-Tags ~ " + req.http.X-Magento-Tags-Pattern);

    return (synth(200, "Purged"));
}
{% endhighlight %}

A minor modification of the standard `default.vcl` will instruct Varnish to purge all assets as well. Line 12-14 ban everything regargless of `X-Magento-Tags` header. But only if Magento2 wants to clear everything indicated by `.*`, leaving explicit purging of some tagged content untouched.

{% highlight c linenos %}
if (req.method == "PURGE") {
    if (client.ip !~ purge) {
        return (synth(405, "Method not allowed"));
    }
    if (!req.http.X-Magento-Tags-Pattern) {
        return (synth(400, "X-Magento-Tags-Pattern header required"));
    }
    ban("obj.http.X-Magento-Tags ~ " + req.http.X-Magento-Tags-Pattern);

    # If all Tags should be purged clear
    # ban everything to catch assets as well
    if (req.http.X-Magento-Tags-Pattern == ".*") {
      ban("req.url ~ .*");
    }

    return (synth(200, "Purged"));
}
{% endhighlight %}









