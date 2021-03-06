---
layout: post
title: Configuring NGINX and CodeIgniter on Ubuntu Oneiric Ocelot on Amazon EC2
date: 2011-07-31 15:39:39.000000000 +01:00
categories:
- development
tags:
- Amazon EC2
- cloud computing
- Nginx
status: publish
type: post
published: true
comments: true
---
<p>Few days ago I started the server setup for a web project @therumpusroom_ and, after receiving the traffic estimates, I thought a single Apache server was not enough to handle the expected load of visitors. For several reasons I want to avoid using a load balancer and multiple Apache instances, hence the decision to implement Nginx with MySql running on a separate dedicated server.</p>
<p>The whole infrastructure lives on Amazon Web Services and the web application - still under development - will rely on CodeIgniter. I have read quite a lot of articles on-line and stolen bits and pieces of configuration files, but none of them entirely reflected what I needed. I feel it is quite a common configuration hence I am writing down here the required steps and some code snippets, both for my personal records and also hoping it can be helpful for someone else with similar issues.</p>
<p><!--more-->The premise: implement a CodeIgniter installation on Amazon EC2 with a dedicated DB server and content delivery network for rich media distribution.</p>
<p>Pre-requisites / specs: Ubuntu 11.10 Oneiric Ocelot 64bit with Nginx web server running on a large instance on Amazon EC2, dedicated MySQL server on Amazon RDS and Cloudfront CDN.</p>
<p>The steps:</p>
<h3>1. choose your Ubuntu installation</h3>
<p>I ended up choosing Oneiric Ocelot 64bit, I am always too tempted to try the latest, anyhow you can always find your own Ubuntu AMI using the super helpful <a title="Ubuntu AMI Locator" href="http://cloud.ubuntu.com/ami/">AMI locator</a> for EC2</p>
<h3>2. start a basic NGINX installation</h3>
<p>I used <a title="Setup NGINX on Ubuntu 11.04 with PHP Fast CGI" href="http://library.linode.com/web-servers/nginx/php-fastcgi/ubuntu-11.04-natty">this guide on Linode</a> to configure Nginx and PHP-FastCGI on Ubuntu 11.04 (Natty) as a starting point, just be aware of the following:</p>
<ul>
<li>ignore the hostname configuration: it did not work for me and it is not relevant to make the web server work properly</li>
<li>start with the suggested config for nginx, but keep in mind you will need to finalize it later</li>
</ul>
<p>also the init.d/php-fastcgi script in the Linode guide gave errors and was not working properly for me, so I have created a simpler version (you may need to manually create pid/socket folders before running the script the first time):</p>
<pre class="brush:shell">PHP_SCRIPT=/usr/bin/php-fastcgi
PID_DIR=/var/run/php-fastcgi
PID_FILE=/var/run/php-fastcgi/php-fastcgi.pid
SOCKET_FILE=/var/run/php-fastcgi/php-fastcgi.socket
RET_VAL=0

case "$1" in
    start)
      $PHP_SCRIPT
      RET_VAL=$?
  ;;
    stop)
      rm $PID_FILE
      rm $SOCKET_FILE
      killall -9 php5-cgi
      RET_VAL=$?
  ;;
    restart)
      rm $PID_FILE
      rm $SOCKET_FILE
      killall -9 php5-cgi
      $PHP_SCRIPT
      RET_VAL=$?
  ;;
    status)
      echo "php-fastcgi running with PID `cat $PID_FILE`"
  ;;
    *)
      echo "Usage: php-fastcgi {start|stop|restart|status}"
      RET_VAL=1
  ;;
esac
exit $RET_VAL</pre>
<p>by this time you should be able to execute some test php code to chech that your FastCGI script is working properly and receiving parameters from the web server just using the default site already enabled.</p>
<h3>3. setup CodeIgniter</h3>
<p>now the interesting part: setting up CodeIgniter with correct locations is not straight forward. There is an <a title="CodeIgniter with NGINX" href="http://codeigniter.com/forums/viewthread/90231/">interesting thread on the official CodeIgniter Forum</a>, pointing the right way but unfortunately it does not entirely solve the problem.</p>
<p>After downloading CodeIgniter and extracting the archive in the document root, the first important step required to see at least the welcome screen is to setup the configuration file so as to receive parameters from the web server, under /application/config/config.php</p>
<pre class="brush:php">$config['uri_protocol'] = 'REQUEST_URI';</pre>
<p>and finally setup the Nginx "virtual host" to serve the correct directories and path infos used by CodeIgniter controllers to receive parameters: in my setup I have the CodeIgniter application folder also serving the main static contents (under /application/public with subfolders: css, img, js). I started from a <a title="CodeIgniter with PATH_INFO on gist" href="https://gist.github.com/1050850">config file find on gist</a> then tweaked to reflect my specific needs. Here is the code:</p>
<pre class="brush:plain">server {
    server_name project.staging.example.com;
    access_log /home/ubuntu/repo/staging/logs/access.log;
    error_log /home/ubuntu/repo/staging/logs/error.log;

    root /home/ubuntu/repo/staging/webdev;
    index index.php index.html;

    location ~* ^/(css|img|js)/(.+)$ {
        root /home/ubuntu/repo/staging/webdev/application/public;
    }

    location / {
        try_files $uri $uri/ @rewrites;
    }

    location @rewrites {
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php/$1 last;
            break;
        }
    }

    location /(application|system) {
        internal;
    }

   location ~ \.php {
        include /etc/nginx/fastcgi_params;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php-fastcgi/php-fastcgi.socket;
        fastcgi_index index.php;
    }
}</pre>
<p>this should be all, hope this helps and feel free to drop a line below for questions.</p>
