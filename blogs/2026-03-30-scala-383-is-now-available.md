---
title: "Scala 3.8.3 is now available!"
url: "https://www.scala-lang.org/news/3.8.3/"
date: "2026-03-31T00:00:00+02:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<h1 id="scala-383-is-now-available">Scala 3.8.3 is now available!</h1>

<h2 id="release-highlights">Release highlights</h2>

<h3 id="local-coverage-exclusions-with--coverage-off-blocks-24486">Local coverage exclusions with <code class="language-plaintext highlighter-rouge">// $COVERAGE-OFF$</code> blocks (<a href="https://github.com/scala/scala3/pull/24486">#24486</a>)</h3>

<p>Coverage-instrumented builds can now disable coverage for a selected region of code, instead of excluding a whole file or class. This is useful for generated code, intentionally defensive branches, or support code that would otherwise distort coverage results.</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">//&gt; using scala 3.8.3</span>
<span class="c1">//&gt; using options --coverage-out coverage-data</span>

<span class="k">class</span> <span class="nc">Parser</span><span class="k">:</span>
  <span class="kt">def</span> <span class="kt">parse</span><span class="o">(</span><span class="kt">input:</span> <span class="kt">String</span><span class="o">)</span><span class="kt">:</span> <span class="kt">Int</span> <span class="o">=</span>
    <span class="nv">input</span><span class="o">.</span><span class="py">toInt</span>

  <span class="c1">// $COVERAGE-OFF$</span>
  <span class="k">def</span> <span class="nf">debugFallback</span><span class="o">(</span><span class="n">input</span><span class="k">:</span> <span class="kt">String</span><span class="o">)</span><span class="k">:</span> <span class="kt">Int</span> <span class="o">=</span>
    <span class="k">if</span> <span class="n">input</span> <span class="o">==</span> <span class="s">"zero"</span> <span class="n">then</span> <span class="mi">0</span>
    <span class="k">else</span> <span class="o">-</span><span class="mi">1</span>
  <span class="c1">// $COVERAGE-ON$</span>

<span class="nd">@main</span> <span class="k">def</span> <span class="nf">CoverageTest</span> <span class="k">=</span> <span class="o">{</span>
  <span class="k">val</span> <span class="nv">parser</span> <span class="k">=</span> <span class="nc">Parser</span><span class="o">()</span>
  <span class="nf">assert</span><span class="o">(</span><span class="nv">parser</span><span class="o">.</span><span class="py">parse</span><span class="o">(</span><span class="s">"42"</span><span class="o">)</span> <span class="o">==</span> <span class="mi">42</span><span class="o">)</span>
<span class="o">}</span>
</code></pre></div></div>

<p>Only the code between the markers is skipped by coverage instrumentation. The rest of the file is still measured as usual.</p>

<h3 id="safe-mode-for-capability-safe-code-25307">Safe mode for capability-safe code (<a href="https://github.com/scala/scala3/pull/25307">#25307</a>)</h3>

<p>Scala 3.8.3 introduces <strong>safe mode</strong>, a new experimental language subset that can be enabled with <code class="language-plaintext highlighter-rouge">import language.experimental.safe</code> or <code class="language-plaintext highlighter-rouge">-language:experimental.safe</code>. As described in the <a href="https://www.scala-lang.org/api/3.8.3/docs/experimental/capture-checking/safe.html">safe mode reference</a>, this is not just “stricter capture checking”: it is a capability-safe subset intended for agent-generated or otherwise untrusted code.</p>

<p>The underlying model is also described in the research paper <a href="https://arxiv.org/abs/2603.00991">Tracking Capabilities for Safer Agents</a>, which proposes using Scala 3 with capture checking as a programming-language-based safety harness for AI agents.</p>

<p>When safe mode is enabled, the compiler rejects unchecked casts and unchecked pattern matches, forbids escape hatches such as <code class="language-plaintext highlighter-rouge">caps.unsafe</code>, <code class="language-plaintext highlighter-rouge">@unchecked</code>, and runtime reflection, turns on capture checking with mutation tracking, and restricts access to global APIs unless they are known-safe or explicitly reviewed.</p>

<p>That last point is what makes the feature practical. Safe code is meant to call a restricted set of APIs directly, while effectful or implementation-dependent behavior can still be exposed through wrappers marked <code class="language-plaintext highlighter-rouge">@assumeSafe</code>. The implementation in <a href="https://github.com/scala/scala3/pull/25307">#25307</a> makes that boundary explicit: <code class="language-plaintext highlighter-rouge">@assumeSafe</code> declarations are themselves written outside safe mode, and safe code calls them from within the restricted subset.</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// app.scala</span>

<span class="c1">//&gt; using scala 3.8.3</span>
<span class="c1">//&gt; using file CheckedMailer.scala</span>
<span class="c1">//&gt; using options -experimental</span>

<span class="k">import</span> <span class="nn">language.experimental.safe</span>

<span class="k">object</span> <span class="nc">PotentiallyUnsafeApp</span><span class="k">:</span>
  <span class="kt">val</span> <span class="kt">address</span> <span class="o">=</span> <span class="nc">EmailAddress</span><span class="o">(</span><span class="s">"team@scala-lang.org"</span><span class="o">)</span>
  <span class="nv">CheckedMailer</span><span class="o">.</span><span class="py">send</span><span class="o">(</span><span class="n">address</span><span class="o">)</span>  <span class="c1">// ok</span>
  <span class="nf">println</span><span class="o">(</span><span class="n">address</span><span class="o">)</span>             <span class="c1">// error: rejected in safe mode</span>
  <span class="nv">address</span><span class="o">.</span><span class="py">asInstanceOf</span><span class="o">[</span><span class="kt">String</span><span class="o">]</span> <span class="c1">// error: rejected in safe mode</span>
  <span class="n">address</span> <span class="k">match</span>
    <span class="k">case</span> <span class="nc">EmailAddress</span><span class="o">(</span><span class="n">rawAddress</span><span class="o">)</span> <span class="k">=&gt;</span> <span class="o">???</span>  <span class="c1">// error: rejected in safe mode</span>
</code></pre></div></div>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// CheckedMailer.scala</span>

<span class="k">import</span> <span class="nn">scala.caps.assumeSafe</span>

<span class="nd">@assumeSafe</span>
<span class="k">object</span> <span class="nc">CheckedMailer</span><span class="k">:</span>
    <span class="kt">def</span> <span class="kt">send</span><span class="o">(</span><span class="kt">to:</span> <span class="kt">EmailAddress</span><span class="o">)</span> <span class="o">=</span>
        <span class="nv">scala</span><span class="o">.</span><span class="py">Console</span><span class="o">.</span><span class="py">out</span><span class="o">.</span><span class="py">println</span><span class="o">(</span><span class="n">s</span><span class="s">"Sending message to $to"</span><span class="o">)</span>

<span class="n">opaque</span> <span class="k">type</span> <span class="kt">EmailAddress</span> <span class="k">&lt;:</span> <span class="kt">String</span> <span class="o">=</span> <span class="nc">String</span>

<span class="k">object</span> <span class="nc">EmailAddress</span><span class="k">:</span>
    <span class="kt">@assumeSafe</span> <span class="kt">def</span> <span class="kt">apply</span><span class="o">(</span><span class="kt">value:</span> <span class="kt">String</span><span class="o">)</span><span class="kt">:</span> <span class="kt">EmailAddress</span> <span class="o">=</span> <span class="n">value</span>
    <span class="k">def</span> <span class="nf">unapply</span><span class="o">(</span><span class="n">value</span><span class="k">:</span> <span class="kt">EmailAddress</span><span class="o">)</span><span class="k">:</span> <span class="kt">Option</span><span class="o">[</span><span class="kt">String</span><span class="o">]</span> <span class="k">=</span> <span class="nc">Some</span><span class="o">(</span><span class="n">value</span><span class="o">)</span>
</code></pre></div></div>

<p>In the example above, the safe code in <code class="language-plaintext highlighter-rouge">app.scala</code> can call <code class="language-plaintext highlighter-rouge">CheckedMailer.send</code>, but the effectful operation is isolated behind an <code class="language-plaintext highlighter-rouge">@assumeSafe</code> boundary. By contrast, direct calls to <code class="language-plaintext highlighter-rouge">println</code>, unchecked <code class="language-plaintext highlighter-rouge">asInstanceOf</code> casts, or <code class="language-plaintext highlighter-rouge">scala.caps.unsafe</code> helpers are rejected in safe mode.</p>

<h3 id="scala-2-jvm-optimizer-ported-to-scala-3-25165">Scala 2 JVM optimizer ported to Scala 3 (<a href="https://github.com/scala/scala3/pull/25165">#25165</a>)</h3>

<p>Scala 3 now includes the port of the Scala 2 JVM backend optimizer. The optimizer is opt-in: compiler flag <code class="language-plaintext highlighter-rouge">-opt</code> enables local bytecode optimizations, while <code class="language-plaintext highlighter-rouge">-opt-inline:...</code> controls which classes and packages may be inlined across call sites. This brings Scala 3 to feature parity with the Scala 2 optimizer and opens the door to performance gains for JVM applications.</p>

<p>Rather than enabling blanket inlining everywhere, it is usually better to start from explicit filters. The <code class="language-plaintext highlighter-rouge">-opt-inline</code> setting accepts a comma-separated list of patterns; <code class="language-plaintext highlighter-rouge">**</code> matches all classes, <code class="language-plaintext highlighter-rouge">a.**</code> matches a package and its subpackages, <code class="language-plaintext highlighter-rouge">&lt;sources&gt;</code> matches classes compiled in the current run, and a leading <code class="language-plaintext highlighter-rouge">!</code> excludes matches. The last matching pattern wins.</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">//&gt; using scala 3.8.3</span>
<span class="c1">//&gt; using options -opt</span>
<span class="c1">//&gt; using options "-opt-inline:&lt;sources&gt;,my.app.**,!java.**,!org.example.**"</span>
<span class="c1">//&gt; using options "-Wopt:at-inline-failed-summary,no-inline-missing-bytecode"</span>
</code></pre></div></div>

<p>In this configuration, the optimizer may inline code from the current compilation (<code class="language-plaintext highlighter-rouge">&lt;sources&gt;</code>) and from <code class="language-plaintext highlighter-rouge">my.app</code> subpackages defined in external dependencies, but not from the JDK or the <code class="language-plaintext highlighter-rouge">org.example</code> packages. This is often a good starting point for applications. For libraries, the conservative choice is usually to inline only from <code class="language-plaintext highlighter-rouge">&lt;sources&gt;</code> or from packages you fully control.</p>

<p>The optimizer port also brings additional settings:</p>

<ul>
  <li><code class="language-plaintext highlighter-rouge">-Wopt:...</code> enables optimizer warnings. Available choices are <code class="language-plaintext highlighter-rouge">all</code> or <code class="language-plaintext highlighter-rouge">at-inline-failed-summary</code>, <code class="language-plaintext highlighter-rouge">at-inline-failed</code>, <code class="language-plaintext highlighter-rouge">any-inline-failed</code>, <code class="language-plaintext highlighter-rouge">no-inline-mixed</code>, <code class="language-plaintext highlighter-rouge">no-inline-missing-bytecode</code>, and <code class="language-plaintext highlighter-rouge">no-inline-missing-attribute</code></li>
  <li><code class="language-plaintext highlighter-rouge">-Yopt-specific:...</code> enables individual optimization passes such as <code class="language-plaintext highlighter-rouge">copy-propagation</code>, <code class="language-plaintext highlighter-rouge">box-unbox</code>, <code class="language-plaintext highlighter-rouge">nullness-tracking</code>, <code class="language-plaintext highlighter-rouge">closure-invocations</code>, or <code class="language-plaintext highlighter-rouge">redundant-casts</code></li>
  <li><code class="language-plaintext highlighter-rouge">-Yopt-inline-heuristics:default|everything|at-inline-annotated</code> adjusts how aggressively the compiler chooses call sites for inlining</li>
  <li><code class="language-plaintext highlighter-rouge">-Yopt-log-inline:&lt;prefix&gt;</code> logs inliner activity for matching methods</li>
  <li><code class="language-plaintext highlighter-rouge">-Yopt-trace:&lt;prefix&gt;</code> traces optimizer progress for matching methods</li>
</ul>

<p>The <code class="language-plaintext highlighter-rouge">-Wopt</code> options let you choose between a one-line summary for failed <code class="language-plaintext highlighter-rouge">@inline</code> calls, detailed per-callsite diagnostics, reporting for heuristic inlining failures, and warnings for cases where inlining could not even be decided because bytecode or Scala inline metadata was unavailable.</p>

<blockquote>
  <p>The <code class="language-plaintext highlighter-rouge">-Y...</code> optimizer flags are primarily intended for debugging and internal use. As with other <code class="language-plaintext highlighter-rouge">-Y</code> settings, they are not stable user-facing interfaces, and their exact behavior may change between releases.</p>
</blockquote>

<p>As with the Scala 2 optimizer, inlining external code comes with a compatibility trade-off: if you compile against one version of a dependency and later run against a different one, any inlined bytecode will not pick up the dependency’s runtime bug fixes or behavior changes. In practice, that means aggressive cross-library inlining is best reserved for applications with tightly controlled runtime classpaths. Read more about binary compatibility of optimized code in the <a href="https://docs.scala-lang.org/overviews/compiler-options/optimizer.html#binary-compatibility">Scala 2 optimizer documentation</a></p>

<p>The long-term plan is to build on this work in Scala 3.9 by enabling optimizations for the Scala standard library and the compiler itself.</p>

<h3 id="-print-lines-is-deprecated-for-removal-but-remains-accepted-as-a-no-op-25330"><code class="language-plaintext highlighter-rouge">-print-lines</code> is deprecated for removal, but remains accepted as a no-op (<a href="https://github.com/scala/scala3/pull/25330">#25330</a>)</h3>

<p>Scala 3.8.3 restores the <code class="language-plaintext highlighter-rouge">-print-lines</code> flag for compatibility, but only as a deprecated no-op. This avoids breaking existing builds in a patch release while giving users time to remove the setting from their build definitions.</p>

<p>The flag no longer has any effect and is scheduled for removal in Scala 3.9.0.</p>

<hr />

<p>For the complete list of changes and contributor credits, see the <a href="https://github.com/scala/scala3/releases/tag/3.8.3">release notes on GitHub</a>.</p>
