---
title: '.plan-26-21: Pint of Science, OxCaml dissertations, and TESSERA 1.1 stirring'
description: After the BBC/ITV media run we had a talk at Pint of Science, two cracking
  Part II dissertations on CE/TESSERA and OxCaml vector RAG, and put TESSERA v1.1
  weights on HuggingFace.
url: https://anil.recoil.org/notes/2026w21
date: 2026-05-24T00:00:00-00:00
preview_image: https://anil.recoil.org/images/videos/6169311f-f1b3-449d-b435-fc040748697b.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Most of my week has been dominated by <a href="https://anil.recoil.org/notes/hedgehog-tessera-week">being on the BBC/ITV/etc</a> and so just a short weeknote this time around!
Aside from all the media coverage, there have been some fun events and hacking going on.</p>
<h2><a href="https://anil.recoil.org/news.xml#pint-of-science-cambridge-about-tessera" class="anchor" aria-hidden="true"></a>Pint of Science Cambridge about TESSERA</h2>
<p><a href="https://toao.com">Sadiq Jaffer</a> spoke at the <a href="https://pintofscience.com/">Cambridge Pint of Science</a>, which was held in the congenial surroundings of the Station Tavern near the train station. The room itself was quite small and long, and was hugely noisy due to being right beside the actual pub, but it was a wonderfully quirky and informal way to present science to a generalist audience.  The specific theme of the event we attended was "<a href="https://pintofscience.co.uk/event/how-is-ai-accelerating-science/">How is AI accelerating science?</a>", and other speakers ranged from <a href="https://www.cst.cam.ac.uk/people/mv487">Moe Vali</a> talking about ultrasound detection of <a href="https://www.nhs.uk/conditions/adenomyosis/">Adenomyosis</a> to <a href="https://www.cai.cam.ac.uk/people/dr-anna-breger">Anna Breger</a> talking about reconstructing <a href="https://www.youtube.com/watch?v=t0OzKoAYVdM">medieval music from ancient transcripts</a>! Sadiq had the pressure piled on being the last speaker at a point when the audience had mostly had their third pint:</p>
<p></p><div class="video-center"><iframe title="Sadiq at Pint of Science Cambridge" width="100%" height="315px" src="https://crank.recoil.org/videos/embed/6169311f-f1b3-449d-b435-fc040748697b" frameborder="0" allowfullscreen="" sandbox="allow-same-origin allow-scripts allow-popups allow-forms"></iframe></div><p></p>
<p>As always though, Sadiq pulled off explaining these complex ideas just brilliantly! His <a href="https://toao.com/static/pint-of-science-2026.pdf">slides</a> were some of the best I've seen yet, as it was done with a combination of his expert knowledge about <a href="https://anil.recoil.org/papers/2025-tessera">TESSERA</a> and the use of <a href="https://www.anthropic.com/news/claude-design-anthropic-labs">Claude Design</a> (the latest iteration of an AI vision model from Anthropic).</p>
<p><a href="https://toao.com/static/pint-of-science-2026.pdf"> <img src="https://anil.recoil.org/images/pintofscience-slides.webp" alt="%c" title="Gorgeous slides about how TESSERA works"> </a></p>
<p>You can also see another iteration of Sadiq speaking about TESSERA in this
<a href="https://crank.recoil.org/w/dxXkfLYocHMbZtdkZLAc8R?start=1m35s">news piece</a> on ITV the day after this talk.</p>
<h2><a href="https://anil.recoil.org/news.xml#cambridge-part-ii-projects-due-in" class="anchor" aria-hidden="true"></a>Cambridge Part II projects due in</h2>
<p>The Cambridge undergrads had to get their Part II dissertation projects in, and in particular I <em>really</em> enjoyed two (which I couldn't directly mark/supervise as I'm on sabbatical, but I cheered on from the sidelines).</p>
<h3><a href="https://anil.recoil.org/news.xml#conservation-actions-using-multimodal-foundation-models" class="anchor" aria-hidden="true"></a>Conservation actions using multimodal foundation models</h3>
<p>The first is Radhika Iyer on applying TESSERA to test the <a href="https://anil.recoil.org/ideas/assessing-conservation-actions-using-tessera">effectiveness of conservation actions</a> which was
a very bold foray into the unknown that she did a brilliant job of writing up.
I'll talk more about this one in a while, but the execution in combining two
very separate (and cutting edge) projects into a cohesive thesis was remarkable
work.</p>
<h3><a href="https://anil.recoil.org/news.xml#hybrid-vector-databases-in-oxcaml" class="anchor" aria-hidden="true"></a>Hybrid vector databases in O(x)Caml</h3>
<p>The other was as ambitious but in a totally different dimension: the first <a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a> project we've supervised here with an undergraduate, with <a href="https://ryan.freumh.org">Ryan Gibb</a> keeping a close eye. <a href="https://github.com/olifog">Oliver Fogelin</a> went off and built a <a href="https://anil.recoil.org/ideas/oxcaml-vector-db">hybrid vector RAG</a> database in OxCaml, including embedding a big stack of arxiv papers and showing a beautiful browser visualisation of the embeddings in an interactive way. You can try it for yourself <a href="https://4047f.com/explorer/3523152">on his site</a>.</p>
<p><img src="https://anil.recoil.org/images/oli-project-1.webp" alt="%c" title="A rapt audience for Oli's demo"></p>
<p><a href="https://4047f.com/explorer/3523152"> <img src="https://anil.recoil.org/images/oli-project-2.webp" alt="%c" title="Try out Oli's explorer for yourself by clicking on the image!"> </a></p>
<p>We had an entertaining session in my office with Oli demonstrating it to us. I <em>almost</em> got it to run out of the box via <code>oix --toolchain=oxcaml --with=https://github.com/olifog/gvecdb-ocaml gvecdb-server</code>, except we got scuppered by a few (valid) build failures in dependent libraries that he had patched locally but not committed. It's very close to being able to work, though!</p>
<p>This is some of the most impressive hacking I've seen in a Part II project in a while, and I'm very much going to try to replace my shaky website search with Oli's code when things calm down a bit this summer!</p>
<h2><a href="https://anil.recoil.org/news.xml#tessera-v11-updates" class="anchor" aria-hidden="true"></a>TESSERA v1.1 updates</h2>
<p>We've had a flurry of activity on preparing a TESSERA 1.1 model update with <a href="https://www.cst.cam.ac.uk/people/zf281">Frank Feng</a> pulling out all the stops on training with help from Nvidia. This will be the first updated iteration of the TESSERA mode that we've issued. More on the cool new features next week, but in the meanwhile some headlines are:</p>
<ul>
<li>We've put up the <a href="https://huggingface.co/geotessera">weights on Huggingface</a> in a new <code>geotessera</code> org. You can find both <a href="https://huggingface.co/geotessera/TESSERA-V-1.0">TESSERA-V-1.0</a> and <a href="https://huggingface.co/geotessera/TESSERA-V-1.1">TESSERA-V-1.1</a> there now.</li>
<li><a href="https://www.tunbury.org/">Mark Elvers</a> has an impressive <a href="https://www.tunbury.org/2026/05/20/processing-uk-azure-spot/">OxCaml inference pipeline</a> we've been using to do spot generation of 1.1, while also collaborating with partners on a full AWS inference run that'll go to Zarr directly. If you need some specific 1.1 embeddings urgently, let me know or <a href="https://github.com/ucam-eo/geotessera/issues">raise an issue</a>.</li>
<li>I'm working on a <a href="https://github.com/ucam-eo/geotessera/pull/250">geotessera 0.9</a> that'll support the switch to hosting on S3 and also support the 1.1 model variant.</li>
</ul>
<h2><a href="https://anil.recoil.org/news.xml#fun-links" class="anchor" aria-hidden="true"></a>Fun Links</h2>
<ul>
<li>I enjoyed this <a href="https://www.youtube.com/watch?v=9M_dq_0ljsc">video interview</a> of economist <a href="https://en.wikipedia.org/wiki/Clara_Mattei">Clara Mattei</a> speaking about what alternatives to capitalism might look like (via <a href="https://haddadi.github.io/">Hamed Haddadi</a>).</li>
<li>Listened to a fantastic podcast on Amol Rajan's Radical on <a href="https://www.bbc.co.uk/sounds/play/m002wkk4">The Future of Food: Can Regenerative Farming Save Our Soil?</a></li>
<li>Read some interesting preprints on how <a href="https://arxiv.org/abs/2605.18667">geospatial embedding models are complementary</a> and also on how <a href="https://arxiv.org/abs/2605.26099">language models need sleep</a> (in order to improve long context attention).</li>
<li>I got <code>oix --toolchain=oxcaml-minus37 strace_tui</code> working with a very <a href="https://github.com/oxcaml/opam-repository/pull/50">WIP release</a> from <a href="https://www.dra27.uk">David Allsopp</a>. This will all make sense if you read <a href="https://blog.janestreet.com/strace-ui-bonsai-term-and-the-tui-renaissance/">Jane Street's post about terminal UIs making a comeback</a>.</li>
<li>Resumed work on <a href="https://notes.roscidus.com/2026/05/24/">Eio again</a> with <a href="https://roscidus.com">Thomas Leonard</a> returning, so been refreshing a bunch of my trees.</li>
<li>Next week I'm off to Edinburgh to speak at <a href="https://www.eventbrite.co.uk/e/rewilding-the-web-diversity-resilience-in-sociotechnical-infrastructure-tickets-1983352473639">Rewilding the Web</a> with Jon Crowcroft. It's due to be a heat wave, so wish me luck!</li>
</ul><h1>References</h1><ul><li>Feng et al (2026). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li>
<li>Plas et al (2026). Better Together: Evaluating the Complementarity of Earth Embedding Models. arXiv. <a href="https://doi.org/10.48550/arXiv.2605.18667" target="_blank"><i>10.48550/arXiv.2605.18667</i></a></li>
<li>Lee et al (2026). Do Language Models Need Sleep? Offline Recurrence for Improved Online Inference. arXiv. <a href="https://doi.org/10.48550/arXiv.2605.26099" target="_blank"><i>10.48550/arXiv.2605.26099</i></a></li></ul>
