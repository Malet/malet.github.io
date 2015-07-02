---
layout: post
title: "CoreOS on a single server"
date: 2015-07-01 23:57:08
tags: coreos docker systemd
categories: docker coreos
---
[CoreOS][coreos] brings some very exciting ideas to the world of Linux, such as
the very nice auto-update features from ChromeOS; it's also very minimalist,
it comes with just a few packages installed (the only way to "install" new
packages is to run them within a docker container).

Unfortunately this also means that when you're first getting started, (and when
you need the comfort of familiar tools the most!), you'll not immediately be
able to find any useful debugging tools. CoreOS's solution to this is
`toolbox`. [Toolbox][toolbox] downloads the `core-fedora-latest` image which
comes with a few more commands such as `telnet`, `dig`; it also allows you to
install any additional tools you need with the `yum` package manager.

The main motivation I had for setting up a single-server CoreOS environment was
to learn about the system, without immediately being overwhelmed by bringing
`fleet` and `etcd` into the equation.

**CoreOS is not designed to be used in a single server configuration**, it's far
more at home in a cluster. It's also far easier to configure if you're application is
essentially stateless (or has it's state is delegated to another server).

The application I wanted to deploy was written in Ruby on Rails, and required
persistent storage for image uploads, and well as requiring a MySQL database.
This definitely doesn't fit the description of what the OS was designed for, so
please take this as an educational process, not a tutorial for your
business-critical systems.

<!-- *This guide assumes that you already have some understanding of docker, and that
you also have a dockerised application which you're hoping to deploy.* -->

If you've not already got a server with CoreOS installed, please install it to
disk using the [instructions on the CoreOS site][coreos-install]. *Please note:
this will completely wipe the target disk and replace its contents with a CoreOS
installation!*.

You'll also want to make sure your dockerised application is in a docker
registry somewhere, I'd recommend using [Docker Hub][dockerhub] - authentication
is a total pain if you're self-hosting a registry (Basic HTTP auth is the
simplest option, but even that requires routing through another nginx container).

The most obvious place to start is to find out how we can do persistant data
storage on a CoreOS instance. The MySQL container will require a docker volumes
to be set up and attached at runtime. We'll also need to find out if there are
any parts of the system which aren't completely wiped between upgrades.

I'll be walking through the setup of the dockerised MySQL service in my next post.

[coreos]: https://coreos.com/ "CoreOS"
[toolbox]: https://coreos.com/docs/cluster-management/debugging/install-debugging-tools/#quick-debugging "CoreOS toolbox"
[coreos-install]: https://coreos.com/docs/running-coreos/bare-metal/installing-to-disk/ "Installing CoreOS to disk"
[dockerhub]: https://hub.docker.com/ "Docker's official registry service"
