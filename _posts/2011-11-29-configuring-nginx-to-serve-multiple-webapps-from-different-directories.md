---
layout: post
title: Configuring NGINX to serve multiple webapps from different directories
date: 2011-11-29 16:54:30.000000000 +00:00
categories:
- development
tags:
- Nginx
status: publish
type: post
published: true
comments: true
---
Few days ago I had to add a wordpress installation within the same environment where a Codeigniter app was already running happily and undisturbed. It took me a while to figure out how to keep separate folders on the filesystem, and serve the blog from a subfolder of the main domain: it ended up that the solution is super simple, but apparently I am not the only one who had similar problems. Symptoms of a bad installation usually result in "no input file specified" messages or, even worse, downloading the php source code with all your precious database passwords shown in clear.

<p>So the premise being:</p>
<ul>
<li>the webapps need to live in sibling folders to keep tidy our github repo, in the example below will be named as /home/ubuntu/repo/webapp (codeigniter) and /home/ubuntu/repo/blog (wordpress)</li>
<li>the main webapp needs to respond to all the requests, while wordpress needs to catch only requests starting with /blog</li>
</ul>
<p>there might be better and more elegant solutions, but this is working for me, including pretty permalinks on wordpress:</p>
<pre class="brush:shell">server {
    server_name your.domain.com;

    access_log /home/ubuntu/repo/logs/access.log;
    error_log /home/ubuntu/repo/logs/error.log;

    # main root, used for codeigniter
    root /home/ubuntu/repo/webapp;
    index index.php index.html;

    # links to static files in the main app, mainly for dev purposes as this is
    # unlikely to be triggered when using a CDN with absolute URLs to assets
    location ~* ^/(css|img|js|flv|swf)/(.+)$ {
        root /home/ubuntu/repo/webapp/application/public;
    }

    # most generic (smaller) request
    # most of the times will redirect to named block @ci
    location / {
        try_files $uri $uri/ @ci;
    }

    # create the code igniter path and perform
    # internal redirect to php location block
    location @ci {
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php/$1 last;
            break;
        }
    }

    # now the meaty part, execute php scripts
    location ~ \.php {
        include /etc/nginx/fastcgi_params;

        # default path of our php script is the main webapp
        set $php_root /home/ubuntu/repo/webapp;

        # but we might have received a request for a blog address
        if ($request_uri ~ /blog/) {
            # ok, this line is a bit confusing, be aware
            # that path to /blog/ is already in the request
            # so adding a trailing /blog here will
            # give a "no input file" message
            set $php_root /home/ubuntu/repo;
        }

        # all the lines below are pretty standard
        # notice only the use of $php_root instead of $document_root
        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;

        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_param SCRIPT_FILENAME $php_root$fastcgi_script_name;

        fastcgi_pass unix:/var/run/php-fastcgi/php-fastcgi.socket;
        fastcgi_index index.php;
    }

    # now the blog, remember this lives in a sibling directory of the main app
    location ~ /blog/ {
        # again, this might look a bit weird,
        # but remember that root directive doesn't drop
        # the request prefix, so /blog is appended at the end
        root /home/ubuntu/repo;
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php/$1 last;
            break;
        }
    }
}</pre>
<p>please feel free to add comments and suggestions, hope this helps.</p>
