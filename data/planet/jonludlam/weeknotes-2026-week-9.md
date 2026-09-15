---
title: Weeknotes 2026 week 9
description:
url: https://jon.recoil.org/blog/2026/03/weeknotes-2026-09.html
date: 2026-03-02T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>Let's make this really terse!</p>
<h2>What did I do?</h2>
<ul>
<li>
<p>Got docs working with github actions on Anil's oxmono monorepo. Results are <a href="https://jonludlam.github.io/oxmono/">here</a>. This includes experimental support for oxcaml modes/layouts.</p>
</li>
<li>
<p>Got markdown mode output into Sherlodoc's db so you can query it - great for agents!</p>
<p><img src="https://jon.recoil.org/search.png" alt="search.png"></p>
</li>
<li>
<p>Widgets in the JS OCaml toplevels - using FRP for the interactions. The neat thing here is that using FRP via Daniel Bunzli's <a href="https://erratique.ch/software/note">note</a> library is that all the interactions are all purely functional, no refs or mutables in sight. You provide a little wrapper scripts that's run in the frontend and the interactions and send back and forth with the worker running the code where it's translated into Events and Signals. My proof-of-concept of this is a widget that works with the <a href="https://leafletjs.com/">leaflet.js</a> library:</p>
</li>
</ul>
<p><video controls="" src="./mapdemo.mov"></video></p>
<p>Demo coming soon!</p>
<ul>
<li>Consolidating all of the Odoc toplevel bits and pieces into the one monorepo. Again, demo of this coming soon!</li>
</ul>
<h2>What am I going to do?</h2>
<ul>
<li>New website!</li>
<li>Odoc plugins showcase</li>
<li>Writing writing writing writing</li>
</ul>

    
