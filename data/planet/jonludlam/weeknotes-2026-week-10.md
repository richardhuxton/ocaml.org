---
title: Weeknotes 2026 week 10
description:
url: https://jon.recoil.org/blog/2026/03/weeknotes-2026-10.html
date: 2026-03-09T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>Here are my weeknotes for the last week, while I'm still writing up some more focused posts on some specific topics - like the experience of putting everything in a monorepo to create this site, and more notes on Claude and Agentic coding in general, and its impact on the world of software. But for now, here's what I've been up to.</p>
<h2>What did I do?</h2>
<ul>
<li>
<p>New site design. The old site was a bit of a mess and was simply reusing odoc's default default styling. I've also rearranged the content a bit to make it more navigable and cohesive.</p>
<p><img src="https://jon.recoil.org/old.png" alt="old.png">
<img src="https://jon.recoil.org/new.png" alt="new.png"></p>
</li>
<li>
<p>TESSERA in the browser is a <a href="https://tee.cl.cam.ac.uk/">hot</a> <a href="https://anil.recoil.org/notes/2026w10">topic</a> right now, so I've applied the work I've been doing with x-ocaml, js_top_worker and odoc plugins to make a <a href="https://jon.recoil.org/notebooks/interactive_map.html">TESSERA notebook</a> that's based on the <a href="https://github.com/ucam-eo/tessera-interactive-map">example notebook</a>.</p>
<p><img src="https://jon.recoil.org/tessera.png" alt="tessera.png"></p>
</li>
<li>
<p>I was interested in whether we'll be able to do inference in reasonable time using these notebooks. <a href="https://onnx.ai/">ONNX</a> has a web version of its runtime, so I got Claude to make some bindings, and checked it was working by doing a sentiment analysis notebook. This is working nicely, so the next step is to do something a bit more useful. Try it <a href="https://jon.recoil.org/reference/onnxrt/sentiment_example.html">here</a>.</p>
</li>
<li>
<p>The docs CI was again causing problems. This time it had decided that it had never built anything, and therefore needed to rebuilt the entire world. However, despite being set up as a custom dedicated runner, all its jobs were queued waiting to start. It turned out that the runner paused itself when the docker partition reached 70%. This was a little surprising on two counts - firstly we don't actually use docker for running the jobs, we use obuilder, which doesn't share space with docker. Secondly, with that in mind, how did it get to 70%? It turned out to be the job logs - including 250 gigs of older logs from a previous instance. Simply blowing those away caused everything to restart and so it's now live again.</p>
</li>
<li>
<p>I met up with <a href="https://ancazugo.github.io/">Andrés C. Zúñiga-González</a> to have a chat about how he's using interactive maps and notebooks. He pointed me at his <a href="https://ancazugo.github.io/blog.html">blog</a>, some of which which is using <a href="https://quarto.org/">quarto</a>, which he rates very highly. An <a href="https://ancazugo.github.io/posts/2025-11-16-tessera_example.html">example of quarto output</a>.</p>
</li>
<li>
<p>Our group seminar this week was <a href="https://tombearpark.com/">Tom Bearpark</a> who talked about his proposed 'Carbon at Risk' measure in order to compare diverse ways of removing carbon from the atomsphere to help with the carbon removal market.</p>
</li>
</ul>
<h2>What's next?</h2>
<ul>
<li>More writing before more coding, I think.</li>
</ul>

    
