---
author: "Joe Miller"
tags:
  - software
  - howto
  - linux
  - centos
comments: true
date: 2011-04-13 19:03:32 -0700
date_gmt: 2011-04-14 02:03:32 -0700
draft: true
title: 'HOWTO: install graylog2 on CentOS 5 with RVM + Passenger'
---

Graylog2 is an open-source self-hosted centralized log management tool. Think of it as a do-it-yourself version of loggly.com, or perhaps a simpler alternative to Splunk. Logs are stored in a MongoDB database.

<!--more-->

I won't go into too much detail, so if you want more info check out [graylog2.org](|http://www.graylog2.org/)

Graylog2 consists of two components: **graylog2-server** , a Java process which receives logs and writes them to a MongoDB database. **graylog2-web-interface** , a Ruby on Rails app that seems happiest with modern versions of Ruby and Rails. CentOS 5, however, ships with an old version of Ruby (1.8.x) which did not play nicely with all of the gems that graylog2 wanted.

If the server is going to be dedicated to only running graylog2, we could probably overwrite the installed Ruby with a newer version. However, the server we wanted to install it on was already running a complex Rails app under mod\_passenger which we did not want to risk breaking.

I decided to see if [RVM - Ruby Version Manager](https://rvm.beginrescueend.com/) - would allow me to setup an isolated Ruby environment just for graylog2 and not disturb the other Ruby apps on the machine. I also wanted to setup an isolated instance of Passenger-standalone for graylog2 then configure apache to listen on port 80 and forwarding requests with mod\_proxy.

Everything worked even better than I expected. In the Ruby world, with so much rapid change, it can be challenging to put together a compatible set of gems for multiple apps running on the same machine. RVM worked so well I will consider using it to provide isolated ruby environments for other apps in the future. I love RVM!

Here is how I put it all together:

Overview:

- Install graylog2-server in /opt/graylog2-server
- Install graylog2-web-interface in /opt/graylog2-web-interface
- graylog2-web-interface runs as unprivileged ‘graylog’ user
- RVM installed to provide a Ruby 1.9.2 and Rails 3.x environment for graylog2-web-interface
- Passenger-standalone installed in the Ruby 1.9.2 environment, running graylog2-web-interface on http://127.0.0.1:3001
- Apache listens on tcp/80 and forwards to Passenger-standalone with mod\_proxy

### MongoDB install

1. install mongodb from [EPEL](http://fedoraproject.org/wiki/EPEL)yum repo
{{< highlight text >}}
# yum install mongodb mongodb-server
# mkdir /var/lib/mongodb
# chown -R mongodb:mongodb /var/lib/mongodb/
{{< / highlight >}}

2. enable mongod to start on boot
{{< highlight text >}}
# chkconfig mongod on
{{< / highlight >}}

3. start mongod (logfile: /var/log/mongodb/mongodb.log)
{{< highlight text >}}
# service mongod start
{{< / highlight >}}

4. run mongo, create an admin user in the admin collection and and a graylog user in a new graylog2 collection
{{< highlight text >}}
# mongo
     use admin
     db.addUser('admin', 'admin-mongo-passwd')
     db.auth('admin', 'admin-mongo-passwd')
     use graylog2
     db.addUser('grayloguser', 'grayloguser-mongo-passwd')
{{< / highlight >}}

### graylog2-server install

based on: [https://github.com/Graylog2/graylog2-server/wiki/Installing](https://github.com/Graylog2/graylog2-server/wiki/Installing)

1. download latest graylog2-server tarball: [https://github.com/Graylog2/graylog2-server/downloads](https://github.com/Graylog2/graylog2-server/downloads)
2. Install into /opt/graylog2-server:
{{< highlight text >}}
# cd /opt
# tar xfvz ~/graylog2-server-0.9.5p1.tar.gz
# ln -s /opt/graylog2-server-0.9.5p1 /opt/graylog2-server
{{< / highlight >}}

3. configure /etc/graylog2.conf
{{< highlight text >}}
# cp graylog2.conf.example /etc/graylog2.conf


edit /etc/graylog2.conf, set monogdb_user, mongodb_password
{{< / highlight >}}

4. create /etc/init.d/graylog2-server
{{< highlight bash >}}
#!/bin/sh
#
# graylog2-server: graylog2 message collector
#
# chkconfig: - 98 02
# description: This daemon listens for syslog and GELF messages and stores them in mongodb
#


CMD=$1
NOHUP=`which nohup`
JAVA_HOME=/usr/java/latest
JAVA_CMD=$JAVA_HOME/bin/java


GRAYLOG2_SERVER_HOME=/opt/graylog2-server


start() {
    echo "Starting graylog2-server ..."
    $NOHUP $JAVA_CMD -jar $GRAYLOG2_SERVER_HOME/graylog2-server.jar > /var/log/graylog2.log 2>&1 &
}


stop() {
        PID=`cat /tmp/graylog2.pid`
    echo "Stopping graylog2-server ($PID) ..."
        kill $PID
}


restart() {
    echo "Restarting graylog2-server ..."
        stop
        start
}


case "$CMD" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    *)
        echo "Usage $0 {start|stop|restart}"
        RETVAL=1
esac
{{< / highlight >}}

5. create /etc/logrotate.d/graylog2-server
{{< highlight text >}}
/var/log/graylog2.log {
       daily
       rotate 90
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
}
{{< / highlight >}}

6. register graylog2-server init script
{{< highlight text >}}
# chmod +x /etc/init.d/graylog2-server
# chkconfig --add graylog2-server
# chkconfig graylog2-server on
{{< / highlight >}}

7. start graylog2-server
{{< highlight text >}}
# service graylog2-server start
# ps aux | grep graylog2
    root 23197 2.3 0.1 1196204 18428 pts/0 Sl 12:37 0:00 java -jar ../graylog2-server.jar
{{< / highlight >}}

### graylog2-web-interface install

based on: [https://github.com/Graylog2/graylog2-web-interface/wiki](https://github.com/Graylog2/graylog2-web-interface/wiki)

1. Install RVM and prepare a new Ruby 1.9.2 environment
{{< highlight text >}}
# bash
{{< / highlight >}}

2. create ‘graylog’ user
{{< highlight text >}}
# adduser -m graylog
{{< / highlight >}}

3. download latest graylog2-web-interface tarball: [https://github.com/Graylog2/graylog2-web-interface/downloads](https://github.com/Graylog2/graylog2-web-interface/downloads)
{{< highlight text >}}
# mkdir /opt
# cd /opt
# tar xfvz ~/graylog2-web-interface-0.9.5.tar.gz
# ln -s /opt/graylog2-web-interface-0.9.5 /opt/graylog2-web-interface
# chown -R graylog /opt/graylog2-web-interface-0.9.5
{{< / highlight >}}

4. Install gems necessary to run graylog2 using Bundler in the ruby 1.9.2 environment
{{< highlight text >}}
# rvm use 1.9.2
# cd /opt/graylog2-web-interface
# gem install bundler
# bundle install
{{< / highlight >}}

5. Configure graylog2-web-interface
  - Edit the various config files in /opt/graylog2-web-interface to suit your environment:
    - email.yml
    - general.yml
    - mongoid.yml (make sure to use the user/pass you setup earlier)

6. Start graylog2-web-interface, create first user
{{< highlight text >}}
# rvm use 1.9.2
# cd /opt/graylog2-web-interface
# RAILS_ENV=production script/rails server
{{< / highlight >}}

Connect to http://127.0.0.1:3000. If everything is working, graylog2 will ask you to create the first user. Shutdown graylog2 (ctrl-C) after you create the first user.

7. Install and configure passenger-standalone
{{< highlight text >}}
# yum -y install curl-devel
# rvm use 1.9.2
# gem install passenger
# gem install file-tail


# cd /opt/graylog2-web-interface
# mkdir tmp log
# chmod -R 777 tmp log
# passenger start
{{< / highlight >}}

Passenger should download, compile, and build everything it needs. If passenger tries to serve requests, ctrl-C to quit.

8. Create /etc/init.d/graylog2-web-interface
{{< highlight bash >}}
#!/bin/bash
#
# graylog2-web-interface: graylog2 web interface
#
# chkconfig: - 98 02
# description: Starts graylog2-web-interface using passenger-standalone. \
# Uses RVM to use switch to a specific ruby version.
#


# config
USER=graylog
APP_DIR=/opt/graylog2-web-interface
RVM_RUBY=1.9.2
ADDR=127.0.0.1
PORT=3000
ENVIRONMENT=production
LOG_FILE=/var/log/graylog2-web-interface.log


# --


CMD_START="cd $APP_DIR; rvm use $RVM_RUBY; passenger start -d \
                    -a $ADDR \
                    -p $PORT \
                    -e $ENVIRONMENT \
                    --user $USER"
CMD_STOP="cd $APP_DIR; rvm use $RVM_RUBY; passenger stop -p $PORT"


CMD_STATUS="cd $APP_DIR; rvm use $RVM_RUBY; passenger status -p $PORT"


. /lib/lsb/init-functions
case "$1" in
  start)
    echo "Starting graylog2-web-interface"
    su - $USER -c "$CMD_START"
    ;;
  stop)
    echo "Stopping graylog2-web-interface"
    su - $USER -c "$CMD_STOP"
    ;;
  status)
   su - $USER -c "$CMD_STATUS"
   ;;
  *)
    echo "Usage: $0 start|stop|status" >&2
    exit 3
    ;;
esac
{{< / highlight >}}

9. Create /etc/logrotate.d/graylog2-web-interface
{{< highlight bash >}}
/opt/graylog2-web-interface/log/*log
       size=256M
       rotate 90
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
}
{{< / highlight >}}

10. Register graylog2-web-interface service
{{< highlight text >}}
# chmod +x /etc/init.d/graylog2-web-interface
# chkconfig --add graylog2-web-interface
# chkconfig graylog2-web-interface on
{{< / highlight >}}

11. Start graylog2-web-interface
{{< highlight text >}}
# service graylog2-web-interface start
{{< / highlight >}}

12. Configure apache to forward requests to passenger

    1. Create /etc/httpd/conf.d/graylog2.conf

{{< highlight text >}}
NameVirtualHost *:80


	ServerName log.mydomain.com
	ServerAlias graylog2.mydomain.com
	ProxyPreserveHost On
	ProxyPass / http://127.0.0.1:3000/
	ProxyPassReverse / http://127.0.0.1:3000/


        CustomLog /var/log/httpd/log.mydomain.com-access_log common
{{< / highlight >}}

  1. Restart apache
{{< highlight text >}}
# service httpd configtest
# service httpd restart
{{< / highlight >}}

### Startup scripts for running multiple Passenger-standalone instances:

If you need a method for setting up multiple Passenger-standalone Rails apps, checkout these scripts. They’re very well done: [http://moeffju.net/blog/passenger-standalone-initscript](http://moeffju.net/blog/passenger-standalone-initscript)

### Alternatives to Apache

rippleAdder posted example configs on the graylog2 mailing list for running graylog2-web-interface with other webservers than Apache, namely unicorn and nginx. These are likely going to be faster than apache, although I haven't benchmarked any webservers with graylog2 yet. You can find his sample configs [here](https://groups.google.com/group/graylog2/browse_thread/thread/9768347210f3863d/aea05b5433656a8d?lnk=gst&q=unicorn#aea05b5433656a8d)
