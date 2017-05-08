---
layout: post
type: post
title: Back from PyData 2017 London
date: 2017-05-08
categories:
- miscellanea
tags: []
status: publish
published: true
---

Back from [#PyData2017 London](https://pydata.org/london2017/) where I gave a talk called "Show me the failures!", showing how the Data Science and Analytics team at Pirelli approach the problem of designing and implementing data products at shop floor.

Thank you again to all the crew, you guys have been amazing: venue, food, schedule were absolutely spot on. Bring it on and looking forward to coming over for #PyData2018! Below the slides and outline of my talk.

<iframe src="//www.slideshare.net/slideshow/embed_code/key/HaSC1VOWDJKDww" width="550" height="450" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

## Description
The Data Science and Analytics group at Pirelli has to deal with factories' day to day that can't be further from the aseptic crunching of data from a keyboard in an office. Our group took the lift, went down at shop floor and started asking questions to try and make their life better: turns out questions flowed the other way round and results were startling.

## Abstract
Pirelli has a 140 year old tradition of manufacturing with 20 factories across 14 countries and headquarter office in Milan. Production flows, logistic, machinery and the whole extended value chain has morphed through decades across a broad range of needs and circumstances.

The creation of a Data Science and Analytics department at the beginning of 2016 has the goal of speeding up change and innovation, starting from areas that are harder to tackle. Some of the most interesting challenges include:

bring data products at shop floor to increase efficiency while being aware of UX principles
keep 2-sided communication alive with wide number of actors, particularly with IT, quality and engineering
encourage active participation by providing accessible analytics tools and an internal Academy training program
activate the virtuous circle of prototyping, feasibility check and production releases for sound product lifecycles
introduce Agile development methodologies in traditional waterfall environments
shape a roadmap with principal stakeholders starting from off-line through live analysis and heading to ahead-of-time predictions
opening a steady communication channel across groups is progressively eroding barriers between white and blue collars, allowing teams to better understand each other requirements and kicking off a broader conversation. At the end of the first year since releasing the first prototype, there is much more on the plate, and groups are now more familiar with concepts of User Experience, release lifecycle, data exploration and agile development.

## Outline
In this talk we are going to show the data science team approach to prototyping and implementation of data products for Pirelli factories, both at shop floor and quality / engineering offices. Different needs or - taking a UX approach - different personas, lead to different outcomes: from large displays mounted in wide warehouses to detailed descriptions of statistical distributions, from near real-time processing of streams of data coming out of sensors to large computations for statistical models made on millions of rows stored in sql tables.

The sheer variety of technologies involved in the process is probably the biggest challenge when deploying at production level: aside standard data processing and machine learning packages, such as Pandas and scikit-learn, our Flask and Django based web infrastructures interact with MsSQL servers, JBoss data virtualisers, a Hadoop cluster and Oracle data warehouse, responsively adapting their output for different contexts with Angular and React front-ends.

The introduction of data products has triggered a little revolution allowing to improve and widen the offer of internal training via an Academy program, in search of competences inside the factories for all people willing to open an editor and write some Python and Javascript.

P.S. link to a gist with my notes from the conference [here](https://gist.github.com/grudelsud/a40704fbd76c03e2870792cb3674fabe)
