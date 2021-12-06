---
id: 158
title: How to build a scalable, caching, resizing image server
date: 2011-11-11T00:37:13-05:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=158
permalink: /2011/11/how-to-build-a-scalable-caching-resizing-image-server/
image: /wp-content/uploads/2011/11/nginx-logo.png
categories:
  - Linux
  - Web
tags:
  - linux
  - nginx
  - php
  - Web
format: gallery
---
Building the next big thing on the web?  It is never a good idea to serve your static files, i.e. style sheets, javascripts and images, from your main application server.  This task should be offloaded to another server for three reasons &#8211;

  1. It ties up resources in your app server while it would be performing other tasks.
  2. App servers are hardly ever optimized for simple file serving.
  3. Browsers can fetch resources in parallel from the two servers.  This can improve the load time significantly.

<!--more-->

Most major websites on the web such as Facebook, Best Buy et.al use this concept.  Some opt to use commercial services called Content Delivery Networks (CDN) to accomplish this task.  Two such companies are Akamai and Limelight.  You can see this for yourself by checking the source of images on Best Buy&#8217;s site, for instance.  Images have the URL:

http://www.bestbuyon.com/sites/default/files/COD\_MW3\_300x145.jpg

If you look up the domain bestbuyon.com, you will see that it is actually being served by multiple Akamai servers.

<pre>dig www.bestbuyon.com

;; QUESTION SECTION:
;www.bestbuyon.com.		IN	A

;; ANSWER SECTION:
www.bestbuyon.com.	86357	IN	CNAME	on.bestbuy.com.edgesuite.net.
on.bestbuy.com.edgesuite.net. 21557 IN	CNAME	a1982.b.akamai.net.
a1982.b.akamai.net.	20	IN	A	65.32.34.146
a1982.b.akamai.net.	20	IN	A	65.32.34.128

;; AUTHORITY SECTION:
b.akamai.net.		1755	IN	NS	n9b.akamai.net.
b.akamai.net.		1755	IN	NS	n3b.akamai.net.
b.akamai.net.		1755	IN	NS	n1b.akamai.net.
b.akamai.net.		1755	IN	NS	n5b.akamai.net.
b.akamai.net.		1755	IN	NS	n0b.akamai.net.
b.akamai.net.		1755	IN	NS	n4b.akamai.net.
b.akamai.net.		1755	IN	NS	n2b.akamai.net.
b.akamai.net.		1755	IN	NS	n8b.akamai.net.
b.akamai.net.		1755	IN	NS	n6b.akamai.net.
b.akamai.net.		1755	IN	NS	n7b.akamai.net.</pre>

These CDN services can be fairly expensive and may be out of scope of a startup budget, or if you are doing a side project.  In my past experience, starting price was around $700/mo.   The good news is that you can create a less ambitious version of a CDN yourself using free, open-source tools.  The set up which I describe below can handle boat-loads of traffic.  If you start outgrowing this, then you already have a successful website and should be earning enough to pay for a CDN!

### Resizing Images on the fly

I have found over the years that it is always best to upload original images and resize them on the fly as needed.  This can be useful in multiple scanarios:

  1.  Think of an application like Facebook.  Users upload images of all different shapes and sizes.  You have to resize the image to fit a certain box area before displaying on the web page.
  2. You have your website all set up and running.  6 months later, you decide to redesign the pages.  Do you recreate all your images from scratch?  The better solution is to let our caching, imaging server do the work.

The catch is that image resizing is a heavy task.  You don&#8217;t want to do that on every request.  Instead, the image server must cache the resized image for subsequent calls.  In our case, resized images are saved to disk and served directly from there.

### Tools of the trade &#8211; Linux, Nginx and PHP

For this project, we will use three open-source tools.  They are listed below in the order in which they have to be installed.  If you are linux-savvy, the whole project should take less than an hour.

#### Any Linux Distribution

Install any recent version on Linux.  I have used Ubuntu 10.04 LTS Server for my work.  But you can use any flavor you like.

#### Nginx

Nginx (pronounced engine-x) is an amazing, light-weight web server that can serve files at blazing speeds while utlizing very few resources.  In fact, I run it on Amazon EC2&#8217;s micro instance which only gives me 640MB of RAM.  The server design is asynchronous.  Unlike Apache, it does not use a threading model and is a lot more scalable.  A nice side-bonus is that the config files are a lot sensible also.

<pre>prompt$ sudo apt-get install nginx</pre>

Next set up a website using the following configuration.  Change domain name and directory location to suit your setup:

<pre>server {
        listen   80;
        server_name  static.example.com;
        root   /var/www/static.example.com/;

        location ~ ^/(.*)/cache/(.*)$
        {
                try_files $uri @resize;
                expires 4h;
        }

        location / {
                expires 4h;
        }

        location ~ \.php(.*)$  {
                fastcgi_pass 127.0.0.1:8000;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_index index.php;
                include /etc/nginx/fastcgi_params;
        }

        location @resize {
                rewrite ^/(.*)/cache/(.*)$ /resize.php?dir=$1&path=$2;
                include /etc/nginx/fastcgi_params;
                fastcgi_pass 127.0.0.1:8000;
        }
}</pre>

The above configuration performs the following actions:

  1. If requested image exists on disk, serve it directly.
  2. If image does not exist on disk, return 404.
  3. If image does not exist on disk and URL contains /cache/, then call PHP script via fast-cgi which creates a resized version of the image, saves it to disk and returns it to caller.

<div>
  As you can see, with the above setup, the image is resized just once.  Future requests are served directly from disk with no overhead of call to the PHP script via fast-cgi.  Additionally, expires 4h set the cache expiration in browser to 4 hours.  You could set this to 10 years or so.  The idea being that a visitor to your website isn&#8217;t re-downloading the common images going from page to page.
</div>

<div>
  <span class="Apple-style-span" style="line-height: 18px;"><br /> </span>
</div>

#### PHP

We will use a short PHP script to perform image resizing.  This script makes use of Imagick library.  So make sure that is installed also.

<pre>prompt$ sudo apt-get install php5-cgi php5-imagick</pre>

Create a file called /var/www/static.example.com/resize.php and enter the following content:

<pre class="brush: php; title: ; notranslate" title="">ini_set("memory_limit","80M");

        # prevent creation of new directories
        $is_locked = false;

        # figure out requested path and actual physical file paths
        $orig_dir = dirname(__FILE__);
        $path = $_GET['path'];
        $tokens = explode("/", $path);
        $file = "/".implode('/', array_slice($tokens,3));
        $orig_file = $orig_dir.$file;

        if (!file_exists($orig_file))
        {
                header("Status: 404 Not Found");
                echo "&lt;br/&gt;PATH=$path&lt;br/&gt;ORIGFILE=$orig_file";
                return 0;
        }

        # check if new directory would need to be created
        $save_path = "$orig_dir/cache/$tokens[2]$file";
        $save_dir = dirname($save_path);

        if(!file_exists($save_dir) && $is_locked)
        {
                header("Status: 403 Forbidden");
                echo "Directory creation is forbidden.";
                return 0;
        }

        # parse out the requested image dimensions and resize mode
        $x_pos = strpos($tokens[2], 'x');
        $dash_pos = strpos($tokens[2], '-');
        $target_width = substr($tokens[2], 0, $x_pos);
        $target_height = substr($tokens[2], $x_pos+1, $dash_pos-$x_pos-1);
        $mode = substr($tokens[2], $dash_pos+1);

        $new_width = $target_width;
        $new_height = $target_height;
        $image = new Imagick($orig_file);
        list($orig_width, $orig_height, $type, $attr) = getimagesize($orig_file);

        # preserve aspect ratio, fitting image to specified box
        if ($mode == "0")
        {
                $new_height = $orig_height * $new_width / $orig_width;
                if ($new_height &gt; $target_height)
                {
                        $new_width = $orig_width * $target_height / $orig_height;
                        $new_height = $target_height;
                }
        }
        # zoom and crop to exactly fit specified box
        else if ($mode == "2")
        {
                // crop to get desired aspect ratio
                $desired_aspect = $target_width / $target_height;
                $orig_aspect = $orig_width / $orig_height;

                if ($desired_aspect &gt; $orig_aspect)
                {
                        $trim = $orig_height - ($orig_width / $desired_aspect);
                        $image-&gt;cropImage($orig_width, $orig_height-$trim, 0, $trim/2);
                        error_log("HEIGHT TRIM $trim");
                }
                else
                {
                        $trim = $orig_width - ($orig_height * $desired_aspect);
                        $image-&gt;cropImage($orig_width-$trim, $orig_height, $trim/2, 0);
                }
        }

        # mode 3 (stretch to fit) is automatic fall-through as image will be blindly resized 
        # in following code to specified box
        $image-&gt;resizeImage($new_width, $new_height, imagick::FILTER_LANCZOS, 1);

        # save and return the resized image file        
        if(!file_exists($save_dir))
                mkdir($save_dir, 0777, true);

        $image-&gt;writeImage($save_path);
        echo file_get_contents($save_path);

        return true;
</pre>

You will need to start up php as a fast-cgi server running on port 8000.  You can download the /etc/init.d/php-fascgi script and /etc/default/php-fastcgi scripts from the following location:

<a href="http://blog.codefront.net/2007/06/11/nginx-php-and-a-php-fastcgi-daemon-init-script/" target="_blank">http://blog.codefront.net/2007/06/11/nginx-php-and-a-php-fastcgi-daemon-init-script/</a>

Once up and running, you should see an output similar to this:

<pre>prompt$ ps -ef | grep php
www-data 25255 31528  0 Nov10 ?        00:00:52 /usr/bin/php-cgi -q -b 127.0.0.1:8000
www-data 25295 31528  0 Nov10 ?        00:00:54 /usr/bin/php-cgi -q -b 127.0.0.1:8000
www-data 31528     1  0 Nov06 ?        00:00:00 /usr/bin/php-cgi -q -b 127.0.0.1:8000</pre>

At this point, all necessary software has been installed.   Next, we&#8217;ll look at the directory structures and how to invoke image resizing.

### How to use our super-duper setup

Accessing an original file:

<pre>http://static.example.com/originals/myface.jpg</pre>

Accessing a resized file:

<pre>http://static.example.com/cache/150x200-0/originals/myface.jpg</pre>

As you can see, it is not difficult to construct a URL for resized image if you know the location of the original.  In the example above, 150&#215;200 are the maximum dimensions of the returned image.  The last number &#8216;0&#8217; is the resizing mode:

<table>
  <tr>
    <td nowrap="nowrap">
      <strong>Mode 0</strong>
    </td>
    
    <td>
      Size the image to fit the box while preserving the aspect ratio. The returned image will be of specified dimension or smaller.
    </td>
  </tr>
  
  <tr>
    <td nowrap="nowrap">
      <strong>Mode 1</strong>
    </td>
    
    <td>
      Stretches the image two fit the specified dimensions. Aspect ratio is not preserved.
    </td>
  </tr>
  
  <tr>
    <td nowrap="nowrap">
      <strong>Mode 2</strong>
    </td>
    
    <td>
      Zoom and crops the image to fill the specified dimension.
    </td>
  </tr>
</table>

### Verify Setup

Once set up, your directory structure should look something like below. Make sure the file are owned by same user as the one running nginx (usually www-data).

<pre>root@myserver:/var/www/static.example.com$ tree
.
├── cache
│   ├── 600x300-2
│   │   └── originals
│   │       └── astro.jpg
│   └── 600x400-0
│       └── originals
│           └── astro.jpg
├── originals
│   └── astro.jpg
└── resize.php</pre>

It is fairly simply to test our setup.  You can request image at varying sizes and ensure they are returned.  One should also check the HTTP Response Header to make sure that the PHP script is not being called every time.

**Original:** http://static.example.com/originals/astro.jpg

<img class="aligncenter" title="Original" src="http://cdn1.tampahost.net/pk/cache/600x400-0/userfiles/images/albums/2/241bb64b-1351-40bf-a9c7-48787d77e302.jpg" alt="" width="563" height="400" /> 

**Resized Mode 0: **http://static.example.com/cache/300&#215;200-0/originals/astro.jpg

<img class="alignnone" title="Resized Mode 0" src="http://cdn1.tampahost.net/pk/cache/300x200-0/userfiles/images/albums/2/241bb64b-1351-40bf-a9c7-48787d77e302.jpg" alt="" width="281" height="200" /> 

**Resized Mode 1: **http://static.example.com/cache/300&#215;100-1/originals/astro.jpg

<img class="alignnone" title="Resized Model 1" src="http://cdn1.tampahost.net/pk/cache/300x100-1/userfiles/images/albums/2/241bb64b-1351-40bf-a9c7-48787d77e302.jpg" alt="" width="300" height="100" /> 

**Resized Mode 2: **http://static.example.com/cache/300&#215;100-2/originals/astro.jpg

<img class="alignnone" title="Resized Mode 2" src="http://cdn1.tampahost.net/pk/cache/300x100-2/userfiles/images/albums/2/241bb64b-1351-40bf-a9c7-48787d77e302.jpg" alt="" width="300" height="100" /> 

**HEAD output on first call to resize (served by PHP):**

<pre>prompt$ HEAD http://static.example.com/cache/300x150-2/originals/astro.jpg
200 OK
Connection: close
Date: Fri, 11 Nov 2011 05:17:45 GMT
Server: nginx/0.7.65
Content-Type: text/html
Client-Date: Fri, 11 Nov 2011 05:17:45 GMT
Client-Peer: 184.72.219.132:80
Client-Response-Num: 1
<span style="color: #ff0000;">X-Powered-By: PHP/5.3.2-1ubuntu4.7</span></pre>

**HEAD output on subsequent calls (served directly by nginx):**

<pre>prompt$ HEAD http://static.example.com/cache/300x150-2/originals/astro.jpg
200 OK
Cache-Control: max-age=14400
Connection: close
Date: Fri, 11 Nov 2011 05:19:00 GMT
Accept-Ranges: bytes
Server: nginx/0.7.65
Content-Length: 44024
Content-Type: image/jpeg
Expires: Fri, 11 Nov 2011 09:19:00 GMT
Last-Modified: Fri, 11 Nov 2011 05:17:45 GMT
Client-Date: Fri, 11 Nov 2011 05:19:00 GMT
Client-Peer: 184.72.219.132:80
Client-Response-Num: 1</pre>

### One last thing

The PHP script in charge if resizing images creates directories as needed.  For example, it an image was requested with the following URL:

<pre>/cache/300x150-2/originals/astro.jpg</pre>

The script would create /cache/300&#215;150-2 directory if it doesn&#8217;t already exist.  This could be used to fill up your disk space by malicious users simply by requesting a particular file with constantly varying dimensions.  In general, when you design a website, you will have used images at fixed dimensions.  These don&#8217;t change until you alter the design.  So during the cruise phase, it is best to set $is_locked = true in the script so that it doesn&#8217;t create new directories.

### Summary (or for the impatient)

  1. Install Linux
  2. sudo apt-get install nginx php5-cgi php5-imagick
  3. Create nginx virtualhost, start php5-cgi process
  4. Download resize.php into directory structure show above
  5. Test