---
title:  "Bringing Magento2 to Production"
date:   2016-08-16 00:00:00
categories: Magento2
---

# Preface
This is a status report of my experience trying to get a Magento 2.0.x shop into production mode. I was fighting quite a while to deploy one of our recent shop projects to live production mode. Sounds very simple and straight-forward on the first glance, if you've read the docs and have some familarity with the Magento 2 system. However, it was very painful and caused me a lot of headache. I want to share some of the pain I had and maybe help others also confronted with this daunting task.

Lets start with some theory. In Magento 2, you have some folders where the system generates files. In development mode, this is done on the fly and you don't experience any problems, except that it is fucking slow on initial requests. These folders are:

* var/di
* var/generation
* pub/static/frontend
* pub/static/_cache
* pub/static/adminhmtl
* pub/static/_requirejs

It sounds pretty reasonable that this _on the fly generation_ is not a good idea for production systems, where performance matters. For this case, Magento provides some tasks to generate these files. These tasks are:

* `bin/magento setup:di:compile`
* `bin/magento setup:static-content:deploy`

This is, where most of my painful experience began. Not to mention that especially `setup:static-content:deploy` eats about 30 minutes of your time (_everytime_) for generating assets for more than 1 store language. I dont't understand why it needs to take so long. Maybe the process can be improved, I don't care for now. Waiting time is just annoying everytime.

However on my way, I discovered some core issues in 2.0.x, which finally forced us to update to Magento 2.1.0. So lets get started with a list of detailed issues, I ran into.

## Switching to HTTPS

This is not a bug, rather a topic not pretty well documented. For live systems you usually want to have https in place. In theory, pretty simple. Install certificate, configure your web server and set _web/secure/base\_url_. However, as we have varnish and nginx and multiple backend servers in place, I couldn't persuade them to propagate the correct http headers to Magento. I didn't want to put more time into debugging this. A workaround for forcing Magento to https internally, is to modify _pub/index.php_ and prepend some variables:

{% highlight php %}
<?php
$_SERVER['HTTPS'] = 'on';
$_SERVER['HTTP_X_FORWARDED_PROTO'] = 'https';
$_SERVER['HTTP_X_FORWARDED_PORT'] = '443';
{% endhighlight %}

This is required to force magento to link all assets with https, because otherwise your stylesheets and javascripts would get linked with http, causing security warnings in the browser. However, this is only half of the story. Also `setup:static-content:deploy` task is affected by this change. In https-mode magento links to `pub/static/_requirejs/frontend/Magento/luma/de_DE/secure/requirejs-config.js`, which had been `pub/static/_requirejs/frontend/Magento/luma/de_DE/requirejs-config.js` previously. See the difference? Only Jesus knows, why Magento uses a different `requirejs-config.js` in a subfolder called `secure` in this case. I wasn't able to get this file out of the deploy task, until I discovered somewhere on google, that you need to set the environment variable HTTPS as well. This sucks very hard and I don't understand why the deploy task can't just generate everything you need. Long story short, just call your generation tasks with `HTTPS="on"`:

{% highlight bash %}
HTTPS="on" php bin/magento setup:static-content:deploy de_DE en_US en_GB
{% endhighlight %}

## Deploy task generates requirejs-config only for default language

I had a lot of 404 loading errors when switching to another language, which were caused by a missing `requirejs-config.js`. It turned out, that this file is only generated for the default language. I could work around this by creating some symlinks and this issue got fixed by Magento 2.1. Here is the respective [Issue on Github](https://github.com/magento/magento2/pull/3297).

## Compile task cannot handle preference and plugin for the same class

This one caused a lot of trouble and finally forced me to update to Magento 2.1. Prior versions are obviously not able to handle both preference class and plugin for the same class. It was very very painful to track this issue down, especially because it only happens if you explicilty use `bin/magento setup:di:compile`, which you don't do locally in development mode. Thanks for this wasted time. Let me provide some explanation. In Magento 2 you have the option to change behavior of core classes either by preference or by plugin. Both is done in your modules `etc/di.xml` file. The preference tag basically changes the requested class to your changed one:

{% highlight xml %}
<preference for="Magento\Catalog\Model\Product" type="Foo\MyModule\Model\Product" />
{% endhighlight %}

Whenever the ObjectManager is called to get an instance of `Magento\Catalog\Model\Product` it will give you an instance of `Foo\MyModule\Model\Product`.

With plugins, it is possible to only exchange certain functions. I don't want to dive too deeply into this topic. For this issue, it is important to know, that magic which is making the plugin system work happens inside of `var/di`. Magento generates Interceptor classes which forward method calls to your plugin.

{% highlight xml %}
<type name="{ObservedType}">
    <plugin name="{pluginName}" type="{PluginClassName}" sortOrder="1" disabled="true"/>
</type>
{% endhighlight %}

In development mode, these required Intercepter classes are generated on the fly and everything works perfectly well. But when using `bin/magento setup:di:compile`, which is expected to generate every necessary Interceptor class inside `var/di`, some unexpected behavoir might occur. You might expect this generation to be totally transparent. It shouldn't matter if it is done on the fly or pre-generated, right? Got ya, made this assumption without your dear friend Magento 2. The reason for this is a core bug in releases below 2.1. Before 2.1, pre-generated Interceptor classes do not inherit from classes defined in preference tag. Instead they just inherit from its base class leaving your defined preference totally useless behind, if some plugin hooks into its methods. Here is the respective [Issue on Github](https://github.com/magento/magento2/pull/5579).


## Old composer version

I wish, I could tell you that all problems were solved by moving to Magento 2.1. Besides some breaking changes in the backend it seemed to work at the first glance. Until I wanted to login to the adminpanel on our production system. I got something like this:

{% highlight bash %}
Warning: is_dir(): open_basedir restriction in effect. File(/etc/pki/tls/certs) is not within the allowed path(s): (/var/www/example.com:/var/www/example.com/wwwroot:/usr/share/pear:/usr/share/php:/usr/share/phpmyadmin:/tmp:/usr/local/lib/php:/usr/share/misc) in /var/www/example.com/wwwroot/vendor/composer/composer/src/Composer/Util/RemoteFilesystem.php on line 914
#1 /var/www/example.com/wwwroot/vendor/composer/composer/src/Composer/Util/RemoteFilesystem.php(915): is_dir('/etc/pki/tls/ce...')
#2 /var/www/example.com/wwwroot/vendor/composer/composer/src/Composer/Util/RemoteFilesystem.php(801): Composer\Util\RemoteFilesystem->getSystemCaRootBundlePath()
#3 /var/www/example.com/wwwroot/vendor/composer/composer/src/Composer/Util/RemoteFilesystem.php(61): Composer\Util\RemoteFilesystem->getTlsDefaults(Array)
{% endhighlight %}

Whamp, greetings from Magento again. Would have been to easy, right? It turned out, that 2.1 was still using an outdated alpha version of composer in its dependency. This was solvable by changing version of composer according to this [Issue on Github](https://github.com/magento/magento2/issues/4359)

{% highlight bash %}
"composer/composer": "1.1.2 as 1.0.0-beta1",
{% endhighlight %}

## File requirejs-min-resolver.min.js missing

Already tired? Keep having fun with your freshly upgraded shop and asset generation. Makes a lot of fun to count all these 404 errors, although `setup:static-content:deploy` has been run. Turns out that again some file is not even generated. This time `requirejs-min-resolver.min.js` is missing. Here is the respective [Issue on Github](https://github.com/magento/magento2/issues/2976), which is currently only solvable by manually patching the core as it is not integrated into a relase AFAIK.

## Failed to open stream var/di/setup.ser

Not a show-stopper but annoying is my next issue on the list. After running `bin/magento setup:di:compile` (remember, this cool task to generate code inside `var/di` and `var/generation`), it was not possible to run other tasks anymore:

{% highlight bash %}
Warning: file_get_contents(/var/www/example.com/releases/20160810171124/var/di/setup.ser): failed to open stream: No such file or directory in /var/www/example.com/releases/20160810171124/vendor/magento/fra...
{% endhighlight %}

As you might expect, here is the respective [Issue on Github](https://github.com/magento/magento2/issues/4795). Workaround is to delete content of `var/di` or move it out of scope temporarily.

## bin/magento always returns status code 0

Nearly reached the end of my list with this one. Also not a show-stopper but annoying, because our lovely `bin/magento` tasks do not return proper status codes if an error occoured. But especially for automated deployments with Magallanes/Jenkins this is fucking mandatory because in bad cases it leads to broken live systems. Respective issues and pull requests are [here](https://github.com/magento/magento2/issues/3060), [here](https://github.com/magento/magento2/pull/3189) and [here](https://github.com/magento/magento2/pull/6171/commits/4d1fe5c84145c6d6cc490bb944e48e08ccea9051).

## Asset merging is very slow

Merging all of your css and javascript into single files is a good idea and recommended. However, currently I would suggest not to use this feature in Magento 2 at all, because it makes your site incredibly slow. I found at least one [Issue on Github](https://github.com/magento/magento2/issues/4710). There might be more and after I had a look into how the progress of asset merging works in Magento 2, my impression is, that the current implementation will never work well at all. Let's have a look into the class which as actually performing this merge:

`lib/internal/Magento/Framework/View/Asset/MergeService.php`

{% highlight php startinline=true %}
<?php
public function getMergedAssets(array $assets, $contentType)
{
    $isCss = $contentType == 'css';
    $isJs = $contentType == 'js';
    if (!$isCss && !$isJs) {
        throw new \InvalidArgumentException("Merge for content type '{$contentType}' is not supported.");
    }

    $isCssMergeEnabled = $this->config->isMergeCssFiles();
    $isJsMergeEnabled = $this->config->isMergeJsFiles();
    if (($isCss && $isCssMergeEnabled) || ($isJs && $isJsMergeEnabled)) {
        $mergeStrategyClass = \Magento\Framework\View\Asset\MergeStrategy\FileExists::class;

        if ($this->state->getMode() === \Magento\Framework\App\State::MODE_DEVELOPER) {
            $mergeStrategyClass = \Magento\Framework\View\Asset\MergeStrategy\Checksum::class;
        }

        $mergeStrategy = $this->objectManager->get($mergeStrategyClass);

        $assets = $this->objectManager->create(
            \Magento\Framework\View\Asset\Merged::class,
            ['assets' => $assets, 'mergeStrategy' => $mergeStrategy]
        );
    }

    return $assets;
}
{% endhighlight %}

The main problem is, that it is done on the fly, *always*. No matter, how hard you deploy your static content with `bin/magento setup:static-content:deploy` you won't get merged assets out of it. They are always generated on the fly into `pub/static/_cache`. Nice one! Just another folder for generated assets, life was just too easy. However, the worst part of this is, that merging of assets is done based on the current asset group, which might be different for different pages. I understand, that we deal with spreaded resources due to requirejs and it is hard to determine which assets will be finally present due to this blown up _xml-dynamic-rewrite-php-crap_, but there must be another way to make asset merging useable. Current solution totally sucks at all. It will just eat up all you server resources for the next hours until all pages have been visited and all possible asset group combinations have been generated.

# Sum up

Honestly? After a few months with Magento 2, it still feels like an ugly damn bitch waiting to face me with its next surprises. You need very solid full stack development skills, a lot of stamina and a very high frustation tolerance bugging through the system. Sure it is Open Source and Magento 2 introduced a lot of improvements like the ObjectManager, DI, namescpaes and composer, but there are other aspects of the system that suck very hard, and live deployment is one of it. As a developer, I want to show results and not explain everytime why trivial things like moving buttons in checkout take so long. Btw, have a look at how the checkout process actually works and try to make minor changes. Couldn't hardly be more complicated. Have fun, watch a horror movie instead, you time is will be equally wasted. I will continue my journey with Magento 2 and hope, it will improve with future releases. But it will be a hard way.
