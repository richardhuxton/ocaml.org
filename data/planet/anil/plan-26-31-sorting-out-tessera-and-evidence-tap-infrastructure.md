---
title: '.plan-26-31: Sorting out Tessera and Evidence TAP infrastructure'
description: A petabyte of TESSERA embeddings moves to Source Cooperative, and Taposaur's
  GROBID metadata index and capability-based downloader take shape for Evidence TAP,
  while Eio gets some native Windows support.
url: https://anil.recoil.org/notes/2026w31
date: 2026-08-02T00:00:00-00:00
preview_image: https://anil.recoil.org/images/cenews-tessera-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I had a bunch of infrastructure work I had to get done this week, and so I went headsdown hacking to sort out both <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> and the <a href="https://anil.recoil.org/projects/ce">Evidence TAP</a> this week!</p>
<p><img src="https://anil.recoil.org/images/cenews-tessera-1.webp" alt="%rc" title="Yours truely in the Cambridge Evening News">
I spent it <a href="https://anil.recoil.org/news.xml#moving-a-petabyte-of-tessera-embeddings">moving a petabyte of TESSERA embeddings</a> onto Source Cooperative, taking <a href="https://anil.recoil.org/news.xml#a-detour-to-investigate-icechunk">a detour to investigate Icechunk</a> before settling on <a href="https://anil.recoil.org/news.xml#transcoding-to-zarr-on-aws-fargate-spot">transcoding to Zarr on AWS Fargate Spot</a>. Then my custom PDF downloader <a href="https://anil.recoil.org/news.xml#arise-taposaur-and-obtain-the-literature-for-evidence-tap">Taposaur arose</a> to obtain the literature for Evidence TAP, by <a href="https://anil.recoil.org/news.xml#getting-pdf-structured-metadata-with-grobid">extracting structured metadata with GROBID</a> and downloading papers with <a href="https://anil.recoil.org/news.xml#a-capability-based-ocaml-paper-downloader">a capability-based OCaml downloader</a>. I continued <a href="https://anil.recoil.org/news.xml#eio-pathfinding-on-windows-and-beyond">Eio work Windows</a> now that 1.4 is out, mirrored some databases for <a href="https://anil.recoil.org/news.xml#the-dash-for-life">the Dash for Life</a>, and wanted to advertise some <a href="https://anil.recoil.org/news.xml#job-opportunities-on-deep-learning-and-sdms">super fun jobs on deep learning SDMs</a>. The <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a> at the end feature an unexpected newspaper appearance by yours truely and my first new unikernel boot trace in some years!</p>
<h2><a href="https://anil.recoil.org/news.xml#moving-a-petabyte-of-tessera-embeddings" class="anchor" aria-hidden="true"></a>Moving a petabyte of TESSERA embeddings</h2>
<p><a href="https://www.tunbury.org/">Mark Elvers</a> and I have been getting the full range of <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> embeddings up on <a href="https://source.coop">Source Cooperative</a> for the <a href="https://anil.recoil.org/notes/2026w29">past few weeks</a>. We split this latest work into two steps: first move the existing npy-based embeddings as-is, and then transcode to Zarr v3. This week the npy ones fully synched (almost a petabyte!) covering TESSERA v1 alongside selectively generated v1.1 and v2-beta ones.</p>
<p>The first half of my <a href="https://github.com/ucam-eo/geotessera/pull/337">geotessera#337</a> PR switches us over to the <code>data.source.coop</code> endpoint for HTTP downloads, which lets us drop our S3-specific logic we had in geotessera 0.9. That code was weirdly problematic and complex; EC2 seems to drop connections quite quickly on bulk downloads, which is what most of our users do. My tests on the Source Coop proxy show Cloudflare R2 doing a great job of edge caching, so this looks like a solid long-term solution that's still using S3 under the hood, but not exposing that to our library users.</p>
<p>I'm also in the Source Coop Slack now, with extremely responsive developers on the other end. I'll cut a geotessera release with the npy support this week, with the Zarr conversion to follow shortly I hope!</p>
<h3><a href="https://anil.recoil.org/news.xml#a-detour-to-investigate-icechunk" class="anchor" aria-hidden="true"></a>A detour to investigate Icechunk</h3>
<p>The Zarr conversion requires transcoding the npy sources over to a fresh Zarr store, so I spent some time exploring how best to do this.</p>
<p>The only alternative that's credible is <a href="https://icechunk.io">Icechunk</a>, Earthmover's transactional storage engine for Zarr that adds git-like version control over object storage. That's obviously very attractive for TESSERA, since we dynamically generate embeddings for a <a href="https://anil.recoil.org/papers/2026-tessera-v2">growing set</a> of model variants and could really use version control. I knocked up a quick version of geotessera using the Icechunk Python library instead of Zarr's, which was a trivial drop-in since <a href="https://icechunk.io/en/latest/getting-started/quickstart/">Icechunk speaks the Zarr Python interface</a>.</p>
<p>Behind the scenes though, Icechunk writes to S3 in a <a href="https://icechunk.io/en/stable/reference/spec-v2-1/">custom format</a>. That format seems rock solid in my testing, but is a blocker for us since we need client access from languages other than Python. There's an <a href="https://icechunk.io/en/stable/reference/icechunk-rust/">Icechunk Rust crate</a> that isn't intended for external binding, a <a href="https://icechunk.io/en/stable/reference/icechunk-js/">JavaScript version</a> for browser use, but nothing I could bind (e.g.) OCaml against myself. There's a <a href="https://github.com/earth-mover/icechunk/pull/1722">vibe-coded PR for a C API and Julia bindings</a>, but it's not high quality enough so was rejected (rightly I think) by the Earthmover team. So I'll reluctantly come back to Icechunk when it grows a more stable FFI story, but won't use it just yet over a 'raw' Zarr v3 publish.</p>
<h3><a href="https://anil.recoil.org/news.xml#transcoding-to-zarr-on-aws-fargate-spot" class="anchor" aria-hidden="true"></a>Transcoding to Zarr on AWS Fargate Spot</h3>
<p>I went back to my <a href="https://anil.recoil.org/notes/tessera-zarr-v3-layout">Zarr v3 geoembeddings layout</a>, with a static store on source.coop spanning 2017-2025 (extensible to future years, though not easily to earlier ones without using the aforementioned Icechunk). Finding the CPU to transcode ~10 million tiles without burning a fortune in egress bandwidth was surprisingly tricky! Running it in Cambridge via our current server (which we're trying to deprecate in favour of cloud hosting) would take around six months (!).
After a number of failed experiments, I came across <a href="https://aws.amazon.com/fargate/pricing/">Amazon Fargate Spot</a>, an order of magnitude cheaper way to run a Docker container in the same region as the embeddings (us-west-2) and serverless so we didn't have to manage all the VMs manually.</p>
<p>The rest of my <a href="https://github.com/ucam-eo/geotessera/pull/337">geotessera#337</a> PR makes the <code>zarr-fill</code> cli stateless/incremental. It scans each zone first via Zarr to determine what's needed for a particular UTM zone, and also stores sampled metadata so the RGB pyramids are fast to calculate at the end. Without the samples we'd have to read the full petabyte back over the network just to run a PCA for the preview you see in <a href="https://tze.geotessera.org">tze.geotessera.org</a>!</p>
<p>Mark did a quick trial run, and our spot instance goes about 100x faster than from Cambridge, so we're going to run every UTM zone in parallel for a rough cost of 1500 quid for the whole world (I'll find out in a few days if our back-of-the-envelope math is right or not). This compares well to the 50 grand bill from my original naive plan!
Once TESSERA v1 is converted, v1.1 will be quick as it has fewer tiles as we've not done a full global run yet. In the future I'm aiming for all our embeddings generation to be Zarr-first, so getting this giant transcode out of the way will be a big boost.</p>
<h2><a href="https://anil.recoil.org/news.xml#arise-taposaur-and-obtain-the-literature-for-evidence-tap" class="anchor" aria-hidden="true"></a>Arise Taposaur and obtain the literature for Evidence TAP!</h2>
<p>I've also been making inroads on the processing pipeline for <a href="https://anil.recoil.org/notes/2026w30">Evidence TAP</a>, now that we have more users. This is codenamed 'Taposaur', and has two major pieces: a custom downloader to deal with the (often ridiculous) hoops publishers make us jump through, and a metadata index over the millions of resulting PDFs that we've snarfed. My goal is to get this piece integrated with <a href="https://samreynolds.org">Sam Reynolds</a> work on <a href="https://www.samreynolds.org/#copilot">the Conservation Copilot</a> this summer so it can have secure (and local model only) access to our fulltext paper database.</p>
<h3><a href="https://anil.recoil.org/news.xml#getting-pdf-structured-metadata-with-grobid" class="anchor" aria-hidden="true"></a>Getting PDF structured metadata with GROBID</h3>
<p>Since we already have millions of downloaded papers from the last year's work, the first thing I cleaned up was the metadata processing. I'm building on three brilliant upstream projects <a href="https://github.com/kermitt2/biblio-glutton">biblio-glutton</a>, <a href="https://github.com/kermitt2/grobid">GROBID</a> and <a href="https://github.com/kermitt2/Pub2TEI">Pub2TEI</a>, which all fit together to mirror public indices like <a href="https://crossref.org">Crossref</a> and provide HTTP APIs that scan a PDF into structured metadata:</p>
<div role="region"><table>
<tbody><tr>
<th>service</th>
<th>port</th>
<th>role</th>
</tr>
<tr>
<td>glutton</td>
<td>8080</td>
<td>bibliographic lookup over 175M+ Crossref records</td>
</tr>
<tr>
<td>grobid</td>
<td>8070</td>
<td>convert PDF to TEI via deep-learning models</td>
</tr>
<tr>
<td>pub2tei</td>
<td>8060</td>
<td>convert publisher XML to TEI</td>
</tr>
<tr>
<td>elasticsearch</td>
<td>9200</td>
<td>glutton's search index</td>
</tr>
</tbody></table></div><p>The 'TEI' here refers to an <a href="https://tei-c.org/release/doc/tei-p5-doc/en/html/SG.html">XML format for scholarly publishing</a> that acts as an interchange format across all the publishers' slightly different schemas. I'll write more on this later, but here's a preview of the web interface:</p>
<p><img src="https://anil.recoil.org/images/taposaur-ss-1.webp" alt="%c" title="The Taposaur index with a subset of our PDFs, and the corpus-wide sharded TEI conversion"></p>
<p>The overall database runs from an OCaml scheduler that dispatches jobs over to a Docker Swarm running Grobid and the other services. It'll take about a month to churn through the full set, and outputs the TEI form into a nice local filesystem that I can then stash on our Ceph storage cluster.</p>
<p><img src="https://anil.recoil.org/images/taposaur-ss-2.webp" alt="%c" title="The author metadata is parsed out of the PDF into structured form. Here's a Bill Sutherland paper for example"></p>
<p>The best thing about TEI is that is has a structured representation of much paper metadata. For example, I grabbed a random paper by <a href="https://www.zoo.cam.ac.uk/directory/bill-sutherland">Bill Sutherland</a> about <a href="https://doi.org/10.1002/ece3.11237">replacing bar charts with histograms</a> and it has ORCIDs and even the corresponding author for the paper.</p>
<p><img src="https://anil.recoil.org/images/taposaur-ss-3.webp" alt="%c" title="Journal, licence, keyword and funder records for the same paper, matched against Crossref via biblio-glutton"></p>
<p>Then there's also the usual Crossref data, and because we have a local database, I can introduce overrides or evidence synthesis specific metadata quite easily for our own use.  I'm very grateful to the Grobid team for open-sourcing their infrastructure; they are also working on some really cool <a href="https://arxiv.org/abs/2210.15600">open mining</a> and <a href="https://arxiv.org/abs/2512.11192">science literature datasets</a> that I'm excited to read more about later in the year.</p>
<h3><a href="https://anil.recoil.org/news.xml#a-capability-based-ocaml-paper-downloader" class="anchor" aria-hidden="true"></a>A capability-based OCaml paper downloader</h3>
<p>The PDF paper downloader itself is surprisingly complex, since it needs quite low-level HTTP control that depends on the quirks of each publisher's API. If we just do a simple <code>curl</code> based download it often goes wrong (i.e. HTML from a Cloudflare bot trap), despite us supplying the right API keys and having full legal permission to fetch the content.
So a while back I defined per-publisher strategies for how the downloader should behave, which was subsequently improved by <a href="https://www.lambdacambridge.com/robin-message">Robin Message</a> and <a href="https://toao.com">Sadiq Jaffer</a>. I've been building a new version in <a href="https://github.com/ocaml-multicore/eio">Eio</a>, porting the excellent Python work Robin did, which was itself based on OCaml code I hacked together last year when first getting started!</p>
<p>The new design was driven by my need to use custom downloaders in some cases, for example the built-in Windows HTTP downloader (necessary for some very obscure literature sources that require a special Windows client). Taposaur therefore uses an Eio frontend providing the high-level download logic, with pluggable backends performing the actual transfers. These backends include <a href="https://curl.se/libcurl/">libcurl</a>, which works with a vast number of real-world quirks; there's also a <a href="https://developer.apple.com/documentation/foundation/urlsession">macOS NSURLSession</a> backend, a browser one using <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API">fetch()</a>, and a cohttp-eio one that isn't quite functional yet due to some missing protocol pieces.</p>
<p>The frontend extends the Eio <a href="https://roscidus.com/blog/blog/2023/04/26/lambda-capabilities/">capability model</a>. Each publisher module can define a more narrow HTTP capability that carries its own API keys and rate limits. For example, Wiley wants a client token in a custom header, drawn here from a rotating pool, and requests paced a few seconds apart. Article requests then redirect to a separate content host, which the capability permits without opening up the rest of the web:</p>
<pre><code class="language-ocaml">let wiley =
  http
  |&gt; Fetch.with_limits ~clock ~min_interval:3.0
  |&gt; Wiley.client ~tokens
in Wiley.pdf ~sw wiley doi
</code></pre>
<p>Springer instead binds an API key as a query parameter (so a URL a caller builds never carries it), batches up to 25 DOIs per query, and expects you to keep one request in flight and sit out a half-hour pause after every 429. The same combinators compose in a different order, with the service's own retry schedule wrapped in before the key is bound:</p>
<pre><code class="language-ocaml">let springer =
  http
  |&gt; Fetch.with_limits ~clock ~max_concurrent:1
  |&gt; Fetch.with_retry ~clock ~random ~config:Springer.retry
  |&gt; Springer.client ~api_key
in
Springer.articles ~sw springer dois
</code></pre>
<p>Note that both of these capabilities are minted from the very same Eio <code>http</code> client; each service module just wraps it with its own polymorphic variant tag. The tags mean the two end up with different types, so passing the Springer client to <code>Wiley.pdf</code> is a compile-time error rather than a confusing burst of 403s at 3am:</p>
<pre><code class="language-ocaml"># let wiley = Wiley.client ~tokens http;;
val wiley : Wiley.t
# let springer = Springer.client ~api_key http;;
val springer : Springer.t
# Wiley.pdf ~sw springer doi;;
Error: The value springer has type Springer.t = Springer.tag ty t
       but an expression was expected of type Wiley.t = Wiley.tag ty t
       Type Springer.tag = [ `Generic | `Springer ]
       is not compatible with type Wiley.tag = [ `Generic | `Wiley ]
       The second variant type does not allow tag(s) `Springer
</code></pre>
<p>However, both can still be driven by the same Eio http calls. A service capability remains an ordinary <code>Fetch</code> client underneath despite the service-specific tag. Generic operations like <code>Fetch.get</code> are polymorphic in the tag, but the service-specific credentials and rate limits still attached to every request:</p>
<pre><code class="language-ocaml">let wiley_pdf = Fetch.get ~sw wiley (Wiley.url ~doi) in
let springer_batch = Fetch.get ~sw springer (Springer.url ~dois) in
(* ... *)
</code></pre>
<p>This means a download strategy can be written once against the plain client interface and handed whichever publisher capability is appropriate. So far this is a pretty cool way to write HTTP interfaces in OCaml where site-specific HTTP overrides are needed.  The work-in-progress code is in my <a href="https://tangled.org/anil.recoil.org/taposaur">taposaur repository</a> on Tangled, and I'll publish the fetch library separately once I've iterated on the design within Taposaur some more and stabilise it all.</p>
<h2><a href="https://anil.recoil.org/news.xml#eio-pathfinding-on-windows-and-beyond" class="anchor" aria-hidden="true"></a>Eio pathfinding on Windows and beyond</h2>
<p>Following on from <a href="https://anil.recoil.org/notes/2026w30">last week's Windows promise</a>, I've started on Eio work with <a href="https://github.com/ocaml-multicore/eio/pull/908">a draft PR implementing Windows native paths</a> as a Pi backend, following the suggestion from <a href="https://roscidus.com">Thomas Leonard</a>. There's enough spec divergence between POSIX and Windows logic that we dispatches via the Eio Pi layer and treat them separately. I'm still fuzzing the Windows parsing logic to find simplications (esp. Cygwin and WSL), as it's remarkably fiddly.</p>
<p>For example, Windows can have drive-local relative paths that have separate cwds per processs:</p>
<pre><code>C:\src&gt; D:
D:\&gt; cd scratch
D:\scratch&gt; C:
C:\src&gt; type D:notes.txt
  --&gt;  reads D:\scratch\notes.txt (D: remembers its own directory)
C:\src&gt; type C:notes.txt
  --&gt;  reads C:\src\notes.txt
C:\src&gt; type C:\notes.txt
  --&gt;  reads \notes.txt at the root of C:
</code></pre>
<p>However, after much spelunking it turns out that these drive-relative paths are
only <em>emulated</em> in <code>cmd.exe</code> these days via magic environment variables, so I
can disregard them entirely for the purposes of Eio support. Argh!</p>
<p>In other minor improvements, Taposaur's millions of PDFs running slowly on Ceph
prompted me to submit <a href="https://github.com/ocaml-multicore/eio/pull/910">a improvement to getdents in
Eio</a>, making traversal of
directories with ~1m files about 30% faster by cutting syscalls by an order of
magnitude.</p>
<p>I've also been extremely enjoying myself playing with the work <a href="https://patrick.sirref.org">Patrick Ferris</a> has done with his new <a href="https://git.sirref.org/merry">merry shell</a>. As part of that, I put up <a href="https://github.com/ocaml-multicore/eio/pull/911">a Eio <code>mknod</code> PR</a> for creating device nodes directly in Eio. This removes one of the C bindings in a prototype wasm shell he's experimenting with.</p>
<h2><a href="https://anil.recoil.org/news.xml#the-dash-for-life" class="anchor" aria-hidden="true"></a>The Dash for Life</h2>
<p>Another place where the literature database will eventually be useful is in <a href="https://shaneweisz.com">Shane Weisz</a> work on <a href="https://dashforlife.org">Dash for Life</a>, the new name for the <a href="https://anil.recoil.org/projects/enki">Enki</a> codename! He has been brilliantly pouring in more and more data sources while preserving the ergonomics of the UI.</p>
<p>As part of my infrastructure push this week, I also created a mirror of the iNaturalist and OpenStreetMap databases internally here as well, with a view to running an API server so that we don't put an undue load on public services for Shane's work.</p>
<p><a href="https://www.dashforlife.org/?taxa=mammals&amp;search=ochotona%20alpina&amp;species=41255"> <img src="https://anil.recoil.org/images/dashforlife-ss-pika.webp" alt="%c" title="Obligatory pika dashboard"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#job-opportunities-on-deep-learning-and-sdms" class="anchor" aria-hidden="true"></a>Job opportunities on deep learning and SDMs</h2>
<p>If you're looking for a <em>really</em> fun research opportunity and want to get into
deep learning, I'm helping my buddy <a href="https://www.zoo.cam.ac.uk/people/andrea-manica">Andrea Manica</a> over at Zoology with a
project on deep learning for population genomics and biodiversity forecasting.
See <a href="https://www.jobs.cam.ac.uk/job/56448/">advert 1</a> on the genomics side:</p>
<blockquote>
<p>The successful candidates will design and implement deep learning models
capable of integrating heterogeneous data sources, including genomic
variation, species occurrence records, climate reconstructions, environmental
layers, and remotely sensed observations. The methods will be applied to
three case studies focussing on African megafauna, European butterflies and
moths, and UK pollinators for which we have extensive genomic resources,
including time series based on museum specimens. The researchers will
contribute directly to the development of a new generation of predictive
biodiversity models that combine mechanistic understanding with
state-of-the-art artificial intelligence.</p>
</blockquote>
<p>And <a href="https://www.jobs.cam.ac.uk/job/56450/">advert 2</a> on the LLM side (we hope to use the Evidence TAP infra here of course):</p>
<blockquote>
<p>The principal aim of this post is the development of agentic LLM-based
systems that can extract, organise, and validate biodiversity information
from the published scientific literature at unprecedented scale. The
successful candidate will design and implement AI workflows capable of
processing more than one million scientific papers to identify and extract
georeferenced information on species distributions, ecological interactions,
demographic processes, environmental associations, and other
biodiversity-relevant data. These data will form a key component of the
CISGeM framework, complementing genomic, climatic, and environmental datasets
and enabling a richer representation of biodiversity dynamics through space
and time. The researcher will contribute directly to the development of a new
generation of biodiversity forecasting models that combine mechanistic
understanding with state-of-the-art artificial intelligence.</p>
</blockquote>
<p>Andrea spent an hour with me at the Mill explaining the intricacies of ensemble-based
deep learning approaches for genomic mapping, so I can't wait to dig into this more
later in the year! Also very closely tied to <a href="https://mynameismwd.org">Michael Dales</a> work on <a href="https://digitalflapjack.com/weeknotes/2026-08-03/">contemporary habitat mapping</a> and <a href="https://aneeshnaik.github.io/">Aneesh Naik</a> on <a href="https://www.aneeshnaik.com/blogposts/20260720_weeknotes_2026_29.html">plant SDMs</a>, just displaced a few hundred thousand years backwards!</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun links</h2>
<p><a href="https://en.wikipedia.org/wiki/Chris_Smith,_Baron_Smith_of_Finsbury">Chris Smith</a>, the Chancellor of Cambridge and former Master of my own Pembroke College, wrote a nice column <a href="https://www.telegraph.co.uk/news/2026/07/24/frank-whittle-engineering-lab-jet-engine-tech-economy/">in the Telegraph on the new Whittle Laboratory</a>, which <a href="https://www.cam.ac.uk/news/the-king-officially-opens-cambridges-new-whittle-laboratory">the King opened last month</a>.</p>
<p><a href="https://simon.peytonjones.org/">Simon Peyton Jones</a> walked into the office and dropped a physical copy of the Cambridge Evening News, featuring yours truely! I'm not entirely sure where they got the interview from; it must have been from the AMD/Vultr press coverage recently. Either way, I've saved the precious paper copy for posterity!</p>
<p><img src="https://anil.recoil.org/images/cenews-tessera-2.webp" alt="%c" title="Note that I neither died nor allowed instant forest loss, just unfortunate cropping"></p>
<p>While updating <a href="https://thicket.dev">OCaml Thicket</a> I noticed a cool PR adding a <a href="https://github.com/oxcaml/oxcaml/pull/6549">baremetal OxCaml</a> runtime variant, and so I messed around over the weekend and got Eio booting on UEFI under KVM. Nothing production worthy in the slightest, but also didn't take <a href="https://github.com/avsm/flambda-backend/commit/b997c2d9ea795d398fcb16e6141865f1be74056c">much extra code</a>.</p>
<pre><code>BdsDxe: loading Boot0002 "UEFI QEMU HARDDISK QM00001 " from PciRoot(0x0)/Pci(0x1F,0x2)/Sata(0x0,0xFFFF,0x0)
BdsDxe: starting Boot0002 "UEFI QEMU HARDDISK QM00001 " from PciRoot(0x0)/Pci(0x1F,0x2)/Sata(0x0,0xFFFF,0x0)
efi_main   : heap ready, starting oxcaml runtime
[   0.00ms] === hello-uefi-ocaml: oxcaml running as a UEFI boot service ===
[   1.03ms] list       : 2000 elements, sum = 2664667000
[   1.83ms] gc         : 7134 minor words, 3 major collections
[   2.64ms] exceptions : caught Failure "nth"
[   3.25ms] floats     : sqrt 2 = 1.414214, cos pi = -1.000000, exp 1 = 2.718282
[   4.25ms] strings    : UEFI-BOOT-SERVICE
[   4.82ms] gop        : 1280x800, stride 1280 px, 4000 KiB framebuffer, 30 modes
[  12.42ms] gop        : filled 1024000 px from oxcaml in 6564 us
[  13.24ms] gop        : centre pixel reads back 0x007f7f80
[  13.97ms] effects    : yielded 1 4 9 16 25, 5 resumes
[  14.65ms] effects    : caught: Failure("from inside a fiber")
[  15.44ms] effects    : fiber recursion depth 100 ok
[  16.26ms] effects    : fiber recursion depth 10000 ok
[  18.46ms] effects    : fiber recursion depth 100000 ok
[  34.50ms] effects    : fiber recursion depth 1000000 ok
[  35.27ms] revents    : before start: ring=0 words=0 nonzero=0
[  36.21ms] revents    : Runtime_events.start () returned
[  37.05ms] revents    : after start:  ring=1 words=196626 nonzero=45
[  38.00ms] revents    : wrote 500 events, nonzero words 45 -&gt; 1545 (+1500)
[  38.93ms] eio        : starting scheduler
[  39.48ms] eio        : fiber A waiting on promise
[  40.15ms] eio        : fiber B sleeping 1ms then resolving
[  41.90ms] eio        : fiber A got 42 from promise
[  42.85ms] eio        : interleaved a1 b1 a2 b2 a3 b3
[  48.52ms] eio        : Fiber.first winner = fast
[  49.16ms] eio        : scheduler drained cleanly
[  49.80ms] === oxcaml finished, returning to efi_main ===
efi_main   : oxcaml returned; powering off
</code></pre><h1>References</h1><ul><li>Madhavapeddy (2026). .plan-26-29: Perfect weather, imperfectly measured, precisely predicted. <a href="https://doi.org/10.59350/9xhzk-z8549" target="_blank"><i>10.59350/9xhzk-z8549</i></a></li>
<li>Feng et al (2026). TESSERA v2: Scaling Pixel-wise Earth Foundation Models. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.03949" target="_blank"><i>10.48550/arXiv.2607.03949</i></a></li>
<li>Madhavapeddy (2026). Streaming millions of TESSERA tiles over HTTP with Zarr v3. <a href="https://doi.org/10.59350/tk0er-ycs46" target="_blank"><i>10.59350/tk0er-ycs46</i></a></li>
<li>Stuart et al (2024). Sea stack plots: Replacing bar charts with histograms. <a href="https://doi.org/10.1002/ece3.11237" target="_blank"><i>10.1002/ece3.11237</i></a></li>
<li>Foppiano et al (2022). Automatic extraction of materials and properties from superconductors scientific literature. arXiv. <a href="https://doi.org/10.48550/arXiv.2210.15600" target="_blank"><i>10.48550/arXiv.2210.15600</i></a></li>
<li>Foppiano et al (2026). SciLaD: A Large-Scale, Transparent, Reproducible Dataset for Natural Scientific Language Processing. arXiv. <a href="https://doi.org/10.48550/arXiv.2512.11192" target="_blank"><i>10.48550/arXiv.2512.11192</i></a></li></ul>
