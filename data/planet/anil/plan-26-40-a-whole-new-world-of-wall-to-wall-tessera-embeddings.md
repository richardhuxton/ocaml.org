---
title: '.plan-26-40: A whole new world (of wall-to-wall Tessera embeddings)'
description: Michaelmas term restarts, with Forester replacing my traditional printed
  FoCS notes, GeoTessera is now wall-to-wall across dual Zarr/Icechunk stores, and
  a trip to Oxford's Intelligent Earth CDT.
url: https://anil.recoil.org/notes/2026w40
date: 2026-10-04T00:00:00-00:00
preview_image: https://anil.recoil.org/images/cri-seminar-26.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I'm almost set for the start of teaching term now and meeting our newest intake
and catching up with everyone after a year away! My <a href="https://anil.recoil.org/notes/focs">1A Foundations of CS</a> starts next week, and I've decided to lock in the <a href="https://anil.recoil.org/notes/forester-teaching-notes">use of Forester</a> for this.
For the first time, we won't be
giving out the traditional <a href="https://www.cl.cam.ac.uk/teaching/2526/FoundsCS/materials.html">printed
notes</a>.
I'm a little worried about not giving away since I've seen students scribbling on them well
into their second years in the past! I'm hoping that the cross-referenced
Forester will be more friendly to the tablets and notebooks that students are
all using these days.</p>
<h2><a href="https://anil.recoil.org/news.xml#geotessera-0110-now-has-wall-to-wall-embeddings" class="anchor" aria-hidden="true"></a>GeoTessera 0.11.0 now has wall-to-wall embeddings!</h2>
<p>As its start-of-term here, <a href="https://coomeslab.org">David Coomes</a> <a href="https://toao.com">Sadiq Jaffer</a> and I will give an intro to Tessera talk
at the CRI next week, so pop along if you're in town!</p>
<p><img src="https://anil.recoil.org/images/cri-seminar-26.webp" alt="%c"></p>
<p>I also shipped <a href="https://github.com/ucam-eo/geotessera/releases/tag/v0.11.0">geotessera v0.11.0</a>, which now
provides <strong>wall-to-wall embeddings for anywhere in the world from 2017-2025</strong>, a major first for the project! Yay!
See the release notes above for details.</p>
<p>There are now <em>two</em> separate full copies of the v1.1 embeddings available, in either
Icechunk or Zarr format, with the choice depending on your particular usecase.
This has been a (validly) confusing point for <a href="https://digitalflapjack.com/weeknotes/2026-10-05/">early users</a>, so here's an explanation of the innards.</p>
<h3><a href="https://anil.recoil.org/news.xml#icechunk-for-bulk-streaming" class="anchor" aria-hidden="true"></a>Icechunk for bulk streaming</h3>
<p>First, the dClimate team <a href="https://blog.dclimate.net/how-we-re-engineered-the-tessera-embeddings-inference-pipeline-into-a-faster-cheaper-cloud-native-open-source-library/">built a custom inference stack</a> that uses <a href="https://icechunk.io">Icechunk</a>, and publishes this in the <code>s3://tessera-embeddings</code> bucket on AWS.
This has been optimised for 'streaming access' with relatively large shard and chunk sizes to minimise round-trip traffic.</p>
<p>However, it can <em>only</em> be accessed using Icechunk-compatible bindings since although Icechunk has a <a href="https://icechunk.io/en/latest/understanding/faq/">Zarr-compatible store interface</a>, its on-disk (via HTTP) format completely differs from standard Zarr. Therefore, a client in something like OCaml that speaks the Zarr standard will <a href="https://anil.recoil.org/notes/2026w31">not be able to interoperate</a> with the Icechunk store.</p>
<h3><a href="https://anil.recoil.org/news.xml#zarr-v3-for-smaller-mobile-friendly-chunks" class="anchor" aria-hidden="true"></a>Zarr v3 for smaller mobile-friendly chunks</h3>
<p>This motivated us to transcode the Icechunk store to a <em>separate</em> pure Zarr v3 format. While we lost some cool management features available with Icechunk (like versioning and snapshots), we gained compatibility with clients that implement pure Zarr (like my own OCaml code for example).  While we were here, I also changed the size of the chunking to be lower to make it more friendly for mobile streaming. This is what allows mobile JavaScript frontends like <a href="https://tze.geotessera.org">TZE</a> to work.</p>
<p>Therefore as a user, with GeoTessera 0.11 you now have a choice between the two:</p>
<ul>
<li>The Zarr store on Source Cooperative is the default for streaming reads.</li>
<li>Pass an <code>.icechunk</code> URL to <code>GeoTesseraZarr()</code> instead if you want the Icechunk repository directly. The production store is at <code>s3://tessera-embeddings/v1.1/dclimate.icechunk</code>. I'll expose this as a builtin option in a future geotessera.</li>
</ul>
<h3><a href="https://anil.recoil.org/news.xml#bypassing-the-source-coop-proxy-for-direct-s3" class="anchor" aria-hidden="true"></a>Bypassing the Source Coop proxy for direct S3</h3>
<p>One reason that many users saw bad performance initially is that the Source Coop has an additional caching layer over Amazon via a Rust proxy (<code>data.source.coop</code>), and this appears to be throttling rapid connections in a way that directly going to S3 isn't. GeoTessera 0.11 now reads straight from the S3 bucket by default to avoid putting any undue pressure on the poor sole proxy server that Source Coop is kindly running for us, with the gateway still available as an opt-in in the geotessera CLI via <code>--via-gateway</code> if you want its Cloudflare edge caching.</p>
<p>Phew! There's a lot going on here with where the embeddings are stored, but I hope we're hitting the right notes as to how to make them as maximally accessible as possible. My next step here is to publish the <a href="https://anil.recoil.org/notes/2026w9">HTTP perma proxy</a> I wrote a few months ago for caching Zarr tiles, in order to provide a gateway proxy in Cambridge that stashes commonly used years locally to avoid lots of network traffic.</p>
<p><a href="https://github.com/ucam-eo/geotessera/releases/tag/v0.11.0"> <img src="https://anil.recoil.org/images/tessera-wall-to-wall.webp" alt="%c" title="Wall to wall Tessera, anywhere in the world! Woohoo!"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#going-to-the-other-place-to-talk-at-the-intelligent-earth-cdt" class="anchor" aria-hidden="true"></a>Going to the Other Place to talk at the Intelligent Earth CDT</h2>
<p>I was invited by the enthusiastic crew to give a talk on on "Technology for Living Evidence" (my overall name for the work we're doing on <a href="https://anil.recoil.org/projects/tessera">Tessera</a>, <a href="https://anil.recoil.org/projects/rsn">habitat</a> and <a href="https://anil.recoil.org/projects/enki">species</a> mapping and <a href="https://anil.recoil.org/projects/ce">evidence</a> ) to the new student cohort at the <a href="https://intelligent-earth.ox.ac.uk/">Intelligent Earth CDT</a> in Oxford.</p>
<p>The venue was the incredibly impressive new <a href="https://www.biology.ox.ac.uk/article/the-life-and-mind-building-a-new-home-for-discovery-and-collaboration">Centre for Life and Mind</a>, which opened about a year ago and still had that 'new building' smell.  I was told it cost a cool ~£200m and is Oxford's largest-ever building project, except for certain other <a href="https://www.oxfordmail.co.uk/news/26565850.oxford-disbelief-237m-bridge-reopens-three-years/">hilariously bad</a> bridges. I'm told it supports 1,400 researchers, &gt;1,000 undergraduates a year, and has shielded EEG rooms, eye-tracking labs, and even a sleep laboratory tucked away somewhere!</p>
<p><img src="https://anil.recoil.org/images/w40-5.webp" alt="%c" title="A buzzing student poster session in Oxford at the Intelligent Earth CDT."></p>
<p>The Intelligent Earth CDT is a UKRI-funded initiative that'll train 100 PhD students years across climate, biodiversity, natural hazards, and environmental solutions.
This is temporally a follow-on to our own <a href="https://ai4er-cdt.esc.cam.ac.uk/">AI4ER</a> program here that's drawing to a close now, so it was also excellent to have <a href="https://yihshe.github.io/">Yihang She</a> along from our group to act as a bridge between generations of students!</p>
<p>The director, <a href="https://intelligent-earth.ox.ac.uk/people/philip-stier">Philip Stier</a>, was a brilliant host and also tutored me on his expert topic of the <a href="https://lacuna.tiptreesystems.com/work/cloudflow-a-flow-matching-model-to-generate-high-resolution-cloud-structures/wrk_c5f82ceb675cdb33b7015ac4a5841deb">physics of clouds</a>. He recently built a <a href="https://icml.cc/virtual/2026/73478">flow-matching model</a> that downscales high-resolution (~1km) cloud structure from the coarse (~25km) fields that climate models calculate.</p>
<p><img src="https://anil.recoil.org/images/w40-3.webp" alt="%c" title="Philip Stier introduces the program for the three days"></p>
<p><img src="https://anil.recoil.org/images/w40-1.webp" alt="%c" title="If nature had a CEO, they'd have their office in this building by jove"></p>
<h3><a href="https://anil.recoil.org/news.xml#the-oxford-cambridge-rail-certainly-isnt-moving-very-fast" class="anchor" aria-hidden="true"></a>The Oxford-Cambridge rail certainly isnt moving very fast</h3>
<p>Getting to Oxford was a nightmare as usual, as the only useful options were trains via London (~3 hours but the line was down), a bus via Bedford (~2.5h), or motorbiking down the single-carriageway A421 (~2h "on a good day"). As <a href="https://radicalhonesty.uk/p/kickstart-britain-connect-oxford">others have noted</a> the East-West railway can't come soon enough:</p>
<blockquote>
<p>Today, travelling between Oxford and Cambridge by public transport is a joke. [...]
With East West Rail, that journey time drops to ninety minutes. [...]
This means a researcher in Cambridge can collaborate with a lab in Oxford without losing a day to the journey</p>
</blockquote>
<p>Despite the <a href="https://en.wikipedia.org/wiki/Varsity_Line">Varsity line</a> dating back well into the 19th century, the campaign to reinstate has been going on for 30 years (!), and under <a href="https://railway-news.com/east-west-rail-presents-revised-plans-for-earlier-delivery/">East West Railway Company's revised April 2026 proposals</a> the full end-to-end Oxford–Cambridge service isn't scheduled until the mid-to-late 2030s. Absolute snail's pace progress here...</p>
<p><img src="https://anil.recoil.org/images/w40-4.webp" alt="%c" title="Excellent talk as always from Yihang representing the Cambridge side!"></p>
<h2><a href="https://anil.recoil.org/news.xml#scrutineer-fixes-keep-rolling" class="anchor" aria-hidden="true"></a>Scrutineer fixes keep rolling</h2>
<p><a href="https://www.infoq.com/news/2026/10/open-source-ai-security/">InfoQ picked up</a> my <a href="https://anil.recoil.org/notes/rumour-is-the-exploit">'rumour is the exploit'</a> post, on how AI agents can basically find working exploit faster than traditional embargoed disclosure can keep up. In the meanwhile, Scrutineer continues to be extremely successful at finding issues, and <a href="https://github.com/hannesm">Hannes Mehnert</a> and Edwin Torok continue to fix many of them (thank you!!). I'm going to catch up on my Patch the Planet planning backlog later as I've been extremely busy preparing for the start of term.</p>
<p>In terms of where activity has been focussed, <a href="https://github.com/mirage/awa-ssh">mirage/awa-ssh</a> joins the list of repos being fixed, with a <a href="https://github.com/mirage/awa-ssh/pull/94">nonce fix</a>, a <a href="https://github.com/mirage/awa-ssh/pull/99">CPU/memory fix</a> and a <a href="https://github.com/mirage/awa-ssh/pull/100">required-authenticator fix</a>. Elsewhere I spotted <a href="https://github.com/mirleft/ocaml-tls/pull/535">ocaml-tls's signature-algorithm fix</a>, <a href="https://github.com/robur-coop/happy-eyeballs/pull/51">happy-eyeballs's empty-address fix</a>, more <a href="https://github.com/Solo5/solo5">Solo5</a> bounds checks (<a href="https://github.com/Solo5/solo5/pull/682">#682</a>/<a href="https://github.com/Solo5/solo5/pull/684">#684</a> closed, <a href="https://github.com/Solo5/solo5/pull/683">#683</a> carried over, plus new <a href="https://github.com/Solo5/solo5/pull/686">dev-type</a> and <a href="https://github.com/Solo5/solo5/pull/687">note-size</a> checks).</p>
<h2><a href="https://anil.recoil.org/news.xml#the-life-tour-and-offsetting-ourselves" class="anchor" aria-hidden="true"></a>The LIFE tour, and offsetting ourselves</h2>
<p>I built an <a href="https://anil.recoil.org/notes/life-zarr-and-everything">interactive LIFE tour</a> using an ensemble
of agents narrating and animating the LIFE metric over the live Zarr store.
This was worryingly quick and good, and with several reactions on the socials like <a href="https://www.linkedin.com/posts/anilmadhavapeddy_frontier-agents-can-build-interactive-tutorials-ugcPost-7511738497364439040-Nqe5/">LinkedIn</a>.</p>
<p>This ranged from <a href="https://www.conservation.cam.ac.uk/staff/dr-alison-eyres">Alison Eyres</a> being very impressed, to <a href="https://www.linkedin.com/posts/anilmadhavapeddy_frontier-agents-can-build-interactive-tutorials-ugcPost-7511738497364439040-Nqe5/">Chris Sandbrook asked how the process actually worked</a> and worrying about the implications. It's fair to say that I am worried as well; I'm feeling awe at the one-shot quality, intrigue at the educational potential for personalisation, and a little shock at the implications for jobs down the line. Will we end up needing to 'offset' our use of AI to support human endeavour, the way we do for <a href="https://anil.recoil.org/notes/carbon-credits-vs-offsets">carbon credits</a>?</p>
<p><a href="https://life-metric.org"> <img src="https://anil.recoil.org/images/life-metric-ss-2.webp" alt="%c" title="A fun interactive guided tour about what the LIFE metric is and some uses for it"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#released-a-new-ocaml-mdx-to-unblock-eio-dev" class="anchor" aria-hidden="true"></a>Released a new OCaml mdx to unblock Eio dev</h2>
<p>I <a href="https://github.com/ocaml/opam-repository/pull/30853">released mdx 2.7.0</a> to add support by <a href="https://roscidus.com">Thomas Leonard</a> for a new <code>mdx_skip</code> <a href="https://github.com/realworldocaml/mdx/pull/483">directive</a> and <a href="https://github.com/realworldocaml/mdx/pull/482">shonfeder's dune 3.24 support fix</a>.  Thanks to a Windows VM that <a href="https://www.tunbury.org/">Mark Elvers</a> setup for me  I no longer need a physical Windows desktop, which has made me much less grumpy. I've resumed working on  <a href="https://github.com/ocaml-multicore/eio/pull/936">Eio's Windows process spawning</a> and <a href="https://github.com/ocaml-multicore/eio/pull/947"><code>nt_path</code> normalisation</a>, as well as dusting off my changes to the <a href="https://github.com/avsm/ocaml-iocp">IOCP bindings</a>. This is the 'relaxing' hacking of the week, but may need to be put on ice as term starts!</p><h1>References</h1><ul><li>Madhavapeddy (2026). .plan-26-31: Sorting out Tessera and Evidence TAP infrastructure. <a href="https://doi.org/10.59350/30e5y-n8p97" target="_blank"><i>10.59350/30e5y-n8p97</i></a></li>
<li>Madhavapeddy (2025). Foundations of Computer Science. <a href="https://doi.org/10.59350/qms3q-ymn65" target="_blank"><i>10.59350/qms3q-ymn65</i></a></li>
<li>Madhavapeddy (2026). Just a rumour of a bug is enough to find a security exploit these days. <a href="https://doi.org/10.59350/tngsm-6rx23" target="_blank"><i>10.59350/tngsm-6rx23</i></a></li>
<li>Madhavapeddy (2026). Using Forester to turn Foundations of CS into interactive evergreen lectures. <a href="https://doi.org/10.59350/sjcvd-hb857" target="_blank"><i>10.59350/sjcvd-hb857</i></a></li>
<li>Madhavapeddy (2025). Disentangling carbon credits and offsets with contributions. <a href="https://doi.org/10.59350/g4ch1-64343" target="_blank"><i>10.59350/g4ch1-64343</i></a></li></ul>
