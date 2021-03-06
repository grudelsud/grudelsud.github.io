---
layout: post
title: Time for my own AngularJS + Gulp + Browserify boilerplate
date: 2014-05-02 11:25:20.000000000 +01:00
categories:
- development
tags: []
status: publish
type: post
published: true
comments: true
---
<p><strong>tl;dr</strong>: just visit my <a href="https://github.com/grudelsud/angularjs-gulp-browserify-boilerplate" target="_blank">Github repo</a> for a simple boilerplate with basic documentation.</p>
<p>I have found 36,749 examples of boilerplates using a miscellanea of AngularJS, Gulp, Browserify, Grunt, RequireJS and all the usual suspects.</p>
<p>But none of them actually did what I needed, although I reckon it is quite simple: define Angular modules in separate files, so the code is nice and clean and all the various services, directives and controllers are pluggable without massive headaches.</p>
<p>At the same time, having completely ignored the existence of Grunt so far (thanks to frontend guys who took care of this), I thought it was a good time for a fresh start with the latest and more fashionables Gulp and Browserify.</p>
<p>I ended up creating my own boilerplate (or seed, if you like), it works defining angular modules:</p>
<pre class="brush:javascript">require('../../vendors/angular/angular');

module.exports = angular.module('boilerplate.controllers', []);

// Just define the list of controllers here while developing the whole app (in this boilerplate, just one):
require('./welcome.js');</pre>
<p>using them:</p>
<pre class="brush:javascript">require('../../vendors/angular/angular');

var module = require('./_module_init.js');

module.controller('WelcomeCtrl', ['$scope', function($scope) {
console.log('welcome controller');
$scope.greetings = 'Hey!'
}]);</pre>
<p>and eventually compiling them in a single js</p>
<pre class="brush:javascript">gulp.task('scripts', function() {
  return gulp.src('scripts/main/app.js')
    .pipe(browserify({
      insertGlobals : true,
      debug : !isProduction
    }))
    .pipe(gulp.dest('build'))
  });</pre>
<p>if anyone is interested, feel free to clone and submit pull requests!</p>
