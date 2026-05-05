---
title: "Scala 3.8.2 is now available!"
url: "https://www.scala-lang.org/news/3.8.2/"
date: "2026-02-24T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<h1 id="scala-382-is-now-available">Scala 3.8.2 is now available!</h1>

<h2 id="release-highlights">Release highlights</h2>

<h3 id="warning-for-for-with-many-vals-and-overloaded-map-25090">Warning for <code class="language-plaintext highlighter-rouge">for</code> with many <code class="language-plaintext highlighter-rouge">val</code>s and overloaded <code class="language-plaintext highlighter-rouge">map</code> (<a href="https://github.com/scala/scala3/pull/25090">#25090</a>)</h3>

<p>Scala 3.8’s <strong>betterFors</strong> (available since 3.7 under <code class="language-plaintext highlighter-rouge">-preview</code>) changes for-comprehension desugaring and removes an intermediate <code class="language-plaintext highlighter-rouge">map</code> used for consecutive <code class="language-plaintext highlighter-rouge">val</code> bindings.</p>

<p>The following code snippet behaves differently at runtime depending on Scala version used for compilation. In Scala 3.7.x and earlier it produces <code class="language-plaintext highlighter-rouge">List((43,29), (43,30), (43,31))</code>, but starting with 3.8 it results in <code class="language-plaintext highlighter-rouge">Map(43 -&gt; 31)</code>.</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">def</span> <span class="nf">f</span><span class="o">(</span><span class="n">x</span><span class="k">:</span> <span class="kt">Int</span><span class="o">)</span><span class="k">:</span> <span class="o">(</span><span class="kt">Int</span><span class="o">,</span> <span class="kt">Int</span><span class="o">)</span> <span class="k">=</span> <span class="o">(</span><span class="mi">1</span><span class="o">,</span> <span class="n">x</span><span class="o">)</span>

<span class="k">val</span> <span class="nv">result</span><span class="k">:</span> <span class="kt">Iterable</span><span class="o">[(</span><span class="kt">Int</span>, <span class="kt">Int</span><span class="o">)]</span> <span class="k">=</span>
  <span class="k">for</span> <span class="c1">// warning: For comprehension with multiple val assignments may change result type</span>
    <span class="o">(</span><span class="n">k</span><span class="o">,</span> <span class="n">v</span><span class="o">)</span> <span class="k">&lt;-</span> <span class="nc">Map</span><span class="o">(</span><span class="mi">1</span> <span class="o">-&gt;</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">2</span> <span class="o">-&gt;</span> <span class="mi">1</span><span class="o">,</span> <span class="mi">3</span> <span class="o">-&gt;</span> <span class="mi">1</span><span class="o">)</span>
    <span class="n">x</span>      <span class="k">=</span> <span class="n">k</span> <span class="o">+</span> <span class="nf">v</span>
    <span class="o">(</span><span class="n">a</span><span class="o">,</span> <span class="n">b</span><span class="o">)</span> <span class="k">=</span> <span class="nf">f</span><span class="o">(</span><span class="n">x</span><span class="o">)</span>
    <span class="o">(</span><span class="n">y</span><span class="o">,</span> <span class="n">z</span><span class="o">)</span> <span class="k">&lt;-</span> <span class="nc">Map</span><span class="o">(</span><span class="mi">42</span> <span class="o">-&gt;</span> <span class="mi">27</span><span class="o">)</span>
  <span class="nf">yield</span> <span class="o">(</span><span class="n">a</span> <span class="o">+</span> <span class="n">y</span><span class="o">,</span> <span class="n">b</span> <span class="o">+</span> <span class="n">z</span><span class="o">)</span>
</code></pre></div></div>

<ul>
  <li><strong>Before (3.7):</strong> A synthetic tuple-producing <code class="language-plaintext highlighter-rouge">map</code> was inserted; that step could make <code class="language-plaintext highlighter-rouge">Map</code> select a generic <code class="language-plaintext highlighter-rouge">map</code> overload and become an <code class="language-plaintext highlighter-rouge">Iterable</code> (e.g. <code class="language-plaintext highlighter-rouge">List</code>)</li>
  <li><strong>In 3.8:</strong> That synthetic step is removed, so the conversion no longer happens and a different <code class="language-plaintext highlighter-rouge">map</code>/<code class="language-plaintext highlighter-rouge">flatMap</code> path can be selected.</li>
</ul>

<p>The previous runtime behaviour can be achieved by explicitly converting first <code class="language-plaintext highlighter-rouge">Map</code> to <code class="language-plaintext highlighter-rouge">Iterable</code> type or by compilation with <code class="language-plaintext highlighter-rouge">-source:3.7</code> settings.</p>

<p>The new warning highlights code where this migration risk exists.</p>

<h2 id="notable-changes">Notable changes</h2>

<ul>
  <li>
    <p>Support <code class="language-plaintext highlighter-rouge">:dep ...</code> to add library dependencies in the Scala REPL <a href="https://github.com/scala/scala3/pull/24131">#24131</a></p>
  </li>
  <li>
    <p>Upgrade to Scala.js 1.20.2 <a href="https://github.com/scala/scala3/pull/24898">#24898</a></p>
  </li>
  <li>
    <p>Bump Scala CLI to v1.12.2 (was v1.11.0) <a href="https://github.com/scala/scala3/pull/25217">#25217</a>:
New aliases for RC and nightly Scala versions.
See the Scala CLI release notes for additional details:
<a href="https://github.com/VirtusLab/scala-cli/releases/tag/v1.12.0">v1.12.0</a>,
<a href="https://github.com/VirtusLab/scala-cli/releases/tag/v1.12.1">v1.12.1</a> and
<a href="https://github.com/VirtusLab/scala-cli/releases/tag/v1.12.2">v1.12.2</a></p>
  </li>
</ul>

<p>For the complete list of changes and contributor credits, see the <a href="https://github.com/scala/scala3/releases/tag/3.8.2">release notes on GitHub</a>.</p>
