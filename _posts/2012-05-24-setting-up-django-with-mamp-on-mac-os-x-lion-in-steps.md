---
layout: post
title: Setting up Django with MAMP on Mac OS X Lion (in ? steps)
date: 2012-05-24 12:31:50.000000000 +01:00
categories:
- development
tags:
- python
status: publish
type: post
published: true
comments: true
---
<p>I started playing with a Django-Python version of my current project and I needed a testing environment on my local Mac-based machine. The thing is I didn't want to use a dedicated MySQL server just for the Django deployment, as it seems really silly when I already have my quiet MAMP running in parallel with all the other CodeIgniter-PHP based projects.</p>
<p>It's harder than you may expect. I had to go through a painful series of steps to eventually have the whole thing working, and taking here some notes so I can have them handy when I'll need to do it again, hopefully it will help someone else. It still needs to have a "dedicated" copy of MySQL installed just to compile, but it won't be used to actually run the server.</p>
<p><strong>step1</strong>. download and install the latest mysql community server from the DMG file, <a href="http://www.mysql.com/downloads/mysql/">here</a>. Then from the command line, add the fresh installation to the local path, this will be used to pick the sources needed for the python module compilation. Also add a variable for the dynamic library linkage, this will be used on execution of your python scripts to find the correct library:</p>
<pre class="brush:shell">export PATH=/usr/local/mysql/bin/:$PATH
export DYLD_LIBRARY_PATH=/usr/local/mysql/lib/</pre>
<p><strong>step2</strong>. assuming we have already a python <a href="http://pypi.python.org/pypi/virtualenv">virtualenv</a> script in place. Create a new virtual environment and activate it:</p>
<pre class="brush:shell">python virtualenv.py django-mysql
. django-mysql/bin/activate</pre>
<p><strong>step3</strong>. now we're ready to install django and mysql-python. the latter should compile with few minor warnings if and only if we have correctly executed step1 (just double check that paths in the export statements are correct)</p>
<pre class="brush:shell">pip install django
pip install mysql-python
python
&gt;&gt;&gt; import MySQLdb
&gt;&gt;&gt; print MySQLdb</pre>
<p>if everything works fine, then the last command above should show the full path of the mysql module compiled during the installation process</p>
<p><strong>step4</strong>. create a django project and configure the settings.py so as to use the MySQL server shipped with MAMP</p>
<pre class="brush:shell">DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'django_test', # Or path to database file if using sqlite3.
        'USER': 'root', # Not used with sqlite3.
        'PASSWORD': 'root', # Not used with sqlite3.
        'HOST': '/Applications/MAMP/tmp/mysql/mysql.sock', # Set to empty string for localhost. Not used with sqlite3.
        'PORT': '', # Set to empty string for default. Not used with sqlite3.
    }
}</pre>
<p>take EXTRA CARE to double check that the <strong>host</strong> variable is set to MAMP's socket, hence <strong>/Applications/MAMP/tmp/mysql/mysql.sock</strong></p>
<p><strong>step5</strong>. in my case, it's really not happening frequently, I also had problems with my locale, as some of the programs I installed messed around with the environment, so I got the error "unknown locale: UTF-8" as soon as I tried to sync the django db. this is solved exporting the correct variables, THEN syncing the db</p>
<pre class="brush:shell">export LC_ALL=en_us.UTF8
export LC_CTYPE=en_us.UTF8
export LANG=en_us.UTF8
python manage.py syncdb</pre>
<p>and this should be all! Just a final note: this post was assembled trying things and using different sources to solve each and every problem appeared along the road, these are the original URLs I had to collate to eventually see my working copy of django: <a href="http://avianbonesyndrome.com/2010/04/04/installing-the-mysqldb-python-module-on-snow-leopard/">dedicated mysql</a> <a href="http://www.blog.bridgeutopiaweb.com/post/how-to-fix-mysql-load-issues-on-mac-os-x/">dynamic library</a> <a href="http://www.marteinn.se/blog/?p=467">mamp socket</a>  <a href="https://www.google.co.uk/search?&amp;q=unknown+locale%3A+UTF-8">unknown locale</a></p>
