---
id: 1045
title: 'OpenSIPS: Basic Accounting'
date: 2016-03-20T22:27:07-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=1045
permalink: /2016/03/opensips-basic-accounting/
image: /wp-content/uploads/2015/11/fMjXFg41_400x400.png
categories:
  - OpenSIPS
tags:
  - opensips
  - sip
  - voip
---
With any phone system, you want to log the calls and certain details related to each call. You may do this to perform billing or generate reports of some kind. By default, OpenSIPS performs very basic logging of events. So for a given call, you may have numerous rows in database and you are supposed to match an INVITE with corresponding BYE (or cancel) event and determine the bill_duration. This is quite a bit of work as you can imagine a lot of things can go wrong between the INVITE and the BYE.

There is another way to log successful calls by using the [dialog module](http://www.opensips.org/html/docs/modules/2.1.x/dialog) of OpenSIPS in conjunction with the standard [acc](http://www.opensips.org/html/docs/modules/2.1.x/acc) (for accounting) module. Essentially, you create a database table that has columns that OpenSIPS expects by default. In addition you create columns for any additional information you would like to store per call. In my example below, I have chosen to store the src\_user, src\_domain, dst\_user and dst\_domain. You are free to store anything else if you prefer. You tell acc what to store in the &#8220;db_extra&#8221; parameter.

Once the module and database are configure, it is just a matter of setting the appropriate flag on INVITE and OpenSIPS will take care of the rest for you.

<pre>mysql&gt; describe acc;
+---------------+------------------+------+-----+---------+----------------+
| Field         | Type             | Null | Key | Default | Extra          |
+---------------+------------------+------+-----+---------+----------------+
| id            | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| method        | char(16)         | NO   |     |         |                |
| src_user      | varchar(64)      | YES  |     | NULL    |                |
| src_domain    | varchar(64)      | YES  |     | NULL    |                |
| dst_user      | varchar(64)      | YES  |     | NULL    |                |
| dst_domain    | varchar(64)      | YES  |     | NULL    |                |
| from_tag      | char(64)         | NO   |     |         |                |
| to_tag        | char(64)         | NO   |     |         |                |
| callid        | char(64)         | NO   | MUL |         |                |
| sip_code      | char(3)          | NO   |     |         |                |
| sip_reason    | char(32)         | NO   |     |         |                |
| time          | datetime         | NO   |     | NULL    |                |
| duration      | int(11) unsigned | NO   |     | 0       |                |
| setuptime     | int(11) unsigned | NO   |     | 0       |                |
| created       | datetime         | YES  |     | NULL    |                |
+---------------+------------------+------+-----+---------+----------------+
</pre>

As always, the following script is for illustration purposes only. It has very few checks and may not work in your network setup. Pay attention to the use of DB_FLAG.

<pre class="brush: cpp; title: ; notranslate" title="">####### Global Parameters #########

debug=3
log_stderror=no
log_facility=LOG_LOCAL0
fork=yes
children=2
auto_aliases=no

mhomed=1
listen=udp:192.168.32.8:5060    # LAN Interface
listen=udp:6.7.8.9:5060   # WAN Interface


####### Modules Section ########

mpath="/usr/lib/opensips/modules/"

loadmodule "signaling.so"
loadmodule "sl.so"
loadmodule "registrar.so"
loadmodule "proto_udp.so"
loadmodule "tm.so"
loadmodule "uri.so"
loadmodule "rr.so"

loadmodule "mi_fifo.so"
modparam("mi_fifo", "fifo_name", "/tmp/opensips_fifo")

loadmodule "acc.so"
loadmodule "dialog.so"
modparam("acc", "db_flag", "DB_FLAG")
modparam("acc", "cdr_flag", "DB_FLAG")
modparam("acc", "db_extra",
    "src_user=$fU;src_domain=$fd;dst_user=$tU;dst_domain=$rd")
modparam("acc", "db_url",
    "mysql://opensips:opensipsrw@127.0.0.1/opensips_new")

####### Routing Logic ########

route{
    # process sequential requests
    if (has_totag()) {
        if (!loose_route() && !t_check_trans() ) {
            sl_send_reply("404","Not here");
            exit;
        }

        t_relay();
        exit;
    }

    if (method == "INVITE") {
        # accounting
        create_dialog("B");
        setflag(DB_FLAG);    
    }

    record_route();

    # relay request in stateful manner to R-URI
    if (!t_relay())
        sl_reply_error();
}
</pre>