---
id: 990
title: 'OpenSIPS Part 1: A Basic Script'
date: 2015-11-20T08:00:19-05:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=990
permalink: /2015/11/a-basic-opensips-script/
image: /wp-content/uploads/2015/11/fMjXFg41_400x400.png
categories:
  - OpenSIPS
  - Technology
---
> **Note:** updated for OpenSIPS 2.2 (long term support) on May 10th, 2017.

Looking at a full-fledged OpenSIPS script for the first time can be a bewildering experience. It takes a lot of reading, testing and learning to get comfortable with it. However, for some instant gratification, you can try this. The script below is allows you to register SIP phones on your LAN and make calls to each other. It does not handle PSTN calls or any other fancy features. It doesn&#8217;t even check for your credentials, so you can pass in anything you like. If two phones are set up with the same extension, both will ring.  
[<img src="http://sumit.tampahost.net/wp-content/uploads/2015/11/OpenSIPS_-Basic.png" alt="OpenSIPS_ Basic" width="500" class="aligncenter size-full wp-image-1027" srcset="https://sumitbirla.me/wp-content/uploads/2015/11/OpenSIPS_-Basic.png 678w, https://sumitbirla.me/wp-content/uploads/2015/11/OpenSIPS_-Basic-300x225.png 300w" sizes="(max-width: 678px) 100vw, 678px" />](http://sumit.tampahost.net/wp-content/uploads/2015/11/OpenSIPS_-Basic.png)

<pre class="brush: cpp; title: ; notranslate" title="">####### Global Parameters #########&lt;/code&gt;
log_stderror=no
log_facility=LOG_LOCAL0
log_level=3
children=2
auto_aliases=no
listen=udp:10.0.9.1:5060  # CHANGE MY IP ADDRESS!


####### Modules Section ########
mpath="/usr/lib/opensips/modules/"

loadmodule "signaling.so"
loadmodule "sl.so"
loadmodule "registrar.so"
loadmodule "proto_udp.so"
loadmodule "tm.so"

loadmodule "mi_fifo.so"
modparam("mi_fifo", "fifo_name", "/tmp/opensips_fifo")

loadmodule "usrloc.so"
modparam("usrloc", "db_mode", 0)


####### Routing Logic ########
route{
    # register user location without authentication
    if (method == "REGISTER") {

        xlog("registering $tU \n");
        save("nuc");
        exit;
    }

    # lookup requested user, if registered, below will replace 
    # R-URI with registered user's contact
    if (method == "INVITE") {

        xlog("calling $tU \n");
        if (!lookup("nuc")) {
            sl_send_reply("404", "User not found");
            exit;
        }
    }

    # relay request in stateful manner to R-URI
    t_relay();
}
</pre>

Run the script and watch your syslog for register and calling messages. You can also view registered users with the following command:

<pre style="font-size: 1.0em; line-height: 1.0em;">root@nuc:/etc/opensips# opensipsctl ul show
Domain:: nuc table=512 records=2
	AOR:: 102
		Contact:: sip:102@192.168.32.118:2048;line=7dga26nf Q=1
			Expires:: 3337
			Callid:: 313434373937393735303238383034-1soqnqfjhn95
			Cseq:: 26
			User-agent:: n/a
			State:: CS_NEW
			Flags:: 0
			Cflags::
			Socket:: udp:192.168.32.8:5060
			Methods:: 7999
			SIP_instance:: &lt;urn:uuid:ecea5aa1-09b1-4f2e-8272-0004134AAB2B>
	AOR:: 101
		Contact:: sip:101@192.168.32.3:52500;rinstance=4d64f6f35eb88aa6 Q=
			Expires:: 1155
			Callid:: 78102MWJiZmRhY2VkMTJkNjZmMjhmZjQyMThhNjZmYjIwMGU
			Cseq:: 12
			User-agent:: X-Lite release 4.9.0 stamp 78102
			State:: CS_NEW
			Flags:: 0
			Cflags::
			Socket:: udp:192.168.32.8:5060
			Methods:: 5919



</pre>

In the next post, I will show how to add a PSTN gateway to this set up so that you can make outgoing calls. Small Steps. I recommend reading up on how SIP works and the structure of a SIP datagram and the various field before proceeding with more advanced topics.