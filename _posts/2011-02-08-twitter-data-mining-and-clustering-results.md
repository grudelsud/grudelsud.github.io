---
layout: post
title: Twitter data mining and clustering results
date: 2011-02-08 12:15:16.000000000 +00:00
categories:
- research
tags:
- FOM
- Telecom Italia
- Working Capital
status: publish
type: post
published: true
---
<p><a href="http://tom.londondroids.com/wp-content/uploads/2011/02/twitter_topic_clusters.png"><img class="aligncenter size-medium wp-image-205" title="twitter_topic_clusters" src="/images/twitter_topic_clusters-544x318.png" alt="" width="544" height="318" /></a></p>
<p>It has been a while since my last update on the Flux of MEME project, but my team was not idle: we have worked a lot both on algorithms and architecture, and now it is time to analyse the first results. Thanks to the research grant awarded by @workingcapital, a first prototype of data mining, topic extraction and clustering application was developed, using Twitter as its main data source.<br />
<!--more--><br />
Our software is structured in 3 main modules:</p>
<ol>
<li>data acquisition: uses <a href="http://dev.twitter.com/pages/streaming_api">Twitter streaming API</a> to fetch contents and store them on our database. Data is filtered in order to store geo-located references only, representing around 1% of the total amount of tweets. Data elements are "enriched" when possible, thanks to a web crawler that fetches content referenced by links inside the tweets body</li>
<li>data analysis: performs a 2 step iteration on content elements, first creating a set of geo-located clusters with <a href="http://en.wikipedia.org/wiki/K-means_clustering">K-means algorithm</a>, then extracting topics with <a href="http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation">Latent Dirichlet allocation (LDA)</a>. A lot of work still needs to be done on this module in order to increase meaningfulness of results and analysis of clusters correlation and prediction</li>
<li>data visualization: it is an AJAX based web front-end for analysis and verification of experimental results</li>
</ol>
<p>The algorithms and architecture are still undergoing a lot of development, but the first results are really encouraging and we have planned another 6 months of activities under the current grant. [EDIT] please check below this non-technical presentation of the main project features.</p>
<div class="prezi-player">
<style type="text/css" media="screen">.prezi-player { width: 540px; } .prezi-player-links { text-align: center; }</style>
<p><object id="prezi_xgcgigujvjru" name="prezi_xgcgigujvjru" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" width="540" height="400"><param name="movie" value="http://prezi.com/bin/preziloader.swf" /><param name="allowfullscreen" value="true" /><param name="allowscriptaccess" value="always" /><param name="bgcolor" value="#ffffff" /><param name="flashvars" value="prezi_id=xgcgigujvjru&amp;lock_to_path=0&amp;color=ffffff&amp;autoplay=no&amp;autohide_ctrls=0" /><embed id="preziEmbed_xgcgigujvjru" name="preziEmbed_xgcgigujvjru" src="http://prezi.com/bin/preziloader.swf" type="application/x-shockwave-flash" allowfullscreen="true" allowscriptaccess="always" width="540" height="400" bgcolor="#ffffff" flashvars="prezi_id=xgcgigujvjru&amp;lock_to_path=0&amp;color=ffffff&amp;autoplay=no&amp;autohide_ctrls=0"></embed></object>
<div class="prezi-player-links">
<p><a title="results achieved during the 1st semester of research - keywords: semantic web, twitter, clustering, geo, topic extraction" href="http://prezi.com/xgcgigujvjru/flux-of-meme-description-of-work/">Flux of MEME - Description of Work</a> on <a href="http://prezi.com">Prezi</a></p>
</div>
</div>
<p>We are completely open to discussion, recommendations and funding proposals, so anyone interested in this topic, please feel free to enquire to learn more about this project.</p>
