---
title: 'Voluntary AI disclosure proposal for OCaml: update 1'
description: An update on the voluntary AI disclosure proposal, digesting the security,
  quality and legal feedback, and some concrete next steps around maintenance intent,
  multi-repository tooling, and reputation.
url: https://anil.recoil.org/notes/opam-ai-disclosure-update
date: 2026-05-14T00:00:00-00:00
preview_image: https://anil.recoil.org/images/faces/avsm.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>A quick update on my <a href="https://anil.recoil.org/notes/opam-ai-disclosure">proposal for voluntary AI disclosure in OCaml code</a>. The <strong>online <a href="https://discuss.ocaml.org/t/a-proposal-for-voluntary-ai-disclosure-in-ocaml-code/17950/28?u=avsm">discussion
thread</a></strong>
has gathered a lot of thoughtful feedback, both public and private, so I
posted a digest of where I think things stand and what we might do next.
It's reproduced here for the record:</p>
<blockquote>
<p>Thanks, everyone, for the thoughtful and polite feedback to my proposal. I've received a lot of private comments as well, from many perspectives, so I'll attempt to digest them here.</p>
<p>The prevailing concerns seem to hinge around quality and security and (to a lesser extent) legalities. This is <em>not</em> to diminish the debate around ethics, but this is such an active and evolving topic that I can't pin much down there yet.</p>
<h2><a href="https://anil.recoil.org/news.xml#security" class="anchor" aria-hidden="true"></a>Security</h2>
<p>This is a growing concern for the opam-repository, and is one I think that goes well beyond CVE tracking. We often use a social signal as opam repo maintainers to "sniff" a packaging PR and browse around the original source to ensure that it's reasonable. In many cases, we offer suggestions to the package submitter, many of whom apply those changes.</p>
<p>Now, however, with LLM generated content, this social signal is demolished since every package comes with confidently verbose reams of text. It's no longer practical to assess code by quickly reading through it, and we'll need some other measure or automation to help out here. I offer no quick solution here, except for some emerging <a href="https://tangled.org/gazagnaire.org/merlint#output-example">type driven linters</a> that can distinguish "bad vibe coding" from the more curated agentically boosted approaches.</p>
<p>A major problem here is that backdoors could slip quite easily into this high volume code, which leads onto the next topic of quality.</p>
<h2><a href="https://anil.recoil.org/news.xml#quality" class="anchor" aria-hidden="true"></a>Quality</h2>
<p>We've resisted measuring popularity by the number of downloads in opam, preferring instead to look for more stable metrics such as the number of downstream dependencies on a package. This signal has been pretty good; there are islands of popular maintainers and packages, and the opam repository serves to aggregate them all and sort out incompatibilities at package submission time via constraints. In other words, the opam repo is a collective database that is more than the sum of the individual packages.</p>
<p>With LLM generated code, there's often a desire to 'throw something over the wall' and not keep it updated. If we accept these sorts of packages into the opam repository, we're not improving the health of our collective database, since <a href="https://nesbitt.io/2026/05/08/weekend-at-bernies.html">unmaintained packages</a> could rapidly accrue dependencies without humans behind them.</p>
<p>Therefore, our <a href="https://github.com/ocaml/opam-repository/blob/master/governance/policies/archiving.md#specification-of-the-x--fields-used-in-the-archiving-process">maintainer intention field</a> might become more important moving forward. I can see us accepting LLM packages (that are beyond a minimum level of slop that we can leave to opam repo maintainer judgement) that are set to a maintenance intent of <code>none</code>. This would, at least, be honest, and a signal that other people are welcome to pick up the baton and iteratively improve that particular effort.</p>
<p>A useful improvement to opam itself may be to <em>avoid</em> packages in the dependency chain that have declared themselves unmaintained.</p>
<h2><a href="https://anil.recoil.org/news.xml#legality" class="anchor" aria-hidden="true"></a>Legality</h2>
<p>This one's the most potentially serious, especially given the diverse and international nature of our contributors (from individuals, to corporates, to academic). Unfortunately, it's also the most in flux; the current legal situation is murky, varies by country, and is being actively legislated almost everywhere.</p>
<p>The goal of my proposal above is voluntary disclosure to make future provenance easier to figure out, but I have doubts it's going to take off: even within my own group, people are reluctant to disclose AI usage for a variety of reasons. Some worry it's a poor social signal, others have it tightly integrated into their workflows and treat it like a code editor, and yet others are not computing experts and do not distinguish.</p>
<p>However, if you do have strong opinions, then now is the time to feed back to your legislative bodies! <a href="https://github.com/samoht">Thomas Gazagnaire</a> pointed out to me that the <a href="https://ec.europa.eu/eusurvey/runner/Art50guidelines">EU is seeking feedback on Article 50</a>, so I'll be submitting a synopsis to that.</p>
<h2><a href="https://anil.recoil.org/news.xml#so-what-do-we-do-next" class="anchor" aria-hidden="true"></a>So what do we do next?</h2>
<p>I have just three concrete suggestions for now:</p>
<h3><a href="https://anil.recoil.org/news.xml#make-maintenance-intent-first-class-in-opam" class="anchor" aria-hidden="true"></a>Make maintenance intent first-class in opam</h3>
<p>We could <strong>promote the x-maintenance-intent field to be a first class opam field</strong>, and actively 'solve around' unmaintained packages. We have this really fancy solver, so why not use it?</p>
<h3><a href="https://anil.recoil.org/news.xml#improve-tooling-for-multiple-package-repositories" class="anchor" aria-hidden="true"></a>Improve tooling for multiple package repositories</h3>
<p>opam supports handling multiple simultaneous package repositories just fine. In fact, we've got two active ones: <a href="https://github.com/ocaml/opam-repository">ocaml/opam-repository</a> and <a href="https://github.com/ocaml/opam-repository-archive">ocaml/opam-repository-archive</a> today.</p>
<p>What's missing is the <strong>tooling to manipulate, filter and merge multiple opam repositories</strong> easily (I pushed <a href="https://github.com/avsm/opam-repomin">repomin</a> for this purpose). Having better tooling here would allow us to (for example) have:</p>
<ul>
<li>an opam repository just for all OCaml compilers. This is extremely useful for the developers and packagers and testers to have just the build rules and patches in one place.</li>
<li>an opam repository that's compatible with Windows, with non-building packages filtered out.</li>
<li>an opam repository that's got just the latest versions of packages (an equivalent of Stack).</li>
<li>an opam repository with only a core of curated and maintained packages that's small and portable.</li>
<li>an opam repository that explicitly accepts 'work in progress' LLM generated outputs, for those who want to live on the agentic bleeding edge.</li>
</ul>
<h3><a href="https://anil.recoil.org/news.xml#is-it-time-to-consider-a-reputation-system" class="anchor" aria-hidden="true"></a>Is it time to consider a reputation system?</h3>
<p>Hannes has worked on <a href="https://hannes.robur.coop/Posts/ConexRunning">conex</a> for many years, but it hasn't been pushed into opam repository due to the significant hassle involved in key management for end users.</p>
<p>Is it now time to bring back a system like this, but with <a href="https://blog.tangled.org/vouching/">vouching</a> as a first-class feature? The good folk at <a href="https://tangled.org">tangled.org</a> have been building in "evidence" to their vouching system, which took me back to the good old days of <a href="https://web.archive.org/web/20170715120119/http://advogato.org/person/Stab">Advogato</a> (for the really oldies among you!).</p>
<p>As with all such efforts, this will require coordination and contribution from all interested in making such change happen :-) I'm very willing to be corrected on anything I've raised above!</p>
</blockquote><h1>References</h1><ul><li>Madhavapeddy (2026). A Proposal for Voluntary AI Disclosure in OCaml Code. <a href="https://doi.org/10.59350/cxypn-ysv27" target="_blank"><i>10.59350/cxypn-ysv27</i></a></li></ul>
