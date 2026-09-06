---
title: Weeks 24-27
description:
url: https://jon.recoil.org/blog/2025/07/week27.html
date: 2025-07-07T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#weeks-24-27" class="anchor"></a>Weeks 24-27</h1>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-07-07</p></li></ul>
<ul class="at-tags"><li class="page-tags"><span class="at-tag">page-tags</span> <p><a href="https://jon.recoil.org/tags/weeknotes.html" title="weeknotes">weeknotes</a> <a href="https://jon.recoil.org/tags/odoc.html" title="odoc">odoc</a> <a href="https://jon.recoil.org/tags/ai.html" title="ai">ai</a></p></li></ul>
<p>It's been a busy few weeks. There's been exam marking for the 1A Foundations of Computer Science course, an Odoc release to plan, and some really interesting new work on using LLMs to summarise OCaml documentation. This post is about anaspect of that last one that I found particularly interesting.</p>
<h2><a href="https://jon.recoil.org/atom.xml#odoc-llm" class="anchor"></a>odoc-llm</h2>
<p>Sadiq and I have been <a href="https://toao.com/blog/ocaml-local-code-models">looking at using LLMs</a> to summarise the documentation produced by Odoc. The idea is to get a concise summary of the purpose of each module, so that we can quickly identify which modules are relevant to a particular task.</p>
<p>For testing this, we need to see how it works on different types of libraries. The first axis I wanted to test on goes between 'well documented' and 'poorly documented', and so I need at least two libraries on opposite ends of the spectrum.</p>
<p>For the 'well documented' case, I chose <code>cmdliner</code>. It's a library that I almost always have to look at the docs for when I'm using it, as, despite using it many many times, the interface doesn't seem to stick in my head.</p>
<p>For the 'poorly documented' case, I chose <code>odoc</code> itself, somewhat ironically. In defence of the odoc authors (me included!), the libraries it provides are simply there for code organisation and aren't meant to be consumed outside of the tool itself!</p>
<h3><a href="https://jon.recoil.org/atom.xml#the-approach" class="anchor"></a>The approach</h3>
<p>The output from Odoc is a set of HTML files, one per module/module type/class/etc., containing the documentation for that item. Our first take on this was to parse the HTML files and extract the text content, which we then fed to an LLM to summarise. However, this was pretty awkward, so we decided to try a PR that <a href="https://github.com/ocaml/odoc/pull/1341">davesnx recently made to Odoc</a> to output markdown instead of HTML.</p>
<p>We look for leaf modules that don't contain any submodules, and start by summarising those, then move onto the parent modules, splicing in the summaries of the children, and so on, up to the top-level modules. We then move on to summarising the whole library, which usually is just a single namespace module, but occasionally is a group of top-level modules.</p>
<p>One of the early prompts for the module <code>Cmdliner.Term.Syntax</code> looked roughly as follows:</p>
<pre>You are an expert OCaml developer. Write a 2-3 sentence description focusing on:
- The specific operations and functions this module provides
- What data types or structures it works with
- Concrete use cases (avoid generic terms like "utility functions" or "common operations")

Do NOT:
- Repeat the module name in the description
- Mention "functional programming patterns" or "code clarity"
- Use filler phrases like "provides functionality for" or "collection of functions"
- Describe how it works with other modules

Module: Syntax
Module Documentation: let operators.
( let+ ) is map.
( and* ) is product.
- val (let+) : 'a t -&gt; ('a -&gt; 'b) -&gt; 'b t (* ( let+ ) is map. *)
- val (and+) : 'a t -&gt;
  'b t -&gt;
  ('a * 'b) t (* ( and* ) is product. *)</pre>
<p>and the output using a small model (qwen3-30b-a3b) was:</p>
<pre>"The module provides (let+) for applying functions to values within a context and (and+) for combining two contexts into a product. It operates on applicative
structures like option, list, or custom types that support these operations. For example, it enables sequential transformation of values in a context or
pairing elements from two separate contexts."</pre>
<p>There are quite a few issues with the input here. Firstly, we've only given the module name, not the full path. Secondly, there's nothing to let the model know what <code>t</code> might be, so it has decided it's a completely generic type. It also has no idea about the context in which this module was found, so it has no idea that it's part of a command-line processing library. By fixing these issues, we end up with the prompt:</p>
<pre>You are an expert OCaml developer. Write a 2-3 sentence description focusing on:
- The specific operations and functions this module provides
- What data types or structures it works with
- Concrete use cases (avoid generic terms like "utility functions" or "common operations")

Do NOT:
- Repeat the module name in the description
- Mention "functional programming patterns" or "code clarity"
- Use filler phrases like "provides functionality for" or "collection of functions"
- Describe how it works with other modules

Module: Cmdliner.Term.Syntax

Ancestor Module Context:
- Cmdliner: Declarative definition of command line interfaces.
Consult the tutorial, details about the supported command line syntax and examples of use.
Open the module to use it, it defines only three modules in your scope.
- Cmdliner.Term: Terms.
A term is evaluated by a program to produce a result, which can be turned into an exit status. A term made of terms referring to command line arguments implicitly defines a command line syntax.

Module Documentation: let operators.
- val (let+) : 'a Cmdliner.Term.t -&gt; ('a -&gt; 'b) -&gt; 'b Cmdliner.Term.t (* ( let+ ) is map. *)
- val (and+) : 'a Cmdliner.Term.t -&gt;
  'b Cmdliner.Term.t -&gt;
  ('a * 'b) Cmdliner.Term.t (* ( and* ) is product. *)</pre>
<p>The output of this improved prompt is much better:</p>
<pre>The module provides operators to map and combine terms, which represent command line argument parsers and their results. `let+` transforms a parsed argument
into a new value, while `and+` merges two independent arguments into a tuple. These enable building structured command line interfaces, such as parsing a
filename and a flag simultaneously, then combining them into a configuration record.</pre>
<p>It still occasionally incorrectly decides that this module provides monadic combinators rather than applicative, but this is where we get better results from using a larger model.</p>
<p>There are quite a few other issues that we've fixed - for example, treating module types differently than modules, and a bug where infix operators were being omitted from the documentation. In one case, it uncovered a bug in the markdown generator where includes weren't getting expanded, which I got <a href="https://github.com/jonludlam/odoc/commit/926cca100c307818e57281c3d40e98f1975f0f95">Claude to fix</a>. My <i>modus operandi</i> has essentially been to look at the output for the test packages, find a summary that looks bonkers, and then look back at the prompt to find that, indeed, the input was missing some crucial information.</p>
<p>One thing I'd quite like to try is to re-open a <a href="https://github.com/ocaml/odoc/pull/655">PR that Drup made</a> as an April Fool's joke back in 2001, which ended up outputting OCaml formatted code. This is actually pretty close to what we might want to give to the LLM - though we'd probably format the comments as markdown, and we'd be replacing types with fully-qualified types as above. Funny how things turn out!</p>
