---
layout: post
type: post
title: Convert Longitude-Latitude to a flat map using Python and PostGIS
date: 2015-05-19
categories:
- development
tags: []
status: publish
published: true
---

I've recently had to develop a web app that shows Tweets locations on a map. It's simple enough to extract a tweet's location (when present), just check the [API Docs for a Tweet object][tweet_obj] and you'll find a *coordinates* field that reportedly:

> Represents the geographic location of this Tweet as reported by the user or client application. The inner coordinates array is formatted as geoJSON (longitude first, then latitude).

The next step is to visualize it on a flat map with the widely accepted [Mercator projection][merc_wiki]. There are a few useful references on [StackOverflow][stack_conv] and [Wolfram][merc_wolfram] that gave me the hints to write these simple python functions:

{% highlight python %}
import math

def get_x(width, lng):
    return int(round(math.fmod((width * (180.0 + lng) / 360.0), (1.5 * width))))

def get_y(width, height, lat):
    lat_rad = lat * math.pi / 180.0
    merc = 0.5 * math.log( (1 + math.sin(lat_rad)) / (1 - math.sin(lat_rad)) )
    return int(round((height / 2) - (width * merc / (2 * math.pi))))
{% endhighlight %}

where *width* and *height* are the size in pixels of the flat projection. The formula works fine, translating the reference from Wolfram to the *get_y* function was simple enough, but the reason behind some details of the function found on StackOverflow (e.g. multiplying *width* by 1.5) seemed a bit arbitrary to me and I was too lazy to find the answers.


Turns out my Postgresql database also has PostGIS extensions installed, so I've decided to put them at use. I found that what we usually simply call *lng-lat* has a formal definition with the standard [WGS84][wgs_wiki], this mapping to [PostGIS' spatial reference][srid] id.4326. On the other hand, the Mercator Projection is also a standard transformation known as [EPSG:3785][epsg] mapping to PostGIS id.3785 (same id, thank god).


It's then possible to transform a WGS84 reference to EPSG:3785 by calling PostGIS functions directly in the SQL query:

{% highlight sql %}
select
    t.latitude,
    t.longitude,
    ST_X(ST_Transform(ST_SetSRID(ST_Point(t.longitude, t.latitude),4326),3785)) as x,
    ST_Y(ST_Transform(ST_SetSRID(ST_Point(t.longitude, t.latitude),4326),3785)) as y,
    created
from tweet as t
{% endhighlight %}

nice! just be aware that transforming lng-lat to EPSG:3785 returns points where the axis origin is at the centre of the map, and boundaries are [defined by the standard][epsg] as -20037508.3428, -19971868.8804, 20037508.3428, 19971868.8804. It's simple to translate the origin of axis to the top left corner and normalize the size in pixels to obtain the same results of the Python function.

uh, one last thing I never managed to permanently store in my brain: LONGITUDE is the X on the map, while LATITUDE is the Y. For me it's easier to remember by visualizing the equivalence X-Y -> LNG-LAT.

## References

- [Tweet Objects][tweet_obj]
- [Mercator Projection on Wiki][merc_wiki]
- [Mercator Projection formula on Wolfram][merc_wolfram]
- [conversion function found on StackOverflow][stack_conv]
- [World Geodetic System on Wiki][wgs_wiki]
- [PostGIS Spatial reference][srid]
- [EPSG:3785 Reference][epsg]

[tweet_obj]: https://dev.twitter.com/overview/api/tweets "Tweet Objects"
[merc_wiki]: http://en.wikipedia.org/wiki/Mercator_projection "Mercator Projection on Wiki"
[merc_wolfram]: http://mathworld.wolfram.com/MercatorProjection.html "Mercator Projection formula on Wolfram"
[stack_conv]: http://stackoverflow.com/questions/16080225/convert-lat-long-to-x-y-coordinates-c "StackOverflow conversion function"
[wgs_wiki]: http://en.wikipedia.org/wiki/World_Geodetic_System "World Geodetic System on Wiki"
[srid]: http://postgis.net/docs/using_postgis_dbmanagement.html#spatial_ref_sys "PostGIS Spatial reference"
[epsg]: http://spatialreference.org/ref/epsg/popular-visualisation-crs-mercator/ "EPSG:3785 Reference"
