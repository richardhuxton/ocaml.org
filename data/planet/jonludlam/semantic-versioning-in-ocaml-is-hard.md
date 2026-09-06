---
title: Semantic Versioning in OCaml is Hard
description:
url: https://jon.recoil.org/blog/2025/04/semantic-versioning-is-hard.html
date: 2025-04-20T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#semantic-versioning-in-ocaml-is-hard" class="anchor"></a>Semantic Versioning in OCaml is Hard</h1>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-04-20</p></li></ul>
<ul class="at-tags"><li class="page-tags"><span class="at-tag">page-tags</span> <p><a href="https://jon.recoil.org/tags/ocaml.html" title="ocaml">ocaml</a></p></li></ul>
<p><a href="https://semver.org">Semantic versioning</a> is a lovely and simple idea that, if it were reliably implemented everywhere, would make life a lot simpler. So, is it possible to make our OCaml libraries stick to this scheme? There are some projects that are trying to do this, including a recent <a href="https://www.outreachy.org">Outreachy</a> project by <a href="https://github.com/azzsal/">Abdulaziz Alkurd</a> mentored by <a href="https://choum.net">panglesd</a> and <a href="https://github.com/nathanreb">Nathan Reb</a>. While this is a great start, there are some subtleties of the OCaml module system that make it a good deal more complex than in other languages.</p>
<h2><a href="https://jon.recoil.org/atom.xml#opam-format.2.3.0-%E2%89%A0-opam-format.2.3.0?" class="anchor"></a>opam-format.2.3.0 ≠ opam-format.2.3.0?</h2>
<p>Let's take the case that hit me this morning. I've been working on <a href="https://github.com/ocurrent/ocaml-docs-ci">ocaml-docs-ci</a> in order to bring the exciting new <a href="https://ocaml.github.io/odoc">odoc 3</a> features to <a href="https://ocaml.org/">ocaml.org</a> for everyone to enjoy. I have it checked out and building locally, but to deploy it to the infrastructure managed by <a href="https://tunbury.org/">Mark Elvers</a> it needs to be packaged up into a Docker image. So I issued the usual <code>docker build .</code> and after it churned through the setup stages and got on to building the project, it hit an error:</p>
<pre>File "src/solver/solver.ml", line 58, characters 75-98:
     let deps = List.map (fun pkg -&gt; OpamPackage.Map.find pkg simple_deps) (OpamPackage.Set.to_list pkgs) in
Error: Unbound value OpamPackage.Set.to_list
Hint: Did you mean of_list?</pre>
<p>Now <code>OpamPackage</code> is a module in the <code>opam-format</code> library, which is easily discovered using the excellent <a href="https://doc.sherlocode.com/?q=OpamPackage">Sherlodoc</a> tool, so I checked what version I had locally, and what version I had in the Docker container, and it turned out I was using exactly the same version -- 2.3.0 -- both locally and in the container. So what's going on?</p>
<p>The problem is that the Dockerfile I was using was using OCaml version 4.14, whereas locally I was using OCaml 5.3. "But how on earth can this cause the API of <code>opam-format</code> to change?" I hear you wail! Well, this is actually one of the simpler outcomes of the way the OCaml module system works. Let's look at <a href="https://github.com/ocaml/opam/blob/2.3.0/src/format/opamPackage.mli">the code</a>.</p>
<p>The first thing we note is the absence of any definition of <code>Set</code> or <code>Map</code> here</p>
<ul><li>where do they come from? It turns out they come from <a href="https://github.com/ocaml/opam/blob/2.3.0/src/format/opamPackage.mli#L49">this line here</a>:</li></ul>
<div><pre class="language-ocaml"><code>include OpamStd.ABSTRACT with type t := t</code></pre></div>
<p>So let's take a look over in <code>opamStd.mli</code> to see what that signature looks like:</p>
<div><pre class="language-ocaml"><code>(** A signature for handling abstract keys and collections thereof *)
module type ABSTRACT = sig

  type t

  val compare: t -&gt; t -&gt; int
  val equal: t -&gt; t -&gt; bool
  val of_string: string -&gt; t
  val to_string: t -&gt; string
  val to_json: t OpamJson.encoder
  val of_json: t OpamJson.decoder

  module Set: SET with type elt = t
  module Map: MAP with type key = t
end</code></pre></div>
<p>OK, so we've found the definitions of <code>Set</code> and <code>Map</code> - they refer to signatures <code>SET</code> and <code>MAP</code> which are defined just above in <a href="https://github.com/ocaml/opam/blob/2.3.0/src/core/opamStd.mli#L17-L98">opamStd.mli</a>. Let's just look at <code>Set</code> since that's where the problem was:</p>
<div><pre class="language-ocaml"><code>module type SET = sig

  include Set.S

  val map: (elt -&gt; elt) -&gt; t -&gt; t

  val is_singleton: t -&gt; bool

  (** Returns one element, assuming the set is a singleton. Raises [Not_found]
      on an empty set, [Failure] on a non-singleton. *)
  val choose_one : t -&gt; elt

  val choose_opt: t -&gt; elt option

  val of_list: elt list -&gt; t
  val to_list_map: (elt -&gt; 'b) -&gt; t -&gt; 'b list
  val to_string: t -&gt; string
  val to_json: t OpamJson.encoder
  val of_json: t OpamJson.decoder
  val find: (elt -&gt; bool) -&gt; t -&gt; elt
  val find_opt: (elt -&gt; bool) -&gt; t -&gt; elt option

  (** Raises Failure in case the element is already present *)
  val safe_add: elt -&gt; t -&gt; t

  (** Accumulates the resulting sets of a function of elements until a fixpoint
      is reached *)
  val fixpoint: (elt -&gt; t) -&gt; t -&gt; t

  (** [map_reduce f op t] applies [f] to every element of [t] and combines the
      results using associative operator [op]. Raises [Invalid_argument] on an
      empty set, or returns [default] if it is defined. *)
  val map_reduce: ?default:'a -&gt; (elt -&gt; 'a) -&gt; ('a -&gt; 'a -&gt; 'a) -&gt; t -&gt; 'a

  module Op : sig
    val (++): t -&gt; t -&gt; t (** Infix set union *)

    val (--): t -&gt; t -&gt; t (** Infix set difference *)

    val (%%): t -&gt; t -&gt; t (** Infix set intersection *)
  end

end</code></pre></div>
<p>Sure enough, there's no <code>to_list</code> function defined in there. Once again though, there's an <code>include Set.S</code> in there. It turns out that that refers to the <code>Set</code> module in the OCaml standard library. We can again <a href="https://github.com/ocaml/ocaml/blob/5.3.0/stdlib/set.mli">look at the source</a>:</p>
<div><pre class="language-ocaml"><code>val to_list : t -&gt; elt list
    (** [to_list s] is {!elements}[ s].
        @since 5.1 *)</code></pre></div>
<p>And there it is. The <code>to_list</code> function has only been in the <code>Set</code> module since version 5.1.</p>
<h2><a href="https://jon.recoil.org/atom.xml#using-ocaml.org-docs" class="anchor"></a>Using ocaml.org docs</h2>
<p>It was pretty difficult to figure that out from the source, but happily there's a better way. We can browse the docs on https://ocaml.org/ - We can look at the docs for the <a href="https://ocaml.org/p/opam-format/2.3.0/doc/OpamPackage/Set/index.html">OpamPackage.Set module</a> which, as of today, does not contain any <code>to_list</code> function. The <code>include Set.S</code> is there with the expansion showing all of the types and values coming from it, so we can click on the <code>Set.S</code> link on the include line which takes us to the documentation for the stdlib from OCaml 4.11.2. Changing the version from the dropdown at the top to something more recent takes us to a page containing the <code>to_list</code> function with the helpful <code>since 5.1</code> annotation.</p>
<p>This is, in fact, a relatively simple example of the sorts of issues that can occur that make semantic versioning a headache. In this example, it was a change due to a difference in the compiler version used, but there's nothing particularly special about that - a package may expose signatures derived from any of its dependencies! So is there anything we can do about this? Obviously, yes!</p>
<h2><a href="https://jon.recoil.org/atom.xml#towards-a-solution" class="anchor"></a>Towards a solution</h2>
<p>Step 1 of any approach to solving this is to be able to identify which bits of a libraries API come from which packages, and therefore which versions of those packages. It turns out there may well be a nice way to piggy-back on a recent feature from Odoc, which was originally intended to help with suppressing suprious warnings.</p>
<p>The problem we were tackling was that if your library ends up exporting a module whose signature is defined in someone else's package, then any warnings that come from it are unfixable. To fix this we added a tag to each signature of a module that indicates which package it originally came from. Odoc is then very careful to keep track of this as it performs its signature manipulations, resulting in an accurate way to know which signature elements came from which package. This fixed the problem of the spurious warnings nicely.</p>
<p>Quite separately, we've got the docs CI that is attempting to build docs for every version of every package. Obviously given the above, in order to exhaustively show all the possible APIs of every library, we should build all possible combinations of every version of every package. Clearly we can't possibly do this, so the docs CI focuses on the goal of building at least one solution for every version of every package.</p>
<p>Now if you combine these two ideas, we can use the builds of the packages with the tracking of the package of the originating signatures to be able to precisely track the differences in API between different versions of a package. This would allow us to build a database of those changes, and with this in hand we can look at what APIs are used in any other package and be able to suggest upper and lower bounds on the versions of its dependencies.</p>
<p>Now wouldn't that be cool?</p>
