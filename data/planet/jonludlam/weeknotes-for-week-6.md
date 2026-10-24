---
title: Weeknotes for week 6
description:
url: https://jon.recoil.org/blog/2026/02/weeknotes-2026-06.html
date: 2026-02-09T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>Highlights:</p>
<ul>
<li><a href="https://jon.ludl.am/experiments/day10-jtw/standalone/index.html">day10 / javascript toplevels integration</a></li>
<li><a href="https://jon.ludl.am/experiments/scrollycoder/">Scrollycode experiments</a></li>
</ul>
<h2>Oxmono</h2>
<p>I spent some time on Anil's oxmono repo getting odoc to work correctly. It turned out that the bug I was working on last week was critically important for this - and that the bugfix was incomplete. One of the issues was to do with identifiers needing to be unique. For example, consider the following code:</p>
<p><x-ocaml mode="interactive">module type S = sig
type t</x-ocaml></p>
<p>include sig
type t</p>
<pre><code>val f : t -&amp;gt; t
</code></pre>
<p>end with type t := t
end
The problem here is that both definitions of `type t` have the same identifier, which causes problems when we move to and from the 'Component' types. The solution was to introduce a 'dummy' parent for the type defined within the include. This works because we never actually render the body of the include into HTML - we render the <em>expansion</em>, which <em>doesn't</em> have <code>type t</code> in it, as it has been substituted out.</p>
<p>The fix I made last week fixed the <a href="https://jon.recoil.org/reference/odoc/odoc.loader/Odoc_loader/index.html">loader</a>, which reads in the <code>cmt</code>/<code>cmti</code> files produced by the compiler. There's one more place where we create these in the code - when we translate from the <a href="https://jon.recoil.org/reference/odoc/odoc.xref2/Odoc_xref2/Component/index.html">Component</a> types back into <a href="https://jon.recoil.org/reference/odoc/odoc.model/Odoc_model/Lang/index.html">Lang</a> types. I was a little curious about whether it was possible to make this happen, so I thought I'd ask Claude to see if it could come up with a scenario where we'd end up in this situation. This was a complete failure, which was a real disappointment to me, as doing this sort of thing is a quite tedious and annoying part of working on odoc.</p>
<p>Meanwhile, I was running odoc on Anil's <a href="https://github.com/avsm/oxmono">oxmono</a> repo, which was using <a href="https://github.com/art-w">art-w</a>'s <a href="https://github.com/ocaml/odoc/pull/1399">PR to upstream oxcaml support</a>. It was failing with an exception that was very familiar, so I pulled in the fix I'd been working on, and that enabled it to get much further. However, it did subsequently fail with another slightly different exception. I had my suspicions at this point that it might be due to the other place, but I thought this again was a good opportunity to test Claude's debugging skills. However, this again was a complete failure. I spend quite a long time prodding it - at least 4 separate sessions - and it really didn't get anywhere close to a solution, despite knowing precisely that the commit we'd made that had fixed the first problem. Two of the four times it ended up telling me that the oxcaml compiler was broken and suggesting that we create an issue!</p>
<p>I'm only very mildly disappointed in this - it's all quite subtle, and something I still end up scratching my head over sometimes, but it would have been wonderful to be able to offload this sort of work!</p>
<p>In any case, the docs now all build on <a href="https://github.com/jonludlam/oxmono/commit/2a53f6857d5b8849a73f5bb3e5244b9ac0f36708">my fork of oxmono</a>.</p>
<h2>Docs CI</h2>
<p>The fix I deployed last week for ocaml-docs-ci was taking forever to complete, so I ended up spending some time investigating this. The problem was happening during the 'prep' phase, which is the first part of the pipeline where we simply build the package to be documented. This is supposed to work by building a graph of all inter-package dependencies across all of the solved packages, so we maximise sharing of built artefacts. Each 'prep' job builds precisely one package by coping in the dependencies from previous prep jobs, then running <a href="https://github.com/jonludlam/opamh">opamh</a> to fix up the metadata so that opam believes it has installed everything itself, then running opam to build the one package required. It was this last step that was going wrong, where it would decide that there had been upstream changes to the compiler itself, and rebuild <em>everything</em>, so rather than a prep job taking a few seconds, it would take a few minutes.</p>
<p>I was totally unable to repro this locally - everything build very quickly and just how it should have done. After much head-scratching I finally realised that the problem was somewhere in the caching. I think what's going on is that we dynamically build an opam repository to make the `opam install` command faster, and that repo contains only the packages that are required to build whatever it is we're building. Those opam files are cached by the docs CI server and passed to the build script as a base64-encoded gzipped tarball inline in the obuilder file (!). This should all be totally consistent as we're also caching all the builds - except for the compiler itself, which comes from the base docker image. This, of course, is the problem. The ocaml compiler opam files had been updated, and then when we reconstructed the opam repo with our cached opam files, opam noticed they had changed (gone <em>backwards</em> in time!) and decided it needed to rebuild the compiler, and therefore <em>everything</em> else. Clearing out the opam-files cache and restarting the builds fixed this entirely, and the full rebuild job completed after about 2 days. I flipped the switch on Saturday night and the docs are now fully up to date again. Phew!</p>
<h2>day10 work</h2>
<p>This was a fun week of large-scale building! I integrated day10 and odoc_driver and js_top_worker and x-ocaml and have now successfully got a docs-ci-like system that's able to build docs and toplevels that can coexist in the one HTML tree. I've not got a full integrated demo yet, but you can see the test cases for this <a href="https://jon.ludl.am/experiments/day10-jtw/standalone/index.html">here</a>. Be sure to take a look at the 'network' tab in the browser dev tools to see what it's doing!</p>
<h2>Scrollycode experiments</h2>
<p>I've long been a fan of <a href="https://pomb.us/">Rodrigo Pombo's</a> work on "building tools for better code reading comprehension", ever since first seeing his post "<a href="https://pomb.us/build-your-own-react/">Build your own React</a>". Claude is <em>fantastically good</em> at doing this sort of thing, so I asked it to go and build me some simple OCaml-focused versions. We came up with 5 variations in the end - and they're all pretty neat! <a href="https://jon.ludl.am/experiments/scrollycoder/">take a look!</a>. The best part of this was that it took me less than half-an-hour to get Claude to do all this.</p>
<h2>Dune PR</h2>
<p>I attended the bi-weekly dune dev meeting to talk about the first part of the dune PR - the bit that Paul Elliot did almost a year ago.</p>
<h2>Coming week</h2>
<p>So the clock is ticking on writing the exam questions for FoCS, so I'll need to be spending time this week on that.</p>

    
