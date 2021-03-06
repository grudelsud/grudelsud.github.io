---
layout: post
title: AIS and ONAV Wine tasting application for Android - ALPHA
date: 2010-06-23 12:42:40.000000000 +01:00
categories:
- development
tags:
- android
- wine &amp; food
status: publish
type: post
published: true
---
<p>I started a personal project months ago, aimed at realizing a web and mobile infrastructure for wine tasting notes. During the same period I quit my old job, moved from my old flat in Florence to my wife's in London, found a new flat and a new job, moved again to the new flat, changed computer (the old one being back to the lab where I used to work) and started a couple of new projects, quoted in my previous posts.</p>
<p>I think it's enough to justify that my original project - for which I found what I think a cute name: "Cellarium" - is currently in stand by. Still I think it's a real shame, mainly for two reasons:</p>
<ol>
<li>there are not official AIS and/or ONAV wine tasting applications for mobiles in the cloud (or at least I didn't found them)</li>
<li>being now in London, I must say that Italian wines are under estimated and are really hard to find (no, I'm not talking about the crap you usually pick up around)</li>
</ol>
<p>I decided thus to put everything under github, and if a brave volunteer fancies a contribution, can easily download the source code here: <a href="http://github.com/grudelsud/com.londondroids.cellarium">http://github.com/grudelsud/com.londondroids.cellarium</a><a rel="attachment wp-att-124" href="http://tom.londondroids.com/2010/06/ais-onav-wine-tasting-application-for-android-alpha/cellarium_vino/"><img class="aligncenter size-medium wp-image-124" title="cellarium_vino" src="/images/cellarium_vino-544x397.png" alt="" width="544" height="397" /></a></p>
<p><!--more-->If you are brave enough to read through the rest of the post, you will discover the following:</p>
<ul>
<li>code is completely undocumented, shame on me, but I didn't have the time to do anything else, it's just there and you can use it at your own risk;</li>
<li>wines and wine tasting notes are stored in a sqlite database (stored in the mobile device) and I don't even remember its structure, the only way to retrieve how things are done is to take a look at the two inner classes defined inside CellariumProvider.java</li>
<li>saving and updating wines work fine, while wine notes still need a lot of coding, both from a data persistence perspective and interface implementation.</li>
</ul>
<p>The application is intended to help while writing down wine tasting notes for both AIS and ONAV standards, as depicted in the following.</p>
<p><a rel="attachment wp-att-125" href="http://tom.londondroids.com/2010/06/ais-onav-wine-tasting-application-for-android-alpha/cellarium_scheda/"><img class="aligncenter size-medium wp-image-125" title="cellarium_scheda" src="/images/cellarium_scheda-544x397.png" alt="" width="544" height="397" /></a></p>
<p>I'd be happy to help/contribute and finish the application, volunteers appreciated. If no one interested in this, I'll probably finish it when I'll retire, using Android v.714.23 on a nuclear-powered-artificial-intelligence-smartphone produced in Mars by Cyberdyne systems.</p>
