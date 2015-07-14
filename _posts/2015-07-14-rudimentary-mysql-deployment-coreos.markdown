---
layout: post
title: "Rudimentary MySQL deployment on CoreOS"
date: 2015-07-14 12:25:08
tags: coreos docker systemd systemctl journalctl
categories: docker coreos
comments: true
published: true
---

Continuing on from ["CoreOS on a single server"][coreos-intro] we'll be setting
up a persistent instance of MySQL for your single-server CoreOS (non-)cluster.

## Persisting after reboots
Whenever CoreOS receives an update, the `/usr` directory is swapped out for a
fresh copy. This means that any files you put here will be gone as soon as the
next update happens. The CoreOS developers have helpfully marked this as a
read-only partition. It should therefore be fairly difficult to accidentally
lose data between reboots.

{% highlight text %}
localhost / # touch /usr/write-test
touch: cannot touch '/usr/write-test': Read-only file system
{% endhighlight %}

For this example we'll be keeping our data in `/home/core/mysql-rubypx`. This
will survive the dual-partition switch and will be equivalently reliable as
storing mysql data on any other linux distribution.

## Creating the unit file
CoreOS uses [systemd][systemd] for system management. We'll be focusing on
learning how to use **systemctl** for managing services; and **journalctl** for
viewing their logs.

Systemd replaces "/etc/init.d/" scripts with [their own ".ini"-like
templates][systemd-unit-format]. Below is the service unit file we'll use to get
our MySQL container running. Use `sudo vi /etc/systemd/system/rubypx-db.service`
to create the following file:

{% highlight ini linenos=table %}
[Unit]
Description=Database
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker run \
  --name rubypx-db \
  -e MYSQL_ROOT_PASSWORD=<password> \
  -v /home/core/mysql-rubypx/:/var/lib/mysql \
  -p 172.17.42.1:3306:3306 \
  mysql
ExecStop=/usr/bin/docker rm -f rubypx-db

[Install]
WantedBy=basic.target
{% endhighlight %}

We assign a docker short name of "rubypx-db" so that we can easily stop the
container in the "ExecStop" step (otherwise we'd have to keep track of the
container ID). The "MYSQL_ROOT_PASSWORD" is set here, as the instance will only
ever be accessible from inside the server anyway. The container is set to bind
to `172.17.42.1`, this is the IP of the "docker0" internal interface, and allows
us to prevent the container exposing an external port on the host.

We'll need to create the directory for mysql to mount as it's data volume, do so
with: `mkdir /home/core/mysql-rubypx`.

## Starting the service
First of all we'll need to enable our unit in systemd: `sudo systemctl enable
rubypx-db`. This will now ensure that the service loads once all of it's
dependencies have been satisfied.

The "WantedBy" field allows us to select which dependencies our service has. The
"basic.target" will load before the GUI (if applicable), and after all of the
basic subsystems (networking, filesystem).

Since we want to start the unit immediately and test it out, we can run `sudo
systemctl start rubypx-db`.

## Testing the service
Now that the unit has been started we can use `sudo journalctl -u rubypx-db` to
view the logs (the "-f" flag can be used to tail the log).

We can also use `sudo systemctl status rubypx-db` to display the status of the
unit:

{% highlight text %}
core@localhost ~ $ sudo systemctl status rubypx-db
● rubypx-db.service - Database
   Loaded: loaded (/etc/systemd/system/rubypx-db.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2015-07-10 21:51:20 UTC; 3 days ago
 Main PID: 620 (docker)
   Memory: 5.9M
      CPU: 2.242s
   CGroup: /system.slice/rubypx-db.service
           └─620 /usr/bin/docker run --name rubypx-db -e MYSQL_ROOT_PASSWORD=<password> -v /home/core/mysql-rubypx/:/var/lib/mysql -p 172.17.42.1:3306:3306 mysql
Jul 10 21:51:35 localhost docker[620]: 2015-07-10 21:51:35 1 [Note] InnoDB: Waiting for purge to start
...
{% endhighlight %}

We can test out our new server by starting the MySQL client in a temporary
container using the following command:

{% highlight bash %}
docker run -it --rm mysql \
  sh -c 'exec mysql \
    -h172.17.42.1 \
    -P3306 \
    -uroot \
    -p<password>'
{% endhighlight %}

We should also be able to be able to connect to our mysql server manually by
using an ssh tunnel, and then connecting to the "docker0" interface ip. Now that
we've got our MySQL database set up, we're ready to deploy our application, I'll
cover this in my next post!

[coreos-intro]: {% post_url 2015-07-01-coreos-single-server %} "CoreOS on a single server"
[pets-vs-cattle]: https://blog.engineyard.com/2014/pets-vs-cattle "Pets vs. Cattle"
[systemd]: http://www.freedesktop.org/wiki/Software/systemd/ "systemd"
[systemd-unit-format]: http://www.freedesktop.org/software/systemd/man/systemd.service.html "Systemd unit format"
