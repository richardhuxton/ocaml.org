---
title: '.plan-26-36: Ground control to major MODIS'
description: Tessera v2 beta2 smooths space at the cost of time, new Pembroke Associate
  Scholars, Scrutineer's git remediation workflow, paying the Windows tax, and retiring
  AI disclosure metadata.
url: https://anil.recoil.org/notes/2026w36
date: 2026-09-06T00:00:00-00:00
preview_image: https://anil.recoil.org/images/pemb-etap-aa-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Got back to Cambridge mid-week and back on the keyboard to start preparing for Michaelmas term, which creeps up fast! <a href="https://anil.recoil.org/news.xml#tessera-v2-beta2-and-the-temporal-axis">Tessera v2 beta2</a> turns out to smooth space at the cost of time, so beta1 may be as far as the v2 series goes without a retrain. Evidence TAP gets <a href="https://anil.recoil.org/news.xml#more-polite-crawling-for-the-evidence-tap">a splendid explainer from Sam</a> and some new Associate Scholars at Pembroke, along with <a href="https://anil.recoil.org/news.xml#text-and-data-mining-reservation">a TDM reservations library</a> and some thoughts on <a href="https://anil.recoil.org/news.xml#working-on-both-ocaml-and-oxcaml">working across OCaml and OxCaml</a>. Elsewhere there's <a href="https://anil.recoil.org/news.xml#scrutineers-git-workflow-for-security-scanning">Scrutineer's git workflow</a> and an invitation to a bug sprint, rather a lot of <a href="https://anil.recoil.org/news.xml#the-windows-tax">the Windows tax</a>, why <a href="https://anil.recoil.org/news.xml#ai-disclosure-is-more-of-a-social-issue-than-technical">AI disclosure is a social problem</a> rather than a technical one, <a href="https://anil.recoil.org/news.xml#a-fourth-recoil-host">a fourth Recoil host</a> running OpenBSD, and the usual <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-v2-beta2-and-the-temporal-axis" class="anchor" aria-hidden="true"></a>Tessera v2 beta2 and the temporal axis</h2>
<p><a href="https://www.tunbury.org/">Mark Elvers</a> and I published more <a href="https://anil.recoil.org/projects/tessera">Tessera</a> v2.0 beta2 to <a href="https://source.coop/tessera/tessera/zarr">source.coop</a> this week to test out some fixes to the improved embeddings.</p>
<p>The v1 embeddings sometimes show a "checkerboard" where Sentinel-1 and Sentinel-2 coverage is thin. This happens in areas where there are few satellite observations for the model to infer embeddings from. v2-beta2 tried to smooth them over by applying <a href="https://modis.gsfc.nasa.gov/about/">MODIS</a> corrections during inference to augment the data, since MODIS is coarser spatially but has more consistent coverage. This actually works pretty well, and <a href="https://www.tunbury.org/">Mark Elvers</a> measured <a href="https://www.tunbury.org/2026/08/31/week-35-2025/#the-tessera-temporal-axis">a 35% reduction in the boundary artefact</a>.</p>
<p>Unfortunately, this does regress another aspect of the embedding temporally.. The v2 model wasn't trained on MODIS, and so inferring with it shifts the embeddings year-on-year in ways the model does not anticipate. Because MODIS observations themselves vary year to year, this breaks the cross-year alignment <a href="https://anil.recoil.org/notes/tessera-v11-out">v1.1 added</a>. <a href="https://patball1.github.io">James G. C. Ball</a> caught this by testing them on the v2 <a href="https://anil.recoil.org/papers/2026-tessera-trentino">Trentino</a> embedding as while intra-year cross-validation is neutral, training on 2018 and testing on 2019 dropped the macro F1 (from 0.586 -&gt; 0.511) wherase the v2-beta1 embeddings instead <em>improve</em> (from 0.576 to 0.626).</p>
<p>So it looks like, somewhat reluctantly, that the v2-beta1 embeddings may be the best we can do in the v2 series without retraining a model with more MODIS observations or (maybe) doing some extensive fine tuning. Opinions from other testers are still incoming, so do get in touch if you have a view. We're still looking for GPU resource to train the v2.5 model on corrected data rather, so get in touch if you can help with those too!</p>
<p>Meanwhile, the wall-to-wall inference v1.1 run for 2017 to 2025 is underway after last month's GPU shortage and should finish in a few weeks. I released <a href="https://github.com/ucam-eo/geotessera/releases/tag/v0.10.2">GeoTessera 0.10.2</a> as a point release to fix v1.1 Zarr URL resolution that broke in last week's <a href="https://anil.recoil.org/notes/2026w35">Source Cooperative migration</a>, and am working on a <a href="https://github.com/ucam-eo/geotessera/pull/403">robustness pass</a> for recoverable Zarr writes and <a href="https://anil.recoil.org/news.xml#the-windows-tax">improved Windows support</a> for Zarr.</p>
<h2><a href="https://anil.recoil.org/news.xml#more-polite-crawling-for-the-evidence-tap" class="anchor" aria-hidden="true"></a>More polite crawling for the Evidence TAP</h2>
<p><a href="https://samreynolds.org">Sam Reynolds</a> has written <a href="https://www.pem.cam.ac.uk/college/news/august-2026-dr-sam-reynolds">a splendid piece for the Pembroke blog</a> on what the Evidence Traceable AI Pipeline is for (as part of his introductory post for College!).</p>
<blockquote>
<p>For the last three years, we have been working with colleagues in Computer
Science on an AI pipeline that finds, classifies, extracts data and
summarises findings from the academic literature using self-hosted
open-access Large Language Models, benchmarking performance against our hard
won, human created database. Underpinning this are pioneering agreements with
the largest scientific publishers, brokered with the help from the University
Library, to download millions of academic papers.  We are also developing
tools that let decision makers interact with this extracted information. This
is the foundation for the Evidence Traceable Accountable Pipeline (E-TAP)
project here at Pembroke, generously funded by the MacArthur Foundation,
through which we hope to generalise this approach for other fields.
<cite>-- <a href="https://www.pem.cam.ac.uk/college/news/august-2026-dr-sam-reynolds">Sam Reynolds, Aug 2026</a></cite></p>
</blockquote>
<p>I'm delighted that <a href="https://samreynolds.org">Sam Reynolds</a> and Mélanie Gréaux have now joined Pembroke as
Associate Scholars alongside <a href="https://toao.com">Sadiq Jaffer</a>. Having the <a href="https://anil.recoil.org/projects/ce">conservation evidence</a>, education experts and the computer scientists all in College
will make for some fun socials, especially as many other colleagues from other
departments have expressed interest in the project as well!</p>
<p><img src="https://anil.recoil.org/images/pemb-etap-aa-1.webp" alt="%c" title="Welcome Mel, Sam and Sadiq to Pembroke!"></p>
<h3><a href="https://anil.recoil.org/news.xml#text-and-data-mining-reservation" class="anchor" aria-hidden="true"></a>Text and Data Mining Reservation</h3>
<p>On the coding side, I released a <a href="https://github.com/ocaml/opam-repository/pull/30616">TDMRep 1.0</a> library to opam. Our download engine is crawling papers at some scale now, and publishers have adopted a standard to express their responses to AI crawlers via the <a href="https://www.w3.org/community/reports/tdmrep/CG-FINAL-tdmrep-20240202/">W3C TDM Reservation Protocol</a>. My <a href="https://tangled.org/anil.recoil.org/ocaml-tdmrep">tdmrep library</a> gives us the ability to read this metadata from OCaml code, and the Taposaur crawler declares its intentions (non-commercial, research use only) when requesting papers as well.</p>
<p>I also pulled out <a href="https://github.com/ocaml/opam-repository/pull/30656">json-pointer 1.0</a> into my <a href="https://anil.recoil.org/notes/tangled-and-ci">tangled</a> repos. This is a query syntax for JSON that's seeing some adoption (e.g. <code>/users/0/name</code>), and I'm using this to parse Semantic Scholar fulltexts in the Evidence TAP corpus.</p>
<p>Since we're using these libraries in my <a href="https://anil.recoil.org/news.xml#working-on-both-ocaml-and-oxcaml">OxCaml httpz stack</a> now, I've also started extracting that code out of our internal monorepo. E-TAP is a "live" service now, with the stack fetching papers and categorising them, so I'm getting more handson experience with OxCaml in production.</p>
<h3><a href="https://anil.recoil.org/news.xml#working-on-both-ocaml-and-oxcaml" class="anchor" aria-hidden="true"></a>Working on both OCaml and OxCaml</h3>
<p>One challenge is that a lot of our high-performance infra (both for ETAP and
Tessera) is built in OxCaml, which is a fast moving target as Jane Street
release compilers quickly and with breaking changes. There's also some very
ugly (but performant) edges to the language as it evolves, such as the use of
ppx to get around the lack of layout polymorphism. While this makes code hard
to edit sometimes, I've found that coding in OCaml and then agentically
translating to OxCaml works very well, since the type system catches layout
issues very reliably.</p>
<p>I got my <a href="https://github.com/oxcaml/opam-repository/pull/59">oxcaml/opam-repository#59</a> merged, which makes it much easier to mix OCaml and OxCaml packages through the <a href="https://anil.recoil.org/notes/oxcaml-opam-guards">guard packages</a>.
However, this approach does neccessitate having full control over dependencies, since writing a
parser in OxCaml is very different from normal OCaml and many dependencies need annotations.  My
<a href="https://anil.recoil.org/notes/oxcaml-httpz">httpz</a> library has matured in recent months from a parser into
a full HTTP stack with a zero-allocation <code>fetch</code> for HTTP clients, and
<code>proffer</code>, a portable HTTP server layer. I've been experimenting with various
strategies to packaging this up which I'll share in the coming weeks.</p>
<p>I've also started <a href="https://tangled.org/anil.recoil.org/httnope">httnope</a>, to
build a conformance corpus that supplies adversarial peers to httpz clients or
servers and checks the observable effects on fresh connections. This is more
useful than testing against a well-behaved HTTP peer since all the
"interesting" failures all found in behaviours no reasonable HTTP server would
send (but bad attackers probably will). I'm bootstrapping this using LLMs
parsing RFCs, but <a href="https://github.com/samoht">Thomas Gazagnaire</a> has also pointed me to some of his work on an FSM
library that he's having good success with, so I'll work more on this...</p>
<h2><a href="https://anil.recoil.org/news.xml#scrutineers-git-workflow-for-security-scanning" class="anchor" aria-hidden="true"></a>Scrutineer's git workflow for security scanning</h2>
<p>The <a href="https://anil.recoil.org/notes/scrutineer-local-llm">Scrutineer security scanner deployment</a> from last week now has a private Git remediation workflow I added.  A gated patch attempt is exported as a cherry-pickable commit on a private Git remote, so I can review with:</p>
<pre><code>$ git fetch scrutineer '+refs/findings/*:refs/remotes/scrutineer/findings/*'
$ git cherry-pick scrutineer/findings/F-63
</code></pre>
<p>...over an ssh tunnel. Triage decisions go back as Git notes under <code>refs/notes/scrutineer/triage/&lt;principal&gt;</code>. This is a local-only prototype for now, but I'll continue to gain experience over the next few weeks.</p>
<p><img src="https://anil.recoil.org/images/scrutineer-ss-cstruct-1.webp" alt="%c" title="Scrutineer's proposed patch for the reverse branch of Cstruct.tail, with the private Git review commands underneath"></p>
<p>There are quite a few bugs the scanners are finding across a variety of repos, but I released the ones found in <a href="https://github.com/mirage/ocaml-cstruct">cstruct</a> via <a href="https://github.com/mirage/ocaml-cstruct/pull/324">#324</a> which corrects indexing and subview offset handling. <a href="https://github.com/samoht">Thomas Gazagnaire</a> reviewed it and I cut <a href="https://github.com/mirage/ocaml-cstruct/releases/tag/v6.3.0">6.3.0</a>. That only leaves about 150 verified bugs to triage elsewhere...</p>
<p>I also got a couple of fixes merged upstream into Scrutineer (<a href="https://github.com/alpha-omega-security/scrutineer/pull/949">#949</a> and <a href="https://github.com/alpha-omega-security/scrutineer/pull/950">#950</a>). Separately, we've been invited to a week-long bug sprint as part of <a href="https://trailofbits.com/patch-the-planet/">Patch the Planet</a>, the Trail of Bits initiative that pairs security engineers with maintainers. Their projects already include Python, PyPI, cURL and Go, so OCaml would be in decent company! I'll email the OCaml security team and a few others next week to arrange something for October.</p>
<h2><a href="https://anil.recoil.org/news.xml#the-windows-tax" class="anchor" aria-hidden="true"></a>The Windows tax</h2>
<p>Unfortunately there comes a time in every open source library's life when it has to pay a Windows tax, and productivity comes to a grinding halt. Since <a href="https://github.com/ucam-eo/geotessera">GeoTessera</a> users are typically ecologists rather than computery people, I have to figure out new bugs whenever I add a new feature (in this case, Zarr).</p>
<p>Every time I SSH'd into my Windows box this week I got random errors, and it turned out to be <a href="https://www.microsoft.com/en-us/msrc/blog/2025/06/redirectionguard-mitigating-unsafe-junction-traversal-in-windows/">RedirectionGuard</a>, a newish mitigation that the builtin OpenSSH server now activates by default. It stops privileged processes following symlinks created by unprivileged users. Unfortunately for some reason my opam install has Flexlink installed via a symlink, which maddeningly lead to obscure toolchain errors <em>only</em> when I was remotely connected and didn't manifest when I was at my keyboard. Argh!</p>
<p>The work in <a href="https://github.com/ucam-eo/geotessera/pull/403">geotessera#403</a> now makes the Zarr stores handle Windows paths consistently. On Linux or macOS I can reproduce a path bug in seconds, but for Windows I have to go through a painful CI loop and also understand a <a href="https://github.com/ocaml-multicore/eio/blob/main/lib_eio/utils/nt_path.ml">very complex path mechanism</a>. I kind of assumed that Python would have good support for abstracting all this, but it feels very similar to OCaml in its difficulty writing portable code...</p>
<p>I also dove into Eio while I was in front of my Windows box, and opened <a href="https://github.com/ocaml-multicore/eio/pull/929">#929</a> to fix anonymous-pipes and then also worked on improving <a href="https://github.com/ocaml-multicore/eio/issues/931">Forester support in #931</a>. Fixing that was a bit of a <a href="https://github.com/ocaml-multicore/eio/pull/932">rabbithole#932</a> but has improved the state of Eio on Windows quite a bit. Just need process and pty support next I think.</p>
<h2><a href="https://anil.recoil.org/news.xml#ai-disclosure-is-more-of-a-social-issue-than-technical" class="anchor" aria-hidden="true"></a>AI disclosure is more of a social issue than technical</h2>
<p>I removed the AI disclosure attributes and opam fields across my monorepo this week, and deleted the <code>ai-disclosure</code> skill from <a href="https://github.com/avsm/ocaml-claude-marketplace">my OCaml Claude marketplace</a>. That reverses what I <a href="https://anil.recoil.org/notes/opam-ai-disclosure">proposed in April</a> and <a href="https://anil.recoil.org/notes/opam-ai-disclosure-update">May</a>.</p>
<p>Disclosure of how people code is a technical answer to a social question. When I wrote the proposal you could still imagine agent-written code as a distinguishable subset of a codebase. But today agents are absolutely everywhere, and the number of models have exploded. Tracking this in detail doesn't seem like a winning strategy.</p>
<p>The <a href="https://github.com/ocaml/opam-repository/tree/master/governance/policies#14-package-contributors-accounts-should-have-a-human-behind-them">opam-repository policy</a> that <a href="https://github.com/dinosaure">Romain Calascibetta</a> started is a much better social solution. I wrote my thoughts on <a href="https://discuss.ocaml.org/t/opam-repository-package-contributors-should-have-a-human-behind-them/18466">OCaml Discuss</a>; we ask only that a human is around to answer our reviewers when they have a question. The opam-repository is where we need to aspire to build community within OCaml, which is only getting harder as automated code generation removes the necessity of collaboration we used to have.</p>
<p>To put this in context, some of the PRs we get to opam-repository are a bit surreal these days. <a href="https://github.com/ocaml/opam-repository/pull/30634">#30634</a> proposed a <code>realdentalcosts</code> library with "stdlib-only OCaml URL helpers", from an organisation that only appeared on GitHub on a few days ago and was spraying packages across several ecosystems at once. <a href="https://bsky.app/profile/giltho.bsky.social/post/3muoop5x6as2m">Sacha Ayoun</a> dubbed this LLM-oriented SEO.
Meanwhile, <a href="https://github.com/ocaml/opam-repository/pull/30643">#30643</a> went the other way with a very pleasant interaction with a <code>vscoqbot</code> bot account for <a href="https://github.com/rocq-prover/vsrocq">rocq-prover/vsrocq</a>. <a href="https://github.com/gares">Enrico Tassi</a> replied within the hour with "sure, tell me what needs to be fixed" and we got the minor issues sorted quickly.</p>
<h2><a href="https://anil.recoil.org/news.xml#a-fourth-recoil-host-at-mythic-beasts" class="anchor" aria-hidden="true"></a>A fourth Recoil host at Mythic Beasts</h2>
<p>I've been provisioning a fourth Recoil host at <a href="https://mythic-beasts.com">Mythic Beasts</a> and putting OpenBSD back on it, so <a href="https://anil.recoil.org/notes/recoil-self-hosting-2026">our routable IPv4 allocation</a> can finally host throwaway services I've been using <a href="https://anil.recoil.org/notes/2026w30">exe.dev</a> for. OpenBSD's <code>vmm</code> suits this well since it brings some sorely needed <a href="https://anil.recoil.org/notes/rewilding-the-web-report">software diversity</a>, and its single-vCPU hypervisor is just fine for this.</p>
<p>Mythic Beasts remain a joy to host with since there's always a real human at the other end who replies (thanks Pete!) quickly. They're plugging a USB stick into the machine so I can netinstall OpenBSD myself, and quickly sorted out routing the custom <code>/28</code> from our block as well.</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun links</h2>
<ul>
<li>There's a cool workshop called <a href="https://terrabytes-workshop.github.io/">TerraBytes</a> on next week (Sep 8th) I'm going to catch some talks online for.</li>
</ul><h1>References</h1><ul><li>Madhavapeddy (2026). A Proposal for Voluntary AI Disclosure in OCaml Code. <a href="https://doi.org/10.59350/cxypn-ysv27" target="_blank"><i>10.59350/cxypn-ysv27</i></a></li>
<li>Madhavapeddy (2026). Tessera v1.1 released, with smoother and temporally stable embeddings. <a href="https://doi.org/10.59350/vcqjp-24y05" target="_blank"><i>10.59350/vcqjp-24y05</i></a></li>
<li>Madhavapeddy (2026). My (very) fast zero-allocation webserver using OxCaml. <a href="https://doi.org/10.59350/9c6bz-kb659" target="_blank"><i>10.59350/9c6bz-kb659</i></a></li>
<li>Madhavapeddy (2026). Self-hosting email the hard way from your own routable IPv4 block up. <a href="https://doi.org/10.59350/gj8re-sca95" target="_blank"><i>10.59350/gj8re-sca95</i></a></li>
<li>Ball et al (2026). Geospatial foundation models enable data-efficient tree species mapping in temperate mountain forests. Elsevier BV. <a href="https://doi.org/10.1016/j.srs.2026.100466" target="_blank"><i>10.1016/j.srs.2026.100466</i></a></li>
<li>Madhavapeddy (2026). Rewilding the Web: my workshop report from Edinburgh. <a href="https://doi.org/10.59350/g40yy-ks003" target="_blank"><i>10.59350/g40yy-ks003</i></a></li>
<li>Madhavapeddy (2025). mlgpx is the first Tangled-hosted package available on opam. <a href="https://doi.org/10.59350/7267y-nj702" target="_blank"><i>10.59350/7267y-nj702</i></a></li></ul>
