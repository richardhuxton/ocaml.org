---
title: 'The Road to Odoc 3: Module Type Of'
description:
url: https://jon.recoil.org/blog/2025/03/module-type-of.html
date: 2025-03-08T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#the-road-to-odoc-3:-module-type-of" class="anchor"></a>The Road to Odoc 3: Module Type Of</h1>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-03-08</p></li></ul>
<ul class="at-tags"><li class="page-tags"><span class="at-tag">page-tags</span> <p><a href="https://jon.recoil.org/tags/ocaml.html" title="ocaml">ocaml</a> <a href="https://jon.recoil.org/tags/odoc.html" title="odoc">odoc</a></p></li></ul>
<p>There are <a href="https://discuss.ocaml.org/t/ann-odoc-3-beta-release/16043">many new and improved features</a> that Odoc 3 brings, but there are also a large number of bugfixes. I thought I'd write about one in particular here, an <a href="https://github.com/ocaml/odoc/pull/1081">overhaul of "module type of"</a> that landed in May 2024.</p>
<h2><a href="https://jon.recoil.org/atom.xml#module-type-of" class="anchor"></a>Module Type Of</h2>
<p>module type of is a language feature of OCaml allowing one to recover the signature of an existing module. For example, if I had a module <code>X</code>:</p>
<div><pre class="language-ocaml"><code>module X = struct
  type t = Foo | Bar
end</code></pre></div>
<p>then I can get back the signature of <code>X</code> using <code>module type of</code>:</p>
<div><pre class="language-ocaml"><code>module type Xsig = module type of X</code></pre></div>
<p>which can be very useful if you’re trying to <a href="https://discuss.ocaml.org/t/extend-existing-module/1389">extend existing modules</a> amongst other things.</p>
<p>OCaml and Odoc treat module type of in somewhat different ways. OCaml internally expands the expression immediately it sees it, and effectively replaces it with the signature - ie, in the above example Xsig is now a signature, not a module type of expression.</p>
<p>In contrast, Odoc would like to keep track of the fact that this signature came from a <code>module type of</code> expression, as it’s very useful to know. If you’re extending a module, your signature might look like:</p>
<div><pre class="language-ocaml"><code>module type UnitExtended = sig
  include module type of Unit
  val extra_unit_function : unit -&gt; unit
end</code></pre></div>
<p>The documentation we produce will expand the contents of the <code>include</code> statement, but keep track of the fact that it came from a <code>module type of</code> expression so the reader can see where these signature items came from. In practice, you'd probably want to use <code>module type of struct include Unit end</code>, which is a bit different from simply <code>module type of Unit</code>, and I'll talk about this at some point in a future post.</p>
<h2><a href="https://jon.recoil.org/atom.xml#the-problem" class="anchor"></a>The problem</h2>
<p>We run into difficulties as soon as we introduce another language feature that operates on signatures: with. Let’s start with a module type <code>S</code>:</p>
<div><pre class="language-ocaml"><code>module type S = sig
  module X : sig
    type t = int
  end

  module type Y =
    module type of X
end</code></pre></div>
<p>We’ll now define a new module <code>X2</code> that we intend to use as a replacement for <code>X</code>:</p>
<div><pre class="language-ocaml"><code>module X2 = struct
  type t = int
  type u = float
end</code></pre></div>
<p>Now we’ll define a new module type <code>T</code> which is <code>S</code> but with <code>X</code> replaced:</p>
<div><pre class="language-ocaml"><code>module type T = S with module X := X2</code></pre></div>
<p>Here you can see that OCaml has expanded the <code>module type of</code> expressions and told us the computed signature. The interesting thing here is that in module type <code>T</code>, module type <code>Y</code> only has a type <code>t</code> in it, not a type <code>u</code>. As above, Odoc wants to keep the <code>module type of</code> expression so the reader can tell where module type <code>Y</code> came from. However, the substitution would do a different thing in this case - we would have the following:</p>
<div><pre class="language-ocaml"><code>module type T = sig
  module type Y = module type of X2
end</code></pre></div>
<p>and the expansion of this would then clearly have both types <code>t</code> and <code>u</code> in it.</p>
<p>So now Odoc has two problems: We need to compute the correct signature, and we need to be able to describe how we computed it.</p>
<h2><a href="https://jon.recoil.org/atom.xml#the-solution" class="anchor"></a>The solution</h2>
<p>The previous solution to this was to have a ‘phase 0’ of odoc which would compute the expansions of all module type of expressions before doing any other work. This was necessary because of a ‘simplfying’ assumption in how we handled the typing environment. The new, simpler approach was to calculate the expansion during the normal flow of work, and never to attempt to recalculate it, but simply operate on the signature. This was a nice big simplification and optimisation that removed a few corner cases in the previous code (including an <a href="https://github.com/ocaml/odoc/blob/v2.4/src/xref2/type_of.ml#L167-L174">infinite loop</a> that we <em>hoped</em> always terminated…!)</p>
<p>The second issue was how to describe it. We still want it clear that this signature was derived from another, but it’s clear we can’t honestly say that in the above example that it’s <code>module type of X2</code>. The answer is that we have applied a transparent ascription to the signature. Essentially, the signature is <code>X2</code> but constrained to only have the fields of <code>X</code>.</p>
<p>This is not a current feature of OCaml, though Jane Street has <a href="https://blog.janestreet.com/plans-for-ocaml-408/">done some work</a> on this, including declaring the syntax: <code>X2 &lt;: X</code>. However, there’s another interesting wrinkle here. <code>X</code> is a module defined in the module type <code>S</code>, so it’s not possible to write a valid OCaml path that points to it – <code>S.X</code> has no meaning. In addition, the right-hand side of the <code>&lt;:</code> operator should be a module type, so we’d actually need to write <code>X2 &lt;: module type of S.X</code> . We’re still figuring out the right thing to do here, so for now Odoc 3 will still pretend that it’s simply <code>module type of X2</code>.</p>
