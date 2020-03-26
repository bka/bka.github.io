---
title:  "Purge Varnish Turpentine via varnishadm"
date:   2020-03-26 08:00:00
categories: Magento
---

Took me some time to figure this command out:

    php shell/varnishadm.php ban req.url "~" "."
