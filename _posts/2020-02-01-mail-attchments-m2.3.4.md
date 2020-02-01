---
title:  "E-Mails are sent as attachments after upgrading to Magento 2.3.4"
date:   2020-02-01 08:00:00
categories: Magento2
---

We recently updated a Magento shop to latest version Magento 2.3.4. First, I was wondering that order confirmation mails were sent as attachments. Meaning there was no content anymore, everything was placed in a file called `attachment.html`.

![Thunderbird attachment example](/images/Auswahl_623.png "Thunderbird attachment example")

After analysing this issue, I stumpled upon this [issue](https://github.com/magento/magento2/issues/25076). Reason was a newly introduced `$part->setDisposition(Mime::DISPOSITION_INLINE);`. Thankfully, this could be patched by applying this [commit](https://github.com/magento/magento2/commit/6976aabdfdab91a9d06e412c2ed619538ed034b6).
