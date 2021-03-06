---
layout: post
title: Particle Animations with Flint
date: 2010-05-25 18:10:43.000000000 +01:00
categories:
- development
tags:
- as3
- projects
status: publish
type: post
published: true
---
<p>In the last few week I have been working on a project that involved the development of a particle animation system. The project is still under development and we are working under NDA: for this reason unfortunately I still cannot post any details about it, even though I will post details as soon as it goes live.</p>
<p>What I can definitely do is show the tests I had to develop using <a href="http://flintparticles.org/">Flint particle animation system</a>, developed by @Richard_Lord and give a brief explanation of the code, since I saw that a lot of people are having troubles with, to say the less, a little bit sparse documentation.</p>
<p>Let's show the results first, it is a tweaking of the logo tweener example found on the website.</p>
<p>[swf src="http://tom.londondroids.com/wp-content/uploads/2010/05/ParticleTest.swf" width=400 height=400 version=10]</p>
<p><!--more-->It was not rocket science at all, since it was only matter of tweaking an already made example, still I noticed from the forum that loads of people got stuck while introducing some changes, and I had to go more in depth of some aspects related to EmitterEvents, ParticleEvents and Easing.</p>
<p>I wanted to use extensively the TweenToZone effect for a series of bitmaps which had to be displayed in the .swf, for this reason I decided it was better to create a class and reuse it when needed. The emitter can also reuse existing particles instead of blasting a complete new set of them, the code is the following.</p>
<pre class="brush:as3">public class Tweener extends Emitter2D
{
    public function Tweener( init:Boolean, bitmapFrom:Bitmap, bitmapTo:Bitmap, particles:uint, lifetime:uint, color1:uint, color2:uint )
    {
        if( init ) {
            counter = new Blast( particles );
            addInitializer( new ColorInit( color1, color2 ) );
            addInitializer( new Position( new BitmapDataZone( bitmapFrom.bitmapData, 0, 0 ) ) );
        }

        addInitializer( new Lifetime( lifetime ) );
        addAction( new Age( Elastic.easeInOut ) );
        addAction( new TweenToZone( new BitmapDataZone( bitmapTo.bitmapData, 0, 0 ) ) );
    }
}</pre>
<p>Be aware of the fact that the Age initializer is using the easing function Elastic.easeInOut, and it is working properly if, and only if, the imported package is org.flintparticles.common.energyEasing.*</p>
<p>Another tricky problem was to create a loop of tweens without necessarily creating a listener function for each emitter, like the example shown in the website. And moreover, I did not figure out an elegant method to retrieve the instance of the emitter which was calling the ParticleDead event. For this reason, I decided to use the EmitterEvent EmitterEmpty and tweak the listener in order to restart the correct emitter as soon as its execution ends. The code is thus easily extendable, using arrays, to an indefinite amount of emitters and related events.</p>
<pre class="brush:as3">public function ParticleTest()
{
    stage.scaleMode = StageScaleMode.NO_SCALE;
    stage.align = StageAlign.TOP_LEFT;

    super();

    _renderer = new PixelRenderer( new Rectangle( 0, 0, 400, 400 ) );
    _renderer.addFilter( new ColorMatrixFilter( [ 1,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,0.92,0 ] ) );
    addChild( _renderer );

    runParticleDemo();
}

private function runParticleDemo() : void {

    var textBmp:Bitmap = new text();
    var imageBmp:Bitmap = new image();

    _loop = 0;

    _emitterLoop0 = new Tweener(true, textBmp, imageBmp, 8000, 9, 0xffff0098, 0xffffffff);
    _emitterLoop1 = new Tweener(true, imageBmp, textBmp, 8000, 9, 0xffff0098, 0xffffffff);

    _renderer.addEmitter( _emitterLoop0 );
    _renderer.addEmitter( _emitterLoop1 );

    _emitterLoop0.addEventListener( EmitterEvent.EMITTER_EMPTY, loopParticleEv );
    _emitterLoop0.start( );
}

public function loopParticleEv( ev:EmitterEvent ): void
{
    switch( _loop % 2 ) {
    case 0:
        _emitterLoop0.removeEventListener( EmitterEvent.EMITTER_EMPTY, loopParticleEv );
        _emitterLoop0.stop();
        _emitterLoop1.addEventListener( EmitterEvent.EMITTER_EMPTY, loopParticleEv );
        _emitterLoop1.start();
        break;
    case 1:
        _emitterLoop1.removeEventListener( EmitterEvent.EMITTER_EMPTY, loopParticleEv );
        _emitterLoop1.stop();
        _emitterLoop0.addEventListener( EmitterEvent.EMITTER_EMPTY, loopParticleEv );
        _emitterLoop0.start();
        break;
    }
    _loop++;
}</pre>
<p>As should be clear enough from the code, listeners are removed from the emitter that has ended its execution and subsequently added to the emitter that is going to start. A global variable named _loop take care of switching to the proper emitter initialization function.</p>
