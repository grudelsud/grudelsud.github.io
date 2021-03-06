---
layout: post
title: Anonymous functions with PHP / Eclipse
date: 2011-06-17 19:33:48.000000000 +01:00
categories:
- development
tags:
- PHP
status: publish
type: post
published: true
---
<p>Eclipse is a great IDE, I don't know how could I live without it, but when it comes to PHP development, the PDT plugin shows some flaws unfortunately. It may happen that you need to declare an <a class="zem_slink" title="Anonymous function" href="http://en.wikipedia.org/wiki/Anonymous_function" rel="wikipedia">anonymous function</a> and get a "compiler" error, as shown below:</p>
<p><img class="aligncenter size-full wp-image-235" title="eclipse_php" src="/images/eclipse_php.png" alt="php fake errors in eclipse pdt" width="544" height="100" /></p>
<p>just ignore it, the code is working perfectly well, so declaring an anonymous function will not give any problem, you just need to cope with the little red error message until the end of the project (or declare the callback as an identified function).</p>
<p>Below the code to declare the inner function for array_filter:</p>
<pre class="brush:php">foreach( $sem_clusters as $sem_cluster ) {
	$terms = explode(';', preg_replace('/\"/', '', $sem_cluster-&gt;terms_meta));
	$terms = array_filter( $terms, function($value) { return strlen($value) &gt; 2; });
	$tf_idf = array_merge( $tf_idf, $terms );
}</pre>
