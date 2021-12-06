---
id: 577
title: .Net on Linux (Fedora Core 2)
date: 2005-09-18T12:00:06-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=577
permalink: /2005/09/net-on-linux-fedora-core-2/
image: /wp-content/uploads/2005/09/200px-Mono_project_logo.svg_.png
categories:
  - Linux
tags:
  - linux fedora
  - mono
---
### Introduction

The .Net platform is Microsoft&#8217;s answer to Java. Java was/is a favorite for enterprise application developers who require cross-platform operability, object oriented design and scalability. However, .Net has been adopted very quickly given the dominance of its creator in IT circles. This platform offers some improvements over J2EE and also allows you to program in one of many languages which all compile to the same intermediate language. The .Net runtime does not care in which language the program was written.

One of the big advantages that .Net provides is the coding of ASPX pages. The concept of code-behind allows clean separation of UI and logic (well, some people still make a pudding out of it). Java offers this functionality too now. Competition is good indeed.

Until now, the major drawback of ASP.Net was that it only ran on Windows platform. Windows isn&#8217;t as secure, reliable or scalable as Unix. Now we have the open source and cross-platform project called [Mono](http://www.mono-project.com/). It implements the .Net runtime which apparently is an open standard. It is a new project, but seems to be fairly usable already. One can program applications in Windows and run on Linux &#8211; an ideal combination for many. The only thing to watch for is the changing nature of this &#8216;standard&#8217;. Microsoft isn&#8217;t known for sticking to standards.

This is a log of my efforts to install Mono on my Linux server. It should also serve as a guide to anyone attempting to do the same. Mono is relatively new and is progressing fast. But the installation process is still intimidating and the setup requires manual configuration.

### My System

  * Fedora Core 2
  * Apache version 2.0.51

### Initial attempts

A few months ago, I had attempted to install individual RPM packages. But there were some circular dependencies which I did not know how to resolve. Then I tried yum, and that seemed to work to a certain extent. But recently, the yum repository for Fedora Core 2 seems to have disappeared. The repositories don&#8217;t seem to be maintained well.

Given this, I download the source code and tried compiling. Mono was easy to compile and install. Simply do the standard ./configure, make and make install. XSP, the standalone server also compiled and installed in similar fashion. mod_mono, the module which allows you to run ASP.Net applications in conjunction with Apache would not compile. It looks for a script from Apache called apxs. This is not installed on Fedora. I found the RPM package for it called apache-devel, but then the script complained about not finding some other files. So I gave up on this. After all any production-type scenario will require integration with Apache. XSP isn&#8217;t designed for heavy-duty work.

### Something that works

This is my most current installation using RPMs. It seems to work even though some of the packages aren&#8217;t specifically for Fedora Core 2. Here is the list of packages that were needed (my main goal being getting ASP.Net to work) &#8211;

  1. mono-core-1.1.9-0.novell.i586.rpm ([download](http://www.go-mono.com/download/x86/mono-1.1/1.1.9/mono-core-1.1.9-0.novell.i586.rpm))
  2. mono-data-1.1.9-0.novell.i586.rpm ([download](http://www.go-mono.com/download/x86/mono-1.1/1.1.9/mono-data-1.1.9-0.novell.i586.rpm))
  3. mono-web-1.1.9-0.novell.i586.rpm ([download](http://www.go-mono.com/download/x86/mono-1.1/1.1.9/mono-web-1.1.9-0.novell.i586.rpm))
  4. xsp-1.1.9-0.novell.noarch.rpm ([download](http://www.go-mono.com/download/noarch/xsp/1.1.9/xsp-1.1.9-0.novell.noarch.rpm))
  5. mod_mono-1.1.9-0.fedora3.novell.i386.rpm ([download](http://www.go-mono.com/download/fedora-3-i386/mod_mono/1.1.9/mod_mono-1.1.9-0.fedora3.novell.i386.rpm))

These packages must be installed in the order above. These are the bare-minumum required. If you need additional function, you may download them from the following website &#8211;

> <http://www.mono-project.com/Downloads>

Notice that the mod\_mono package is for Fedora Core 3, but it also works for Core 2. Download each of the packages and type &#8216;rpm &#8211;install package\_name&#8217;

### Testing the installation

The first order of business is to familiarize yourself with the files that were installed.

<table border="1">
  <tr>
    <td>
      C# compiler executable
    </td>
    
    <td>
      /usr/bin/mcs
    </td>
  </tr>
  
  <tr>
    <td>
      Mono runtime executable
    </td>
    
    <td>
      /usr/bin/mono
    </td>
  </tr>
  
  <tr>
    <td>
      Class Libraries
    </td>
    
    <td>
      /usr/lib/mono
    </td>
  </tr>
  
  <tr>
    <td>
      XSP binaries
    </td>
    
    <td>
      /usr/lib/xsp
    </td>
  </tr>
  
  <tr>
    <td>
      XSP startup scripts
    </td>
    
    <td>
      /usr/bin/xsp<br /> /usr/bin/xsp2
    </td>
  </tr>
  
  <tr>
    <td>
      Sample ASP.Net pages
    </td>
    
    <td>
      /usr/lib/xsp/test
    </td>
  </tr>
  
  <tr>
    <td>
      mod_mono.so (for Apache)
    </td>
    
    <td>
      /etc/httpd/modules/
    </td>
  </tr>
</table>

While browsing these directories, you will notice two different versions of XSP and libraries &#8211; 1.0 and 2.0. These correspond to the old and the new .Net versions.

### Compiling Hello World!

Listed below is the most basic, command-line C# program. Create a file called hello.cs and type in the code below.

hello.cs

<div>
  <pre>public class Hello1
{
   public static void Main()
   {
      System.Console.WriteLine("Hello, World!");
   }
}</pre>
</div>

You are now ready to compile your very first program.

<div>
  [sumit@europa utils]$ mcs hello.cs<br /> [sumit@europa utils]$
</div>

This creates a file called &#8216;hello.exe&#8217; which you can execute using mono runtime.

<div>
  [sumit@europa utils]$ mono hello.exe<br /> Hello, World!
</div>

&nbsp;

### Testing XSP

If everything worked as expected, you can start testing standalone web server written in C# called XSP. By default, it will process aspx pages present in the current directory. It listens on port 8080. To check this, do the following &#8211;

<div>
  <pre>[sumit@europa utils]$ cd /usr/lib/xsp/test
[sumit@europa test]$ xsp
xsp
Listening on port: 8080
Listening on address: 0.0.0.0
Root directory: /usr/lib/xsp/test
Hit Return to stop the server.</pre>
</div>

Using any browser, point to the server at port 8080, you should see the sample applications installed in /usr/lib/xsp/test.

### Testing with Apache

The final step is to install mod_mono so that we can use Apache to serve .aspx pages. To accomplish this, open up /etc/httpd/conf/httpd.conf using a text editor and insert the lines highlighed in blue below &#8211;

<div>
  <pre>LoadModule file_cache_module modules/mod_file_cache.so
LoadModule mem_cache_module modules/mod_mem_cache.so
LoadModule cgi_module modules/mod_cgi.so
LoadModule mono_module modules/mod_mono.so</pre>
</div>

<div>
  <pre>Alias /icons/ "/var/www/icons/"

&lt;Directory "/var/www/icons"&gt;
    Options Indexes MultiViews
    AllowOverride None
    Order allow,deny
    Allow from all
&lt;/Directory&gt;

#
# Mono ASP.Net applications
#
MonoServerPath /usr/lib/xsp/2.0/mod-mono-server2.exe

Alias /samples "/usr/lib/xsp/test"
MonoApplications "/samples:/usr/lib/xsp/test"

&lt;Directory /usr/lib/xsp/test&gt;
    SetHandler mono
&lt;/Directory&gt;</pre>
</div>

Note that I chose to use version 2.0 in the MonoServerPath. It is upto you which one you want to use. Now restart the web server.

<div>
  <pre>[root@europa test]# /etc/init.d/httpd restart
Stopping httpd:                                            [  OK  ]
Starting httpd:                                            [  OK  ]</pre>
</div>

Open your browser and point to http://servername/samples/. You should see the same page which you saw with XSP on port 8080. The only difference is that Apache handles the .aspx requests now by forwarding them to a slightly modified version of XSP called mod-mono-server2.exe. You can verify that the web server did indeed start this up by looking at the process list owned by apache. One of the lines should contain mod-mono-server2.exe

<div>
  ps -ef | grep apache
</div>

By now you should have a fairly good understanding of the Mono installation. The next step is to start coding some interesting applications.