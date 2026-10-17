---
title: Language integrated LLMs as an OCaml function
description: Using a local DeepSeek model as an ordinary OCaml library and building
  sandboxed agents from simple primitives
url: https://anil.recoil.org/notes/language-integrated-llms
date: 2026-06-14T00:00:00-00:00
preview_image: https://anil.recoil.org/images/humpty-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Fable <a href="https://www.theverge.com/ai-artificial-intelligence/949553/anthropic-fable-5-mythos-5-government-national-security">cut out on me</a> at 1am on Saturday while I was sweeping over the OCaml runtime looking for concurrency bugs. There have been <a href="https://x.com/rosstaylor90/status/2066067747738431504">excellent takes</a> on the sovereignity implications of this, and I figured I'd roll my sleeves up and get serious about using the open weights models. DeepSeek's models have been getting more capable since their <a href="https://anil.recoil.org/notes/deepseek-r1-advances">first release</a>, and <a href="https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash">v4 Flash</a> is small enough to run on my Mac (admittedly, very high-end Macs with 128GB/512GB of RAM respectively for my laptop and desktop).</p>
<p>The question is whether the agentic CLIs I've <a href="https://anil.recoil.org/notes/aoah-2025">been using</a> can be easily replaced.  The best way to learn how a system works is to build it <a href="https://anil.recoil.org/projects/unikernels">unikernel style</a>, and so I aimed to expose the LLM as a normal OCaml library.
This avoids routing via <a href="https://github.com/anthropics/claude-code/issues/8382">bloated CLIs</a>, and
lets the linking application drive the agentic loop according to its specific needs.</p>
<p>What makes this practical is <a href="https://github.com/antirez">Antirez</a>'
<a href="https://github.com/antirez/ds4">Dwarfstar</a>, a self-contained
native inference engine that supports <a href="https://developer.apple.com/metal/">Apple Metal</a> and portable(ish) C.
I bound this directly to OCaml 5 and <a href="https://github.com/ocaml-multicore/eio">Eio</a> as
<strong><a href="https://tangled.org/anil.recoil.org/ocaml-deepseek">ocaml-deepseek</a></strong>, and now a
plain function call on my laptop gets me an LLM in my application.</p>
<p><img src="https://anil.recoil.org/images/humpty-ss-1.webp" alt="%c" title="The Humpty OCaml deepseek agent in full (local) poetic flow"></p>
<p>For example, I can now embed Deepseek inference directly into the OCaml webserver that drives this very
site in order to look for suspicious bot activity, and because it's open
weights and running locally, there's no dependency on external services!</p>
<pre><code class="language-ocaml">(* A traffic-triage agent in-process in OCaml. The agent is handed two
   OCaml function tools and works out for itself how to combine them. *)
let agent =
  Agent.create engine ~system:"You are a web-traffic analyst."
    ~tools:[
      Toolbox.read ~dir:logs;   (* read-only sandboxed handle to the logs dir *)
      query_db ~conn;           (* a SELECT-only tool over the local database *)
    ]
in
Agent.send agent ~on_event
  "Cross-reference today's 404 spikes in the access log against the \
   client IPs in the requests table. Anything coordinated indicating a bad bot?"
</code></pre>
<p>The log reader and the database query are just two OCaml functions the model is
allowed to call, each scoped and sandboxed (using Eio) to exactly what it needs. The model decides when and
how to combine them.</p>
<h2><a href="https://anil.recoil.org/news.xml#trying-out-humpty-the-ocaml-agent" class="anchor" aria-hidden="true"></a>Trying out Humpty the OCaml agent</h2>
<p>I've <a href="https://github.com/ocaml/opam-repository/pull/30053">submitted</a> the package to opam, so <code>opam install deepseek</code> or <code>opam pin add deepseek https://tangled.org/anil.recoil.org/ocaml-deepseek.git</code> should work.
The package also ships a binary called humpty<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> with two variants: <code>humpty-metal</code> for Apple Silicon and a portable <code>humpty-cpu</code> that should run anywhere (slowly).</p>
<p>There are four subcommands that we'll use to explain how to build an agent up in OCaml:
first <a href="https://anil.recoil.org/news.xml#choose-the-right-deepseek-model"><code>list</code></a> the available models and <a href="https://anil.recoil.org/news.xml#grab-the-deepseek-model-weights"><code>download</code></a> one, then <a href="https://anil.recoil.org/news.xml#an-llm-is-a-stateless-request-reply-function"><code>chat</code></a> with it statelessly, and then wrap that into an <a href="https://anil.recoil.org/news.xml#adding-state-to-make-an-agentic-ocaml-library"><code>agent</code></a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#choose-the-right-deepseek-model" class="anchor" aria-hidden="true"></a>Choose the right Deepseek model</h3>
<p>Before we can get started you'll first need the <a href="https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash">open model weights</a> downloaded.
<a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/bin/humpty.ml#L101-134"><code>humpty list</code></a>
prints a the catalogue of available weights:</p>
<pre><code class="language-bash">$ humpty-metal list
Models (download dir: /Users/avsm/.local/share/ds4)

      TARGET                 ALIASES   DESCRIPTION
  [ ] q2-imatrix             q2        2-bit Flash routed experts (~81 GB); for 96-128 GB RAM.
  [*] q2-q4-imatrix          q2q4      Mixed Flash quant (~98 GB); higher quality for 128 GB.
  [ ] q4-imatrix             q4        4-bit Flash routed experts (~153 GB); for 256 GB+ RAM.
  [ ] pro-q2-imatrix         pro-q2    PRO q2 single file (~430 GB); for 512 GB RAM.

[*] = present, [ ] = not downloaded
</code></pre>
<p>Pick one based on how much RAM you have; I use <code>q2q4</code> on my laptop (with 128GB RAM),
and the extremely beefy <code>pro-q2</code> on my Mac Studio (with 512GB RAM).
There are also split files for running the model distributed across several machines, which I'll
skip here for now.</p>
<h3><a href="https://anil.recoil.org/news.xml#grab-the-deepseek-model-weights" class="anchor" aria-hidden="true"></a>Grab the Deepseek model weights</h3>
<p>Once you've chosen, <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/bin/humpty.ml#L66-97"><code>humpty download q4</code></a>
(or <code>pro-q2</code>, or whichever) shells out to the Hugging Face CLI to fetch the GGUF.
You'll either need the <a href="https://huggingface.co/docs/huggingface_hub/en/guides/cli">Huggingface CLI installed</a> or have <a href="https://github.com/astral-sh/uv">uvx</a> in your path.</p>
<p>Once this gets doing go have a cup of tea while the gigabytes of LLM weights
download, and then we'll start to build an agent from the camel up!</p>
<h2><a href="https://anil.recoil.org/news.xml#building-an-agent-from-the-ground-up" class="anchor" aria-hidden="true"></a>Building an agent from the ground up</h2>
<p>I first want to pin down what an "agent" actually means, as the term seems to have
accreted much mystique this year. The whole OCaml Deepseek stack is a small library you
can read through quickly, so let me build an agent up from scratch.
The code below links to the <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/commit/1edcdad29a19924f988c7adef4343cb189dcb4a2">Tangled source</a>.</p>
<h3><a href="https://anil.recoil.org/news.xml#an-llm-is-a-stateless-request-reply-function" class="anchor" aria-hidden="true"></a>An LLM is a stateless request-reply function</h3>
<p>A basic LLM takes in a text prompt, performs inference on some weights, and generates a text reply back.
To illustrate this in our OCaml code, we need to load the model weights and spin up an
<code>engine</code> with a cache directory for the compiled Metal kernels (if using the
Apple GPU version):</p>
<pre><code class="language-ocaml">let engine = Deepseek.V4.create ~cache ~model ~domain_mgr ~sw () in
V4.generate engine "Explain monads in one sentence." ~on_token:print_string;
- : unit

=&gt; Monads are a design pattern that allows you to chain operations together
   while automatically handling extra behavior like error handling, state, or
   side effects, by wrapping values in a context and providing a way to
   transform and combine them.
</code></pre>
<p><a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/v4.mli#L41-47"><code>Deepseek.V4.create</code></a>
opens the <a href="https://huggingface.co/docs/hub/gguf">GGUF</a> model file and, the first
time it runs, materialises the embedded Metal shaders in the <code>cache</code>. Generating a
reply is then a single call to <code>V4.generate</code> that
encodes the supplied prompt, runs a prefill, and samples one token at a time into the <code>on_token</code> callback until the
end-of-sequence marker.
All the inference is done in the Metal library in a separate OCaml domain,
so we can continue to use other Eio fibres in our main application.</p>
<p>You can try this single-shot request/response using <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/bin/humpty.ml#L44-62"><code>humpty chat</code></a>, which keeps no memory between runs and can't take any action beyond showing the reply.</p>
<pre><code class="language-bash">$ humpty-metal chat 'Explain algebraic effects in OCaml 5 in one sentence'

  Algebraic effects in OCaml 5 allow functions to suspend execution and invoke
  user-defined handlers for operations (like state, exceptions, or generators)
  via a lightweight, type-safe mechanism that integrates with the language's
  existing type system and is used primarily for effectful programming, such as
  with the new `Effect` library for handling delimited continuations.
</code></pre>
<p>A "conversation" is therefore just a list of role-tagged messages (e.g. system,
user, assistant, tool) that we concatenate in our library into a prompt string
for the LLM.</p>
<h2><a href="https://anil.recoil.org/news.xml#how-the-stateless-llm-asks-for-effects-to-the-external-world" class="anchor" aria-hidden="true"></a>How the stateless LLM asks for effects to the external world</h2>
<p>The single-step text-to-text function from earlier emits text in an agreed "shape" so we can figure out what to do next based on its output.
DeepSeek has trained their model to understand a little markup language called <a href="https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/blob/main/encoding/encoding_dsv4.py">DSML</a>, which looks something like this:</p>
<pre><code class="language-xml">&lt;｜DSML｜tool_calls&gt;
&lt;｜DSML｜invoke name="edit"&gt;
&lt;｜DSML｜parameter name="path" string="true"&gt;/tmp/x.c&lt;/｜DSML｜parameter&gt;
&lt;｜DSML｜parameter name="line" string="false"&gt;42&lt;/｜DSML｜parameter&gt;
&lt;/｜DSML｜invoke&gt;
&lt;/｜DSML｜tool_calls&gt;
</code></pre>
<p>DSML is a pseudo-XML language that's interspersed in the text responses from the agent.
Those bars are actually the full-width vertical line <code>｜</code> (U+FF5C) and not an ASCII <code>|</code>. DeepSeek reserves the rarer codepoint for DSML's control tokens, so they can't be produced by ordinary text or code the model emits.</p>
<p>We've got a <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/main/dsml/lib/dsml.mli">DSML implementation in OCaml</a>, which
<a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/dsml/lib/dsml.mli#L83-98">parses the XML into an OCaml record type</a>:</p>
<pre><code>type thinking_mode = Chat | Thinking

type reasoning_effort = High | Max

type task = Action | Query | Authority | Domain | Title | Read_url

type tool_call = { id : string option; name : string; arguments : string }

type parsed_message = {
  content : string;
  reasoning_content : string;
  tool_calls : tool_call list;
}
</code></pre>
<p>Parsing a raw text reply splits it into the visible content the user will see,
the model's hidden <a href="https://en.wikipedia.org/wiki/Reasoning_model">reasoning content</a>, and any tool calls it
emitted along the way. A <code>tool_call</code> is just a string name plus its arguments
as a JSON string, with an optional identifier to pair the result back up.</p>
<p>The other three types are knobs on <em>how</em> the model replies rather than what it
returns:</p>
<ul>
<li>a <code>thinking_mode</code> of <code>Chat</code> answers directly while <code>Thinking</code> reasons in a <code>&lt;think&gt;</code> block first</li>
<li><code>reasoning_effort</code> turns that reasoning up to maximum at the cost of slower inference time</li>
<li><code>task</code> is a quick-instruction hint into DeepSeek's internal pipeline as to whether this turn is an <code>Action</code>, a <code>Query</code>, etc.</li>
</ul>
<h3><a href="https://anil.recoil.org/news.xml#making-custom-tool-functions-in-ocaml" class="anchor" aria-hidden="true"></a>Making custom tool functions in OCaml</h3>
<p>We now need to define specific tools that the model knows about. We do this by defining a bidirectional codec that decodes the model's JSON/DSML arguments into a typed OCaml value, and renders that value back. Here's a simple <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/tool.mli#L11-20">touch</a> tool:</p>
<pre><code>let touch =
  let open Dsml.Codec in
  Invoke.map "touch" (fun path -&gt; path)
  |&gt; Invoke.param ~enc:Fun.id "path" string ~description:"file to create"
  |&gt; Invoke.seal
in
Tool.v ~description:"Create an empty file." touch
  (fun path -&gt;
    Out_channel.with_open_text path ignore;
    "created " ^ path)
</code></pre>
<p>We just wrap the effect we are doing (in this case, just writing an empty file)
with the JSON metadata to let the model know <em>when</em> and <em>how</em> to invoke the
tool as it works its way through the prompt. Unlike most policy languages, we
describe these in plain text since we're dealing with an LLM, and it decides
when the description of the tool should be applied in the conversation.</p>
<p>There's still no state here, but we can now use the DSML library to build up a full
prompt string by keeping track of all the messages:</p>
<pre><code class="language-ocaml">val encode_messages :
  ?context:message list -&gt;
  ?drop_thinking:bool -&gt;
  ?add_default_bos_token:bool -&gt;
  ?reasoning_effort:reasoning_effort -&gt;
  thinking_mode -&gt;
  message list -&gt;
  string
(** [encode_messages mode messages] renders the conversation to the prompt
    string. [context] prepends an already-encoded prefix and suppresses the
    leading token; [drop_thinking] (default true) drops reasoning from turns
    before the last user message; [reasoning_effort] [Max] maximises reasoning.
*)
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#adding-state-to-make-an-agentic-ocaml-library" class="anchor" aria-hidden="true"></a>Adding state to make an agentic OCaml library</h2>
<p>We're now ready to add state to this!  An "agent" is just a wrapper around the LLM that does three things:</p>
<ul>
<li>remember the conversation so far for <code>encode_messages</code></li>
<li>keep the engine's KV-cache warm via <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/v4.mli#L95-108">a persistent session</a></li>
<li>run the tool callbacks as they arrive from the LLM</li>
</ul>
<p>The loop is expressed as a simple <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/agent.mli#L27-33">OCaml event datatype</a>:</p>
<pre><code class="language-ocaml">type event =
  | Reasoning of string          (* the model's &lt;think&gt; text *)
  | Content of string            (* a chunk of the reply *)
  | Tool_call of Dsml.tool_call
  | Tool_result of string * string
  | Done

val send : t -&gt; on_event:(event -&gt; unit) -&gt; string -&gt; unit
</code></pre>
<p>When the LLM responds each turn, plain text means the turn is <code>Done</code>. Tool
calls get looked up by their name, then run, and the results folded back into
the conversation as <code>tool</code> messages.  All the agent function does is just run
this to a fixed point until the model answers with text alone.</p>
<h3><a href="https://anil.recoil.org/news.xml#writing-custom-tools-using-os-sandboxing-and-eio-capabilities" class="anchor" aria-hidden="true"></a>Writing custom tools using OS sandboxing and Eio capabilities</h3>
<p>Here's where the unikernel-style magic shows up though. Given that a tool is
just something we define ourselves, then we can start to take advantage of
OCaml itself!  And in particular, I want better security and more fine-grained
tool calls that are tailored to my applications and not generic shell scripts
that are difficult to sandbox.</p>
<p><a href="https://anil.recoil.org/papers/2023-ocaml-eio">Eio is a library</a> built over <a href="https://anil.recoil.org/papers/2021-pldi-retroeff">OCaml 5's effects</a> that
follows an <a href="https://en.wikipedia.org/wiki/Object-capability_model">object-capability</a>
discipline to eliminate ambient authority where possible.
In our <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/toolbox.mli#L4-22">Toolbox</a> module,
we define some example Eio tools:</p>
<pre><code class="language-ocaml">val read  : dir:_ Eio.Path.t -&gt; Tool.t
val write : dir:_ Eio.Path.t -&gt; Tool.t
val dns   : net:_ Eio.Net.t -&gt; Tool.t
val bash  : proc:_ Eio.Process.mgr -&gt; Tool.t
</code></pre>
<p>Notice that each OCaml signature here takes in a capability for that particular
tool:</p>
<ul>
<li><code>read</code> and <code>write</code> can only access the directory you pass as <code>~dir</code> and nothing above it, since Eio uses <em><a href="https://linux.die.net/man/2/openat">openat(2)</a></em> to sandbox the call.</li>
<li><code>dns</code> reaches the network only because it holds a net capability</li>
<li><code>bash</code> spawns processes only because it holds a <a href="https://github.com/ocaml-multicore/eio/blob/main/README.md#running-processes">process manager</a>.</li>
</ul>
<p>When using these from applications, we can select the exact sandboxing we need, or just write our own tool functions with application-specific logic. This is exactly what <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/bin/humpty.ml#L154-201"><code>humpty agent</code></a> does, confining the file and shell tools to a workspace directory you pass with <code>--dir</code>:</p>
<pre><code>(* Sandbox the filesystem tools to [workspace] so that tools only have access there *)
Eio.Path.with_open_dir Eio.Path.(fs / workspace) @@ fun ws -&gt;
let agent =
  Agent.create engine ~system ~thinking:think
    ~tools: [
        Toolbox.read ~dir:ws;
        Toolbox.write ~dir:ws;
        Toolbox.dns ~net;
        Toolbox.bash ~proc; ] in ...
</code></pre>
<p>The <code>fs</code>, <code>net</code> and <code>proc</code> values all come from Eio's standard environment, and
its now up to the programmer to decide how to dole them out.
If you want a read-only agent, then just drop <code>write</code> and <code>bash</code>.</p>
<p>Since a tool is just a <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek/blob/1edcdad29a19924f988c7adef4343cb189dcb4a2/lib/tool.mli#L26"><code>Tool.v</code></a>
function over a codec and a handler, callers can add their own depending on the business
logic of the app. For example, I've now got some tools in this webserver that query the
size of the connection pool, another that can search the in-memory logs, and so on.</p>
<h3><a href="https://anil.recoil.org/news.xml#managing-deterministic-and-reproducibility" class="anchor" aria-hidden="true"></a>Managing deterministic and reproducibility</h3>
<p>One of the advantages of running a local model is that we have better control
over the determinism of the inference. Aside from obvious factors like the weights
and the inference code being the same, we normally still get different results from
GPU based inference due to the parallelism.</p>
<p>However, if we don't mind taking a slowdown, the CPU backend here supports saving
and passing in the same seed:</p>
<pre><code>$ humpty-cpu chat 'tell me a joke about camels in one sentence' -v
humpty-cpu: [INFO] model: DeepSeek V4 Flash (vocab 129280)
humpty-cpu: [INFO] seed: 1690400691090126 (random; pass --seed N to reproduce)
Why did the camel cross the desert? To get to the other hump-side!

$ humpty-cpu chat 'tell me a joke about camels in one sentence' -v
humpty-cpu: [INFO] model: DeepSeek V4 Flash (vocab 129280)
humpty-cpu: [INFO] seed: 1690392111533869 (random; pass --seed N to reproduce)
Why did the camel cross the desert? Because it was sick of the hump-drum of the same old sand dunes.

$ humpty-cpu chat 'tell me a joke about camels in one sentence' --seed 1690400691090126
Why did the camel cross the desert? To get to the other hump-side!
</code></pre>
<p>I was surprised, however, to find that the deterministic seed also worked in the Metal
backend!</p>
<pre><code>$ humpty-metal chat 'tell me a joke about camels in one sentence' -v
humpty-metal: [INFO] model: DeepSeek V4 Flash (vocab 129280)
humpty-metal: [INFO] seed: 2042575750328474 (random; pass --seed N to reproduce)
Why did the camel cross the desert? Because it was tired of the hump-drum of its daily routine.

$ humpty-metal chat 'tell me a joke about camels in one sentence' -v
humpty-metal: [INFO] model: DeepSeek V4 Flash (vocab 129280)
humpty-metal: [INFO] seed: 2043271538873965 (random; pass --seed N to reproduce)
Why do camels never get stuck in traffic? Because they have built-in "hump" day passes.

$ humpty-metal chat 'tell me a joke about camels in one sentence' -v --seed 2043271538873965
humpty-metal: [INFO] model: DeepSeek V4 Flash (vocab 129280)
humpty-metal: [INFO] seed: 2043271538873965
Why do camels never get stuck in traffic? Because they have built-in "hump" day passes.
</code></pre>
<h2><a href="https://anil.recoil.org/news.xml#implications-of-using-llms-as-a-library" class="anchor" aria-hidden="true"></a>Implications of using LLMs as a library</h2>
<p>We've seen so far that the LLM model can be exposed as a mostly pure function;
and that a tool is a function whose type says what it may touch; and that an
agent is just a loop over them.</p>
<p>Crucially, there's no need for serialisation protocols, REST APIs, MCP
authentication and many of the other layers built over them unless the
application wants it.  One of the big advantages of unikernel-style libraries
is that necessary functionality can be specialised at compile-time much more
easily.</p>
<h3><a href="https://anil.recoil.org/news.xml#building-a-safe-bastion-using-dikjstra-monads-or-refinements" class="anchor" aria-hidden="true"></a>Building a safe bastion using Dikjstra monads or refinements</h3>
<p>This library approach is orthogonal to the idea of "language integrated" LLMs, which
involve discharging verification conditions by having LLMs attempt to synthesise
proofs of statements embedded within the source code.
Ranjit Jhala <a href="https://x.com/RanjitJhala/status/2065280989367357725">observed</a> that:</p>
<blockquote>
<p>"Language integrated" is a drum I've been beating on for a while (e.g. with
<a href="https://youtube.com/watch?v=F2tYCxb30WU">refinement types</a>), but in the age of
LLMs I wonder if it <em>really</em> matters, if the AIs are going to also be generating
the proofs?
<cite> -- <a href="https://x.com/RanjitJhala/status/2065280989367357725">Ranjit Jhala, June 2026</a></cite></p>
</blockquote>
<p><a href="https://github.com/yminsky">Yaron Minsky</a> <a href="https://x.com/yminsky/status/2065403677309915570">noted that</a> <em>"language-integrated
assertions and modular specification look like a form of intelligence amplification for whatever is doing the proofs"</em>. I also found
<a href="https://cs.brown.edu/~sk/">Shriram Krishnamurthi</a> and Flatt's <a href="https://arxiv.org/abs/2606.01522">"Type-Error Ablation and AI Coding Agents"</a> paper that found richer type errors help an agent
fix code, and that a type system helps it more than test failures do.</p>
<p>All this means is that we needn't stop at just using types to encode our safety properties. Since each tool is an ordinary OCaml function, nothing stops them being synthesised or checked by a proof assistant like <a href="https://www.fstar-lang.org/">Fstar</a>, so that a tool can ship with a machine-checked guarantee that it stays inside the policy its signature advertises.</p>
<p><a href="https://anil.recoil.org/papers/2024-hope-bastion.pdf"> <img src="https://anil.recoil.org/images/bastion-fig1-ss.webp" alt="%c" title="The design of our Bastion agentic system in F* and OCaml"> </a></p>
<p>We explored this last year in a <a href="https://anil.recoil.org/papers/2024-hope-bastion">HOPE abstract</a> via <a href="https://arxiv.org/abs/1903.01237">Dijkstra monads</a>, since they let you reason precisely about the effects a computation is allowed to have. <a href="https://patrick.sirref.org">Patrick Ferris</a> <a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a> and I sketched how to
modularise some more dependently typed policy reasoning in <a href="https://anil.recoil.org/papers/2024-hope-bastion">Bastion</a>. Eio's capabilities are really just a lightweight version of the sorts of things you can express in a full proof framework, and as Ranjit notes above, LLMs make it easier than ever to just pick the right specification language for the problem at hand.</p>
<p>Another obvious direction to take my library-agentic-harness is to link it in with
OCaml's <a href="https://ocaml.org/manual/5.4/api/compilerlibref/Compiler_libs.html">compiler-libs</a>
to build a much more tightly integrated agent debugger that doesn't need to go
via CLI tooling!</p>
<h3><a href="https://anil.recoil.org/news.xml#letting-a-service-watchand-mutateitself" class="anchor" aria-hidden="true"></a>Letting a service watch...and mutate...itself</h3>
<p>What else could we do with an LLM we can call as cheaply as any function, on hardware you
own? (I'm assuming we have spare CPU/GPU here!)</p>
<p>The experiment I'm mid-way through is wiring this into my <a href="https://anil.recoil.org/notes/oxcaml-httpz">zero-allocation OxCaml webserver</a>, which already emits custom <a href="https://ocaml.org/manual/runtime-events.html">runtime events</a> alongside OCaml's native GC and scheduler ones. Normally they pile up in a ring buffer nobody reads until something's already broken. The idea is to spend idle CPU on them, so when the server isn't busy, I hand a window of recent events to my agent function
in a separate domain and check to see if the latency is drifting, or if that event is firing far more than yesterday (based on summary stats), and so on.
Whether a local model is any good at this is something I don't know yet, but I'm enjoying experimenting.</p>
<p>As we discussed in the <a href="https://anil.recoil.org/notes/rewilding-the-web-report">rewilding the web</a> workshop and our <a href="https://anil.recoil.org/papers/2025-internet-ecology">Internet ecology</a> paper, self-hosted services unfortunately
don't just <a href="https://anil.recoil.org/notes/recoil-self-hosting-2026">run themselves</a> and need tending to.
I could therefore imagine building enough local tooling so that the agent harness would recompile
and re-exec the binary autonomously in response <a href="https://anil.recoil.org/notes/internet-immune-system">to external stimuli</a>...</p>
<p>The <a href="https://tangled.org/anil.recoil.org/ocaml-deepseek">code's here</a> if
you'd like to pull it apart. I'd love to hear more about any improbable stunts or agentic harnesses you build yourself! In the future, I'll investigate expanding this out to CUDA and also beyond just Deepseek.</p>
<p><small class="credits">(Update: typos fixed thanks to Lachlan Kermode)</small></p>
<div class="footnotes"><ol><li><p></p><p><code>humpty</code> is the OCaml binary, and named due to the <a href="https://github.com/antirez/ds4">Dwarfstar</a> needing a camel connection.</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy et al (2025). Steps towards an Ecology for the Internet. Association for Computing Machinery. <a href="https://doi.org/10.1145/3744169.3744180" target="_blank"><i>10.1145/3744169.3744180</i></a></li>
<li>Madhavapeddy (2025). Deepdive into Deepseek advances. <a href="https://doi.org/10.59350/r06z7-0ht06" target="_blank"><i>10.59350/r06z7-0ht06</i></a></li>
<li>Madhavapeddy (2026). My (very) fast zero-allocation webserver using OxCaml. <a href="https://doi.org/10.59350/9c6bz-kb659" target="_blank"><i>10.59350/9c6bz-kb659</i></a></li>
<li>Madhavapeddy (2026). Self-hosting email the hard way from your own routable IPv4 block up. <a href="https://doi.org/10.59350/gj8re-sca95" target="_blank"><i>10.59350/gj8re-sca95</i></a></li>
<li>Sivaramakrishnan et al (2021). Retrofitting effect handlers onto OCaml. ACM. <a href="https://doi.org/10.1145/3453483.3454039" target="_blank"><i>10.1145/3453483.3454039</i></a></li>
<li>Madhavapeddy (2026). Rewilding the Web: my workshop report from Edinburgh. <a href="https://doi.org/10.59350/g40yy-ks003" target="_blank"><i>10.59350/g40yy-ks003</i></a></li>
<li>Madhavapeddy (2026). The Internet needs an antibotty immune system, stat. <a href="https://doi.org/10.59350/snnnf-asc02" target="_blank"><i>10.59350/snnnf-asc02</i></a></li>
<li>Krishnamurthi et al (2026). Type-Error Ablation and AI Coding Agents. arXiv. <a href="https://doi.org/10.48550/arXiv.2606.01522" target="_blank"><i>10.48550/arXiv.2606.01522</i></a></li>
<li>Maillard et al (2019). Dijkstra Monads for All. arXiv. <a href="https://doi.org/10.48550/arXiv.1903.01237" target="_blank"><i>10.48550/arXiv.1903.01237</i></a></li></ul>
