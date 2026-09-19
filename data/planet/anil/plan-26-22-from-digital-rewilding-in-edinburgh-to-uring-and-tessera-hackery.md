---
title: '.plan-26-22: From digital rewilding in Edinburgh to uring and Tessera hackery'
description: Rewilding the Web workshop in Edinburgh, an OCaml io_uring binding refresh,
  and GeoTessera 0.9 moves the embeddings to AWS alongside a fresh HuggingFace org.
url: https://anil.recoil.org/notes/2026w22
date: 2026-05-31T00:00:00-00:00
preview_image: https://anil.recoil.org/images/26w22-ed-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Most of this week was either off for the May bank-holiday long weekend or up in Edinburgh at this <a href="https://anil.recoil.org/notes/rewilding-the-web-report">great workshop</a>, with plenty of hacking on the long train journeys in between.</p>
<h2><a href="https://anil.recoil.org/news.xml#rewilding-the-web-in-edinburgh" class="anchor" aria-hidden="true"></a>Rewilding the Web in Edinburgh</h2>
<p>Jon Crowcroft and I went up to Edinburgh for Kate Nave's <em>Rewilding the Web: Diversity &amp; Resilience in Sociotechnical Infrastructure</em> workshop, an interdisciplinary mix of economists, ecologists, philosophers, techies and authors. The notes are in the <a href="https://anil.recoil.org/notes/rewilding-the-web-report">workshop report</a>.</p>
<p>I came back with a huge reading list, learnt the word "coopetition", and gathered a giant list of follow-ups from our <a href="https://anil.recoil.org/papers/2025-internet-ecology">Internet ecology paper</a>. Jon and I did a double act on antibotty networks and code self-modification, and -- unlike the response six months ago at Aarhus -- nobody in the room treated it as sci-fi. The shift on coding agents into the mainstream is happening fast.</p>
<p><img src="https://anil.recoil.org/images/26w22-ed-1.webp" alt="%c" title="Wander Edinburgh University"></p>
<h2><a href="https://anil.recoil.org/news.xml#ocaml-uring-refreshes" class="anchor" aria-hidden="true"></a>ocaml-uring refreshes</h2>
<p><a href="https://roscidus.com">Thomas Leonard</a> is back <a href="https://notes.roscidus.com/2026/05/24/">hacking on eio</a> and working through the PR backlog (<code>Unix.file_descr</code> conversion to lose <code>Obj.magic</code>, MDX hang detection, a big liburing 2.14 update to get the latest goodies).
Spurred on by all this activity, I spent a chunk of the week filling in some coverage gaps so I can start using uring again in my TESSERA code. My <a href="https://github.com/ocaml-multicore/ocaml-uring/pull/147">PR #147</a> adds bindings for <code>shutdown</code>, <code>socket</code>, <code>renameat</code> and <code>symlinkat</code>.</p>
<p>The other PR is smaller but highlights a slightly more obscure API in Linux. <a href="https://github.com/ocaml-multicore/ocaml-uring/pull/142">PR #142</a> fixes a bug where the supported-attribute check for <code>statx</code> was inverted. Eio doesn't currently use that code path so we hadn't noticed. <code>statx</code> is a slightly odd syscall: it negotiates which attributes the kernel will fill in via a request-mask / returned-mask handshake, and it's easy to get the boolean direction wrong on the OCaml side. It's also not entirely clear under what conditions the kernel will let this mask get out of sync from the request...</p>
<h2><a href="https://anil.recoil.org/news.xml#geotessera-09-and-a-huggingface-home-for-the-models" class="anchor" aria-hidden="true"></a>GeoTessera 0.9 and a HuggingFace home for the models</h2>
<p>On the <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> side, I've been getting <a href="https://github.com/ucam-eo/geotessera/pull/250">GeoTessera 0.9</a> ready to land. The release does two things: it migrates the embeddings host from our Cambridge infrastructure to <code>s3://tessera-embeddings/</code> on AWS <code>us-west-2</code> (the <a href="https://anil.recoil.org/notes/2026w17">AWS Open Data sync</a> that <a href="https://www.tunbury.org/">Mark Elvers</a> and I have been doing), and adds support for our forthcoming TESSERA v1.1 model alongside the existing v1.0.</p>
<p>Since we're now on S3, we've dropped the SHA256-based registry in favour of
S3's built-in <code>x-amz-checksum-crc64nvme</code> header, which simplifies the integrity
check and lets us simplify the existing download path. The <code>geotessera-registry s3scan</code> tool now auto-discovers every <code>(version, variant, year)</code> under any S3
prefix and shards the listing by longitude. A one-year scan went from ~11
minutes to ~47 seconds, which makes regenerating the manifests cheap enough to
do routinely. Cache freshness now also uses ETag / <code>If-None-Match</code> in addition
to <code>If-Modified-Since</code>, so clients won't miss updates when local mtime drifts.</p>
<p>To prepare for the v1.1 release, I opened up a new <a href="https://huggingface.co/geotessera">geotessera org on Hugging Face</a> and uploaded model cards for <a href="https://huggingface.co/geotessera/TESSERA-V-1.0">TESSERA-V-1.0</a> and <a href="https://huggingface.co/geotessera/TESSERA-V-1.1">TESSERA-V-1.1</a>. The card format follows the <a href="https://geoembeddings.org/model-card.html">geospatial embeddings model card template</a> that came out of the <a href="https://anil.recoil.org/notes/2026w12">Clark University embeddings sprint</a> earlier in the spring. <a href="https://mlisaius.github.io/">Madeline Lisaius</a> did a lot of the work pulling that template together, and it's good to see the community standard land on something concrete that other model authors can reuse!</p>
<p>Most users will continue to pull pregenerated embeddings via the GeoTessera library rather than the raw weights, but having a canonical HF home for the model itself was overdue.</p>
<p>I'll post properly about v1.1 once the release is fully out. The v1.0-v1.1
transition is a no-op for downstream code since you just point at a
new manifest and grab new embeddings. Users should just see their performance
increase without any effort, as the model backing the embeddings has improved!</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun Links</h2>
<ul>
<li><a href="https://blog.janestreet.com/strace-ui-bonsai-term-and-the-tui-renaissance/">Jane Street on strace_tui</a> got me to refresh my own <a href="https://anil.recoil.org/notes/aoah-2025-9">Bonsai code</a> and get it running under oi. Working for me but still polishing for release!</li>
<li>Started reading through more <a href="https://github.com/geocaml">geocaml</a> code after seeing <a href="https://digitalflapjack.com/weeknotes/geocaml-hacking/">the Lidar viewer</a>.</li>
<li>I gotta tidy up my bleeding edge <a href="https://anil.recoil.org/notes/oxcaml-httpz">oxcaml-httpz</a> code for <a href="https://digitalflapjack.com/weeknotes/webbplats-update/">others to use</a> in their site. Something for next week!</li>
<li>Learnt a lot about how <a href="https://mcj.vc/inevitable-podcast/dg-matrix">power transformers work</a> in the latest MCJ podcast.</li>
</ul><h1>References</h1><ul><li>Madhavapeddy et al (2025). Steps towards an Ecology for the Internet. Association for Computing Machinery. <a href="https://doi.org/10.1145/3744169.3744180" target="_blank"><i>10.1145/3744169.3744180</i></a></li>
<li>Madhavapeddy (2026). My (very) fast zero-allocation webserver using OxCaml. <a href="https://doi.org/10.59350/9c6bz-kb659" target="_blank"><i>10.59350/9c6bz-kb659</i></a></li>
<li>Madhavapeddy (2026). Rewilding the Web: my workshop report from Edinburgh. <a href="https://doi.org/10.59350/g40yy-ks003" target="_blank"><i>10.59350/g40yy-ks003</i></a></li></ul>
