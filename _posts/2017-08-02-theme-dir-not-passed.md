---
title:  "Exception: Required parameter theme_dir was not passed"
date:   2017-08-02 08:00:00
categories: Magento2
---

If you are trying to send an E-Mail in a cronjob or a console command and encounter following error that the required parameter `theme_dir` was not passed like this:

    [InvalidArgumentException]
    Required parameter 'theme_dir' was not passed

Try to wrap your code inside an `emulateAreaCode` call, which fixes this issue.

    // @var $appState Magento\Framework\App\State
    $appState->emulateAreaCode('frontend', function () {
        // stuff
    });
