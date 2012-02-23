---
date: '2011-07-30 20:41:34'
layout: post
slug: log-monitoring-with-graylog2-on-ubuntu-10-10
status: publish
title: Log monitoring with graylog2 on ubuntu 10.10
wordpress_id: '105'
categories:
- Server Administration
tags:
- analytics
- gelf
- gelf client
- graylog2
- log monitoring
- server administration
- ubuntu
---

.. role:: code
   :class: inline-code

When you get to a point where you are managing several application/web servers, you will come to understand that it is quite often that you have to go through the different logs that you have on each server. Maybe you want to troubleshoot a crash or you want to find cases of intrusion.

**Update**: Found a great post about logging: `Advanced logging in Linux <http://blog.theroux.ca/syslog/advanced-logging-on-linux/>`_.

.. more

Along came Graylog2
===================

Recently, I have come to the point where I need to manage several servers and troubleshooting them has become a bit of a pain. I set out to look for a good log monitoring system and the first option I saw was Splunk. It looked great but trying to set it up, I felt it was too complicated. So I kept searching and the next thing I came upon was `Graylog2 <http://graylog2.org/>`_.

Graylog2 uses MongoDB as it's data backend and is made clustering ready. The web interface is pretty simple, and I'm looking forward to the new Analytics features that is coming in 0.9.6 version.

Setting up Graylog2
===================

I set out to set it up in a little virtual server with Ubuntu 10.10 installed. The stable version at the time was 0.9.5.

I followed the instructions on their site and got it up and running. I don't know if the following part worked, maybe because I had a configuration wrong.

.. sourcecode:: console    
   :caption: bash

    ~$ cd bin
    ~$ ./graylog2ctl start

But I tried with the direct invokation and fixed the problems. But the problem was surely not running it as root.

I also installed the web interface without any problems following the instructions from their site.

Configuration, syslog and gelf
==============================

This is the part that was confusing. I had never had any experience with syslog and I thought Graylog2 worked with syslog so I tried to set it up. But apparently it does not. Graylog2 is kind of a syslog server in it self, it accepts syslog message from port 514 (default syslog port).

So from your client machines you can just forward your rsyslog messages (rsyslog because that's what Ubuntu has now instead of syslog) to the graylog server:

.. sourcecode:: console    
   :caption: /etc/rsyslog.d/graylog.conf

    $template Graylog2Friendly,"<%PRI%> %TIMESTAMP% %HOSTNAME% %APP-NAME% %PROCID% %MSGID% %STRUCTURED-DATA% %msg%\n"
    $ActionForwardDefaultTemplate Graylog2Friendly
    
    *.* @192.168.0.2 # this should be IP of the graylog2 server

This will forward all your logs to the graylog server. See Rsyslog manual to figure out how to send specific messages.

Sending GELF messages
=====================

You can test if your graylog2 server is receiving gelf messages (or syslog messages for that matter) by using the following technique from the shell:

.. sourcecode:: console    
   :caption: bash

    ~$ echo 'hello, this is a syslog message' > /dev/udp/192.168.0.2/514
    ~$ echo '{
      "host":"192.168.0.3",
      "version":"1.0",
      "facility":"testing",
      "short_message":"testing gelf message"
    }' > /dev/udp/192.168.0.2/12022

The IP after /dev/udp is the IP of the Graylog2 server. The number after the IP (514 or 12022) is the port, 514 for syslog, and 12022 for gelf. You can configure the ports in the graylog2 configuration file, which should be in :code:`/etc/graylog2.conf` if you followed the instructions on the installation wiki page as it is.

Client libraries for Graylog2
=============================

The Graylog2 team has provided client libraries for Ruby (`gelf-rb <https://github.com/Graylog2/gelf-rb>`_) and PHP (`gelf-php <https://github.com/Graylog2/gelf-php>`_). I have also created `ubelt-graylog2-logger <https://github.com/andho/ubelt-graylog2-logger>`_, a PHP implementation that can be used with `Zend_Log <http://framework.zend.com/manual/en/zend.log.html>`_.
