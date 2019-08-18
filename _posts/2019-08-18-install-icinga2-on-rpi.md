---
layout: post
title:  "Installing Icinga2 on A Raspberry Pi"
date:   2019-08-18 00:02:44 -0400
categories: jekyll update
---



Installing Icinga on Raspberry Pi

You can configure Icinga as an agent for your existing icinga system, or as a standalone
monitoring system

Why use Icinga?


Well, firstly it's Open Source. If you have used Nagios before, you will
see that a lot of things are similair (since it's based on Nagios).  And
if you have used Nagios, you will also know how rich Nagios' plugin 
community is.  There is just about a plugin for anything, and a good amount
of those work with Icinga.


It's fast!  Even running on my Raspberry Pi 3 with a large amount of checks, it's extremely quick.

The user interface is extremely clean and responsive.  I usually use my phone 
to keep an eye on things, and it's always been a positive experience.


These are instructions to install Icinga on Raspbian flavor of OS.


First you will want to add the key

```
curl https://packages.icinga.com/icinga.key | sudo apt-key add -
echo "deb http://packages.icinga.com/raspbian icinga-stretch main" \
  | sudo tee /etc/apt/sources.list.d/icinga.list
```

After that is complete, you will want to run an update

```
sudo apt update
```

The packages that you will require are icinga2 and icingaweb2

```
sudo apt install icinga2 icingaweb2
```

It should only take a few minutes

After it's finished installing, check to see if the icinga2 service is running

```
 sudo systemctl status icinga2
 ```

 You should see something like this

 ```
 ● icinga2.service - Icinga host/service/network monitoring system
   Loaded: loaded (/lib/systemd/system/icinga2.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/icinga2.service.d
           └─limits.conf
   Active: active (running) since Sat 2019-08-17 16:28:51 EDT; 4min 40s ago
 Main PID: 13015 (icinga2)
   CGroup: /system.slice/icinga2.service
           ├─13015 /usr/lib/arm-linux-gnueabihf/icinga2/sbin/icinga2 --no-stack-rlimit daemon --close-stdio -e /
           └─13054 /usr/lib/arm-linux-gnueabihf/icinga2/sbin/icinga2 --no-stack-rlimit daemon --close-stdio -e /

Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Instantiated 1 U
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Instantiated 215
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Instantiated 1 U
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Instantiated 3 S
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Instantiated 3 T
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ScriptGlobal: Dumping variab
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Triggering Start
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/ConfigItem: Activated all ob
Aug 17 16:28:51 raspberrypi icinga2[13015]: [2019-08-17 16:28:51 -0400] information/cli: Closing console log.
Aug 17 16:28:51 raspberrypi systemd[1]: Started Icinga host/service/network monitoring system.
```

You will need plugins otherwise icinga will not know how to perform checks. You can install those by 

```
apt-get install monitoring-plugins
```

Next, enable the icinga2 service to run on system startup
```
sudo systemctl enable icinga2
```

Setting up Icinga Web 2 
Icinga 2 can be used with Icinga Web 2 and a variety of modules. This  explains how to set up Icinga Web 2.

The DB IDO (Database Icinga Data Output) feature for Icinga 2 take care of exporting all configuration and status information into a database.

For this installation, I will use mysql instead of Postgres

```
apt-get install mysql-server mysql-client
```

Next run the My SQL installation
```
mysql_secure_installation
```

Install the icinga2-ido-mysql packages
```
apt-get install icinga2-ido-mysql
```


Setting up the MySQL database ¶
Set up a MySQL database for Icinga 2:


# mysql -u root -p

CREATE DATABASE icinga;
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'localhost' IDENTIFIED BY 'icinga';
quit




After creating the database you can import the Icinga 2 IDO schema usin
g the following command. Enter the root password into the prompt when asked.


mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/mysql.sql


Enabling the IDO MySQL module ¶
The package provides a new configuration file that is installed in /etc/icinga2/features-available/ido-mysql.conf. You can update the database credentials in this file.

All available attributes are explained in the IdoMysqlConnection object chapter.

You can enable the ido-mysql feature configuration file using icinga2 feature enable:


# icinga2 feature enable ido-mysql
Module 'ido-mysql' was enabled.
Make sure to restart Icinga 2 for these changes to take effect.
Restart Icinga 2.


systemctl restart icinga2

Webserver ¶
The preferred way of installing Icinga Web 2 is to use Apache as webserver in combination with PHP-FPM. If you prefer Nginx, please refer to the Icinga Web 2 documentation.

Debian/Ubuntu:


apt-get install apache2



Enable port 80 (http). Best practice is to only enable port 443 (https) and use TLS certificates.

iptables:
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
service iptables save

Setting Up Icinga 2 REST API ¶
Icinga Web 2 and other web interfaces require the REST API to send actions (reschedule check, etc.) and query object details.

You can run the CLI command icinga2 api setup to enable the api feature and set up certificates as well as a new API user root with an auto-generated password in the /etc/icinga2/conf.d/api-users.conf configuration file:

icinga2 api setup
Edit the api-users.conf file and add a new ApiUser object. Specify the permissions attribute with minimal permissions required by Icinga Web 2.


vim /etc/icinga2/conf.d/api-users.conf

object ApiUser "icingaweb2" {
  password = "Wijsn8Z9eRs5E25d"
  permissions = [ "status/query", "actions/*", "objects/modify/*", "objects/query/*" ]
}
Restart Icinga 2 to activate the configuration.


systemctl restart icinga2


After you have installed Icinga.  The next step is to get Icinga Web 2
setup and configured.  Please pay special attention to these steps as they
could be a little tricky.  


Installing Icinga Web 2 ¶
You can install Icinga Web 2 by using your distribution’s package manager to install the icingaweb2 package. Below is a list with examples for various distributions. The additional package icingacli is necessary to follow further steps in this guide. The additional package libapache2-mod-php is necessary on Ubuntu to make Icinga Web 2 working out-of-the-box if you aren’t sure or don’t care about PHP FPM.

Debian:

```
apt-get install icingaweb2 libapache2-mod-php icingacli
```


Preparing Web Setup 

You can set up Icinga Web 2 quickly and easily with the Icinga Web 2 setup wizard which is available the first time you visit Icinga Web 2 in your browser. When using the web setup you are required to authenticate using a token. In order to generate a token use the icingacli:

```
icingacli setup token create
```

In case you do not remember the token you can show it using the icingacli:

```
icingacli setup token show
```

Preparing Web Setup on Debian 
On Debian, you need to manually create a database and a database user prior to starting the web wizard. This is due to local security restrictions whereas the web wizard cannot create a database/user through a local unix domain socket.

```
MariaDB [mysql]> CREATE DATABASE icingaweb2;

MariaDB [mysql]> GRANT ALL ON icingaweb2.* TO icingaweb2@localhost IDENTIFIED BY 'CHANGEME';
```

You may also create a separate administrative account with all privileges instead.

Note: This is only required if you are using a local database as authentication type.

Starting Web Setup 

Finally visit Icinga Web 2 in your browser to access the setup wizard and complete the installation: /icingaweb2/setup.

Note for Debian

Use the same database, user and password details created above when asked.

The setup wizard automatically detects the required packages. In case one of them is missing, e.g. a PHP module, please install the package, restart your webserver and reload the setup page.

If you have SELinux enabled, please ensure to either have the selinux package for Icinga Web 2 installed, or disable it.