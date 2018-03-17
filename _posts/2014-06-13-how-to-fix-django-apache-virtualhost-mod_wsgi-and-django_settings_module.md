---
layout: post
title: How to fix Django, Apache VirtualHost, mod_wsgi and DJANGO_SETTINGS_MODULE
date: 2014-06-13 12:31:56.000000000 +01:00
categories:
- development
tags: []
status: publish
type: post
published: true
comments: true
---
I've spent countless hours trying to fix Django installations running on legacy Apache servers, these usually recurring every few months, a time span long enough to forget how the last fix was done. And for some reason, the docs are not mentioning this crucial features AT ALL! In the official <a href="https://docs.djangoproject.com/en/1.6/howto/deployment/wsgi/modwsgi/">Django + mod_wsgi documentation page</a>, they don't mention something so irrelevant such as THE MAIN DJANGO SETTINGS FILE?

<p>For future memory, here's a solution that makes me happy, and hopefully next time will only be a few minutes headache (although I already know it won't be the case...)</p>
<p>- copy the default wsgi.py to a new file called <strong>apache_wsgi.py</strong></p>
<p>- modify the new <strong>apache_wsgi.py</strong> so that it reads:</p>

{% highlight python %}
import os
import mod_wsgi
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "project_name.settings.%s" % mod_wsgi.process_group)

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
{% endhighlight %}

<p>- now open your <strong>virtualhost</strong> configuration file and check it contains the following</p>

{% highlight python %}
WSGIScriptAlias / /path/to/your/apache_wsgi.py
WSGIDaemonProcess pick_settings_file_name processes=2 threads=15 python-path=/path/to/virtualenv/lib/python2.7/site-packages:/path/to/django/project
WSGIProcessGroup pick_settings_file_name
{% endhighlight %}

<p>- almost there! create a <strong>/settings</strong> folder sibling of settings.py, put an empty __init__.py in it, then move settings.py inside that folder, rename it to common.py and create a new <strong>pick_settings_file_name.py</strong> with a single liner in it</p>

{% highlight python %}
from .common import *
{% endhighlight %}

<p>VOILAAAAAAAAAAAAAAA!!!!</p>
<p>just restart your apache and everything will work. in your pick_settings_file_name.py you will have configurations specific for your environment (i.e. multiple copies of the file for dev, staging: project_dev.py, project_staging.py)</p>
