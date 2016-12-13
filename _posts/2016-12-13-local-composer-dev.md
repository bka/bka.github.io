---
title:  "local module development for Magento2 with composer"
date:   2016-12-13 00:00:00
categories: Magento2
---

# Preface

One thing I like about Magento2 is, that it is completly based on [composer](https://getcomposer.org/). It means you are able to separate development of Magento2 modules from the actual store you are testing it on. This is especially useful, if you are having multiple modules installed in multiple stores.


# The problem

Suppose I have a module, I want to use in my current store. To install it with composer I would add it to my `composer.json` like this:

    {
        "repositories": [
            {
                "type": "vcs",
                "url": "https://github.com/bka/magento2-delete-orders.git"
            }
        ],
        "require": {
            "bka/delete-orders": "~1.0.0"
        }
    }

After running `composer install`, a copy of the module is placed in `vendor/bka/delete-orders`. Now, the problem is you cannot really make changes to this module without breaking a sweat. The module inside `vendor` is just a copy and not git repo. One working way would be:

* make changes to your original repository
* commit and push
* run composer update inside your project
* test your change

Not a satisfying workflow.

# The solution

The golden trick in this case is to specify a local repo with type path like this:

    {
      "repositories": [
          {
              "type": "path",
              "url": "/home/bernd/work/magento2-modules/*"
          }
      ]
    }

Now, when running `composer install` the content is symlinked.

    composer install
     Loading composer repositories with package information
     Installing dependencies (including require-dev)
       - Installing bka/delete-orders (1.0.0)
           Symlinked from /home/bernd/work/magento2-modules/magento2-delete-orders/
    Writing lock file
    Generating autoload files

## Global configuration

An even better approach is, to define the path to your shared modules globally for all projects inside a `~/.composer/conig.json` like this:

    {
        "repositories": [
            {
                "type": "path",
                "url": "/home/bernd/work/magento2-modules/*"
            }
        ],
        "config": {
            "minimum-stability": "dev",
            "prefer-stable": true
        }
    }

This cool thing about this is, that this config only applies on your local system. When deploying, the pushed version in the regular repo will be used.
