---
title: '.plan-26-41: Hello, is it Part II you''re looking for?'
description: The Foundations of Computer Science notes are now done as a Forest with
  live OCaml toplevels, and Part II projects sorted.
url: https://anil.recoil.org/notes/2026w41
date: 2026-10-11T00:00:00-00:00
preview_image: https://anil.recoil.org/images/w41-photos.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Start of term is truly crazy around here, so I'll save longer form thoughts for later in the month when things calm down a little.</p>
<p>Most of my focus was on teaching and administration to prepare for the term.  I finished the Cambridge 1A <a href="https://anil.recoil.org/notes/focs">Foundations of Computer Science</a> course notes as a <a href="https://anil.recoil.org/notes/forester-teaching-notes">Forest</a>, complete with live in-browser OCaml toplevels. These are now on the <a href="https://www.cl.cam.ac.uk/teaching/2627/FoundsCS/forest/index/">course webpage</a> and ready for lectures next week. This may well be the last time this course (now <a href="https://www.cl.cam.ac.uk/teaching/1998/IA50.html">about three decades old</a>) runs in its current form, so I'm a little nostalgic heading into delivering it!</p>
<h2><a href="https://anil.recoil.org/news.xml#cst-part-ii-projects-this-year" class="anchor" aria-hidden="true"></a>CST Part II projects this year</h2>
<p>There is a traditional crush of emails this week every year since the returning third year CST students all have to find projects and propose them in the first week of term. There's no 'clearing service' coordination here, so I usually get around forty requests for meetings. This year will see some particularly diverse and fun projects; I'm glad to see the spirit of computing exploration is still alive and kicking with our undergrads!</p>
<ul>
<li>Anuraag Datta is designing Scour, a <a href="https://anil.recoil.org/ideas/scour-music-patterns">language for finding patterns and rule violations in musical scores</a>, supervised by Richard Sharp. This one wins the 'most original' award, as we'll be "debugging Bach"!</li>
<li>Eden Cheung is building an <a href="https://anil.recoil.org/ideas/lean-io-uring-backend">io_uring IO backend for Lean</a>, taking advantage of the executable nature of code written in Lean4. This is really to explore how we might specify shared memory protocols more mechanically, as I believe they're the future of <a href="https://anil.recoil.org/notes/icfp25-post-posix">OS interfaces</a>.</li>
<li><a href="https://github.com/Arya-Golkari">Arya Golkari</a> is porting <a href="https://pypi.org/project/spytial-diagramming/">sPyTial</a> to OCaml as a <a href="https://anil.recoil.org/ideas/spytial-ocaml-port">programmatic diagramming library</a> for ML-style data structures (including module signatures and functors). I must ping <a href="https://cs.brown.edu/~sk/">Shriram Krishnamurthi</a> about this to get the latest developments on their end!</li>
<li>Rohan Wadhawan is writing a <a href="https://anil.recoil.org/ideas/ocaml-to-shaders">compiler from OCaml to GLSL and WGSL shaders</a>, supervised by <a href="https://patrick.sirref.org">Patrick Ferris</a>.</li>
<li>Boris Hall is building an OCaml eDSL and runtime for <a href="https://anil.recoil.org/ideas/incremental-traceable-edsl">incremental computation with traceable data types</a>, also supervised by <a href="https://patrick.sirref.org">Patrick Ferris</a>.</li>
<li>Matt Occleshaw is writing an <a href="https://anil.recoil.org/ideas/ocaml-osm-dsl">embedded OCaml DSL for OpenStreetMap</a> whose queries can separate well known results from probable ones, also supervised by <a href="https://patrick.sirref.org">Patrick Ferris</a>. This is a good example of layering a language over a <a href="https://en.wikipedia.org/wiki/Folksonomy">folksonomy</a> that has somewhat shifting semantics as the users decide where to steer it.</li>
<li>Ethan Smith is building an <a href="https://anil.recoil.org/ideas/omniscient-debugger-beginners">omniscient debugger for beginner Python programmers</a>, supervised by Laurie Gale. We break the OCaml monopoly with some micro-Python here!</li>
<li>Minkyum Kim is building an <a href="https://anil.recoil.org/ideas/tessera-inverted-spatial-index">inverted spatial index for Tessera embedding search</a>, supervised by <a href="https://toao.com">Sadiq Jaffer</a>. We're aiming for sub-second searches over global embeddings on a single GPU, so there's some optimisation to do here.</li>
<li>Finley Stirk is measuring and handling <a href="https://anil.recoil.org/ideas/tessera-gbif-uncertainty">GPS uncertainty in GBIF point labels for Tessera</a>, supervised by <a href="https://mynameismwd.org">Michael Dales</a> and <a href="https://aneeshnaik.github.io/">Aneesh Naik</a>. GBIF is full of 'elephants in the ocean' due to the GPS uncertainty inherent in any observations, so this will hopefully greatly improve our habitat mapping efforts.</li>
<li>Soham Shah is continuing work on <a href="https://anil.recoil.org/ideas/autoscaling-geospatial-yirgacheffe">autoscaling geospatial computation with Python and Yirgacheffe</a>, supervised by <a href="https://mynameismwd.org">Michael Dales</a>, to make it easier for ecologists to run global-scale analyses without becoming systems experts.</li>
</ul>
<p>Next week is all about responding to the MPhil students and Part IIIs for their (longer form) projects. Fun times!</p>
<h2><a href="https://anil.recoil.org/news.xml#back-to-the-cambridge-community" class="anchor" aria-hidden="true"></a>Back to the Cambridge community</h2>
<p>We had the annual <a href="https://www.conservation.cam.ac.uk">Discover the Conservation Research
Institute</a> event, with a completely packed
room of new faces and familiar ones. Absolutely delightful atmosphere all
around.</p>
<p><img src="https://anil.recoil.org/images/w41-photos.webp" alt="%c" title="A packed Discover CRI!"></p>
<p>This was then followed by the usual three-minute group pitches by everyone, so
I grabbed a couple of candid camera shots.</p>
<p><img src="https://anil.recoil.org/images/w41-photos-2.webp" alt="%c" title="Andrew Balmford advertises the upcoming Student Conference in Conservation Science">
<img src="https://anil.recoil.org/images/w41-photos-3.webp" alt="%c" title="David Coomes explains the Centre for Landscape Regeneration"></p>
<p>I also had the pleasure of the matriculation dinner for the Pembroke College undergraduates, welcoming our four new arrivals with the traditional selfie. This was followed by a day of Director of Studies meetings with the full cohort to make sure all their projects and supervisions were in order.</p>
<p><img src="https://anil.recoil.org/images/w41-photos-4.webp" alt="%c" title="Welcome Pembroke freshers!"></p>
<p>And after that, it was over to the Computer Science department for our faculty meetings, which I am now attending again as sabbatical is over!</p>
<p><img src="https://anil.recoil.org/images/w41-photos-5.webp" alt="%c" title="The Computer Lab faculty assembles"></p>
<h2><a href="https://anil.recoil.org/news.xml#ocamls-module-language" class="anchor" aria-hidden="true"></a>OCaml's module language</h2>
<p>I also snatched a few minutes to jot down my thoughts on <a href="https://anil.recoil.org/notes/ocaml-modules-agentic">why OCaml's module language is a great fit for agentic programming</a>. This
is really just an explainer of the interface/implementation split that OCaml
has supported for 25+ years, but the post was quite widely read by a new
audience who are now using coding assistants and so switching between languages
at a rapid pace.</p><h1>References</h1><ul><li>Madhavapeddy (2025). Foundations of Computer Science. <a href="https://doi.org/10.59350/qms3q-ymn65" target="_blank"><i>10.59350/qms3q-ymn65</i></a></li>
<li>Madhavapeddy (2026). OCaml's module language is the perfect fit for agentic programming. <a href="https://doi.org/10.59350/g6hbj-qmc76" target="_blank"><i>10.59350/g6hbj-qmc76</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2026). Using Forester to turn Foundations of CS into interactive evergreen lectures. <a href="https://doi.org/10.59350/sjcvd-hb857" target="_blank"><i>10.59350/sjcvd-hb857</i></a></li></ul>
