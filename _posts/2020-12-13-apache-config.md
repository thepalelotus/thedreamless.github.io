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

The DocumentRoot directive specifies the location of the HTML files that make up the pages of the website. The line should look like this:

```
#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/var/www/html"
```

If you wanted to change the location where the website files are stored, this configuration item is used to do that. For example, you might want to use a different name for the www subdirectory to make the identification of the website more explicit:

```
DocumentRoot "/var/mywebsite/html"
```

For example, the server might receive a request for the following document:

```
http://mywebsite.xyz/fnord.html
```

The server will look for the following file in the directory specified using the `DocumentRoot` directive: 

```
/var/mywebsite/html/fnord.html
```

Or, if the default parameter is used:

```
/var/www/html/fnord.html
```

### ServerRoot

The ServerRoot directive sets the directory in which the server lives. Typically it will contain the subdirectories conf/ and logs/:

```
ServerRoot "/etc/httpd"
```

The contents of this directory should look like this (which should be obvious since `httpd.conf` is located in `/etc/httpd/conf`):

```
[root@apache2 conf]# ls -la /etc/httpd
total 12
drwxr-xr-x  5 root root   9 Dec 13 18:53 .
drwxr-xr-x 55 root root 136 Dec 13 18:54 ..
drwxr-xr-x  2 root root   5 Dec 20 13:48 conf
drwxr-xr-x  2 root root   6 Dec 13 18:53 conf.d
drwxr-xr-x  2 root root  13 Dec 13 18:53 conf.modules.d
lrwxrwxrwx  1 root root  19 Nov  4 03:21 logs -> ../../var/log/httpd
lrwxrwxrwx  1 root root  29 Nov  4 03:21 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx  1 root root  10 Nov  4 03:21 run -> /run/httpd
lrwxrwxrwx  1 root root  19 Nov  4 03:21 state -> ../../var/lib/httpd
```

### ServerAdmin

The `ServerAdmin` sets the contact address that the server includes in any error messages it returns to the client. If the httpd doesn't recognize the supplied argument as an URL, it assumes, that it's an email-address and prepends it with mailto: in hyperlink targets. It's recommended to use an email address. If you want to use an URL, it should point to another server under your control. Otherwise users may not be able to contact you in case of errors or other issues related to your server:

```
ServerAdmin root@mywebsite.xyz
```

### User

The `User` directive sets the userid used by the server to answer requests. `User`'s setting determines the server's access. Any files inaccessible to this `user` will also be inaccessible to your website's visitors. The default for `User` is `apache`. 

It is important that the `user` should only have the privleges so as to access files which are intended to be visibile to the outside world. The `User` should **not** be allowed to execute any code which is not intended to be in response to HTTP requests. For security reasons, AThe User should not be allowed to execute any code which is not intended to be in response to HTTP requests. Apache does not allow the `User` to be assigned to `root`, as this will create vulnerabilities in the server.

The parent `httpd` process first runs as `root` during normal operations but is then immediately handed off to the `apache user`. The server must start as `root` because it needs to bind to a port below 1024 (80 for HTTP and 443 for HTTPS). Ports below 1024 are reserved for system use, so they can not be used by anyone but root. Once the server has attached itself to its port, however, it hands the process off to the User before it accepts any connection requests.

### Group

The `Group` directive is similar to the `User`. The `Group` sets the `group` under which the server will answer requests. The default `Group` is `apache`. 


### ErrorLog

`ErrorLog` names the file where server errors are logged. As this directive indicates, the error log file for your Web server is `/var/log/httpd/error_log`:

```
ErrorLog "logs/error_log"
```

#### LogLevel

`LogLevel` sets how verbose the error messages in the error logs will be. `LogLevel` can be set (from least verbose to most verbose) to `emerg`, `alert`, `crit`, `error`, `warn`, `notice`, `info` or `debug`. The default `LogLevel` is `warn`. 

```
LogLevel warn
```

### CustomLog (Access Log)

The server access log records all requests processed by the server. The location and content of the access log are controlled by the CustomLog directive. The LogFormat directive can be used to simplify the selection of the contents of the logs:

A typical configuration for the access log might look as follows.

```
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
CustomLog "logs/access_log" combined

```

This defines the nickname `common` and associates it with a particular log format string. The format string consists of percent directives, each of which tell the server to log a particular piece of information. Literal characters may also be placed in the format string and will be copied directly into the log output. The quote character (") must be escaped by placing a backslash before it to prevent it from being interpreted as the end of the format string. The format string may also contain the special control characters "\n" for new-line and "\t" for tab.

The above configuration will write log entries in a format known as the Common Log Format (CLF). This standard format can be produced by many different web servers and read by many log analysis programs. The log file entries produced in CLF will look something like this:

```
127.0.0.1 - - [20/Dec/2020:13:50:58 +0000] "GET / HTTP/1.1" 200 67 "-" "curl/7.61.1"
10.126.85.1 - - [20/Dec/2020:14:40:12 +0000] "GET / HTTP/1.0" 200 67 "-" "Lynx/2.9.0dev.5 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.6.13"
```

In the two examples above I pulled the main index.html using `curl` and `lynx`. In the following section I will break down what each line means in comparison to the parameters set in the default `LogFormat`:

- **127.0.0.1** or **10.126.85.1** (`%h`)
	- This is the IP address of the client (remote host) which made the request to the server. The IP address reported here is not necessarily the address of the machine at which the user is sitting. If a proxy server exists between the user and the server, this address will be the address of the proxy, rather than the originating machine.

- **-** (`%l`)
	- The "hyphen" in the output indicates that the requested piece of information is not available. In this case, the information that is not available is the RFC 1413 identity of the client determined by `identd` on the clients machine.

- **-** (`%u`)
	- Another hyphen indicates that the requested information is not available. This is the userid of the person requesting the document as determined by HTTP authentication. If the document is not password protected (it is not), this part will be "-" just like the previous one.

- **[20/Dec/2020:14:40:12 +0000]** (`%t`)
	- The time that the request was received.
  
- **"GET / HTTP/1.0"** (`\"%r\"`)
	- The request line from the client is given in double quotes. The request line contains a great deal of useful information. First, the method used by the client is GET. Second, the client requested the resource `/` or the DocumentRoot, upon which will pull the index.html unless otherwise specified, and third, the client used the protocol HTTP/1.0 and HTTP/1.1 for the curl request.

- **200** (`%>s`)
	- This is the status code that the server sends back to the client. The HTTP 200 OK success status response code indicates that the request has succeeded.
  
- **67** (`%b`)
	- The last part indicates the size of the object returned to the client, not including the response headers. If no content was returned to the client, this value will be "-". To log "0" for no content, use %B instead.

- **-** (`\"%{Referer}i\"`)
	 - The "Referer" (sic) HTTP request header. This gives the site that the client reports having been referred from. It shows a `-` because I went directory to this URL, and therefore there was no referral.
   
- **"Lynx/2.9.0dev.5 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.6.13"** (`\"%{User-Agent}i\"`)
	- The User-Agent HTTP request header. This is the identifying information that the client browser reports about itself.

### Multiple Access Logs

Multiple access logs can be created simply by specifying multiple CustomLog directives in the configuration file. For example, the following directives will create three access logs:

```
LogFormat "%h %l %u %t \"%r\" %>s %b" common
CustomLog logs/access_log common
CustomLog logs/referer_log "%{Referer}i -> %U"
CustomLog logs/agent_log "%{User-agent}i"
```

The first contains the basic CLF information, while the second and third contain referer and browser information. The last two CustomLog lines show how to mimic the effects of the ReferLog and AgentLog directives.
