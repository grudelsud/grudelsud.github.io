---
layout: post
title: Visualizing Cluster of Tweets in London during the Royal Wedding
date: 2012-02-01 09:52:22.000000000 +00:00
categories:
- development
- research
tags:
- FOM
- processing
status: publish
type: post
published: true
---
<p>So the little storm has arrived! I am now the proud (and tired) father of little Benjamin, and for this reason blocked at home with little or no time for doing anything but changing nappies and cooking super-proteic food for Val. But somehow I found some time amuse myself with a little piece of Processing and wrote a simple code to visualize tweets on a map of London during the Royal Wedding. Easy enough to foresee, tweets during the day are creating nice clusters around Buckingham Palace, Westminster Abbey and the super posh hotel where the Middleton's used to stay.</p>
<p>Here is the result:<br />
<iframe width="540" height="304" src="http://www.youtube.com/embed/xaofMNCP_yY?rel=0" frameborder="0" allowfullscreen></iframe></p>
<p>To display this data I reused the information stored during the first phase of my Flux of MEME project, fetched from twitter with the Streaming API implementation in its Java flavour twitter4j. Processing is reading the information in XML directly from the database, hence a little PHP backend is providing the XML descriptor for all the posts locations.</p>
