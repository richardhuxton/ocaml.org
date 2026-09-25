---
title: '.plan-26-33: Zarro rides out and evidence papers pour in'
description: TESSERA 1.0 is now fully available as Zarr with global RGB previews,
  a weather downscaling preprint, and Evidence TAP gets a public website with progress
  on the downloader and parser.
url: https://anil.recoil.org/notes/2026w33
date: 2026-08-16T00:00:00-00:00
preview_image: https://anil.recoil.org/images/tze-scoop-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Am back in Ireland again this week, after a quick stint back in Cambridge to catch the <a href="https://www.bbc.co.uk/news/articles/c75g9g50y9eo">90% eclipse of the heart</a>.
The major preprint that went out this week was on <a href="https://anil.recoil.org/notes/weather-downscaling-tessera">weather downscaling with TESSERA embeddings</a>. I also did a chunk of teaching prep (October is coming up fast!) and started <a href="https://anil.recoil.org/notes/forester-teaching-notes">porting Foundations of CS to Forester</a>.</p>
<p>The week itself then went on getting <a href="https://anil.recoil.org/news.xml#tessera-dons-the-mask-of-zarro">TESSERA 1.0 onto Source Cooperative in Zarr</a> with global RGB previews to go with it, putting up <a href="https://anil.recoil.org/news.xml#evidence-tap-goes-public">a public website for Evidence TAP</a>, teaching <a href="https://anil.recoil.org/news.xml#tap-tap-tap-is-this-downloader-on">the paper downloader some manners</a>, and <a href="https://anil.recoil.org/news.xml#working-on-metadata-quality">pointing a VLM at the PDFs</a>. There's some <a href="https://anil.recoil.org/news.xml#ocaml-hacking">OCaml hacking</a> and <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a> at the end.</p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-dons-the-mask-of-zarro" class="anchor" aria-hidden="true"></a>Tessera dons the mask of Zarro</h2>
<p>We finally have <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> 1.0 fully on
<a href="https://source.coop">source.coop</a> in Zarr format, with 1.1 well on its way!
Since we finally have enough disk space to stage everything thanks to source.coop, I also generated global RGB preview pyramids. You can now use <a href="https://tze.geotessera.org">TZE</a> to zoom in and out of a dimensionality-reduced view of the 128 embedding dimensions in glorious technicolour:</p>
<p><a href="https://tze.geotessera.org"> <img src="https://anil.recoil.org/images/tze-scoop-1.webp" alt="%c" title="We have a global RGB preview for the first time"> </a></p>
<p>And of course you can keep zooming in the browser and inspect individual shards:</p>
<p><img src="https://anil.recoil.org/images/tze-scoop-2.webp" alt="%c" title="One shard near Le Mans in UTM zone 31, with its per-year embedding norms"></p>
<p>The RGB previews make some of the inference artefacts obvious; just look for
the checkerboarding and the banding across the Amazon. This doesn't actually
affect downstream inference accuracy too much, but it's very ugly. Luckily this
is all <a href="https://anil.recoil.org/notes/tessera-v11-out">fixed in TESSERA v1.1</a> and even more so <a href="https://anil.recoil.org/papers/2026-tessera-v2">in v2.0</a>, and I can't wait to see
those previews to compare next week!</p>
<p></p><div class="video-center"><iframe title="Tessera 1.0 to 1.1 embeddings" width="100%" height="315px" src="https://crank.recoil.org/videos/embed/297de7c9-9cea-4051-8b27-041fffa90e72" frameborder="0" allowfullscreen="" sandbox="allow-same-origin allow-scripts allow-popups allow-forms"></iframe></div><p></p>
<p>On the client side, <a href="https://github.com/ucam-eo/geotessera/pull/357">geotessera#357</a> switches client downloads over to source.coop for the npy layout as well as Zarr, and picks up our new dataset variants like <code>v2-2B-L~beta1</code> so that users can choose between v1/1.1/2.0~beta easily. The Zarr portion of the PR follows the <a href="https://anil.recoil.org/notes/tessera-zarr-v3-layout">layout</a> I settled on earlier in the year. I'll release all this in a new version of geotessera early next week.</p>
<p>The giant transcode of the embeddings finished after days (and hundreds of terabytes) of back and forth between our compute cluster in Cambridge and the AWS S3 store. At the nth moment, <a href="https://www.tunbury.org/">Mark Elvers</a> and I decided to rename some directories and rediscovered that <a href="https://www.tunbury.org/2026/08/17/week-33-2025/#re-versioning-sourcecoop">S3 doesn't do renames</a>, which led to another day of delay while six million individual server-side copies happened. Mark found that a plain <code>aws s3 cp</code> stays entirely server-side, whereas specifying a <code>CRC64NVME</code> checksum would have dragged 400TB back through Cambridge. Ho hum...</p>
<h2><a href="https://anil.recoil.org/news.xml#papers-and-teaching" class="anchor" aria-hidden="true"></a>Papers and teaching</h2>
<p><a href="https://www.linkedin.com/in/pedro-marques-sousa">Pedro Sousa</a> put his first preprint online on how "<a href="https://anil.recoil.org/papers/2026-weather-downscaling">Earth observation embeddings are effective sub-grid descriptors for probabilistic weather downscaling</a>". This is joint work with <a href="https://toao.com">Sadiq Jaffer</a>, <a href="https://www.cst.cam.ac.uk/people/ray25">Robin Young</a>, <a href="https://willtebbutt.github.io/">Will Tebbutt</a> and <a href="https://rich-turner-group.github.io/">Rich Turner</a>, and since its my first foray into weather prediction I wrote <a href="https://anil.recoil.org/notes/weather-downscaling-tessera">a longer splainer</a> on how weather downscaling works and why a frozen annual <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> embedding beats handcrafted terrain descriptors.</p>
<p>I also <a href="https://anil.recoil.org/notes/forester-teaching-notes">ported 1A Foundations of CS over to Forester</a> and the lecture notes are now a forest of transcluded trees with stable URLs, and there's also an OCaml toplevel compiled into the browser with <code>js_of_ocaml</code> so every code transcript in the course is runnable (and editable) in place.
Also a reminder that <a href="https://icfp26.sigplan.org/">ICFP 2026</a> kicks off on the 24th of August in Indianapolis, where <a href="https://ryan.freumh.org">Ryan Gibb</a> will be presenting his <a href="https://anil.recoil.org/papers/2026-package-calculus">package management calculus paper</a>. Do say hi to him if you're there!</p>
<h2><a href="https://anil.recoil.org/news.xml#evidence-tap-goes-public" class="anchor" aria-hidden="true"></a>Evidence TAP goes public</h2>
<p>There's a website up now at <a href="https://evidencetap.org">evidencetap.org</a>! It's a very barebones site, but it does list what's going on and who is involved, and we'll populate it more over the coming months (especially by Mélanie Gréaux, who wrote up a fantastic AI4Good trip report from her recent travels).</p>
<p>One important reason to get the site up is to have our <a href="https://evidencetap.org/text-and-data-mining">text and data mining policy</a> online, which we need in order to keep publishers informed about our intended use for the large corpus of papers we're downloading.</p>
<h3><a href="https://anil.recoil.org/news.xml#tap-tap-tap-is-this-downloader-on" class="anchor" aria-hidden="true"></a>TAP TAP TAP, is this downloader on?</h3>
<p>So why are we are hoovering up quite so many papers in the first place? Our vision for <a href="https://anil.recoil.org/papers/2025-evidence-tap">living evidence databases</a> has to screen, appraise and extract from the full text of a study rather than just its abstract. We're pinning down inclusion criteria for building evidence databases that be used to conduct arbitrary rapid reviews on demand, which (e.g. in conservation) might be what the intervention was, where, on what, etc. Doing that continuously across conservation and now education requires holding the full corpus here so that local-only models can read it without any of it leaking to a third party and also be reproducible.</p>
<p><img src="https://anil.recoil.org/images/taposaur-conversion-aug26.webp" alt="%c" title="The ingestion is now at over 9m of the papers, with around 4m left to go from the backlog"></p>
<p>Those ten million papers we've obtained are a mixture of closed- and open-access publisher XML and PDFs that still need converting to TEI XML. This involves a pool of Pub2TEI instances (each of which tops out at around 10k documents an hour) which put the backlog at about a fortnight. I added another 128-core box and it's now at a peak of 53,000 documents an hour. The backlog's now about two days, after which we can get on with metadata analysis.</p>
<p>The paper downloader itself is surprisingly complex because of all the per-publisher logic.
Publishers throttle and block us regularly, with a variety of error codes. An HTTP 500 is now treated as a general "slow down": we wait five minutes by default and block all other requests to that origin before asking again. A 403's body is also now read and interpreted per-publisher.</p>
<p>Some sites simply refuse our machines until we're on an IP allowlist, so all fetch traffic now leaves through a SOCKS5 proxy on one stable, allowlistable IP address.
The table below shows some of the reasons we need a custom fetcher at all. Each domain has its own request rate, authentication mechanism, and IP source, all easily customisable. We'll eventually need over 200+ publishers in here!</p>
<div role="region"><table>
<tbody><tr>
<th>Strategy</th>
<th>Gets</th>
<th>Authentication</th>
<th>Quirks</th>
</tr>
<tr>
<td>Elsevier</td>
<td>XML (full text)</td>
<td><code>X-ELS-APIKey</code>/<code>X-ELS-Insttoken</code> headers</td>
<td>Bad key = 401, unavailable = 403</td>
</tr>
<tr>
<td>Springer (PDF)</td>
<td>PDF</td>
<td>Network IP</td>
<td>One request per 30s</td>
</tr>
<tr>
<td>Springer (XML)</td>
<td>XML (JATS)</td>
<td><code>api_key</code> query param</td>
<td>Books answer 403 (access rights). Occasionally 404s instead of 500</td>
</tr>
<tr>
<td>Wiley</td>
<td>PDF (TDM API)</td>
<td><code>Wiley-TDM-Client-Token</code> header</td>
<td>403 if not entitled to the content (books, old papers). Throttling is 500 rather than a 429</td>
</tr>
<tr>
<td>TandF</td>
<td>PDF</td>
<td>Network IP</td>
<td>403 if the IP allowlist expires</td>
</tr>
<tr>
<td>PLOS</td>
<td>XML</td>
<td>None</td>
<td>Also available in bulk via <code>allofplos</code>. Super easy to fetch.</td>
</tr>
<tr>
<td>CUP</td>
<td>PDF</td>
<td>Network IP</td>
<td>30s pacing per request, often times out</td>
</tr>
<tr>
<td>CORE</td>
<td>PDF</td>
<td><code>Bearer</code> token</td>
<td>Search IDs ≠ output IDs, so the returned DOI needs verifying</td>
</tr>
</tbody></table></div><p>The winner by far is PLOS, which provides a simple Python <a href="https://github.com/plos/allofplos">allofplos</a> library to fetch fulltexts.
<em>(Update: And as reminded by <a href="https://toao.com">Sadiq Jaffer</a>, the Royal Society archives who basically told us to download whatever we needed!).</em></p>
<h3><a href="https://anil.recoil.org/news.xml#working-on-metadata-quality" class="anchor" aria-hidden="true"></a>Working on metadata quality</h3>
<p>Our Taposaur pipeline converts as much of the fulltext to TEI XML as it can, but it does miss things in complex paper layouts. On a suggestion from <a href="https://toao.com">Sadiq Jaffer</a>, I've been experimenting with <a href="https://huggingface.co/nvidia/NVIDIA-Nemotron-Parse-2.0">Nemotron Parse 2.0</a> alongside a <a href="https://huggingface.co/Qwen/Qwen3.8-27B-FP8">Qwen 3.8 27B</a> VLM to get high quality metadata and references out of even the older PDFs in our database.</p>
<p>My fledgling 'parsosaur' tool started with pure Nemotron to parse out bounding boxes. You can <a href="https://www.cl.cam.ac.uk/~avsm2/2024-ai-conhorizon/">browse the output for our 2024 conservation horizon scan</a> paper, which shows the PDF and the parsed TEI side by side. This works surprisingly well as even the magazine-style layout of <a href="https://anil.recoil.org/papers/2024-ai-conhorizon">Trends in Ecology &amp; Evolution</a> is matched reasonably accurately to the full text in the TEI, right down to the author affiliations. Mouse over either side and it'll match the scroll position of the other.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/2024-ai-conhorizon/"> <img src="https://anil.recoil.org/images/parsosaur-1.webp" alt="%c" title="The PDF and its extracted TEI, with the abstract bounding box highlighted"> </a></p>
<p>Nemotron does less well at interpreting complex diagrams. I <a href="https://www.cl.cam.ac.uk/~avsm2/2023-ncc-permanence/">chucked it</a> at our <a href="https://anil.recoil.org/papers/2023-ncc-permanence">Nature Climate Change paper on impermanence</a>, whose figures are dense and multi-panel and it degenerated into a loop of near-identical cells, which at least it marked as unreliable:</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/2023-ncc-permanence/"> <img src="https://anil.recoil.org/images/parsosaur-2.webp" alt="%c" title="Six panels of forecast release schedules in, seven columns of repetition artefacts out"> </a></p>
<p>Sadiq set me up a local Qwen 3.8 27B endpoint, and my initial experiments using the VLM to interpret diagrams when supplied with the full text of the paper are going really well. I'll try to cut a release of Parsosaur next week, as <a href="https://shaneweisz.com">Shane Weisz</a> also expressed interest in learning more about VLM usage for his <a href="https://anil.recoil.org/projects/enki">Dash for Life</a> dashboards.</p>
<h2><a href="https://anil.recoil.org/news.xml#ocaml-hacking" class="anchor" aria-hidden="true"></a>OCaml hacking</h2>
<p>Most of my OCaml time went into the infrastructure above, but a few things escaped into the wild.</p>
<h3><a href="https://anil.recoil.org/news.xml#making-eio-traces-free-again" class="anchor" aria-hidden="true"></a>Making Eio traces free again</h3>
<p>While profiling Taposaur I've been adding custom events in various places. <a href="https://roscidus.com">Thomas Leonard</a> and <a href="https://github.com/samoht">Thomas Gazagnaire</a> discussed on <a href="https://github.com/ocaml-multicore/eio/issues/914">eio#914</a> that our OCaml runtime events allocate 23 words per event as soon as the runtime events shared memory ring is active. Since a lot of runtime events usage is about finding performance bugs, it's not ideal that the tracing calls themselves allocate memory.</p>
<p>My fix upstream in <a href="https://github.com/ocaml/ocaml/pull/14984">ocaml/ocaml#14984</a> is hopefully a simplification. There's a per-domain write-buffer cache for custom events that was formerly a list accessed through closures, and I changed it to a single atomic bytes slot held in domain-local storage. Now, the steady state doesn't allocate at all and the buffer is only checked out for the duration of the serializer call. Thanks to <a href="https://toao.com">Sadiq Jaffer</a> for for the review; I'm still not entirely sure I understand the interplay between the C and OCaml runtimes here, but the new version is at least simpler. The fix will be in OCaml 5.6 later this year.</p>
<h3><a href="https://anil.recoil.org/news.xml#escaping-the-docroot-in-cohttp" class="anchor" aria-hidden="true"></a>Escaping the docroot in cohttp</h3>
<p>More seriously, I've been triaging bug reports from Jane Street and with their help fixed a path traversal in cohttp's <code>resolve_local_file</code>, which could be escaped using urlencoded path components. <a href="https://github.com/mirage/ocaml-cohttp/pull/1145">cohttp#1145</a> urldecodes before resolving and also exposes a <code>Cohttp.Path.normalise</code> function so that anyone gating requests on a path prefix can apply the the same normalisation logic, rather than authorising a decoded path and then serving an undecoded one:</p>
<pre><code class="language-ocaml">let callback _conn req _body =
    let path = Cohttp.Path.normalise (Cohttp.Request.uri req) in
    match String.split_on_char '/' path with
    | "admin" :: _ when not (authorised req) -&gt; Server.respond_not_found ()
    | _ -&gt; Server.respond_file ~fname:path ()
</code></pre>
<p>This affects cohttp-lwt, cohttp-mirage and cohttp-async, but not cohttp-eio, which has its own path resolution and uses sandboxed fd operations anyway to prevent traversal. I also experimented with stripping control characters as well, but after discussing with the reporter decided to back that out for compatibility but left the tests in place for a future major bump.</p>
<p><em>(I would ordinarily not talk about security issues until the fix is merged, but in the new agentic world just opening the PR is enough to reveal the existence of the issue, and so I figured getting more eyes on this early is now better while waiting for reviewers. If you use cohttp, please do inspect the patch.)</em></p>
<h3><a href="https://anil.recoil.org/news.xml#prometheus-and-the-opam-repository-treadmill" class="anchor" aria-hidden="true"></a>Prometheus and the opam-repository treadmill</h3>
<p>Following on from the <a href="https://anil.recoil.org/notes/2026w32">prometheus-1.4 release</a> last week, <a href="https://roscidus.com">Thomas Leonard</a> merged <a href="https://github.com/mirage/prometheus/pull/65">prometheus#65</a> to move Lwt-specific logic out of the core package entirely.</p>
<p>Defining and recording a metric no longer pulls in any concurrency library, since it just involves updating an in-memory scoreboard. <a href="https://www.tunbury.org/">Mark Elvers</a> has <a href="https://www.tunbury.org/2026/08/17/week-33-2025/#prometheus-14">already</a> pushed the migration through <a href="https://github.com/ocurrent/opam-health-check/pull/112">opam-health-check</a> and <a href="https://github.com/ocurrent/ocluster/pull/265">ocluster</a>, which is a good sign that the deprecation path works as advertised. Next up is the <a href="https://github.com/mirage/prometheus/pull/71">Eio backend</a> as an independent package and the next major version should be good to go.</p>
<p>As many of the other maintainers are holiday, I also did a bunch of <a href="https://github.com/ocaml/opam-repository">opam-repository</a> merging, I also fixed <a href="https://github.com/oxcaml/opam-repository/pull/59">oxcaml/opam-repository#59</a> to let upstream dune 3.24.2, eio 1.4 and mdx 2.6.0 through the <a href="https://anil.recoil.org/notes/oxcaml-opam-guards">guards</a> so all three now build with OxCaml.  My <a href="https://anil.recoil.org/notes/bushel-lives">website monorepo</a> continues to build on <code>ox-minus39</code> so I can update this website, yay!</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun links</h2>
<ul>
<li>There's a brilliant mobile game on being a prime minister simulator, <a href="https://www.bbc.co.uk/news/articles/cq56pzqy6jvo">No 10: Full Confidence</a>, which briefly ranked above Minecraft on the UK App Store chart (!). I managed two and a half years before being ousted by my own cabinet.</li>
<li><a href="https://www.cst.cam.ac.uk/people/zf281">Frank Feng</a> sent on a report from China that they've been using in-house <a href="https://www.eet-china.com/vote/p/2/27/37?lang=en">Moffett S40 Computing Card</a> to accelerate Tessera inference significantly. I'd heard about these when I went with the Royal Society delegation to China in 2023, but it's the first time I'm seeing sparse-weight AI accelerators 'in the wild'.</li>
<li>Fascinating article on "<a href="https://www.quantamagazine.org/why-are-rivers-so-mathematical-20260810/">Why Are Rivers So Mathematical?</a>" in Quanta. I've been thinking about how to model roads and rivers generatively using Tessera to help with flood predictions, so this article is well timed!</li>
<li>It's been an insanely fast week of progress on local LLMs, with <a href="https://artificialanalysis.ai/models/qwen3-8-27b">Qwen 3.8</a> performing incredibly, and I've been trialling <a href="https://github.com/antirez/ds4/issues/807">Deepseek v4 Pro 813</a> as well. More on that next week!</li>
</ul><h1>References</h1><ul><li>Madhavapeddy (2026). Tessera v1.1 released, with smoother and temporally stable embeddings. <a href="https://doi.org/10.59350/vcqjp-24y05" target="_blank"><i>10.59350/vcqjp-24y05</i></a></li>
<li>Jaffer et al (2025). AI-assisted Living Evidence Databases for Conservation Science. Cambridge Open Engage. <a href="https://doi.org/10.33774/coe-2025-rmsqf" target="_blank"><i>10.33774/coe-2025-rmsqf</i></a></li>
<li>Feng et al (2026). TESSERA v2: Scaling Pixel-wise Earth Foundation Models. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.03949" target="_blank"><i>10.48550/arXiv.2607.03949</i></a></li>
<li>Sousa et al (2026). Earth observation embeddings are effective sub-grid descriptors for probabilistic weather downscaling. arXiv. <a href="https://doi.org/10.48550/arXiv.2608.12271" target="_blank"><i>10.48550/arXiv.2608.12271</i></a></li>
<li>Balmford et al (2023). Realizing the social value of impermanent carbon credits. <a href="https://doi.org/10.1038/s41558-023-01815-0" target="_blank"><i>10.1038/s41558-023-01815-0</i></a></li>
<li>Madhavapeddy (2026). Streaming millions of TESSERA tiles over HTTP with Zarr v3. <a href="https://doi.org/10.59350/tk0er-ycs46" target="_blank"><i>10.59350/tk0er-ycs46</i></a></li>
<li>Madhavapeddy (2026). Using Forester to turn Foundations of CS into interactive evergreen lectures. <a href="https://doi.org/10.59350/sjcvd-hb857" target="_blank"><i>10.59350/sjcvd-hb857</i></a></li>
<li>Reynolds et al (2024). The potential for AI to revolutionize conservation: a horizon scan. <a href="https://doi.org/10.1016/j.tree.2024.11.013" target="_blank"><i>10.1016/j.tree.2024.11.013</i></a></li>
<li>Madhavapeddy (2025). Arise Bushel, my sixth generation oxidised website. <a href="https://doi.org/10.59350/0r62w-c8g63" target="_blank"><i>10.59350/0r62w-c8g63</i></a></li>
<li>Gibb et al (2026). Package Managers à la Carte: A Formal Model of Dependency Resolution. <a href="https://doi.org/10.1145/3828699" target="_blank"><i>10.1145/3828699</i></a></li></ul>
