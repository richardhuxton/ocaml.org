---
title: Open MPhil/Part II student projects for 2026-2027
description: A refreshed set of Part II and MPhil projects, split between environmental
  science problems that need computer science, and more conventional systems and programming
  languages topics.
url: https://anil.recoil.org/notes/student-projects-2026
date: 2026-09-09T00:00:00-00:00
preview_image: https://anil.recoil.org/images/eeg-xmas-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I've refreshed my <a href="https://anil.recoil.org/ideas">project ideas</a> for incoming <a href="https://www.cst.cam.ac.uk/teaching/part-ii">CST Part II</a> and <a href="https://www.cst.cam.ac.uk/teaching/masters">MPhil</a> students who are starting in October 2026 (term's starting soon!).
As ever, these aren't an exhaustive list but a good starting point for things
we might work on together.  See also <a href="https://svr-sk818-web.cl.cam.ac.uk/keshav/wiki/index.php/Main_Page">Srinivasan Keshav</a>'s <a href="https://svr-sk818-web.cl.cam.ac.uk/keshav/teaching/projects.html">own list</a> for even more.</p>
<p>The open projects fall into two rough batches: applying
compsci to solve difficult <a href="https://anil.recoil.org/projects/plancomp">environmental problems</a>, and more
conventional systems / programming languages work on verification, effects or <a href="https://anil.recoil.org/projects/unikernels">unikernels</a>.
Almost everything on the list is pitched at an MPhil-level project, but it's
fairly easy to pitch it up and down to a Part II/III depending on your enthusiasm.</p>
<p>You have a <em>much</em> better chance of getting my attention if you
show evidence of thinking and researching the idea a bit before getting in
touch. I unfortunately receive dozens and dozens of LLM-generated CVs a week
these days and easily miss generic messages.</p>
<h2><a href="https://anil.recoil.org/news.xml#tough-planetary-computing-projects" class="anchor" aria-hidden="true"></a>Tough planetary computing projects</h2>
<p>I want to point to three of the <a href="https://anil.recoil.org/projects/tessera">Tessera</a> ideas in particular,
because it's a <em>great</em> time to be working in this field. The last year
has seen a huge <a href="https://anil.recoil.org/notes/geotessera-a-year-on">surge of interest</a> in geospatial foundation models.</p>
<p><a href="https://anil.recoil.org/ideas/tessera-ghost-roads">"Generative detection of ghost roads"</a> seems easy
at first glance: surely we should easily be able to spot all roads from space using satellites?
It turns out that a great many roads simply aren't mapped, especially in the tropics, and road
building almost always precedes forest loss events. The
<a href="https://doi.org/10.1038/s41586-024-07303-5">2024 study</a> that established this
took some 7,000 hours of volunteer effort to hand-map the tropical
Asia-Pacific, and found between three and six times (!) more roads than the global
datasets knew about.</p>
<p>This project observes that per-pixel and patch classifiers are throwing away some
obvious things we know about roads: for example, almost all roads connect to other roads!
This structural prior means we can experiment with some fun machine learning
approaches to bolt topology-preserving losses onto segmentation heads.
We could also try the <a href="https://arxiv.org/abs/1802.03680">RoadTracer</a> technique and have an agent walk outwards
from known seeds in the embedding space, deciding at each step whether the road continues. Or we could
treat crowd-sourced data like OpenStreetMap as a partial graph and train a model to propose missing
edges, sampling repeatedly to get a distribution over where unmapped roads might be.</p>
<p><a href="https://anil.recoil.org/ideas/tessera-pnv">"Generating potential natural vegetation maps"</a> asks what the land
surface of the world would look like today if humans hadn't existed.
Global species extinction metrics like <a href="https://anil.recoil.org/projects/life">LIFE</a> (<a href="https://anil.recoil.org/papers/2024-life">paper</a>) need
baselines like this to determine the impact of human actions. And more locally,
we could also use a 'what-if' machine to figure out the impact of (e.g.) replacing
an agricultural field with a forest on the local biome.</p>
<p>This is quite a challenging machine learning problem since we need to find
the (current) parts of the planet unaffected by humans, and then learn how
climate, terrain and soil relate to what the land surface does when it is left
alone, and then apply generative techniques to the now-human regions. It builds
directly on our <a href="https://anil.recoil.org/ideas/tessera-habitat-maps">global habitat mapping</a> work. Validation
could start with held-out wild regions and comparisons against the existing PNV
maps, and we'd have to brainstorm further ideas for a map generated using
current climatic conditions...</p>
<p><a href="https://anil.recoil.org/ideas/tessera-interpretable-downscaling">"Interpretable downscaling of local weather predictions"</a>
is a lot of fun if you want to get into matters of both climate and biodiversity!
We now know from <a href="https://www.linkedin.com/in/pedro-marques-sousa">Pedro Sousa</a>'s work that <a href="https://anil.recoil.org/papers/2026-weather-downscaling">satellite embeddings improve local forecasting</a>
significantly, but <a href="https://anil.recoil.org/notes/weather-downscaling-tessera">not why</a>.
We need to explore techniques where we can attribute the gained accuracy back to recognisable surface properties in the Tessera embeddings, or turn the problem around and figure out how to ascribe weights to individual station observations.
This would be great fun for anyone interested in diving into probabilistic modelling.</p>
<p><a href="https://anil.recoil.org/ideas/literature-range-habitat-maps">"Scanning the literature for species range and habitat maps"</a>
is one for LLM and VLM aficionados. We're building an agentic pipeline over the millions of conservation fulltexts in <a href="https://anil.recoil.org/papers/2025-evidence-tap">Evidence TAP</a> to
recover habitat classes, elevation limits and range polygons for taxa that
have never had a Red List assessment. There's a <em>lot</em> of difficult vision work here, since
these papers have (sometimes hand-drawn) diagrams where some obscure knowledge about a
species was recorded 50 years ago. Quite often, the range maps are georeferenced with
respect to some local landmarks.  This project suits someone interested in evaluation
design and in getting VLMs to behave over long documents.</p>
<p><a href="https://anil.recoil.org/ideas/agentic-field-guides">"Agentic LLMs and local field guides"</a> is for the birders
among you! Anyone who's been travelling knows that the local guides have much more
information about a given region than a general book. So rather than training massive
classifiers, we're working out whether a multimodal model given a regional field
guide ('The Birds of Peru') can do VLM search and reason its way to an identification. This
long tail (pun intended) of local species is often where biodiversity occurrence sightings
are weakest, and so improving performance there will make a big difference
to the accuracy of species mapping, and to efforts like a <a href="https://anil.recoil.org/ideas/living-iucn-redlist">living IUCN Red List</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#systems-and-programming-languages" class="anchor" aria-hidden="true"></a>Systems and programming languages</h2>
<p>Systems and PL work remains very much of interest, of course; there's plenty
to be done! Here are some of the things I've been thinking about recently:</p>
<ul>
<li><a href="https://anil.recoil.org/ideas/lean-dijkstra-automata">Compiling Lean specifications into OxCaml enforcement automata</a>
(MPhil). Write a cross-layer effect specification once in <a href="https://lean-lang.org">Lean 4</a> as a Dijkstra
monad, then compile it into proof obligations as well as a runtime automaton in
<a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a>, with a proof that the latter represents the former. I did a much
cruder version of this twenty years ago with <a href="https://anil.recoil.org/papers/2009-icfem-spl">SPL</a>.
Suits someone who wants both proofs and executable fast code.</li>
<li><a href="https://anil.recoil.org/ideas/lean-io-uring-backend">An io_uring backend for Lean</a> (MPhil). What the heck
does a shared memory protocol have to do with a proof assistant, I hear you ask? Lean 4
can emit executables, but they currently do slow blocking IO. I thought it'd be fun
to not only specify the submission/completion ring pair in Lean itself, but also to wire it up into the runtime and
prove properties about the single-producer/single-consumer invariants. See my
<a href="https://anil.recoil.org/notes/icfp25-post-posix">post-POSIX talk</a> for some background.</li>
<li><a href="https://anil.recoil.org/ideas/antibotty-testbed">An antibotty defensive testbed</a> (MPhil). In the wild
west of the modern Internet, the time-to-exploit a vulnerability is <a href="https://anil.recoil.org/notes/rumour-is-the-exploit">now negative</a>. This project asks whether
a mitigation to some new AI cyberattack can be synthesised, verified and deployed <em>faster</em> than an
agent can write the exploit in the first place! I was thinking about a small total rule language with safety
obligations, and enforced by a <a href="https://mirage.io">MirageOS</a> gateway. Suits someone interested in doing lightweight formal
methods and some unikernel plumbing to get some systems experience.</li>
<li><a href="https://anil.recoil.org/ideas/tracing-hdl-with-effects">A hardware description language using OCaml effects</a>
(MPhil). <a href="https://github.com/janestreet/hardcaml">HardCaml</a> builds a circuit as a data structure via some fairly
demanding OCaml module-system work. Could OCaml 5 effect handlers let us instead
describe the circuit by evaluating it directly, as <a href="https://github.com/clash-lang/clash-compiler">Clash</a> does?
This suits someone with an interest in language design who wants to get into hardware synthesis.</li>
</ul>
<p>Most of these have supervisors beyond just me who have expertise deep in
the adjacent fields. Several of the projects also sit beside larger programmes
in my group such as <a href="https://anil.recoil.org/projects/tessera">TESSERA</a>, <a href="https://anil.recoil.org/projects/enki">Enki</a>, <a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a> or
<a href="https://anil.recoil.org/projects/ce">Conservation Evidence</a>, where you have loads of other people to talk to.</p>
<p>So get brainstorming, and I'm looking forward to seeing you all at the start of term!</p>
<p><img src="https://anil.recoil.org/images/eeg-xmas-1.webp" alt="%c" title="The EEG group last Christmas wearing silly hats, proving we exist. Join usss!"></p><h1>References</h1><ul><li>Eyres et al (2025). LIFE: A metric for mapping the impact of land-cover change on global extinctions. <a href="https://doi.org/10.1098/rstb.2023.0327" target="_blank"><i>10.1098/rstb.2023.0327</i></a></li>
<li>Jaffer et al (2025). AI-assisted Living Evidence Databases for Conservation Science. Cambridge Open Engage. <a href="https://doi.org/10.33774/coe-2025-rmsqf" target="_blank"><i>10.33774/coe-2025-rmsqf</i></a></li>
<li>Sousa et al (2026). Earth observation embeddings are effective sub-grid descriptors for probabilistic weather downscaling. arXiv. <a href="https://doi.org/10.48550/arXiv.2608.12271" target="_blank"><i>10.48550/arXiv.2608.12271</i></a></li>
<li>Madhavapeddy (2025). It's time to go post-POSIX at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/mch1m-8a030" target="_blank"><i>10.59350/mch1m-8a030</i></a></li>
<li>Madhavapeddy (2026). Just a rumour of a bug is enough to find a security exploit these days. <a href="https://doi.org/10.59350/tngsm-6rx23" target="_blank"><i>10.59350/tngsm-6rx23</i></a></li>
<li>Madhavapeddy (2009). Combining Static Model Checking with Dynamic Enforcement Using the Statecall Policy Language. Springer. <a href="https://doi.org/10.1007/978-3-642-10373-5_23" target="_blank"><i>10.1007/978-3-642-10373-5_23</i></a></li>
<li>10.1038/s41586-024-07303-5<a href="https://doi.org/10.1038/s41586-024-07303-5" target="_blank"><i>10.1038/s41586-024-07303-5</i></a></li></ul>
