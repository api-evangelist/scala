---
title: "Porting the Scala 2 optimizer to Scala 3"
url: "https://www.scala-lang.org/blog/2026/03/23/porting-the-optimizer.html"
date: "2026-03-23T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<blockquote>
  <p>This post covers work done under the <a href="https://www.scala-lang.org/blog/2026/01/27/sta-invests-in-scala.html">Sovereign Tech Fund investment</a> umbrella: <a href="https://contributors.scala-lang.org/t/standard-library-now-open-for-improvements-and-suggestions/7337">Maintenance of the Standard Library/Core Library Modules and APIs</a>. The work is coordinated by the <a href="https://scala.epfl.ch/">Scala Center</a>.</p>
</blockquote>

<p>We are porting the <em>optimizer</em> from the Scala 2 compiler to the Scala 3 compiler, improving the performance of Scala 3 applications without requiring developers to write more complex code. In early microbenchmarks of code written in a high-level functional style, we’re seeing 10-30% faster execution. But what exactly is this optimizer and why is it necessary?</p>

<p><em>This work is available as of version <a href="https://github.com/scala/scala3/releases/tag/3.8.3-RC3">3.8.3-RC3</a> of the Scala compiler.</em></p>

<p>Scala is a modern and concise language, meaning you can write code expressing <em>what</em> you want to do by composing primitives, rather than <em>how</em> to do it step-by-step.
High-level code is easier to read and maintain, in addition to being shorter to write.</p>

<p>Scala’s expressiveness enables you to write what you mean:</p>
<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">addOneMap</span><span class="o">(</span><span class="n">a</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">Int</span><span class="o">])</span> <span class="k">=</span> <span class="nv">a</span><span class="o">.</span><span class="py">map</span><span class="o">(</span><span class="k">_</span> <span class="o">+</span> <span class="mi">1</span><span class="o">)</span>
</code></pre></div></div>
<p>However, from a CPU’s point of view, a high-level API like <code class="language-plaintext highlighter-rouge">map</code> is conceptually more complex than the equivalent low-level code.
It’s a function call that is passed another function, leading to a “<a href="https://shipilev.net/jvm/anatomy-quarks/16-megamorphic-virtual-calls/">megamorphic</a>” call site that is harder for the JVM to optimize, and the function argument might need a closure to be allocated on the heap if it captures local values.
Compiled naïvely, the high-level call wouldn’t be as fast as this equivalent low-level loop:</p>
<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">addOneLoop</span><span class="o">(</span><span class="n">a</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">Int</span><span class="o">])</span> <span class="k">=</span>
  <span class="k">val</span> <span class="nv">l</span> <span class="k">=</span> <span class="nv">a</span><span class="o">.</span><span class="py">length</span>
  <span class="k">val</span> <span class="nv">r</span> <span class="k">=</span> <span class="k">new</span> <span class="nc">Array</span><span class="o">[</span><span class="kt">Int</span><span class="o">](</span><span class="n">l</span><span class="o">)</span>
  <span class="k">var</span> <span class="n">i</span> <span class="k">=</span> <span class="mi">0</span>
  <span class="k">while</span> <span class="n">i</span> <span class="o">&lt;</span> <span class="n">l</span> <span class="k">do</span>
    <span class="nf">r</span><span class="o">(</span><span class="n">i</span><span class="o">)</span> <span class="k">=</span> <span class="nf">a</span><span class="o">(</span><span class="n">i</span><span class="o">)</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="n">i</span> <span class="o">+=</span> <span class="mi">1</span>
  <span class="n">r</span>
</code></pre></div></div>
<p>You don’t want to write this loop, because its purpose is a lot less obvious and it is harder to maintain, but without compiler help, you might have to write it if that function is critical to your application’s performance.
(Why would adding one to an array be critical to your app’s performance? Because you’re reading a pedagogical blog post and thus suspending disbelief!)</p>

<p>If you’ve ever written a <code class="language-plaintext highlighter-rouge">map</code> function yourself, you may recognize the loop above as being fairly close to how such a function is implemented, though the actual Scala implementation is a little more complex than you’d think because it needs to handle the mismatch between Scala’s generic array type and the JVM’s erased generics:</p>
<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">map</span><span class="o">[</span><span class="kt">B</span><span class="o">](</span><span class="n">a</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">A</span><span class="o">],</span> <span class="n">f</span><span class="k">:</span> <span class="kt">A</span> <span class="o">=&gt;</span> <span class="n">B</span><span class="o">)(</span><span class="k">implicit</span> <span class="n">ct</span><span class="k">:</span> <span class="kt">ClassTag</span><span class="o">[</span><span class="kt">B</span><span class="o">])</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">B</span><span class="o">]</span> <span class="k">=</span>
  <span class="k">val</span> <span class="nv">len</span> <span class="k">=</span> <span class="nv">a</span><span class="o">.</span><span class="py">length</span>
  <span class="k">val</span> <span class="nv">r</span> <span class="k">=</span> <span class="k">new</span> <span class="nc">Array</span><span class="o">[</span><span class="kt">B</span><span class="o">](</span><span class="n">len</span><span class="o">)</span>
  <span class="k">var</span> <span class="n">i</span> <span class="k">=</span> <span class="mi">0</span>
  <span class="o">(</span><span class="n">a</span><span class="k">:</span> <span class="kt">Any</span> <span class="kt">@unchecked</span><span class="o">)</span> <span class="k">match</span>
    <span class="k">case</span> <span class="n">a</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">AnyRef</span><span class="o">]</span>  <span class="k">=&gt;</span> <span class="nf">while</span> <span class="o">(</span><span class="n">i</span> <span class="o">&lt;</span> <span class="n">len</span><span class="o">)</span> <span class="o">{</span> <span class="nf">r</span><span class="o">(</span><span class="n">i</span><span class="o">)</span> <span class="k">=</span> <span class="nf">f</span><span class="o">(</span><span class="nf">a</span><span class="o">(</span><span class="n">i</span><span class="o">).</span><span class="py">asInstanceOf</span><span class="o">[</span><span class="kt">A</span><span class="o">]);</span> <span class="n">i</span> <span class="k">=</span> <span class="n">i</span><span class="o">+</span><span class="mi">1</span> <span class="o">}</span>
    <span class="k">case</span> <span class="n">a</span><span class="k">:</span> <span class="kt">Array</span><span class="o">[</span><span class="kt">Int</span><span class="o">]</span>     <span class="k">=&gt;</span> <span class="nf">while</span> <span class="o">(</span><span class="n">i</span> <span class="o">&lt;</span> <span class="n">len</span><span class="o">)</span> <span class="o">{</span> <span class="nf">r</span><span class="o">(</span><span class="n">i</span><span class="o">)</span> <span class="k">=</span> <span class="nf">f</span><span class="o">(</span><span class="nf">a</span><span class="o">(</span><span class="n">i</span><span class="o">).</span><span class="py">asInstanceOf</span><span class="o">[</span><span class="kt">A</span><span class="o">]);</span> <span class="n">i</span> <span class="k">=</span> <span class="n">i</span><span class="o">+</span><span class="mi">1</span> <span class="o">}</span>
    <span class="cm">/* ... 7 cases omitted for brevity ...  */</span>
  <span class="n">r</span>
</code></pre></div></div>
<p>This method’s complexity is necessary to handle every possible array you can throw at it. You could also implement it with less code by using reflection to handle any possible array with just one code path, but that would lead to slower execution.
But in our <code class="language-plaintext highlighter-rouge">addOne</code> case, we only need the <code class="language-plaintext highlighter-rouge">Array[Int]</code> part. Type-checking the array isn’t needed, nor is casting <code class="language-plaintext highlighter-rouge">a(i)</code> inside the loop, and the whole <code class="language-plaintext highlighter-rouge">ClassTag</code> machinery could go away too.</p>

<h2 id="optimizing-for-the-best-of-both-worlds">Optimizing for the best of both worlds</h2>

<p>How can we get the best of both worlds? By having an <em>optimizer</em> in the compiler that simplifies code and heuristically determines when it is beneficial to <em>inline</em> code to enable further simplifications. Scala 2 has had such an optimizer for years now, <a href="https://docs.scala-lang.org/overviews/compiler-options/optimizer.html">as documented here</a>, and it’s finally time to port it to Scala 3! Let’s see why it works in a little more detail.</p>

<p>The optimizer takes care of turning our single-line <code class="language-plaintext highlighter-rouge">addOneMap</code> example into our fast <code class="language-plaintext highlighter-rouge">addOneLoop</code> version. 
The key technique to perform this is <em>inlining</em>: expanding the code of <code class="language-plaintext highlighter-rouge">map</code> into the function where it is called.
This allows the optimizer to remove redundant branches and operations, such as removing the <a href="https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html">boxing</a> of primitive types.
The <code class="language-plaintext highlighter-rouge">match</code> is simplified to just one of its branches, the <code class="language-plaintext highlighter-rouge">asInstanceOf</code> and <code class="language-plaintext highlighter-rouge">ClassTag</code> disappear, and we end up with the fast version.</p>

<p>Inlining is powerful but can also have big drawbacks.
For starters, your CPU keeps frequently-executed code in an “instruction cache” that is very fast to access.
When common functions get bigger, fewer of them fit in that cache, so execution might become slower because fetching the code is slower even if the code itself has fewer function calls.</p>

<p>Furthermore, inlining is not always possible.
A classic case is recursion: you cannot infinitely inline a function into itself.
Some more tricky problems include the fact that you cannot inline a method that accesses a class’s private fields outside of that class.
(At least not on the JVM, Scala.js does not have that problem.)</p>

<p>In this case, there is a heuristic modeling the fact that if a function call uses a function literal as argument, inlining it is probably worth it.
There are other heuristics, such as one related to generic array operations in general, one that forces inlining of “forwarder” functions that merely call another function with minor changes to their arguments, and so on.</p>

<p>The JVM already has an optimizer as part of Just-In-Time compilation, but it’s designed to make Java code fast, and typical Java code does not look like typical Scala code, so there are some important optimization opportunities not covered by the JVM’s optimizer.
Furthermore, the Scala compiler can optimize code at compile-time based on knowledge that is internal to the compiler and subject to change between versions, such as which Scala runtime functions are guaranteed to be pure or to always return non-null references.</p>

<p>Early results show that for individual methods such as the <code class="language-plaintext highlighter-rouge">addOne</code> example above, the optimizer successfully turns the high-level version into an exact equivalent of the low-level one, which translates to 10-30% gains in some microbenchmarks.</p>

<h2 id="limitations">Limitations</h2>

<p>Heuristics mostly lead to better performance, but performance is such a complex topic that no set of heuristics can guarantee improvements.
It’s possible that enabling the optimizer on your specific codebase could regress some scenarios, which is why you should benchmark any performance-related change just as you would test any correctness-related change.
You can override the heuristics with <code class="language-plaintext highlighter-rouge">@inline</code> and <code class="language-plaintext highlighter-rouge">@noinline</code> annotations, but these should be a last-resort solution that you re-evaluate frequently as the compiler and the JVM improve.</p>

<p>The other key limitation of the optimizer is that you can only use it if you know your dependencies at run-time are the same as the ones you had at compile time.
For instance, if you compile against library <code class="language-plaintext highlighter-rouge">org.example</code> version <code class="language-plaintext highlighter-rouge">1.3.6</code>, and tomorrow the maintainers of <code class="language-plaintext highlighter-rouge">org.example</code> release version <code class="language-plaintext highlighter-rouge">1.3.7</code> that fixes a bug in a small function, this bugfix only has an effect on the already-deployed version of your code if it actually calls that function, not if the optimizer inlined the buggy version into your code.</p>

<p>Thus, the optimizer targets <em>application</em> code as well as the <em>standard library</em>, and is an opt-in compiler setting.
You should only use it for <em>library</em> code if you carefully select what inlining is allowed, as explained below.
Typically this means only inlining code within your own library, or within dependencies you control.</p>

<h2 id="using-the-optimizer">Using the optimizer</h2>

<p>Pass the flag <code class="language-plaintext highlighter-rouge">-opt</code> to the compiler to enable non-inlining optimizations, and <code class="language-plaintext highlighter-rouge">-opt-inline:...</code> arguments to enable inlining calls to specific packages.</p>

<p>For instance, <code class="language-plaintext highlighter-rouge">-opt-inline:**,!java.**,!org.example.*</code> tells the optimizer that inlining is allowed for all functions except those anywhere in the <code class="language-plaintext highlighter-rouge">java</code> package and those directly in the <code class="language-plaintext highlighter-rouge">org.example</code> package.
More details are available with <code class="language-plaintext highlighter-rouge">-opt-inline:help</code>.</p>

<h2 id="future-directions">Future directions</h2>

<p>Now that the optimizer is at feature parity with its Scala 2 incarnation, we hope to bring further optimizations to Scala 3.
For instance, <code class="language-plaintext highlighter-rouge">Range</code>-based abstractions cannot always be fully eliminated today, but could with further work on the optimizer.
If you’re interested, you can contribute either by participating as described below, or by <a href="https://github.com/scala/scala3/blob/main/CONTRIBUTING.md">contributing</a> to the compiler itself!</p>

<h2 id="participation">Participation</h2>

<p>The Scala Center has been entrusted with coordinating the commissioned Scala work for the Sovereign Tech Fund. The Scala Center is an independent, not-for-profit center sponsored by <a href="/blog/2023/09/11/scala-center-fundraising.html">corporate members and individual backers like you</a> to promote and facilitate Scala. If you would like to participate and/or see more of these types of efforts, please reach out to your manager to see if your company can donate engineering time or membership to the Scala Center.</p>

<p>See <a href="/blog/2023/09/11/scala-center-fundraising.html">The Scala Center Fundraising Campaign</a> for more details.</p>
