---
title: '.plan-26-28: What fun papers piled up while I was out at sea'
description: Back from the Arctic into a heatwave, hacking on Eio for the TESSERA
  sync engine, the Conservation Evidence team demoing at Parliament, and TESSERA on
  stage at the RAISE Summit in Paris.
url: https://anil.recoil.org/notes/2026w28
date: 2026-07-12T00:00:00-00:00
preview_image: https://anil.recoil.org/images/natgeo-endurance-port.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I reluctantly disembarked from the NatGeo Endurance on Thursday and returned from the <a href="https://anil.recoil.org/notes/2026w27">Arctic</a> to find the UK in the grip of a <a href="https://www.bbc.co.uk/weather/articles/c0jyx16ww8jo">proper heatwave</a>! The train home almost undid me with the temperature transition of ~40C; in the end I think the London-Cambridge train took more time than the Iceland to London hop. Still, always nice to be home, if only for a week before my next trip!</p>
<p><img src="https://anil.recoil.org/images/natgeo-endurance-port.webp" alt="%c"></p>
<p>A pile of papers landed while I was at sea: the <a href="https://anil.recoil.org/news.xml#tessera-v2-checkpoint-paper">TESSERA v2 checkpoint</a> along with a Trentino tree species study, and two <a href="https://anil.recoil.org/news.xml#over-to-greek-with-programming-language-papers">programming language papers</a> on type slicing and package calculus. I also managed during the travels to get some <a href="https://anil.recoil.org/news.xml#hacking-on-eio-for-the-tessera-syncher">Eio hacking</a> done that I need for our <a href="https://anil.recoil.org/projects/tessera">TESSERA</a> embeddings sync engine. While I was away gallivanting around the world, my hardworking colleagues presented <a href="https://anil.recoil.org/news.xml#conservation-evidence-takes-the-stage-at-parliament">Conservation Evidence in Parliament</a>, and also <a href="https://anil.recoil.org/news.xml#tessera-in-paris">showed off TESSERA in Paris</a> at the <a href="https://www.raisesummit.com/">RAISE Summit</a>. There are the usual <a href="https://anil.recoil.org/news.xml#fun-links">fun links</a> at the end!</p>
<p>I didn't entirely escape admin work on the expedition either, since I had reviews to
finish for EuroSys and for Remote Sensing of Environment.</p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-v2-checkpoint-paper" class="anchor" aria-hidden="true"></a>TESSERA v2 checkpoint paper</h2>
<p>On the geospatial side, <a href="https://www.cst.cam.ac.uk/people/zf281">Frank Feng</a> lead an exciting progress update on <a href="https://anil.recoil.org/papers/2026-tessera-v2">TESSERA v2</a>. This was a large scaling <a href="https://x.com/avsm/status/2074547485620527365">effort</a> along with <a href="https://toao.com">Sadiq Jaffer</a>, <a href="https://coomeslab.org">David Coomes</a>, <a href="https://svr-sk818-web.cl.cam.ac.uk/keshav/wiki/index.php/Main_Page">Srinivasan Keshav</a> and many others, and we used a big allocation of UKRI AIRR GPUs for the hero run which we got <a href="https://www.ukri.org/opportunity/airr-compute-opportunity-ai-for-science/">awarded back in January</a> on <a href="https://isambard.ac.uk">Isambard 2</a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#whats-new-in-tessera-v2" class="anchor" aria-hidden="true"></a>What's new in TESSERA v2</h3>
<p>There are some key innovations that distinguish this new model from <a href="https://anil.recoil.org/papers/2025-tessera">v1.0</a> or <a href="https://anil.recoil.org/notes/tessera-v11-out">v1.1</a>. The first is the sheer <em>scale</em> of the training, as we have a huge amount of available training data from the satellites that we didn't have capacity to add. In the paper, we describe the ablations we did to find the bottlenecks in the model architecture via hundreds of sweeps. A simple rule of thumb that emerged from the sweeps was that as the training budget grows, the model's encoder and satellite data should grow together while the projector stays fixed, which gave us a simple rule for allocating compute.</p>
<p>A few 100,000 hours of GPU time later, this resulted in a lovely new 1-billion parameter model (with a 2-billion one still in training). This many parameters improved accuracy quite a bit, but also would slow down inference considerably.  We don't have the cash to rack up costs when we need to generate new map embeddings for 1.8 million tiles for the whole planet!</p>
<p>To address this, the second innovation is to use a teacher/student distillation and generate a family of smaller models that are used for inference. These student models are dubbed TESSERA-v2-1B-L or TESSERA-v2-1B-M depending on the number of parameters pre- and post-distillation. Remarkably, the new TESSERA-v2-1B-M model has <em>fewer</em> parameters than the original TESSERA v1 and outperforms it (and every other GeoFM model) we could find.</p>
<p><img src="https://anil.recoil.org/images/tessera2-b1.webp" alt="%c" title="Team TESSERA breathes a sigh of relief after the giant hero run shows stellar performance"></p>
<p>Tessera v2 also features <a href="https://arxiv.org/abs/2205.13147">Matryoshka embeddings</a>, a training method that imposes an ordering on the dimensions so there's more information stashed in the earlier ones than the later ones (recall that there are a total of 128 dimensions per 10m2 map tile). This means that just 1/8th of the bands give 90% of the performance in many downstream tasks.</p>
<p>Reducing the amount of data involved in manipulating the embeddings is really important as we work towards much smaller datasets with the <a href="https://aria.org.uk/insights/doubling-down-on-earth-scale-data">Earth Compress team</a> (who I <a href="https://anil.recoil.org/notes/cng-london-2026">heard from at CNG</a> in the spring) and others. It would be amazing to run the TESSERA on a normal feature phone before the end of the year! With v2, all you have to do is to chop the 128 dimensions off earlier and you have everything you need. The only hitch is that I'm not sure how to encode this in Zarr efficiently, which is the <a href="https://anil.recoil.org/notes/tessera-zarr-v3-layout">dimension slicing problem</a> I flagged back when we sharded the v1 embeddings and <a href="https://anil.recoil.org/notes/tessera-embeddings-convention">threw a clanker into the conventions</a>. That's something to figure out later this summer with <a href="https://icechunk.io">Icechunk</a> and friends, having heard <a href="https://anil.recoil.org/notes/2026w25">Deepak Cherian talk about it</a> last month!</p>
<p><img src="https://anil.recoil.org/images/tessera2-b2.webp" alt="%c" title="The Matryoshka embeddings show the increase in accuracy as we add in more dimensions to the tasks"></p>
<p>The TESSERA-1B parameter model is finished and the 2B model is still training, so there is much more exciting news to come this summer! We're really thankful to the Nvidia team (esp. Pembroke alumni <a href="https://ira-shokar.github.io/">Ira Shokar</a> and the irrepressible <a href="https://developer.nvidia.com/blog/author/niallrobinson/">Niall Robinson</a>, who I <a href="https://anil.recoil.org/notes/cng-london-2026">caught up with in London</a> about exactly this) for their help optimising our kernels before deploying onto Isambard 2.</p>
<h3><a href="https://anil.recoil.org/news.xml#mapping-trees-in-trentino" class="anchor" aria-hidden="true"></a>Mapping trees in Trentino</h3>
<p>Trentino, aside from being home to the most <a href="https://www.formaggideltrentino.it/en/trentingrana">amazing cheese</a>, is also home to a variety of <a href="https://www.visittrentino.info/en/articles/practical-info/flora-of-Trentino">tree biodiversity</a>. We used TESSERA to try to spot the different plant species from space! This turned into a really interesting analysis of how to perform a more complex downstream task than the <a href="https://anil.recoil.org/papers/2025-tessera-tasks">environmental mapping benchmarks</a> we published last year:</p>
<blockquote>
<p>Here, we evaluate two geospatial foundation-model embeddings, AlphaEarth and
Tessera1, for tree species classification in the Trentino region of northern
Italy, using parcel-level forest inventories as reference data (18 species
and species groups). We compare their performance against conventional
Sentinel-1+2 satellite composites across a series of controlled experiments
examining classification accuracy, label efficiency, classifier complexity,
robustness to label impurity, and temporal transferability</p>
<p>Foundation-model embeddings consistently outperform composite-based
multispectral satellite baselines [...], reaching near-asymptotic accuracy
with as few as 5% of available training parcels and preserving ecologically
meaningful structure aligned with functional and taxonomic groupings.</p>
<p>However, realising this advantage requires a nonlinear classifier: a compact
neural network provides better results than classic machine learning (i.e.
Random Forest) and performs as well as deeper neural networks, while a linear
classifier on foundation-model embeddings underperforms a neural network on
conventional composites.
<cite>-- <a href="https://doi.org/10.1016/j.srs.2026.100466">Geospatial foundation models enable data-efficient tree species mapping in temperate mountain forests, Ball et al 2026</a></cite></p>
</blockquote>
<p>The paper is now <a href="https://anil.recoil.org/papers/2026-tessera-trentino">published</a> in the journal <a href="https://www.sciencedirect.com/science/article/pii/S2666017226001045">Science of Remote Sensing</a>, led by <a href="https://www.linkedin.com/posts/james-ball-b406a15a_really-pleased-to-share-that-our-paper-is-share-7481937380652904448-J6Q3/">James Ball</a>. It shows that geospatial foundation models can map tree species in temperate mountain forests with far fewer manual labels than previously required.</p>
<p>This is possible because the pretraining process for TESSERA eliminates a lot of redundancy that's present when manually processing satellite data. Note that the results in this paper weren't with the TESSERAv2 I mentioned above, so it'll be exciting to rerun this analysis when we finalise the 2B model.</p>
<p><a href="https://doi.org/10.1016/j.srs.2026.100466"> <img src="https://anil.recoil.org/papers/2026-tessera-trentino" alt="%c"> </a></p>
<p>There was also a new version of our <a href="https://anil.recoil.org/papers/2024-hyper-tropical-mapping">tropical tree mapping paper</a> uploaded (also lead by James), which studies when imaging spectroscopy can and cannot separate species in the tropics. We discovered that <a href="https://en.wikipedia.org/wiki/Phenology">phenological</a> regularity (the strength and consistency of seasonal leaf-cycling) was the single strongest predictor of separability across species.</p>
<p>This second paper doesn't use geospatial foundation models yet as we started it before TESSERA even existed in 2024, and instead uses hyperspectral sensors that fly lower. I'm hopeful that in the future we can train a new version of TESSERA that incorporates <a href="https://www.enmap.org/">hyperspectral sensors like ENMAP</a>, but that depends on data and compute availability.</p>
<h2><a href="https://anil.recoil.org/news.xml#over-to-greek-with-programming-language-papers" class="anchor" aria-hidden="true"></a>Over to Greek with programming language papers</h2>
<p>We now shift from the glorious Italian alps over to more esoteric lambdas for some programming language research next...</p>
<h3><a href="https://anil.recoil.org/news.xml#bidirectional-type-slicing" class="anchor" aria-hidden="true"></a>Bidirectional type slicing</h3>
<p><a href="https://maxcarroll0.github.io/blog/">Max Carroll</a> worked with me for his Part II project last year in <a href="https://anil.recoil.org/ideas/gradual-type-error-debugging">type level debugging</a>. He also recently completed his <a href="https://anil.recoil.org/ideas/bidirectional-type-slicing">Part III project</a> supervised by me and <a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a>, and won the Computer Lab year prize for best part III dissertation!  Max worked on the topic of <a href="https://anil.recoil.org/papers/2026-bidirectional-type-slicing">bidirectional type slicing</a>:</p>
<blockquote>
<p>Development tools report what type an expression has, but not why it has that
type. This paper develops a theory of type slicing: a programmer selects a
term, queries any part of its type information, and receives a program slice
that is sufficient to reproduce the queried type. We formulate type slicing
for bidirectional type systems, where synthesis slices explain the type a
term synthesises and analysis slices explain the type expected by its
surrounding context.
<cite>-- <a href="https://arxiv.org/abs/2607.12197">Bidirectional Type Slicing, Carroll et al, 2026</a></cite></p>
</blockquote>
<p>A huge well done to Max on the hard work that went into this year's project, and for the well deserved award. You can read his <a href="https://doi.org/10.48456/tr-1006">Part II dissertation</a> and <a href="https://doi.org/10.48456/tr-1007">Part III dissertation</a> in our technical library, or the more digestible <a href="https://arxiv.org/abs/2607.12197">arXiv version</a>.</p>
<p>This is extremely cool theory research for any future language we develop for the <a href="https://anil.recoil.org/papers/2025-fairground">planetary wiki</a> because it forms the basis for how a compiler can help a human or coding agent understand <em>why</em> a program is being rejected with a minimal slice.
Max will be around in Cambridge over the summer continuing to work on the project, including building a Hazel implementation. Go find him if you're interested in this topic!</p>
<p><img src="https://anil.recoil.org/images/max-type-hacking-1.webp" alt="%c" title="Max last year talking about his Part II project in my office."></p>
<h3><a href="https://anil.recoil.org/news.xml#package-calculus" class="anchor" aria-hidden="true"></a>Package calculus</h3>
<p>The ICFP 2026 camera ready version of our <a href="https://anil.recoil.org/papers/2026-package-calculus">package calculus</a> work, lead by <a href="https://ryan.freumh.org">Ryan Gibb</a>, is also done and
<a href="https://arxiv.org/abs/2602.18602">uploaded</a> after many, many iterations. There's been a large number of
typographical improvements after a long session with <a href="https://simon.peytonjones.org/">Simon Peyton Jones</a>. The artefact evaluation process also went without a hitch, with all the reviewers able to build the Lean proofs fine.</p>
<p>Ryan's now hacking away on an implementation of the calculus so we can perform
bidirectional translation across package managers, so the next step is more
systemsy.  We also keep <a href="https://nesbitt.io/2026/07/04/this-week-in-package-management.html">running into</a> new package managers that require
translation; it's incredible how many there are out in the wild these days!</p>
<h2><a href="https://anil.recoil.org/news.xml#hacking-on-eio-for-the-tessera-syncher" class="anchor" aria-hidden="true"></a>Hacking on Eio for the TESSERA syncher</h2>
<p>I continued the <a href="https://anil.recoil.org/notes/2026w26">Eio hacking</a> that I started before the Arctic trip while on the ocean. The <a href="https://www.nationalgeographic.com/expeditions/ships/national-geographic-endurance/">NatGeo Endurance</a> had a satellite connection that was sporadically active depending on the weather conditions!</p>
<p><img src="https://anil.recoil.org/images/natgeo-endurance-control.webp" alt="%c" title="The Endurance ship had an engine room with more radar and satellite downlinks than I could count!"></p>
<p>The tool I'm working on has to move a great many terabytes of embeddings around quickly, so it exercises many corners of the Linux IO stack. It's surprisingly hard to juggle all the various bottlenecks when copying files (direct IO, dealing with partially implemented <a href="https://github.com/s3fs-fuse/s3fs-fuse">FUSE S3</a> semantics, and so on), so this has been a useful exercise for me to get up to speed on all this again.</p>
<p>On the filesystem side, I switched Eio's file stat over to <a href="https://github.com/ocaml-multicore/eio/pull/865">statx and populating the block size</a> so we can size reads, and bumped our <a href="https://github.com/ocaml-multicore/eio/pull/867">minimum liburing to 2.15.0</a> to get at some newer functionality. I also helped to fix up the <a href="https://github.com/ocaml-multicore/eio/pull/872">chmod sandbox symlink support</a> and corrected the <a href="https://github.com/ocaml-multicore/eio/pull/864">flags we pass to chown</a>; dealing with sandboxing symlinks is really very complicated!</p>
<p>On the networking side, I helped clean up <a href="https://github.com/ocaml-multicore/eio/pull/873">truncation checks in getsockopt</a> and <a href="https://github.com/ocaml-multicore/eio/pull/874">portable sockopt errors</a> while figuring out how Windows works. There were a couple of lower level correctness fixes;  one <a href="https://github.com/ocaml-multicore/eio/pull/870">retries on EINTR when writing to a pipe</a> in the fork machinery, and another makes sure all <a href="https://github.com/ocaml-multicore/eio/pull/871">read and write functions hold the file descriptor open</a> while they're in use.</p>
<p>Finally, I tidied up <a href="https://github.com/ocaml-multicore/eio/pull/881">Windows build</a> as best as I could remotely, with some unixsupport declarations thanks to a pointer from <a href="https://www.dra27.uk">David Allsopp</a>. It turns out that <code>CAMLextern</code> does matter for Windows since it controls DLL visibility. In OCaml, the <code>Unix</code> module is <em>also</em> a compatibility shim that runs on Windows, which means that the various <code>eio.unix</code> modules do in fact need to link on Windows. If this doesn't make any sense to you, then I recommend you just move on :-)
Thanks as always to <a href="https://roscidus.com">Thomas Leonard</a> for the <a href="https://notes.roscidus.com/2026/07/10/">careful reviews</a>!</p>
<h2><a href="https://anil.recoil.org/news.xml#conservation-evidence-takes-the-stage-at-parliament" class="anchor" aria-hidden="true"></a>Conservation Evidence takes the stage at Parliament</h2>
<p>While I was somewhere off the coast of north-eastern Greenland, the <a href="https://anil.recoil.org/projects/ce">Conservation Evidence</a> crew took our work to the Houses of Parliament. <a href="https://profiles.imperial.ac.uk/a.christie">Alec Christie</a>, <a href="https://samreynolds.org">Sam Reynolds</a> and <a href="https://toao.com">Sadiq Jaffer</a>, along with <a href="https://www.zoo.cam.ac.uk/directory/bill-sutherland">Bill Sutherland</a>, demoed our <a href="https://anil.recoil.org/papers/2025-evidence-tap">Evidence TAP</a> traceable AI pipeline at the <a href="https://www.linkedin.com/feed/update/urn:li:activity:7480249165898436611/">Georgina Mace Centre for the Living Planet debate</a> sponsored by <a href="https://www.jackrankin.org.uk/news/jack-rankin-mp-sponsors-georgina-mace-centre-living-planet-debate">Jack Rankin MP</a>.</p>
<p><img src="https://anil.recoil.org/images/hol-nature-1.webp" alt="%c" title="Team CE presenting at the Houses of Parliament (image credit: Alec Christie)"></p>
<p>The evening focussed on how <a href="https://www.imperial.ac.uk/news/articles/natural-sciences/life-sciences/2026/ai-and-robotics-take-centre-stage-at-imperial-led-biodiversity-debate-in-parliament/">AI and robotics might help biodiversity</a> preservation efforts. Here's the Imperial College summary of the event:</p>
<blockquote>
<p>Additional exhibits included AI-powered conservation tools capable of
synthesising large scientific datasets to support evidence-based biodiversity
decision-making, as well as bio-inspired robotic systems designed for safer
environmental monitoring in complex ecosystems such as forests and waterways.</p>
<p>The exhibition highlighted how AI and robotics are already being applied to
real-world environmental challenges, from biodiversity monitoring to climate
innovation and conservation decision-making.
<cite>-- <a href="https://www.imperial.ac.uk/news/articles/natural-sciences/life-sciences/2026/ai-and-robotics-take-centre-stage-at-imperial-led-biodiversity-debate-in-parliament/">Emily Govan, 2026</a></cite></p>
</blockquote>
<p>From chatting to the Sadiq when I got back, a lot of the questions from the audience were around how quickly they could assemble their own evidence database, and how they might deploy their own version. Most of the policymakers deploying these systems are being <a href="https://anil.recoil.org/notes/red-pill-conservation">forced into using black-box AI</a> because of the <a href="https://anil.recoil.org/notes/coar-prc">access difficulties</a> in building their own local libraries of information.</p>
<p><img src="https://anil.recoil.org/images/hol-nature-2.webp" alt="%c" title="Showing the traceable pipeline to guests in the pavilion (image credit: Alec Christie)"></p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-in-paris" class="anchor" aria-hidden="true"></a>TESSERA in Paris</h2>
<p><a href="https://toao.com">Sadiq Jaffer</a> had a busy week as he also legged it to Paris to present TESSERA at the <a href="https://www.linkedin.com/feed/update/urn:li:share:7480946409609822208/">RAISE Summit</a> on the 8th/9th July, held at the Carrousel du Louvre.
This was quite a large AI event, and Vultr invited us to their booth to showcase the <a href="https://discover.vultr.com/tessera-case-study">TESSERA case studies</a> they've been working on. Since we're an open model, they worked through how to reproduce the pipeline and build an inference pipeline using their cloud infra.</p>
<p></p><div class="video-center"><iframe title="TESSERA: From Satellite Data to Fingerprints (RAISE Summit)" width="100%" height="315px" src="https://watch.eeg.cl.cam.ac.uk/videos/embed/42cea889-76a5-4f72-a542-cdd3d87a1241" frameborder="0" allowfullscreen="" sandbox="allow-same-origin allow-scripts allow-popups allow-forms"></iframe></div><p></p>
<p>This hardware is what made <a href="https://geotessera.org/blog/2026-06-09-tessera-v1-1">TESSERA v1.1</a> possible, so we're really grateful to Vultr and AMD for their support here! An especially big shoutout to Kasia Hilborne who has been an enthusiastic supporter of the project <a href="https://anil.recoil.org/notes/2026w3">since the start</a>, and who Sadiq finally met in person (I'm still looking out for when I can make it to one of these events myself!).</p>
<p><img src="https://anil.recoil.org/images/scale-sadiq.webp" alt="%c" title="Sadiq presenting TESSERA at SCALE. It was so loud they all had to wear headphones..."></p>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun links</h2>
<p><a href="https://mynameismwd.org">Michael Dales</a> chalks up another <a href="https://digitalflapjack.com/weeknotes/monument-maintenance/">great post</a> about a 3000 year old horse, and a commentary on open source maintenance. He'a got a great point about how open source code maintainance is in a weird place due to LLM driven contributions, and how to balance this new 'enthusiasm':</p>
<blockquote>
<p>Another issue in balancing the needs of the volunteer community and the needs
of the horse is that Andy has something of a "success failure" on his hands:
too many people want to maintain the horse now. Word of mouth has spread by
people like me who went and had a good time: you get to sit on a hill, feel
like you've contributed to something beyond yourself that will outlive you,
and you get to hang out with the kind of person who thinks it's a good idea
to sit on a hill and feel like you've contributed to something beyond
yourself that will outlive you. So perhaps you have too many people chalking
now that leads to you having to remove chalk the next year.
<cite>-- <a href="https://digitalflapjack.com/weeknotes/monument-maintenance/">Too much of a good thing: part 2, Michael Dales</a></cite></p>
</blockquote>
<p>Then, a nice independent validation of TESSERA turned up in a <a href="https://arxiv.org/abs/2607.08945">paper on mapping cocoa in Cote d'Ivoire</a>. They compared very high resolution imagery against several coarser inputs, and TESSERA came out as the best of the decametric options at an F1 of 0.86, not far behind the 0.92 of half metre imagery that costs way more to obtain.</p>
<p><a href="https://aneeshnaik.github.io/">Aneesh Naik</a> also has a great <a href="https://www.aneeshnaik.com/blogposts/20260703_weeknotes_2026_27.html">weeknote</a> where he mentions
TESSERA embeddings comfortably beating the Sentinel-2 baseline on the
GeoLifeCLEF species distribution challenge. We had a go at this challenge back
in 2022, so I'm keen to take a serious swipe at the next competition when it
runs again!</p>
<p>Till next week, stay cool everyone!</p><h1>References</h1><ul><li>Madhavapeddy (2026). Discussing effective conservation with all the UK Chief Scientists. <a href="https://doi.org/10.59350/qjrmv-38130" target="_blank"><i>10.59350/qjrmv-38130</i></a></li>
<li>Feng et al (2026). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li>
<li>Feng et al (2026). Applications of the TESSERA Geospatial Foundation Model to Diverse Environmental Mapping Tasks. SSRN. <a href="https://doi.org/10.2139/ssrn.6142416" target="_blank"><i>10.2139/ssrn.6142416</i></a></li>
<li>Madhavapeddy (2026). Tessera v1.1 released, with smoother and temporally stable embeddings. <a href="https://doi.org/10.59350/vcqjp-24y05" target="_blank"><i>10.59350/vcqjp-24y05</i></a></li>
<li>Jaffer et al (2025). AI-assisted Living Evidence Databases for Conservation Science. Cambridge Open Engage. <a href="https://doi.org/10.33774/coe-2025-rmsqf" target="_blank"><i>10.33774/coe-2025-rmsqf</i></a></li>
<li>Madhavapeddy (2025). Publish, Review, Curate to upend scholarly publishing. <a href="https://doi.org/10.59350/fpc9w-ccj82" target="_blank"><i>10.59350/fpc9w-ccj82</i></a></li>
<li>Feng et al (2026). TESSERA v2: Scaling Pixel-wise Earth Foundation Models. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.03949" target="_blank"><i>10.48550/arXiv.2607.03949</i></a></li>
<li>Ball et al (2026). Geospatial foundation models enable data-efficient tree species mapping in temperate mountain forests. Elsevier BV. <a href="https://doi.org/10.1016/j.srs.2026.100466" target="_blank"><i>10.1016/j.srs.2026.100466</i></a></li>
<li>Madhavapeddy (2026). TESSERA now supports the Zarr geo-embeddings convention proposal. <a href="https://doi.org/10.59350/c3hrq-zsx02" target="_blank"><i>10.59350/c3hrq-zsx02</i></a></li>
<li>Ball et al (2026). Phenological regularity, not functional traits, determines whether tropical tree species can be mapped from imaging spectroscopy. bioRxiv. <a href="https://doi.org/10.64898/2026.05.06.722428" target="_blank"><i>10.64898/2026.05.06.722428</i></a></li>
<li>Madhavapeddy (2026). Streaming millions of TESSERA tiles over HTTP with Zarr v3. <a href="https://doi.org/10.59350/tk0er-ycs46" target="_blank"><i>10.59350/tk0er-ycs46</i></a></li>
<li>Madhavapeddy (2026). .plan-26-25: Planetary scale plans, Windows file-descriptor scale problems. <a href="https://doi.org/10.59350/b3vvx-n70" target="_blank"><i>10.59350/b3vvx-n70</i></a></li>
<li>Omar et al (2025). A FAIR Case for a Live Computational Commons. Association for Computing Machinery. <a href="https://doi.org/10.1145/3759536.3763802" target="_blank"><i>10.1145/3759536.3763802</i></a></li>
<li>Carroll et al (2026). Bidirectional Type Slicing. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.12197" target="_blank"><i>10.48550/arXiv.2607.12197</i></a></li>
<li>Gibb et al (2026). Package Managers à la Carte: A Formal Model of Dependency Resolution. <a href="https://doi.org/10.1145/3828699" target="_blank"><i>10.1145/3828699</i></a></li>
<li>Carroll (2026). Type error debugging in Hazel. <a href="https://doi.org/10.48456/tr-1006" target="_blank"><i>10.48456/tr-1006</i></a></li>
<li>Carroll (2026). Polymorphic type slicing. <a href="https://doi.org/10.48456/tr-1007" target="_blank"><i>10.48456/tr-1007</i></a></li>
<li>Kusupati et al (2024). Matryoshka Representation Learning. arXiv. <a href="https://doi.org/10.48550/arXiv.2205.13147" target="_blank"><i>10.48550/arXiv.2205.13147</i></a></li>
<li>Gibb et al (2026). Package Managers à la Carte: A Formal Model of Dependency Resolution. arXiv. <a href="https://doi.org/10.48550/arXiv.2602.18602" target="_blank"><i>10.48550/arXiv.2602.18602</i></a></li>
<li>Orlowski et al (2026). Is sub-metre resolution necessary for cocoa mapping? A landscape-stratified evaluation of very high resolution imagery, decametric Earth Observation inputs, and operational products in Cote d'Ivoire. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.08945" target="_blank"><i>10.48550/arXiv.2607.08945</i></a></li></ul>
