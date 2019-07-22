In most cases, HASS will be used programtically, and if you are following best practices, you probably
should be storing most of these in Github.  Most people, will add all of their configuration in
configuration.yaml including secrets and API keys.  You can see where this poses a security risk.

Home Assistant uses a file called secrets.yaml which lives in your Home Assistant configuration directory.

You will also want to make sure you add this file in your gitignore file.  This file specifies 
