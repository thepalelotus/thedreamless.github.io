# Webserver Configuration with Apache2

This guide will cover the configuration of and Apache2 webserver on a centOS system. The centOS system itself is running in a container with it's own local network configured, as per the default setting when first running LXD.

The goal of this guide is show a minimal Apache setup to be used as a reference for troubleshooting common issues.

## Installing Apache2

To install Apache2 in a centOS system, you'll need to actually install the `httpd` package, as it is named in this system. You can easily check and make sure this package is not already installed, by checking `systemctl status httpd`, which should give you the following output:

```
[root@apache2 ~]# systemctl status httpd
Unit httpd.service could not be found.
```

At this point we can use `yum` to install the `httpd` package, and then use the same command from before to ensure it has been installed:

```
[root@apache2 ~]# yum install -y httpd
 <----snip---->
 
 Complete!
 
 [root@apache2 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd.service(8)
```

`httpd` has been installed and now the output of `systemctl status` shows us this service is available, but currently listed as `inactive (dead)`, which means the service is  not running, obviously.

## Starting the Apache Service

Now that `httpd` has been installed, we can start the service using the `start` parameter, and then use `enable` to make sure `httpd` starts on every reboot:

```
[root@apache2 ~]# systemctl start httpd
[root@apache2 ~]# systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[root@apache2 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-12-13 15:47:44 UTC; 1min 3s ago
     Docs: man:httpd.service(8)
 Main PID: 251 (httpd)
   Status: "Running, listening on: port 80"
    Tasks: 212 (limit: 37307)
   Memory: 20.8M
   CGroup: /system.slice/httpd.service
           ├─251 /usr/sbin/httpd -DFOREGROUND
           ├─252 /usr/sbin/httpd -DFOREGROUND
           ├─253 /usr/sbin/httpd -DFOREGROUND
           ├─254 /usr/sbin/httpd -DFOREGROUND
           └─255 /usr/sbin/httpd -DFOREGROUND

Dec 13 15:47:44 apache2 systemd[1]: Starting The Apache HTTP Server...
Dec 13 15:47:44 apache2 httpd[251]: AH00558: httpd: Could not reliably determine the server's fully qualified d
omain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Dec 13 15:47:44 apache2 systemd[1]: Started The Apache HTTP Server.
Dec 13 15:47:44 apache2 httpd[251]: Server configured, listening on: port 80

```

We can see from the output of `systemctl status httpd` that the service is now `Active: active (running) since Sun 2020-12-13 15:47:44 UTC; 1min 3s ago`. A few other useful commands that we will need later is `systemctl reload httpd`, which is used for reloading Apache after making changes to configuration files, so as to apply the changes.

### Restarting

If for any reason you need to restart the service, you can run the following command:

```
[root@apache2 ~]# systemctl restart httpd
```

As with the `start` command, no output being delivered to the terminal means it restarted, but you can always use `systemctl status httpd` to verify.

### Stopping and Disabling

You can stop the service using:

```
[root@apache2 ~]# systemctl stop httpd
```

And then if you need to stop the service from running on reboot, use the `disable` parameter:

```
[root@apache2 ~]# systemctl disable httpd
Removed /etc/systemd/system/multi-user.target.wants/httpd.service.
```

This will then show a state of `Active: inactive (dead)`, when viewing the status of the service.

## Configuring your Apache Server

The primary Apache configuration file is `/etc/httpd/conf/httpd.conf`. It contains a lot of configuration statements that don't need to be changed for a basic installation. In fact, only a few changes must be made to this file to get a basic website up and running. The file is very large so, rather than clutter this article with a lot of unnecessary stuff, I will show only those directives that you need to change.

### Listen

The first item to change is the Listen statement, which defines the IP address and port on which Apache is to listen for page requests. Right now, you just need to make this website available to the local machine, so use the localhost address. The line should look like this when you finish:

```
#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on specific IP addresses as shown below to
# prevent Apache from glomming onto all bound IP addresses.
#
#Listen 12.34.56.78:80
Listen 80
```

With this directive (`Listen 80`) set to the IP address of the localhost, Apache will listen only for connections from the local host. If you want the web server to listen for connections from remote hosts, you would use the host's external IP address.

### DocumentRoot

The DocumentRoot directive specifies the location of the HTML files that make up the pages of the website. That line does not need to be changed because it already points to the standard location. The line should look like this:

```
#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/var/www/html"
```

The Apache installation RPM creates the /var/www directory tree. If you wanted to change the location where the website files are stored, this configuration item is used to do that. For example, you might want to use a different name for the www subdirectory to make the identification of the website more explicit. That might look like this:

```
#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/var/mywebsite/html"
```

These are the only Apache configuration changes needed to create a simple website. For this little exercise, only one change was made to the httpd.conf file—the Listen directive. Everything else is already configured to produce a working web server.
