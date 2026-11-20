---
title: OCaml's module language is the perfect fit for agentic programming
description: Write interfaces first, statically type check the design, then hand each
  module to an agent one at a time. OCaml's module language makes this a natural workflow
  for future formal methods too.
url: https://anil.recoil.org/notes/ocaml-modules-agentic
date: 2026-10-09T00:00:00-00:00
preview_image: https://anil.recoil.org/images/rwo.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I've been doing a fair bit of <a href="https://anil.recoil.org/notes/cresting-the-ocaml-ai-hump">agentic OxCaml programming</a> in the past year, and consistently
finding that OCaml is a <a href="https://anil.recoil.org/notes/cresting-the-ocaml-ai-hump">perfect fit</a> across
the <a href="https://anil.recoil.org/notes/life-zarr-and-everything">spectrum of languages</a> I've been developing in recently.
If you're not familiar with OCaml, this post is a quick guide as to why I think this.</p>
<p>As codebases get larger, a coding agent is better at filling in a 'well-specified hole' than
cramming in a million-line codebase into its context window.
OCaml has a clean separation between <code>.mli</code> files (an interface definition for a module) and the <code>.ml</code> implementations.
I'll follow with with a toy project that has two modules, but the technique scales to million-line codebases.  We'll start by <a href="https://anil.recoil.org/news.xml#first-build-up-only-the-module-interfaces">writing only the interfaces</a>,
then <a href="https://anil.recoil.org/news.xml#building-a-test-client-for-this-interface">exercise them with a test client</a>,
then <a href="https://anil.recoil.org/news.xml#fill-in-the-module-implemenations-one-at-a-time">fill in the implementations</a>.
After that we get more exotic and <a href="https://anil.recoil.org/news.xml#refining-even-more-with-oxcaml-modes">refine with OxCaml modes</a>
and finish by <a href="https://anil.recoil.org/news.xml#going-deeper-down-the-refinment-rabbithole">speculating where formal proofs could go</a>.</p>
<h2><a href="https://anil.recoil.org/news.xml#first-build-up-only-the-module-interfaces" class="anchor" aria-hidden="true"></a>First build up only the module interfaces</h2>
<p>We have a module <code>Money</code> that tracks our cash (i.e it shoudl never be negative). If you need help with the syntax then the <a href="https://dev.realworldocaml.org/guided-tour.html">RWO guided tour</a> may be helpful.</p>
<pre><code class="language-ocaml">(* lib/money.mli *)

type t

val add : t -&gt; t -&gt; t

val sub : t -&gt; t -&gt; t option
(** [sub a b] is [None] if [b &gt; a]. *)

val to_string : t -&gt; string
(** [to_string m] is e.g. ["£3.05"]. *)
</code></pre>
<p>The <code>Book</code> module is a list of deposits and withdrawals.</p>
<pre><code class="language-ocaml">(* lib/book.mli *)

type t

val empty : t
val deposit : Money.t -&gt; t -&gt; t

val withdraw : Money.t -&gt; t -&gt; (t, [ `Insufficient of Money.t ]) result
(** [`Insufficient bal] carries the balance that was too small. *)

val balance : t -&gt; Money.t
</code></pre>
<p>Notice that we don't have any implementations yet, but that the
types we have defined can reference each other's module despite this.
This OCaml project can be made to compile via a dune directive
to suppress the need for an implementation as well.</p>
<pre><code>(library
 (name ledger)
 (modules_without_implementation money book))
</code></pre>
<p>Now the magic begins, as we can typecheck our project from just
this descrption of how they should work together.</p>
<pre><code>$ dune build @check
</code></pre>
<p>This interface compiles without warnings, so it's time to exercise our fledgling
interface with a test binary!</p>
<h2><a href="https://anil.recoil.org/news.xml#building-a-test-client-for-this-interface" class="anchor" aria-hidden="true"></a>Building a test client for this interface</h2>
<p>By writing a binary next, we can test if the interface we just defined is
sufficiently precise to actually use externally.</p>
<pre><code class="language-ocaml">(* bin/main.ml *)
open Ledger

let () =
  let ( let* ) = Option.bind in
  let r =
    let* ten = Money.of_pence 1000 in
    let* three = Money.of_pence 305 in
    let b = Book.deposit ten Book.empty in
    match Book.withdraw three b with
    | Ok b -&gt; Some (Money.to_string (Book.balance b))
    | Error _ -&gt; None
  in
  print_endline (Option.value r ~default:"failed")
</code></pre>
<p>The first realisation when we compile it is that our interface was
too abstract, so we have no way to make a <code>Money.t</code>! This makes the build
fail:</p>
<pre><code>$ dune build @check
File "bin/main.ml", line 5, characters 15-29:
5 |     let* ten = Money.of_pence 1000 in
                   ^^^^^^^^^^^^^^
Error: Unbound value "Money.of_pence"
</code></pre>
<p>We therefore edit our interface to add the constructor function:</p>
<pre><code class="language-ocaml">(* lib/money.mli *)
type t

val of_pence : int -&gt; t option
(** [of_pence n] is [None] if [n &lt; 0]. *)
</code></pre>
<p>Now the binary typechecks, and fails at linking time complaining that there's
no implementation. But because the type checker has passed it, we know that the
interface is good enough to be worth implementing!</p>
<pre><code>$ dune build ./bin/main.exe
Error: No implementations provided for the following modules:
         "Ledger__Money" referenced from bin/.main.eobjs/native/dune__exe__Main.cmx
         "Ledger__Book" referenced from bin/.main.eobjs/native/dune__exe__Main.cmx
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#fill-in-the-module-implemenations-one-at-a-time" class="anchor" aria-hidden="true"></a>Fill in the module implemenations one at a time</h2>
<p>We can now hand <code>Money</code> to the coding agent with an instruction to read the
interface (<code>.mli</code>) files, and start implementing the module implementations in
dependency order with tests per module (such as <a href="https://blog.janestreet.com/testing-with-expectations/">expect tests</a>, which keep the expected output next to the code).</p>
<pre><code class="language-ocaml">(* lib/money.ml *)
type t = int

let of_pence n = if n &lt; 0 then None else Some n
let add = ( + )
let sub a b = if b &gt; a then None else Some (a - b)
let to_string m = Printf.sprintf "£%d.%02d" (m / 100) (m mod 100)
</code></pre>
<p>The dune link error now only complains about <code>Ledger__Book</code>. The agent
then moves onto writing the <code>Book</code> module, with similar instructions
to only read the interface files.</p>
<p>This then brings up another problem with the interface, as writing <code>Book.empty</code> shows needs
a starting balance. At this point the agent uses its context-driven discretion to either
use <code>of_pence</code>, or add a helper function to <code>Money</code>:</p>
<pre><code>File "lib/book.ml", line 3, characters 12-22:
3 | let empty = Money.zero
                ^^^^^^^^^^
Error: Unbound value "Money.zero"
</code></pre>
<p>If the user (or agent goal) agrees to reassess the interface design, the Book interface gains a <code>zero</code> function.</p>
<h3><a href="https://anil.recoil.org/news.xml#stopping-agents-from-taking-shortcuts" class="anchor" aria-hidden="true"></a>Stopping agents from taking shortcuts</h3>
<p>A tempting shortcut for an agent in <code>Book</code> is to treat money as a plain integer and do
the operations directly within that implementation:</p>
<pre><code class="language-ocaml">(* lib/book.ml *)
type t = Money.t

let empty = Money.zero
let deposit m b = Money.add m b
let withdraw m b = if m &gt; b then Error (`Insufficient b) else Ok (b - m)
let balance b = b
</code></pre>
<p>This results in a type error in OCaml though:</p>
<pre><code>Error: The implementation "lib/book.ml"
       does not match the interface "lib/.ledger.objs/byte/ledger__Book.cmi":
       Values do not match:
         val withdraw : int -&gt; int -&gt; (int, [&gt; `Insufficient of int ]) result
       is not included in
         val withdraw : t -&gt; t -&gt; (t, [ `Insufficient of t ]) result
       Type "int" is not compatible with type "t"
</code></pre>
<p>The agent can't subtract pence directly, since the OCaml interfaces
enforce that only the <code>Money</code> module can perform this operation over a value
of that type.
Conveniently, the compiler rejects it with a message that's helpful enough for the agent
to write the correct implementation from the <code>Money</code> interface.</p>
<pre><code class="language-ocaml">(* lib/book.ml *)
type t = Money.t

let empty = Money.zero
let deposit m b = Money.add m b

let withdraw m b =
  match Money.sub b m with
  | Some b' -&gt; Ok b'
  | None -&gt; Error (`Insufficient b)

let balance b = b
</code></pre>
<p>This now fully builds end-to-end, yay!</p>
<pre><code>$ dune build ./bin/main.exe &amp;&amp; ./_build/default/bin/main.exe
£6.95
</code></pre>
<p>The great thing about this technique is that it scales to enormous
projects with hundreds of mli files, and this agentic workflows allows
for a cheap definition of a complex set of interfaces before embarking on
the expensive implementations. OCaml's separate compilation keeps build
times very fast so we have a quick edit/compile loop.</p>
<h2><a href="https://anil.recoil.org/news.xml#refining-even-more-with-oxcaml-modes" class="anchor" aria-hidden="true"></a>Refining even more with OxCaml modes</h2>
<p>We don't have to stop at just OCaml interfaces though! <a href="https://oxcaml.org">OxCaml</a> is a language
extension from Jane Street that provides <a href="https://doi.org/10.1145/3674642">mode annotations</a> that
can extend this workflow. (If you want to learn more, we ran an
<a href="https://anil.recoil.org/notes/icfp25-oxcaml">OxCaml tutorial</a> at ICFP 2025.)</p>
<p>We can now run an agentic pass to refine our interfaces to have even more checks:</p>
<pre><code class="language-ocaml">(* lib/money.mli *)
type t : immutable_data

val add : t @ local -&gt; t @ local -&gt; t
val to_string : t @ local -&gt; string
</code></pre>
<pre><code class="language-ocaml">(* lib/book.mli *)
type t : immutable_data
</code></pre>
<p>The <code>immutable_data</code> annotations promises that a <code>Money.t</code> or <code>Book.t</code> contains no mutable
state in its implementation, so it can (e.g.) be shared freely between parallel threads.</p>
<p>The <code>@ local</code> ensures that a function doesn't hold onto its argument, so the caller can pass in a stack-allocated
value and not have to have <a href="https://anil.recoil.org/notes/oxcaml-httpz">heap allocations</a>.</p>
<p>An agent that decides to "optimise" <code>Book</code> with a mutable balance now fails:</p>
<pre><code class="language-ocaml">type t = { mutable bal : Money.t }
</code></pre>
<pre><code>Error: The implementation "lib/book.ml"
       does not match the interface "lib/.ledger.objs/byte/ledger__Book.cmi":
       Type declarations do not match:
         type t = { mutable bal : Money.t; }
       is not included in
         type t : immutable_data
       The kind of the first is
           mutable_data with Money/2.t @@ forkable unyielding many
         because of the definition of t at file "lib/book.ml", line 1, characters 0-34.
       But the kind of the first must be a subkind of immutable_data.
       The first mode-crosses less than the second along:
         contention: mod uncontended ≰ mod contended
         visibility: mod read_write ≰ mod immutable
</code></pre>
<p>This error is admittedly a little opaque to a human user (something that's being
worked on in OxCaml), but it's fine for an agent with an <a href="https://github.com/avsm/ocaml-claude-marketplace/blob/main/plugins/ocaml-dev/skills/oxcaml/SKILL.md">OxCaml skill</a>.
(I run these agents in a <a href="https://anil.recoil.org/notes/ocaml-claude-dev">sandboxed devcontainer</a>.) Crucially, these mode annotations helped to stop an agent introducing a subtle error that may have corrupted data when used across multiple processsors.</p>
<h2><a href="https://anil.recoil.org/news.xml#going-deeper-down-the-refinment-rabbithole" class="anchor" aria-hidden="true"></a>Going deeper down the refinment rabbithole</h2>
<p>All the OxCaml modes earliy are statically defined by the compiler, which is getting increasingly capable. The <a href="https://people.mpi-sws.org/~bpeters/papers/mode-crossing.pdf">ICFP 2026 mode crossings</a> paper this summer shows how the compiler automatically strengthens modes for values of certain types, which is how our <code>Money.t</code> can declare <code>immutable_data</code> succinctly. But wouldn't it be cool if we could also express <a href="https://x.com/JulesJacobs5/status/2108144426640175371">arbitrary logical conditions</a> in the interfaces?!</p>
<p>Here's a sketch of <code>Money</code> using a <a href="https://github.com/ocaml-gospel/gospel">Gospel-style specification</a>. I've not actually compiled this one, but you'll get the idea:</p>
<pre><code class="language-ocaml">(* lib/money.mli *)
type t
(*@ model pence : integer
    invariant pence &gt;= 0 *)

val sub : t -&gt; t -&gt; t option
(*@ r = sub a b
    ensures match r with
            | None -&gt; a.pence &lt; b.pence
            | Some c -&gt; c.pence = a.pence - b.pence *)
</code></pre>
<p>The comment is now a machine-checked contract, so every implementation must
guarantee it satisfies these pre- and post-conditions.</p>
<p>About two decades ago, Patrick Rondon and Ranjit Jhala worked on a <a href="https://github.com/ucsd-progsys/dsolve">liquid OCaml</a> that had these features (<a href="https://doi.org/10.1145/1379022.1375602">PLDI 2008 paper</a>). I'm really excited that it's <a href="https://x.com/JulesJacobs5/status/2108144426640175371">heading</a> back into modern OxCaml as I've been jealous of <a href="https://ucsd-progsys.github.io/liquidhaskell/">Liquid Haskell</a> for a long time :-)</p>
<p>The beautiful thing about using OCaml's module system as a basis for these formal extensions is that separate compilation architecture I sketched above means that the edit/feedback loop is fast even on million-line codebases. The layering of annotations also lets us make code progressively more specified without piling on huge numbers of unit tests.</p>
<p>This is context efficient for agents <em>and</em> preserves human sanity as code gets more complex. We're also only beginning to investigate how to visualise such constraints in our user interfaces, like the work ongoing in <a href="https://hazel.org">Hazel</a> and our own work on <a href="https://anil.recoil.org/papers/2026-bidirectional-type-slicing">bidirectional type slicing</a> to debug type errors (also this last paper just got conditionally accepted into POPL 2027, which I'm super excited about and will wrote more on later!)</p><h1>References</h1><ul><li>Madhavapeddy (2025). Cresting the OCaml AI humps. <a href="https://doi.org/10.59350/nn1d6-xgt62" target="_blank"><i>10.59350/nn1d6-xgt62</i></a></li>
<li>Madhavapeddy (2025). Holding an OxCaml tutorial at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/55bc5-x4p75" target="_blank"><i>10.59350/55bc5-x4p75</i></a></li>
<li>Carroll et al (2026). Bidirectional Type Slicing. arXiv. <a href="https://doi.org/10.48550/arXiv.2607.12197" target="_blank"><i>10.48550/arXiv.2607.12197</i></a></li>
<li>Lorenzen et al (2024). Oxidizing OCaml with Modal Memory Management. <a href="https://doi.org/10.1145/3674642" target="_blank"><i>10.1145/3674642</i></a></li>
<li>Rondon et al (2008). Liquid types. <a href="https://doi.org/10.1145/1379022.1375602" target="_blank"><i>10.1145/1379022.1375602</i></a></li></ul>
