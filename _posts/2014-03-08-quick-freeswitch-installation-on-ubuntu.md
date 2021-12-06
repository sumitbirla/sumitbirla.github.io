---
id: 587
title: Quick FreeSWITCH Installation on Ubuntu
date: 2014-03-08T09:13:19-05:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=587
permalink: /2014/03/quick-freeswitch-installation-on-ubuntu/
image: /wp-content/uploads/2014/03/freeswtich-logo.png
categories:
  - FreeSWITCH
  - Linux
tags:
  - freeswitch
  - linux
  - voip
---
<a href="http://sumit.tampahost.net/2014/03/quick-freeswitch-installation-on-ubuntu/freeswtich-logo/" rel="attachment wp-att-608"><img src="http://sumit.tampahost.net/wp-content/uploads/2014/03/freeswtich-logo.png" alt="" title="FreeSWITCH Logo" width="297" height="75" class="aligncenter size-full wp-image-608" /></a>

The following steps for installing FreeSWITCH 1.2.stable on a fresh installation of Ubuntu 12.04.4 LTS. The idea is to document all the necessary packages, config files etc. to get going quickly. This includes ODBC/MySQL libraries which I almost always use with FreeSWITCH.

The following packages need to be installed before FreeSWITCH will compile:

  * build-essential
  * autoconf
  * git
  * unixodbc-dev
  * libmyodbc
  * zlib1g-dev
  * libjpeg-dev
  * libncurses5-dev
  * libssl-dev

<pre>% sudo apt-get install build-essential autoconf git unixodbc-dev libmyodbc 
zlib1g-dev libjpeg-dev libncurses5-dev libssl-dev</pre>

Next get the source code from FreeSWITCH repository and configure it:

<pre>% cd /usr/local/src
  % git clone -b v1.2.stable git://git.freeswitch.org/freeswitch.git
  % cd freeswitch
  % ./bootstrap.sh
  % ./configure --enable-core-odbc-support</pre>

Once above is done, a new file called module.conf will be created in /usr/local/src/freeswitch. Edit this file and uncomment the modules you need, for example:

<pre>applications/mod_blacklist
  applications/mod_callcenter
  applications/mod_curl
  applications/mod_spy
  applications/mod_directory
  applications/mod_distributor
  formats/mod_shout
</pre>

Now compile and install FreeSWITCH, sample sounds and music-on-hold.

<pre>% make install sounds-install moh-install</pre>

Create a user name \`freeswitch\`

<pre>% useradd freeswitch</pre>

Change permissions to allow service to run as user \`freeswitch\`:

<pre>% chown -R freeswitch:freeswitch /usr/local/freeswitch
</pre>

Add bin directory to path:

<pre>% vi /etc/profile
  % export PATH=$PATH:/usr/local/freeswitch/bin
</pre>

&nbsp;

### Set up ODBC

Create /etc/odbc.ini

<pre>[freeswitch]
Driver          = /usr/lib/x86_64-linux-gnu/odbc/libmyodbc.so
SERVER          = mydbserver.com
PORT            = 3306
DATABASE        = freeswitch
OPTION          = 67108864
USER            = freeuser
PASSWORD        = freepass
</pre>

&#8230; and /etc/odbcinst.ini

<pre>[MySQL]
Description     = MySQL driver
Driver          = /usr/lib/x86_64-linux-gnu/odbc/libmyodbc.so
Setup           = /usr/lib/x86_64-linux-gnu/odbc/libodbcmyS.so
UsageCount      = 1
FileUsage       = 1
Threading       = 0
</pre>

Test odbc installation my executing \`isql freeswitch\` on the command line.

&nbsp;

### Create init script

Create /etc/init.d/freeswitch and put the following contents in it:

<pre>#!/bin/sh
### -*- mode:shell-script; indent-tabs-mode:nil; sh-basic-offset:2 -*-
### BEGIN INIT INFO
# Provides: freeswitch
# Required-Start: $network $remote_fs $local_fs
# Required-Stop: $network $remote_fs $local_fs
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: FreeSWITCH Softswitch
# Description: FreeSWITCH Softswitch
### END INIT INFO

# Author: Travis Cross 

PATH=/sbin:/usr/sbin:/bin:/usr/bin
DESC=freeswitch
NAME=freeswitch
DAEMON=/usr/local/freeswitch/bin/freeswitch
DAEMON_ARGS="-u freeswitch -g freeswitch -rp -nc -nonat"
USER=freeswitch
GROUP=freeswitch
RUNDIR=/var/run/$NAME
PIDFILE=/usr/local/freeswitch/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
WORKDIR=/usr/local/freeswitch/lib/

[ -x $DAEMON ] || exit 0
[ -r /etc/default/$NAME ] && . /etc/default/$NAME
. /lib/init/vars.sh
. /lib/lsb/init-functions

do_start() {
  # Directory in /var/run may disappear on reboot (e.g. when tmpfs used for /var/run).
  mkdir -p $RUNDIR
  chown -R $USER:$GROUP $RUNDIR
  chmod -R ug=rwX,o= $RUNDIR

  start-stop-daemon --start --quiet \
    --pidfile $PIDFILE --exec $DAEMON --name $NAME --user $USER \
    --test &gt; /dev/null \
    || return 1
  ulimit -s 240
  start-stop-daemon --start --quiet \
    --pidfile $PIDFILE --exec $DAEMON --name $NAME --user $USER \
    --chdir $WORKDIR -- $DAEMON_ARGS $DAEMON_OPTS \
    || return 2
  return 0
}

stop_fs() {
  start-stop-daemon --stop --quiet \
    --pidfile $PIDFILE --name $NAME --user $USER \
    --retry=TERM/30/KILL/5
}

stop_fs_children() {
  start-stop-daemon --stop --quiet \
    --exec $DAEMON \
    --oknodo --retry=0/30/KILL/5
}

do_stop() {
  stop_fs
  RETVAL="$?"
  [ "$RETVAL" -eq 2 ] && return 2
  stop_fs_children
  [ "$?" -eq 2 ] && return 2
  rm -f $PIDFILE
  return "$RETVAL"
}

do_reload() {
  start-stop-daemon --stop --quiet \
    --pidfile $PIDFILE --name $NAME --user $USER \
    --signal HUP
}

case "$1" in
  start)
    [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC " "$NAME"
    do_start
    case "$?" in
      0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
      2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
    esac
    ;;
  stop)
    [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
    do_stop
    case "$?" in
      0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
      2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
    esac
    ;;
  status)
    status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
    ;;
  reload|force-reload)
    log_daemon_msg "Reloading $DESC" "$NAME"
    do_reload
    log_end_msg $?
    ;;
  restart)
    log_daemon_msg "Restarting $DESC" "$NAME"
    do_stop
    case "$?" in
      0|1)
        do_start
        case "$?" in
          0) log_end_msg 0 ;;
          1|*) log_end_msg 1 ;;
        esac
        ;;
      *) log_end_msg 1 ;;
    esac
    ;;
  *)
    echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" &gt;&2
    exit 3
    ;;
esac

exit 0</pre>