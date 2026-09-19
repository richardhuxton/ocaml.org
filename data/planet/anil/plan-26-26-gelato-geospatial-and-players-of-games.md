---
title: '.plan-26-26: Gelato, geospatial, and players of games'
description: Spoke at CHIA's annual conference on AI for a changing world, as well
  as the first Cloud-Native Geospatial Forum outside the US, and started moving TESSERA's
  embeddings onto Source Cooperative.
url: https://anil.recoil.org/notes/2026w26
date: 2026-06-28T00:00:00-00:00
preview_image: https://anil.recoil.org/images/chia26-3.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>A blistering heatweave threw everything off this week; we even had a big power
cut through central Cambridge that drove many of us to seek haven in gelato
shops.</p>
<p>While melting, I spoke at <a href="https://anil.recoil.org/news.xml#speaking-at-the-annual-chia-conference">CHIA's annual conference</a> in the Cambridge Union on AI for Science, and also wrote up the <a href="https://anil.recoil.org/news.xml#cloud-native-geospatial-forum">Cloud-Native Geospatial Forum</a> London notes, the first such gathering outside the US. Plus the usual <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a> at the end.</p>
<p><img src="https://anil.recoil.org/images/chia26-3.webp" alt="%c" title="On stage at the Cambridge Union at the annual CHIA conference (image credit: Anna Korhonen)"></p>
<h2><a href="https://anil.recoil.org/news.xml#speaking-at-the-annual-chia-conference" class="anchor" aria-hidden="true"></a>Speaking at the annual CHIA conference</h2>
<p>I gave a talk in the <a href="https://cus.org/">Cambridge Union</a> for the <a href="https://www.chia.cam.ac.uk/">CHIA</a> <a href="https://www.ai.cam.ac.uk/events/chia-s-annual-conference-ai-for-a-changing-world/">annual conference</a> about "AI for a changing world".  I spoke there on <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> and our work on using it to find biodiversity worldwide. Afterwards, there was a panel hosted by <a href="https://www.arct.cam.ac.uk/staff/dr-ramit-debnath-mphil-esd-phd-gates-cantab">Ramit Debnath</a> about what young scientists need to consider with the advent of AI in the field.</p>
<p><img src="https://anil.recoil.org/images/chia26-1.webp" alt="%rc">
It's definitely a strange moment to be giving career advice to budding scientists. On one hand, data driven machine learning has opened many, many doors to finding <a href="https://royalsociety.org/news-resources/projects/science-in-the-age-of-ai/">new discoveries</a>. But on the other hand, it's never been a worse time to be <a href="https://digitaleconomy.stanford.edu/publication/canaries-in-the-coal-mine-six-facts-about-the-recent-employment-effects-of-artificial-intelligence/">young and job hunting</a> due to so many entry level jobs disappearing abruptly.  Still, the Cambridge students are as upbeat and as full of energy as ever, so I'm confident that the youth will find their way just fine; I just wish we could do more to help from the professorial end!</p>
<p>On a similar note, I was delighted to see the happy faces of our Pembroke undergraduates upon finishing their exams. They've all worked really hard this year and I'm delighted for them irrespective of whatever the results say! I did run into Emily, Shrey and Sophie at 6am when I was out for my morning jog and they were just returning from St John's May Ball...</p>
<p><img src="https://anil.recoil.org/images/cam-mayball-early.webp" alt="%c" title="Is it early to be out, or late, or both?"></p>
<h2><a href="https://anil.recoil.org/news.xml#cloud-native-geospatial-forum" class="anchor" aria-hidden="true"></a>Cloud Native Geospatial Forum</h2>
<p>I wrote up <a href="https://anil.recoil.org/notes/cng-london-2026">notes from the Cloud-Native Geospatial Forum</a>, the first such gathering outside the US, held in the Jellicoe during London Climate Action Week. It was a cracking collection of 50 practitioners geeking out over geospatial coordinate systems and Zarr access patterns and that sort of thing. My favourite talk was the <a href="https://www.barriosvisibles.org/en">Barrios Visibles</a> work that surfaced 3.4 million people missing from Argentina's official record of informal settlements! The overall theme that came up repeatedly was provenance and trust, which aligned with last week's <a href="https://anil.recoil.org/notes/2026w25">PROPL</a> discussions too.</p>
<p>After the event was done, Isaac Corley is helping me out with a transfer to the <a href="https://source.coop">Source Coop</a> of the TESSERA embeddings. This will, I hope, solve a big headache we have with distributing the growing number and variations of the core model. We've been working on the <a href="https://www.tunbury.org/2026/06/26/week-26-2025/">v2 release</a> which will be even more data (and excitement) when out soon!</p>
<p><img src="https://anil.recoil.org/images/cng-london26-13.webp" alt="%c" title="A full house at CNG London"></p>
<h2><a href="https://anil.recoil.org/news.xml#hacking-on-eio" class="anchor" aria-hidden="true"></a>Hacking on Eio</h2>
<p>I continued to port code over to Eio from my internal trees, most notably with a nice HTTP client capability. I haven't quite had a chance to finish this up well enough to publish yet, but will do so when back from holiday in a few weeks. I've also made progress on Eio Windows, but got a bit stuck with getting in a rabbit hole with IORing and RIO. I've decided to stick to IOCP for the first refresh and save the fancy ring stuff for later on.</p>
<p>Meanwhile, <a href="https://roscidus.com">Thomas Leonard</a> kindly reviewed and merged my longstanding <a href="https://github.com/ocaml-multicore/eio/pull/575">Eio sockopts PR</a> and I also debugged <a href="https://github.com/ocaml-multicore/eio/pull/531">FreeBSD ptys for that feature</a> to add pty support into Eio processes directly.</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun Links</h2>
<ul>
<li>This week's Signals and Threads is on <a href="https://signalsandthreads.com/building-a-data-warehouse-from-scratch/">Building a Data Warehouse from Scratch</a> and had the neat tidbit that JS builds databases with temporarily kill switches built in so they don't lose control of the on/off at any time. This is kind of different from the usual 'distribute and stay alive at any cost' that Kubernetes encourages...</li>
<li>I enjoyed the <a href="https://www.bbc.com/audio/play/m002y1jd">Last Call</a> on Amol Rajan's podcast, with the sad fact that 2 pubs are closing every day (!) in the UK now. Apparently in some places, a pint (about six quid) only returns a 2% margin to the pub after costs are taken into account, so it's no surprise they're not viable right now.</li>
<li>Our book club went through <a href="https://en.wikipedia.org/wiki/The_Player_of_Games">The Player of Games</a> this time. It was really fun re-reading it after many years, and I've always been a bit surprised more Iain M Banks books aren't picked up for a TV or film adaptation. This one is particularly perfect, especially when they go to the fire world towards the end!</li>
</ul>
<p>I'm on vacation for the next couple of weeks heading up to the deep north to find some arctic foxes. Stay cool everyone!</p><h1>References</h1><ul><li>Madhavapeddy (2026). .plan-26-25: Planetary scale plans, Windows file-descriptor scale problems. <a href="https://doi.org/10.59350/b3vvx-n70" target="_blank"><i>10.59350/b3vvx-n70</i></a></li></ul>
