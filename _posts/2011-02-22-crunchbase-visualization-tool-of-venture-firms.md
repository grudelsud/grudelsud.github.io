---
layout: post
title: Crunchbase visualization tool of Venture Firms
date: 2011-02-22 14:33:42.000000000 +00:00
categories:
- development
tags:
- Crunchbase
status: publish
type: post
published: true
---
<p><img class="aligncenter size-medium wp-image-218" title="venture-firm-visualization-tool-crunchbase" src="/images/venture-firm-visualization-tool-crunchbase-544x343.png" alt="" width="544" height="343" /></p>
<p>Over the past few days a couple of friends asked me more or less the same thing: retrieve a collection of Venture Firms and visualize a simple subset of important data, such as geo location, average investment, and short description. I have hence decided to spend a little time with some PHP/Javascript and implemented a simple application.</p>

<p>The application crawls pages from <a class="zem_slink" title="CrunchBase" rel="homepage" href="http://www.crunchbase.com/">Crunchbase</a> to retrieve a set of financial organizations, then update their locations using their API. Details on each organization are fetched through an AJAX request to Crunchbase's API after a marker on the map is clicked. A vector graph is displayed showing the list of the investments of the firm.</p>
<p>The application uses Raphael JS for creating the <a title="Scalable Vector Graphics" rel="wikipedia" href="http://en.wikipedia.org/wiki/Scalable_Vector_Graphics">SVG</a> graph, therefore works on Firefox and Safari only [EDIT: it was then updated to google chart api to maximize browser compatibility and now fully compliant].</p>
