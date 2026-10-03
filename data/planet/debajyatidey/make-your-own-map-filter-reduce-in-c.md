---
title: Make Your Own Map, Filter, Reduce in C++
description: Learn how to implement map, filter, and reduce functions in C++14, and
  modularize utilities for a cleaner programming approach
url: https://debajyatidey.hashnode.dev/make-your-own-map-filter-reduce-in-cpp
date: 2025-06-01T15:47:35-00:00
preview_image: https://cdn.hashnode.com/res/hashnode/image/upload/v1748792500989/e40f013b-1e86-429d-93cf-6c6f3247ac44.gif
authors:
- Debajyati's Blog
source:
ignore:
---

<p>As you know, C++ is an imperative programming language.</p>
<p>Although over the years they have added features of functional programming languages like promises and lambdas, that doesn’t change the core of the language.</p>
<p>However, if you come from a background of languages (like JavaScript, Python, OCaml) with heavy promotion of and decent reliance on immutability, you may find yourself frustrated or disappointed when not seeing built-in support for map, filter and reduce (reduce is also known as fold, fold_left or fold_right) functions.</p>
<p>Although from C++ 17, there comes very powerful and helpful functions like <code>std::reduce</code>, <code>std::inclusive_scan</code>, <code>std::exclusive_scan</code> in C++ that do very well transformations of elements of a data structure and folding a data structure. They are designed for generic values (types), so thus they work with any type and any custom operator.</p>
<p>But this benefit only applies to you if you use C++17 or higher. If you are constrained in an environment where Your compiler only supports C++14 or even lower (I heard few companies still use C++ 5.4 neglecting all the modern type-safe features, weird but it’s their choice), you won’t have access to these utilities.</p>
<p>In this article we will go through possible implementations of map, filter and reduce in C++, modularizing the utilities and hence providing a cleaner approach to solving Problems. For this article, we will be using <strong>C++14</strong> all the time.</p>
<p><strong>So, let’s get started!</strong></p>
<p><img src="https://i.redd.it/brh4qx4q4bb71.gif" alt="" class="image--center mx-auto"></p>
<h1>An Immutable Way of C++</h1>
<p>Let’s begin with the map function. First try to make a map function for a vector of integers.</p>
<pre><code class="lang-cpp"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;vector&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;algorithm&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;functional&gt;</span></span>

<span class="hljs-function"><span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt; <span class="hljs-title">map</span><span class="hljs-params">(<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt;&amp; <span class="hljs-built_in">list</span>,<span class="hljs-built_in">std</span>::function&lt;<span class="hljs-keyword">int</span>(<span class="hljs-keyword">int</span>)&gt; func)</span> </span>{
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt; result;
    result.reserve(<span class="hljs-built_in">list</span>.size());
    <span class="hljs-built_in">std</span>::transform(<span class="hljs-built_in">list</span>.begin(), <span class="hljs-built_in">list</span>.end(), <span class="hljs-built_in">std</span>::back_inserter(result), func);
    <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p>So, it is quite easy when writing for a specific data type.</p>
<p>And our function works pretty well.</p>
<p><img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1748161978609/cf6da2eb-a8ad-4cd2-bee6-06c129ef5536.png" alt="Running The code in an online C++ compiler" class="image--center mx-auto"></p>
<p>Similarly, we can implement filter and reduce for vector&lt;int&gt; also -</p>
<p><strong>Filter function -</strong></p>
<pre><code class="lang-cpp"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;vector&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;algorithm&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;functional&gt;</span></span>

<span class="hljs-function"><span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt; <span class="hljs-title">filter</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt;&amp; <span class="hljs-built_in">list</span>, <span class="hljs-built_in">std</span>::function&lt;<span class="hljs-keyword">bool</span>(<span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;)&gt; predicate)</span> </span>{
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt; result;
    <span class="hljs-built_in">std</span>::copy_if(<span class="hljs-built_in">list</span>.begin(), <span class="hljs-built_in">list</span>.end(), <span class="hljs-built_in">std</span>::back_inserter(result), predicate);
    <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p><strong>Reduce Function -</strong></p>
<pre><code class="lang-cpp"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;vector&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;numeric&gt; // for std::accumulate</span></span>

<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">reduce</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt;&amp; <span class="hljs-built_in">list</span>, <span class="hljs-built_in">std</span>::function&lt;<span class="hljs-keyword">int</span>(<span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;, <span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;)&gt; reducer, <span class="hljs-keyword">int</span> initial)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-built_in">std</span>::accumulate(<span class="hljs-built_in">list</span>.begin(), <span class="hljs-built_in">list</span>.end(), initial, reducer);
}
</code></pre>
<p><img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1748181026240/c7fe70bb-b199-4536-8757-e98d6bdbef07.png" alt="Using the reduce and filter functions on an example array in CodeBlocks IDE and printing the results in command prompt (STDOUT)" class="image--center mx-auto"></p>
<p>Well, it seems like we don’t need a reduce function anymore, just use <code>std::accumulate</code> and it would be done, right?</p>
<p><strong>NO!</strong> You’re wrong!</p>
<p><strong>Because</strong> the above implementation of reduce is <strong>FLAWED</strong>!<br>A reduce function requires that its 2nd argument (the binary operator or function) is both cumulative and associative. Because it performs the operations out of order, sometimes even parallelly (if an execution policy is assigned).</p>
<p>Let me clear the concepts first, about folding and reducing -</p>
<ol>
<li><p><strong>Folding</strong> has to have a direction, either left or right, reducing doesn’t need a direction.</p>
</li>
<li><p><strong>Reducing</strong> is a process of accumulating elements of a linear data structure where the binary operation is assumed to be commutative and associative. Hence, if the order of elements in operation and accumulation goes different from the order of elements stored in the data structure, it will have <strong>ZERO</strong> casualties.</p>
</li>
<li><p>In case of <strong>Folding</strong>, it doesn’t matter if the order of accumulation is the same as order of elements or say the binary operator is commutative or associative or not. Because the <strong>Execution Order</strong> of Folding is Strictly sequential, left-to-right or right-to-left.</p>
</li>
<li><p>The result of a <strong>fold_left</strong> or <strong>fold_right</strong> is always deterministic regardless the operator is commutative and associative or not. But if the operator or function is <strong>NOT</strong> associative and commutative, reduce will cause indeterministic results (due to unordered execution).</p>
</li>
</ol>
<p>What I have implemented above is actually a <strong>fold_left</strong> function. What <code>std::accumulate</code> actually does is a left fold reduction.</p>
<p>So, what the reduce function should actually be is something like this -</p>
<pre><code class="lang-c"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdio.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdlib.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;omp.h&gt;  // For parallel execution</span></span>

<span class="hljs-comment">// Example cumulative and associative function: addition</span>
<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">add</span><span class="hljs-params">(<span class="hljs-keyword">int</span> a, <span class="hljs-keyword">int</span> b)</span> </span>{
    <span class="hljs-keyword">return</span> a + b;
}

<span class="hljs-comment">// Cumulative and associative reduce function</span>
<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">reduce</span><span class="hljs-params">(<span class="hljs-keyword">int</span>* <span class="hljs-built_in">array</span>, <span class="hljs-keyword">size_t</span> length, <span class="hljs-keyword">int</span> (*binary_op)(<span class="hljs-keyword">int</span>, <span class="hljs-keyword">int</span>), <span class="hljs-keyword">int</span> identity)</span> </span>{
    <span class="hljs-keyword">int</span> result = identity;

    <span class="hljs-comment">// Parallel reduction</span>
    <span class="hljs-meta">#<span class="hljs-meta-keyword">pragma</span> omp parallel for reduction(+:result)</span>
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">size_t</span> i = <span class="hljs-number">0</span>; i &lt; length; ++i) {
        result = binary_op(result, <span class="hljs-built_in">array</span>[i]);
    }

    <span class="hljs-keyword">return</span> result;
}

<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">int</span> arr[] = {<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>, <span class="hljs-number">5</span>};
    <span class="hljs-keyword">size_t</span> length = <span class="hljs-keyword">sizeof</span>(arr) / <span class="hljs-keyword">sizeof</span>(arr[<span class="hljs-number">0</span>]);

    <span class="hljs-comment">// Using the reduce function with addition operation</span>
    <span class="hljs-keyword">int</span> sum = reduce(arr, length, add, <span class="hljs-number">0</span>);

    <span class="hljs-built_in">printf</span>(<span class="hljs-string">"Sum: %d\n"</span>, sum);
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}
</code></pre>
<p>Here, <code>&lt;omp.h&gt;</code> is the header for using OpenMP (Open Multi-Processing). OpenMP is a library that supports multi-platform shared-memory parallel programming in C, C++, and Fortran.</p>
<p>Although the code is written in C. You can change it into suitable C++ syntax. I am leaving that up to you.</p>
<h2>How the Parallel Reduction Works</h2>
<ul>
<li><p><code>#pragma omp parallel for reduction(+:result)</code>: This is the core of the parallel execution using OpenMP.</p>
<ul>
<li><p><code>#pragma omp parallel for</code>: This directive tells the OpenMP runtime to parallelize the following <code>for</code> loop across multiple threads. Each thread will execute a portion of the loop iterations concurrently.</p>
</li>
<li><p><code>reduction(+:result)</code>: This clause specifies a reduction operation.</p>
<ul>
<li><p><code>+</code>: Indicates that the reduction operation is addition. OpenMP will handle the partial sums computed by each thread and combine them safely at the end.</p>
</li>
<li><p><code>result</code>: The variable on which the reduction is performed. OpenMP ensures that updates to <code>result</code> from different threads are properly synchronized to avoid race conditions.</p>
</li>
</ul>
</li>
</ul>
</li>
<li><p>After each thread finishes its assigned iterations, OpenMP combines the partial sums from all the threads using the specified operation (addition in this case) and stores the final result in the original <code>result</code> variable in the shared memory. This combining process is done in a thread-safe manner.</p>
</li>
</ul>
<p>Now let’s see how we can define our own <strong>fold_left</strong> and <strong>fold_right</strong>.</p>
<p><strong>The Easy Way</strong> -</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">fold_left</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt;&amp; <span class="hljs-built_in">list</span>, <span class="hljs-built_in">std</span>::function&lt;<span class="hljs-keyword">int</span>(<span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;, <span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;)&gt; reducer, <span class="hljs-keyword">int</span> initial)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-built_in">std</span>::accumulate(<span class="hljs-built_in">list</span>.begin(), <span class="hljs-built_in">list</span>.end(), initial, reducer);
}

<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">fold_right</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">int</span>&gt;&amp; <span class="hljs-built_in">list</span>, <span class="hljs-built_in">std</span>::function&lt;<span class="hljs-keyword">int</span>(<span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;, <span class="hljs-keyword">const</span> <span class="hljs-keyword">int</span>&amp;)&gt; reducer, <span class="hljs-keyword">int</span> initial)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-built_in">std</span>::accumulate(<span class="hljs-built_in">list</span>.rbegin(), <span class="hljs-built_in">list</span>.rend(), initial, reducer);
}
</code></pre>
<p><strong>The Hard Way</strong> -</p>
<pre><code class="lang-cpp"><span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
<span class="hljs-function">AccType <span class="hljs-title">fold_left</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; vec,
                                AccType initial_acc,
                                Func func)</span> </span>{
    AccType accumulator = initial_acc;
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">auto</span> itr = vec.begin(); itr != vec.end(); ++itr) {
        accumulator = func(accumulator, *itr);
    }
    <span class="hljs-keyword">return</span> accumulator;
}

<span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
<span class="hljs-function">AccType <span class="hljs-title">fold_right</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; vec,
                                 AccType initial_acc,
                                 Func func)</span> </span>{
    AccType accumulator = initial_acc;
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">auto</span> ritr = vec.rbegin(); ritr != vec.rend(); ++ritr) {
        accumulator = func(*ritr, accumulator);
    }
    <span class="hljs-keyword">return</span> accumulator;
}
</code></pre>
<p>Yes, I used templating syntax for making them reusable for different data types.</p>
<p>You can try out the functions defined like this way -</p>
<pre><code class="lang-cpp"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;bits/stdc++.h&gt;</span></span>

<span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
<span class="hljs-function">AccType <span class="hljs-title">fold_left</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; vec,
                                AccType initial_acc,
                                Func func)</span> </span>{
    AccType accumulator = initial_acc;
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">auto</span> itr = vec.begin(); itr != vec.end(); ++itr) {
        accumulator = func(accumulator, *itr);
    }
    <span class="hljs-keyword">return</span> accumulator;
}

<span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
<span class="hljs-function">AccType <span class="hljs-title">fold_right</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; vec,
                                 AccType initial_acc,
                                 Func func)</span> </span>{
    AccType accumulator = initial_acc;
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">auto</span> ritr = vec.rbegin(); ritr != vec.rend(); ++ritr) {
        accumulator = func(*ritr, accumulator);
    }
    <span class="hljs-keyword">return</span> accumulator;
}

<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-keyword">double</span>&gt; vec = {<span class="hljs-number">1.5</span>,<span class="hljs-number">2.5</span>,<span class="hljs-number">3.5</span>,<span class="hljs-number">4.5</span>,<span class="hljs-number">5.5</span>,<span class="hljs-number">6.5</span>,<span class="hljs-number">7.5</span>,<span class="hljs-number">8.5</span>,<span class="hljs-number">9.5</span>,<span class="hljs-number">0.5</span>};
    <span class="hljs-keyword">double</span> sum = fold_left(vec, <span class="hljs-number">0.0</span>, [](<span class="hljs-keyword">double</span> num, <span class="hljs-keyword">double</span> nyum){
        <span class="hljs-keyword">return</span> (num + nyum);
    });
    <span class="hljs-keyword">double</span> mul = fold_right(vec, <span class="hljs-number">1.0</span>, [](<span class="hljs-keyword">double</span> n, <span class="hljs-keyword">double</span> m)-&gt;<span class="hljs-keyword">double</span>{
        <span class="hljs-keyword">return</span> (n * m);
    });
    <span class="hljs-built_in">std</span>::<span class="hljs-built_in">cout</span> &lt;&lt; <span class="hljs-built_in">std</span>::fixed &lt;&lt; <span class="hljs-built_in">std</span>::setprecision(<span class="hljs-number">5</span>) &lt;&lt; <span class="hljs-string">"The sum would be - "</span> &lt;&lt; sum &lt;&lt; <span class="hljs-string">"\nAnd the multiplication would be - "</span> &lt;&lt; mul &lt;&lt; <span class="hljs-built_in">std</span>::<span class="hljs-built_in">endl</span>;
    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}
</code></pre>
<p>The code obviously works, and you can see it below -</p>
<p><img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1748781460822/93387e08-1ff3-422b-9668-ae087fa195af.png" alt="Trying out the fold_left and fold_right functions with a main function defined in an online C++ compiler" class="image--center mx-auto"></p>
<p>Now as you have understood the possible implementation of map, filter and reduce, let’s write a module encapsulating all the functions we have defined till now with some extras.</p>
<p>Look the idea is that, in OCaml, the standard library provided a List Module which had some incredibly useful functions, from map, filter, reduce to pipeline operator, some more variations of them and also some extras few different ones.</p>
<p>Availability of utility functions like this can save you lot of time by not having to write cluttered code or repetitive specific function definitions.</p>
<p>So, let’s make it clear, we will be defining these functions in our module -</p>
<h3><strong>1.</strong> <code>map</code></h3>
<p><strong>Application</strong>: Tail-recursive transformation of a range of elements using a function.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">map</span><span class="hljs-params">(begin_itr, end_itr, func)</span></span>;
</code></pre>
<hr>
<h3><strong>2.</strong> <code>mapi</code></h3>
<p><strong>Application</strong>: Index-aware mapping function (not tail-recursive); applies function with access to index and value.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">mapi</span><span class="hljs-params">(begin_itr, end_itr, func)</span></span>;
</code></pre>
<hr>
<h3><strong>3.</strong> <code>map2</code></h3>
<p><strong>Application</strong>: Applies a binary function elementwise on two vectors.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">map2</span><span class="hljs-params">(begin1, end1, begin2, end2, func)</span></span>;
</code></pre>
<hr>
<h3><strong>4.</strong> <code>map2D</code> (Version 1)</h3>
<p><strong>Application</strong>: Applies a function to each element in a 2D vector (matrix-like structure).</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt;&gt; <span class="hljs-title">map2D</span><span class="hljs-params">(input_2d_vector, func)</span></span>;
</code></pre>
<hr>
<h3><strong>5.</strong> <code>map2D</code> (Version 2)</h3>
<p><strong>Application</strong>: Applies an outer and inner transformation over a 2D vector.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt;&gt; <span class="hljs-title">map2D</span><span class="hljs-params">(input_2d_vector, func_outer, func_inner)</span></span>;
</code></pre>
<hr>
<h3><strong>6.</strong> <code>filter</code></h3>
<p><strong>Application</strong>: Filters elements based on a predicate.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; <span class="hljs-title">filter</span><span class="hljs-params">(begin_itr, end_itr, predicate)</span></span>;
</code></pre>
<hr>
<h3><strong>7.</strong> <code>reduce</code></h3>
<p><strong>Application</strong>: Reduces a vector into a single value using a binary function and an initial accumulator.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">reduce</span><span class="hljs-params">(input_vector, initial_acc, func)</span></span>;
</code></pre>
<hr>
<h3><strong>8.</strong> <code>fold_left</code></h3>
<p><strong>Application</strong>: Folds (reduces) from left to right over a range using a binary function.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">fold_left</span><span class="hljs-params">(begin_itr, end_itr, initial_acc, func)</span></span>;
</code></pre>
<hr>
<h3><strong>9.</strong> <code>fold_right</code></h3>
<p><strong>Application</strong>: Folds (reduces) from right to left using reverse iterators.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">fold_right</span><span class="hljs-params">(begin_ritr, end_ritr, initial_acc, func)</span></span>;
</code></pre>
<hr>
<h3><strong>10.</strong> <code>apply</code></h3>
<p><strong>Application</strong>: Function application (pipeline-style transformation of a single value).</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> U <span class="hljs-title">apply</span><span class="hljs-params">(value, func)</span></span>;
</code></pre>
<hr>
<h3><strong>11.</strong> <code>forall</code></h3>
<p><strong>Application</strong>: Tests if all elements in a range satisfy a predicate.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">bool</span> <span class="hljs-title">forall</span><span class="hljs-params">(begin_itr, end_itr, predicate)</span></span>;
</code></pre>
<hr>
<h3><strong>12.</strong> <code>split</code></h3>
<p><strong>Application</strong>: Splits a vector of pairs into two vectors.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">pair</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T1&gt;, <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T2&gt;&gt; <span class="hljs-title">split</span><span class="hljs-params">(input_pairs)</span></span>;
</code></pre>
<hr>
<h3><strong>13.</strong> <code>combine</code></h3>
<p><strong>Application</strong>: Combines two vectors into a vector of pairs.</p>
<pre><code class="lang-cpp"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">pair</span>&lt;T1, T2&gt;&gt; <span class="hljs-title">combine</span><span class="hljs-params">(list1, list2)</span></span>;
</code></pre>
<hr>
<h3><strong>14.</strong> <code>operator|</code> <em>(free function)</em></h3>
<p><strong>Application</strong>: Provides pipeline operator-like syntax for function application.</p>
<pre><code class="lang-cpp">U <span class="hljs-keyword">operator</span>|(value, func);
</code></pre>
<hr>
<h3>Summary</h3>
<div class="hn-table">
<table>
<thead>
<tr>
<td>Function Name</td><td>Purpose</td></tr>
</thead>
<tbody>
<tr>
<td><code>map</code>, <code>mapi</code>, <code>map2</code>, <code>map2D</code> (v1 &amp; v2)</td><td>Transformation / Mapping</td></tr>
<tr>
<td><code>filter</code>, <code>reduce</code>, <code>fold_left</code>, <code>fold_right</code></td><td>Filtering and Aggregation</td></tr>
<tr>
<td><code>apply</code></td><td>Partial Application / Pipelining</td></tr>
<tr>
<td><code>forall</code></td><td>Logical test (universal quantification)</td></tr>
<tr>
<td><code>split</code>, <code>combine</code></td><td>Vector decomposition and composition</td></tr>
</tbody>
</table>
</div><h1>Let’s Implement</h1>
<p>This will be our header file -</p>
<pre><code class="lang-cpp"><span class="hljs-meta">#<span class="hljs-meta-keyword">ifndef</span> FUNCTIONAL_LIB_HPP</span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> FUNCTIONAL_LIB_HPP</span>

<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;vector&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;utility&gt;</span></span>

<span class="hljs-keyword">namespace</span> functional_lib {

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Functional</span> {</span>
<span class="hljs-keyword">public</span>:
    <span class="hljs-comment">// map (Tail Recursive)</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">map</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator begin_itr,
                             <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                             Func func)</span></span>;

    <span class="hljs-comment">// mapi (Not Tail Recursive - index aware map)</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">mapi</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator begin_itr,
                              <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                              Func func)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T1, <span class="hljs-keyword">typename</span> T2, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt; <span class="hljs-title">map2</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T1&gt;::const_iterator begin_itr1,
                              <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T1&gt;::const_iterator end_itr1,
                              <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T2&gt;::const_iterator begin_itr2,
                              <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T2&gt;::const_iterator end_itr2,
                              Func func)</span></span>;

    <span class="hljs-comment">// map2D - Version 1: Single callback</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt;&gt; <span class="hljs-title">map2D</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;&gt;&amp; input_2d_vector, Func func)</span></span>;

    <span class="hljs-comment">// map2D - Version 2: Two callbacks</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> FuncOuter, <span class="hljs-keyword">typename</span> FuncInner&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt;&gt; <span class="hljs-title">map2D</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;&gt;&amp; input_2d_vector, FuncOuter func_outer, FuncInner func_inner)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> Predicate&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt; <span class="hljs-title">filter</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator begin_itr,
                                 <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                                 Predicate predicate)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">reduce</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;&amp; input_vector, AccType initial_acc, Func func)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">fold_left</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator begin_itr,
                              <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                              AccType initial_acc,
                              Func func)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> AccType, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> AccType <span class="hljs-title">fold_right</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_reverse_iterator begin_ritr,
                               <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_reverse_iterator end_ritr,
                               AccType initial_acc,
                               Func func)</span></span>;

    <span class="hljs-comment">// Partial application function (what pipeline operator does)</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> U <span class="hljs-title">apply</span><span class="hljs-params">(T value, Func func)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> Predicate&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">bool</span> <span class="hljs-title">forall</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator begin_itr,
                        <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                        Predicate predicate)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T1, <span class="hljs-keyword">typename</span> T2&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">pair</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T1&gt;, <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T2&gt;&gt; <span class="hljs-title">split</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">pair</span>&lt;T1, T2&gt;&gt;&amp; input_pairs)</span></span>;

    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T1, <span class="hljs-keyword">typename</span> T2&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;<span class="hljs-built_in">std</span>::<span class="hljs-built_in">pair</span>&lt;T1, T2&gt;&gt; <span class="hljs-title">combine</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T1&gt;&amp; list1, <span class="hljs-keyword">const</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T2&gt;&amp; list2)</span></span>;

<span class="hljs-keyword">private</span>:
    <span class="hljs-comment">// Helper for tail-recursive map</span>
    <span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
    <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">map_helper</span><span class="hljs-params">(<span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator itr,
                           <span class="hljs-keyword">typename</span> <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;T&gt;::const_iterator end_itr,
                           Func func,
                           <span class="hljs-built_in">std</span>::<span class="hljs-built_in">vector</span>&lt;U&gt;&amp; result_vector)</span></span>;
};

<span class="hljs-comment">// Pipeline operator equivalent (free function for cleaner syntax)</span>
<span class="hljs-keyword">template</span> &lt;<span class="hljs-keyword">typename</span> T, <span class="hljs-keyword">typename</span> U, <span class="hljs-keyword">typename</span> Func&gt;
U <span class="hljs-keyword">operator</span>|(T value, Func func);


} <span class="hljs-comment">// namespace functional_lib</span>

<span class="hljs-meta">#<span class="hljs-meta-keyword">endif</span> <span class="hljs-comment">// FUNCTIONAL_LIB_HPP</span></span>
</code></pre>
<p>Now, the thing is I won’t just give you the source code (implementation of the functions), Hehe…</p>
<p>Try with all your efforts, don’t just ChatGPT it, try to have little fun, with programming!</p>
<p>In the end, if you think you still want to see my implementation, for comparison purpose, I suppose. Comment it. I will link a GitHub gist in the comment section.</p>
<h1>Concluding</h1>
<p>So, hey, make sure you try to implement the source file!<br>I wanted to make sure you just don’t mindlessly read, or copy paste but actually mindfully read and understand the content.</p>
<p>If no one comments about if they successfully implemented all the functions in the header file or not then I will realize no one actually read it, just scrolled through it 😔🥲.</p>
<p>It’s not something very hard, so I expect you to able to do this. And BTW if you are not able to define all of them, it’s okay, just post whatever functions you implemented in a comment.</p>
<p>I will love to read your implementation and maybe find some better techniques so that I can improve my own code.</p>
<p>So, that’s it! Thank you for reading!</p>
<p><img src="https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExMTVodDY5NGNicTVpMWFrbzIzZW83NHB2aDhvbmcxNnJ5a2JjbTA2cyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/2uA1EH7nwJT65sJJPl/giphy.gif" alt="" class="image--center mx-auto"></p>
<p>If you found this POST helpful, if this blog added some value to your time and energy, please <strong>show some love</strong> by giving the article some <strong>likes</strong> and <strong>share it</strong> with your <strong>developer friends</strong>. It will encourage me to create more content like this.</p>
<p>Feel free to connect with me at - <a href="https://twitter.com/intent/follow?screen_name=ddebajyati" target="_blank"><strong>Twitter</strong></a>, <a href="https://www.linkedin.com/in/debajyati-dey" target="_blank"><strong>LinkedIn</strong></a> or <a href="https://github.com/Debajyati" target="_blank"><strong>GitHub</strong></a> :)</p>
<p>Happy Coding 🧑🏽‍💻👩🏽‍💻! Have a nice day ahead! 🚀</p>

