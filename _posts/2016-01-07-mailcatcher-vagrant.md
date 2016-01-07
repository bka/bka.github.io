---
title:  "Using Mailcatcher with Vagrant and Docker"
date:   2016-01-07 17:30:00
categories: PHP
---

[MailCatcher](http://mailcatcher.me/) is a nice tool to catch generated E-Mails. However, the default installation
is meant for standard localhost set up. When usign [Vagrant](https://www.vagrantup.com/),
[Docker](https://www.docker.com/) or both for isolating different projects it
would be nice to use a single installation of MailCatcher on your development machine. This example covers a set up
for PHP projects. Usually, when setting up MailCatcher with PHP it is required to configure ```sendmail_path``` in
your ```php.ini``` like this:

    sendmail_path = /usr/bin/env catchmail -f some@from.address

The drawback with this approach is, that catchmail is coming from the MailCatcher installation and therefore it
would be necessary to install MailCatcher on the virtual machine provided by Vagrant or inside the corresponding
docker container. I was however looking for a solution to avoid this overhead and use a single installation on my
host system. It is a little bit tricky because the default sendmail is not able to forward mails to another host/port
without a lot of configuration overhead. With [mini_sendmail](http://acme.com/software/mini_sendmail/) however,
I found a working solution with following ```php.ini``` configuration:

    sendmail_path = /usr/bin/mini_sendmail -s192.168.56.1 -p1025 -t -i

From inside a virtual machine, 192.168.56.1 would be the address of the host system and port 1025 the port where
CatchMail is listening. It is also required to define the address where CatchMail should listen because the default
value is localhost and a connection would not be possible. This is why CatchMail needs to be started with the argument
smtp-ip:

    catchmail --smtp-ip 192.168.0.1

# Inside Docker

However, I had one issue using mini_sendmail inside a docker container. It failed with:

    can't determine username

I currently don't know the reason for this. It must have something to do with the glibc method getLogin() used
by mini_sendmail which was not available inside my docker container for whatever reason. However, a dirty workaround
was to change the line

    username = getlogin();

to

    username = "root";

and recompile mini_sendmail which did the trick.
