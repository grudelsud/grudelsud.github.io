---
layout: post
title: jquery+php regex checker and reverse geo-coding service
date: 2012-03-05 17:10:10.000000000 +00:00
categories:
- development
tags:
- jquery
- PHP
status: publish
type: post
published: true
---
<p>It happened a million times: I had to check a regex in PHP and did not have a handy tool to see the result. I usually linger on Rubular, which is a great service but based on Ruby, and sometimes the results are a bit different [update 2015 - just use <a href="https://regex101.com/">regex101</a>, best tool around]. I decided to implement a simple regex checker similar to Rubular, but based on php preg_match.</p> 

<p>Another service that I need to use daily is reverse geo-coding, adding an easy way to find latitude and longitude of a specific point on the map, again there are tons of services around, but none is really what I needed. So I decided to develop my own, and share them (you might break them easily, they're just simple tools for manual use, hence there are API limits, and security wasn't my focus, so please don't hit them too hard). I might add a few things from time to time, depending on my needs, but feel free to drop me a line if you either want to contribute or feel something really missing.</p>

<p>
	
	<img class="aligncenter size-medium wp-image-351" title="php+jquery regex checker" src="/images/php+jquery-regex-checker-544x395.png" alt="" width="544" height="395" />

	<img class="aligncenter size-medium wp-image-352" title="reverse geo-coding" src="/images/reverse-geo-coding-544x395.png" alt="" width="544" height="395" />

these applications are no longer updated and will be discontinued soon, keeping the old URLs here just for the records</p>
<ul>
	<li>http://dev.londondroids.com/tools/index.php/main/preg_match</li>
	<li>http://dev.londondroids.com/tools/index.php/main/geocode</li>
</ul>
