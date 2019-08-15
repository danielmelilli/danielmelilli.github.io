---
layout: post
title:  "Using Secure Secret in HASS"
date:   2019-07-22 00:02:44 -0400
categories: jekyll update
---


In most cases, HASS will be used programtically, and if you are following best practices, you probably
should be storing most of your configuration in Github.  

Most people, will add all of their configuration in
configuration.yaml including secrets and API keys.  You can see where this poses a security risk.

Home Assistant uses a file called secrets.yaml which lives in your Home Assistant configuration directory.

You will also want to make sure you add this file in your gitignore file.  This file specifies the secrets (Passwords, Keys, and more) that HASS uses to authenticate with different integrations

HASS has great documentation on how secrets.yaml works. 

https://www.home-assistant.io/docs/configuration/secrets/

Another alternative to using secrets.yaml, is to use Credstash and store them in AWS. Maybe I'll post about that another day!

