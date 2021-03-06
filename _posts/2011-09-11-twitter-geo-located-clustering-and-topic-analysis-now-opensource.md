---
layout: post
title: Twitter geo-located clustering and topic analysis, now opensource!
date: 2011-09-11 17:59:42.000000000 +01:00
categories:
- development
- research
tags:
- FOM
- Telecom Italia
- Working Capital
status: publish
type: post
published: true
---
<p>A year has passed since the beginning of the trial of Flux of MEME, the project I have presented during the <a href="http://www.workingcapital.telecomitalia.it/2010/08/flux-of-meme/">Working Capital tour</a>, and it is now time to analyze what has been learned and show what has been developed to conclude this R&amp;D phase and deliver results to Telecom Italia.</p>
<p><!--more--></p>
<h4 dir="ltr">the initial idea</h4>
<p>It’s worthwhile giving a quick description of the context: Twitter is a company formed in 2006 which has received several rounds of funding by venture capitals over the past few years, this leading to today's valuation of $1.2B, still during the summer of 2009 the service was not yet mature and widespread as it may look now. At that time the development of the Twitter API had just started, this probably being one of the few sources, if not the only one, for geo-referenced data. The whole concept of communication in the form of public gossip, mediated by a channel that accepts 140 characters per message, was appearing in the world of social networks for the first time.<br />
This lead to the base idea of crunching this data stream, which most importantly include the geographical source, then summarize the content, so as to analyze the space-time evolution of the concepts described and, ultimately, make a prediction of how they could migrate in space and time.</p>
<h4 dir="ltr">A practical use</h4>
<p>It could allow you to control and curb the trend of potentially risky situations (such as social network analysis has been useful during the recent riots in London) or even define marketing strategies targeted to the local context.</p>
<h4 dir="ltr">The implementation</h4>
<p>A consistent initial phase of research allowed to have an overview on different aspects: the ability to capture the information from Twitter, the structure of captured data, the ability to obtaining geo-located information, the classification of languages of the tweets, the enrichment of content through discovery of related information, the possible functions for spatial clustering, the algorithms for topic extraction, the definition of views useful for an operator and finally the ability to perform a trend analysis on the information extracted. All of this has resulted in a substantial amount of programming code, its outcome being a demonstrator for the validity of the initial theory.</p>
<p><img class="size-medium wp-image-301" title="03_search_result" src="/images/03_search_result-544x341.png" alt=""  />
space-time evolution of the concept &quot;earthquake&quot; in a limited subset of data captured during the period May 2011"</p>

<p><img class="size-medium wp-image-302" title="07_dragzoom_stat" src="/images/07_dragzoom_stat-544x341.png" alt="" />
distribution of groups of tweets source languages ​​over Switzerland and northern Italy</p>
<p><span style="font-weight: bold;">The future of the project</span></p>
<p>The development done so far has had two important results: firstly, it allowed to demonstrate the validity of the initial idea, and secondly it has revealed the requirements needed by the system to be fully functional. The main problem lays in the architecture implemented for the demonstrator, which at the moment relies on a limited amount of data (for obvious reasons of availability of resources): this immediately proved the necessity of scaling up the application environment in a more complex architecture for distributed computing  The market and/or Telecom Italia will eventually decide if this second phase of development can be faced.</p>
<h4 dir="ltr">References</h4>
<ul>
<li>Source code and documentation - <a href="https://github.com/grudelsud/fom/">https://github.com/grudelsud/fom/</a></li>
<li>Algorithm for the classification of space - <a href="http://en.wikipedia.org/wiki/Cluster_analysis">http://en.wikipedia.org/wiki/Cluster_analysis</a></li>
<li>Algorithm for extracting topic - <a href="http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation">http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation</a></li>
</ul>
