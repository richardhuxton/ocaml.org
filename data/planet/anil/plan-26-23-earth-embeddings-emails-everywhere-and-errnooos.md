---
title: '.plan-26-23: Earth Embeddings, Emails Everywhere, and ERRNOOOs'
description: TESSERA on the ESA homepage and at CVPR, GeoTessera 0.9 stabilising onto
  S3/Zarr, io-uring in OCaml, carbon credits in New Scientist and WSJ, and musings
  on internet malware again.
url: https://anil.recoil.org/notes/2026w23
date: 2026-06-07T00:00:00-00:00
preview_image: https://anil.recoil.org/images/tze-explorer-v1.1-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Got some hacking done this week after the <a href="https://anil.recoil.org/notes/hedgehog-tessera-week">media whirlwind</a>, mostly to get <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> 1.1 out of the door and a stabilisation push on <a href="https://github.com/ucam-eo/geotessera">GeoTessera 0.9</a>.
A quick map of the weekly is <a href="https://anil.recoil.org/news.xml#tessera-v11-progress">TESSERA v1.1 progress</a> (including <a href="https://anil.recoil.org/news.xml#geotessera-09">GeoTessera 0.9 onto S3+Zarr</a>); <a href="https://anil.recoil.org/news.xml#io-uring-and-ocaml">io-uring and OCaml</a>; distributed systems covering <a href="https://anil.recoil.org/news.xml#on-distributed-matters">self-hosted email</a>, <a href="https://anil.recoil.org/news.xml#bluesky-and-standardsite">Bluesky/standardsite embeds</a>, <a href="https://anil.recoil.org/news.xml#on-ai-and-individual-choice">Brent Yorgey on AI and individual choice</a>, and <a href="https://anil.recoil.org/news.xml#a-wild-internet-ecology-enters-the-fray">two new Internet-ecology preprints</a>; the kick-off of <a href="https://anil.recoil.org/news.xml#evidence-tap-kicks-off">Evidence TAP</a>; coverage of our <a href="https://anil.recoil.org/news.xml#carbon-credit-assessments-in-the-new-scientist-and-wsj">REDD+ work in New Scientist and the WSJ</a>; book club on <a href="https://anil.recoil.org/news.xml#the-stone-book-quartet">Stone Book Quartet</a>; and <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-v11-progress" class="anchor" aria-hidden="true"></a>TESSERA v1.1 progress</h2>
<p>Hacking continues to get the next iterations of the model and access
libraries out as progress marches on in project <a href="https://anil.recoil.org/projects/tessera">TESSERA</a>! Some highlights of coverage this week were:</p>
<ul>
<li>The <a href="https://anil.recoil.org/papers/2025-tessera">TESSERA v1.0 paper</a> is at <a href="https://anil.recoil.org/notes/cvpr-tessera-esa">CVPR 2026</a> in Denver, with <a href="https://github.com/mahuna13">Jovana Knezevic</a> out there spreading the embeddings gospel!</li>
<li>We were on the front page of the <a href="https://www.esa.int/Applications/Observing_the_Earth/Copernicus/Tessera_AI_model_offers_accessible_way_to_view_Earth">European Space Agency</a>, which was incredibly cool. I grabbed a screenshot of ESA's own HQ from the <a href="https://tze.geotessera.org">TZE Explorer</a> which is the hero image on that piece.</li>
<li><a href="https://phys.org/news/2026-06-tessera-ai-accessible-view-earth.html">phys.org</a> and <a href="https://orbitaltoday.com/2026/06/05/meet-tessera-the-ai-turning-satellite-images-into-earths-fingerprints/">Orbital Today</a> picked it up too.</li>
</ul>
<h3><a href="https://anil.recoil.org/news.xml#geotessera-09" class="anchor" aria-hidden="true"></a>GeoTessera 0.9</h3>
<p>I spent a chunk of the week on getting <a href="https://github.com/ucam-eo/geotessera/pull/278">GeoTessera 0.9 #278</a> stable. This shifts our embedding downloads over to <code>s3://tessera-embeddings/</code> on AWS (moving everything on the download path to be S3-specialised), and adds support for the new <a href="https://huggingface.co/geotessera/TESSERA-V-1.1">TESSERA v1.1 model</a> variant alongside v1.0. The major improvements in the model are better year-on-year temporal embeddings, and higher quality on tiles with fewer observational passes. 1.1 is looking like a pretty awesome iteration on the base model so far.</p>
<p>Thanks to <a href="https://www.tunbury.org/">Mark Elvers</a> and <a href="https://aneeshnaik.github.io/">Aneesh Naik</a> for <a href="https://aneeshnaik.github.io/blogposts/20260605_weeknotes_2026_23.html">help debugging</a> the performance issues that came out of the shift to Zarr. The issues arise from a lack of concurrency in retrieving HTTP chunks (<a href="https://github.com/zarr-developers/zarr-python/pull/3004">zarr-python#3004</a>) and an accidentally <a href="https://github.com/ucam-eo/geotessera/pull/281">uncompressed coordinate array</a>.</p>
<p>I'm a little surprised that AWS is so much slower in HTTP latency than our University server, but I think we don't have edge caching enabled (CloudFront). I'm hoping to push out a release in the next few days so that we can debug performance against that rather than blocking all the users trying to get their <a href="https://github.com/ucam-eo/geotessera/issues?q=is:issue%20label:embedding-request">mittens on 1.1 embeddings</a>.</p>
<p>I did also deploy a quick <a href="https://tze.geotessera.org/?store=v1.1">v1.1 update to TZE</a> so we can browse the embeddings that we do have, and it results in some very pretty visuals.</p>
<p><a href="https://tze.geotessera.org/?store=v1.1"> <img src="https://anil.recoil.org/images/tze-explorer-v1.1-ss-1.webp" alt="%c"> </a></p>
<h3><a href="https://anil.recoil.org/news.xml#io-uring-and-ocaml" class="anchor" aria-hidden="true"></a>io-uring and OCaml</h3>
<p>Since <a href="https://roscidus.com">Thomas Leonard</a> is back and <a href="https://notes.roscidus.com/2026/06/01/">hacking on Eio</a> I've been doing a refresh of the <a href="https://github.com/ocaml-multicore/ocaml-uring">ocaml-uring</a> libraries. I need to do a lot of Zarr data copying in the coming weeks and the obvious way to do this is via extremely zero-copy OxCaml code. There's nothing that makes the systems hacker in me happier than having an excuse to write some high performance OCaml code!</p>
<p>Aside from fixing a bunch of bugs, I've added <a href="https://github.com/ocaml-multicore/ocaml-uring/pull/147">shutdown, socket, renameat and symlinkat</a>, and <a href="https://github.com/ocaml-multicore/ocaml-uring/pull/149">fallocate, fsync and ftruncate</a>, and exposed <a href="https://github.com/ocaml-multicore/ocaml-uring/pull/152">Linux-specific errnos</a> with hilarious names like <a href="https://amok.recoil.org/@avsm/116698423254334531">EOWNERDEAD, ENOTRECOVERABLE, ERFKILL, EHWPOISON</a>.</p>
<p>I've got another separate tree with OxCaml specific bindings (mostly using
<code>caml_alloc_local</code> to go full stack alloc) for my <a href="https://anil.recoil.org/notes/oxcaml-httpz">zero-alloc httpz server</a> to use, but more on that after I make the server sweat a
little more. <a href="https://www.tunbury.org/">Mark Elvers</a> has also been <a href="https://www.tunbury.org/2026/06/08/ubuntu-26-04/">upgrading our CI system</a> to Linux 7.0 so we can test these features more easily, as well as valiantly <a href="https://www.tunbury.org/2026/06/03/emulated-riscv-workers/">trying to keep RISC-V alive</a> on our nodes despite that architecture's best attempts to become irrelevant through hardware scarcity.</p>
<h2><a href="https://anil.recoil.org/news.xml#on-distributed-matters" class="anchor" aria-hidden="true"></a>On distributed matters</h2>
<p>I put up a <a href="https://anil.recoil.org/notes/recoil-self-hosting-2026">detailed post on self-hosting email</a> that's attracting <a href="https://lobste.rs/s/cw7vxa/self_hosting_email_hard_way_from_your_own">lively discussion</a> on the Interwebs. This came out of the <a href="https://anil.recoil.org/notes/rewilding-the-web-report">Rewilding the Web</a> workshop I went to last week. I'm going to do a series of blogs on various self-hosting matters over the coming months, as there's clearly appetite for people who want to get control of their own data again.</p>
<h3><a href="https://anil.recoil.org/news.xml#bluesky-and-standardsite" class="anchor" aria-hidden="true"></a>Bluesky and standardsite</h3>
<p>Aside from email, another important distributed system I use is the 'social database' that underpins Bluesky. One of the promises of this underlying database is that multiple services built over it can interoperate. <a href="https://anil.recoil.org/notes/disentangling-git-with-bluesky">Tangled Git hosting</a> is one such service, but what else is there?</p>
<p><img src="https://anil.recoil.org/images/bsky-standardsite-1.webp" alt="%rc" title="Notice the 'View publication' button below the post that represents the embed">
Well, this week we saw Bluesky <a href="https://atproto.com/blog/standard-site-bluesky-timeline">ship a new feature</a> that makes publishing to <a href="https://standard.site">standardsite</a> records much more worthwhile: the Bluesky client now renders long-form Standard.site document embeds nicely inline. Seeing one of my own <code>standardsite</code> documents show up as a proper rich embed in someone else's feed is exactly the kind of small-pieces-loosely-joined interop I keep hoping the network will grow into.</p>
<blockquote>
<p>Standard.site defines a few Lexicons for publishing websites—such as a
publication (like a website or blog), a document (like an article or post),
and one for subscriptions (for tracking which publications to follow). Taken
together, these describe longform writing in an atmospheric way, similar to
how the Bluesky lexicons describe a social network.
<cite>-- <a href="https://atproto.com/blog/standard-site-bluesky-timeline">What the Standard.site lexicons do</a>, June 2026</cite></p>
</blockquote>
<p>Using my <a href="https://tangled.org/anil.recoil.org/ocaml-atp">OCaml ATproto</a> library, I can automatically add records into the ATProto public relay, and they'll get picked up by Bluesky. For example, this site's record is <a href="https://pdsls.dev/at://did:plc:nhyitepp3u4u6fcfboegzcjw/site.standard.publication/3mchoxkwlsx2y">browsable here</a>:</p>
<p><a href="https://pdsls.dev/at://did:plc:nhyitepp3u4u6fcfboegzcjw/site.standard.publication/3mchoxkwlsx2y"> <img src="https://anil.recoil.org/images/pdsls-dev-1.webp" alt="%c"> </a></p>
<h3><a href="https://anil.recoil.org/news.xml#on-ai-and-individual-choice" class="anchor" aria-hidden="true"></a>On AI and individual choice</h3>
<p>There's an excellent essay by Brent Yorgey called "<a href="http://ozark.hendrix.edu/~yorgey/forest/00FD/"><em>To my students</em></a>" on how to carry yourself through a software industry that's being entirely disrupted by AI at the moment. It's written as advice to his compsci students, and it's helped me form up my thoughts around <a href="https://anil.recoil.org/notes/opam-ai-disclosure">AI disclosure proposal</a>:</p>
<blockquote>
<p>Don't believe self-serving lies about technologies being "inevitable" or
"here to stay". You don't have to just go along with the dominant narrative.
You can make deliberate choices and help others to do the same.
<cite>-- <a href="http://ozark.hendrix.edu/~yorgey/forest/00FD/">Brent Yorgey, <em>"To my students"</em></a>, 2026</cite></p>
</blockquote>
<p>He concludes with "be motivated by love instead of fear", which is a bloody brilliant way to think about the current choices we all face. My view on self hosting and open source is that I want to preserve the right for future generations to have their own agency about how digital technologies will guide their lives. I'm entirely for the use of AI <em>if all parties involved are informed and consenting</em>, which is what motivated my <a href="https://anil.recoil.org/notes/opam-ai-disclosure-update">disclosure proposal</a> the other week.</p>
<p>It's also great to see another OCaml-powered <a href="https://www.forester-notes.org/index/index.xml">Forester</a> blog spring up in the wild. Well done on growing a userbase with such excellently thoughtful content, <a href="https://jonmsterling.com">Jon Sterling</a>!</p>
<h3><a href="https://anil.recoil.org/news.xml#a-wild-internet-ecology-enters-the-fray" class="anchor" aria-hidden="true"></a>A wild Internet ecology enters the fray</h3>
<p>A couple of preprints popped up this week that are relevant to the <a href="https://anil.recoil.org/papers/2025-internet-ecology">internet ecology</a> work I've been seeding on <a href="https://anil.recoil.org/notes/ecology-at-aarhus">antibotty networks</a> and self-modifying code:</p>
<ul>
<li><a href="https://arxiv.org/abs/2606.03811">AI Agents Enable Adaptive Computer Worms</a> from Nicolas Papernot's lab (with Cambridge's own Hanna Foerster among the authors) demonstrates a worm that reasons about each target and decides how to attack it using an open-weight LLM model. Like Morris' original worm, it parasitically runs the LLM on the machines it compromises.  It was covered in the <a href="https://www.nytimes.com/2026/06/02/technology/scientists-find-way-to-supercharge-dangerous-computer-worms-with-ai.html">New York Times</a> as well as <a href="https://www.heise.de/en/news/IT-researchers-demonstrate-adaptive-AI-worm-11318259.html">heise</a>.</li>
<li><a href="https://arxiv.org/abs/2605.11086">ExploitGym</a> is a benchmark of ~900 real-world vulnerability instances for measuring whether AI agents can turn theoretical vulnerabilities into working exploits. They showed, somewhat worryingly, that the three major frontier models all found <em>different</em> sets of exploits in the same code. There's a lot of latent bugginess, which hopefully can be turned into <a href="https://doi.org/10.1098/rspb.2025.2377">cryptic resilience</a> rather than pure vulnerability!</li>
</ul>
<h2><a href="https://anil.recoil.org/news.xml#evidence-tap-kicks-off" class="anchor" aria-hidden="true"></a>Evidence TAP kicks off</h2>
<p>After <a href="https://anil.recoil.org/notes/2026w7">playing with LEGO</a> earlier in the year, we had the first meeting of our new Evidence TAP project, which is a broadening of <a href="https://anil.recoil.org/projects/ce">Conservation Evidence</a> into new fields such as education. On the education side, we had <a href="https://www.educ.cam.ac.uk/people/staff/gibson/">Jenny Gibson</a> and Mélanie Gréaux who has just <a href="https://www.languagesciences.cam.ac.uk/news/researcher-profile-melanie-greaux">returned after her PhD</a> to come back and work on education evidence synthesis fulltime with us! <a href="https://www.linkedin.com/feed/update/urn:li:activity:7465687486401265664/">Welcome Mel, back to Cambridge!</a>:</p>
<blockquote>
<p>In Cambridge, I am joining a fantastic multidisciplinary team working on an
ambitious project to build an AI-facilitated evidence platform helping
policymakers and practitioners to find evidence-based answers to their
questions: what works, for whom, and in which contexts? In the age of AI, we
are presented with an opportunity to re-think our approach and access to
evidence systems. If done well, it can accelerate the democratisation of
knowledge and strengthen our ethical engagement with data. I’ll be leading
work on evidence systems for early childhood education.
<cite>-- <a href="https://www.linkedin.com/feed/update/urn:li:activity:7465687486401265664/">Mélanie Gréaux</a>, LinkedIn, 2026</cite></p>
</blockquote>
<p>It was very cool to traipse over to the <a href="https://commons.wikimedia.org/wiki/File:Donald_Mcintyre_Building,_Faculty_of_Education,_Cambridge.jpg">Donald Mcintyre Building</a> beside Homerton College with <a href="https://www.cst.cam.ac.uk/people/eft20">Eleanor Toye Scott</a>, <a href="https://toao.com">Sadiq Jaffer</a>, <a href="https://www.zoo.cam.ac.uk/directory/prof-lynn-dicks">Lynn Dicks</a>, Rob Doubleday and <a href="https://www.zoo.cam.ac.uk/directory/bill-sutherland">Bill Sutherland</a> and see the inside of a building I hadn't seen before after many years in Cambridge!</p>
<p>More on the Evidence TAP in the coming weeks; I need to knock up a website for the project next week to give us a blogging area!  Having three departments involved means that we need more coordination than usual, but our <a href="https://eeg.zulipchat.com">Zulip</a> is serving well so far.</p>
<p><img src="https://anil.recoil.org/images/evitap-education-1.webp" alt="%c" title="In the Donald Mcintyre building for the first time (for me)!"></p>
<h2><a href="https://anil.recoil.org/news.xml#carbon-credit-assessments-in-the-new-scientist-and-wsj" class="anchor" aria-hidden="true"></a>Carbon credit assessments in the New Scientist and WSJ</h2>
<p><a href="https://www.newscientist.com/article/2525921-carbon-credits-are-flawed-but-they-can-still-help-save-forests/"> <img src="https://anil.recoil.org/images/new-scientist-cc-ss-1.webp" alt="%rc"> </a>
I missed this a few weeks ago, but it turns out we had a couple of articles covering our research <a href="https://anil.recoil.org/notes/redd-overcrediting">on REDD+ overcrediting</a> in both New Scientist and the Wall Street Journal! The paper involved was "<a href="https://anil.recoil.org/papers/2025-redd-evals">Learning lessons from over-crediting to ensure additionality in forest carbon credits</a>".</p>
<p>First, New Scientist had a nice piece on "<a href="https://www.newscientist.com/article/2525921-carbon-credits-are-flawed-but-they-can-still-help-save-forests/">Carbon credits are flawed, but they can still help save forests</a>" (<a href="https://archive.is/TfRRu">archive.is mirror</a>):</p>
<blockquote>
<p>Carbon credits bought by companies to offset their emissions really have
reduced deforestation, but not by as much as credit developers claim, according
to a rigorous analysis.</p>
<p>[...] So who's right? Both, according to a growing body of research. Last
month, one of the most rigorous studies yet found that most early projects
did successfully reduce deforestation. But they sold credits for almost 11
times more forest on average than they actually saved.
<cite>-- <a href="https://www.newscientist.com/article/2525921-carbon-credits-are-flawed-but-they-can-still-help-save-forests/">New Scientist</a></cite></p>
</blockquote>
<p>Then the Wall Street Journal also had a piece on "<a href="https://www.wsj.com/pro/sustainable-business/the-way-companies-aim-for-net-zero-is-flawed-its-also-working-07099d0e">The Way Companies Aim for Net Zero Is Flawed. It's Also Working</a>" (<a href="https://archive.is/stF0V">archive.is mirror</a>):</p>
<blockquote>
<p>Two academic papers published last month show corporate climate efforts
having a positive impact on reducing deforestation and cutting emissions.</p>
<p>[...] "What is really positive about carbon credits is that despite being a
complicated economic instrument for achieving these outcomes, they do at
least make it possible for us to directly fund efforts that reduce
deforestation on the ground," said Tom Swinfield, one of the report's
authors. "They allow you to circumvent all the complex politics and get to
the heart of the problem."
<cite><a href="https://www.wsj.com/pro/sustainable-business/the-way-companies-aim-for-net-zero-is-flawed-its-also-working-07099d0e">The Way Companies Aim for Net Zero Is Flawed. It’s Also Working</a>, WSJ, May 2026</cite></p>
</blockquote>
<h2><a href="https://anil.recoil.org/news.xml#the-stone-book-quartet" class="anchor" aria-hidden="true"></a>The Stone Book Quartet</h2>
<p>In our Christ's book club organised by <a href="https://www.educ.cam.ac.uk/people/staff/gibson/">Jenny Gibson</a>, we had <a href="https://github.com/mor1">Richard Mortier</a> recommend Alan Garner's <a href="https://en.wikipedia.org/wiki/The_Stone_Book_Quartet">Stone Book Quartet</a>. I did struggle with the Cheshire dialect in his prose, but absolutely loved the opening of the first "The Stone Book", where a child climbs to the top of a steeple and is then taken deep underground to be shown a cave painting in an opening so narrow only a child can crawl through.</p>
<p>I always get the bug to go and visit the places in the books I've read, like trekking out to Dartmouth after finishing Julian May's <a href="https://en.wikipedia.org/wiki/Galactic_Milieu">Galactic Milieu saga</a>, and after this one I very much want to go and walk <a href="https://en.wikipedia.org/wiki/Alderley_Edge">Alderley Edge</a>, the bit of Cheshire that runs all the way through Garner's writing and that has had human settlements for a very loong time.</p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun Links</h2>
<ul>
<li><a href="https://github.com/justincormack">Justin Cormack</a> recommends <a href="https://press.princeton.edu/books/hardcover/9780691272443/the-irrational-decision"><em>The Irrational Decision: How We Gave Computers the Power to Choose for Us</em></a> by Ben Recht, on how a 1940s statistical definition of rationality became the dominant framework for decision-making everywhere.</li>
<li>Lovely little library from Aneesh Naik that I want to use called <a href="https://github.com/aneeshnaik/datahues">datahues</a> which generates perceptually-uniform colour ramps by interpolating in <a href="https://bottosson.github.io/posts/oklab/">Oklab</a> space, so you don't see banding patterns in your data that aren't really there. Seems perfect for</li>
</ul><h1>References</h1><ul><li>Madhavapeddy (2026). A Proposal for Voluntary AI Disclosure in OCaml Code. <a href="https://doi.org/10.59350/cxypn-ysv27" target="_blank"><i>10.59350/cxypn-ysv27</i></a></li>
<li>Madhavapeddy et al (2025). Steps towards an Ecology for the Internet. Association for Computing Machinery. <a href="https://doi.org/10.1145/3744169.3744180" target="_blank"><i>10.1145/3744169.3744180</i></a></li>
<li>Feng et al (2026). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li>
<li>Madhavapeddy (2026). My (very) fast zero-allocation webserver using OxCaml. <a href="https://doi.org/10.59350/9c6bz-kb659" target="_blank"><i>10.59350/9c6bz-kb659</i></a></li>
<li>Madhavapeddy (2026). Self-hosting email the hard way from your own routable IPv4 block up. <a href="https://doi.org/10.59350/gj8re-sca95" target="_blank"><i>10.59350/gj8re-sca95</i></a></li>
<li>Madhavapeddy (2025). Presenting our Ecology of the Internet ideas at Aarhus 2025. <a href="https://doi.org/10.59350/p45b8-kvt85" target="_blank"><i>10.59350/p45b8-kvt85</i></a></li>
<li>Swinfield et al (2026). Learning lessons from over-crediting to ensure additionality in forest carbon credits. Nature Publishing Group. <a href="https://doi.org/10.1038/s41467-026-71552-3" target="_blank"><i>10.1038/s41467-026-71552-3</i></a></li>
<li>Madhavapeddy (2025). Socially self-hosting source code with Tangled on Bluesky. <a href="https://doi.org/10.59350/r80vb-7b441" target="_blank"><i>10.59350/r80vb-7b441</i></a></li>
<li>Madhavapeddy (2026). Rewilding the Web: my workshop report from Edinburgh. <a href="https://doi.org/10.59350/g40yy-ks003" target="_blank"><i>10.59350/g40yy-ks003</i></a></li>
<li>Zhang et al (2026). Cryptic species are widespread across vertebrates. <a href="https://doi.org/10.1098/rspb.2025.2377" target="_blank"><i>10.1098/rspb.2025.2377</i></a></li>
<li>Guan et al (2026). AI Agents Enable Adaptive Computer Worms. arXiv. <a href="https://doi.org/10.48550/arXiv.2606.03811" target="_blank"><i>10.48550/arXiv.2606.03811</i></a></li>
<li>Wang et al (2026). ExploitGym: Can AI Agents Turn Security Vulnerabilities into Real Attacks?. arXiv. <a href="https://doi.org/10.48550/arXiv.2605.11086" target="_blank"><i>10.48550/arXiv.2605.11086</i></a></li></ul>
