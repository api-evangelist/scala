---
title: "State of the TASTy reader and Scala 2.13 ↔ Scala 3 compatibility"
url: "https://www.scala-lang.org/blog/state-of-tasty-reader.html"
date: "2026-02-20T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<p>With the release of <a href="https://scala-lang.org/news/3.8/">Scala 3.8</a>, Scala 2.13 and Scala 3 interoperability is no longer bidirectional.</p>

<p>Every <strong>Scala 3 version supports consuming Scala 2.13</strong> artifacts. There are no reasons or plans to change that state.</p>

<p>The <strong>Scala 2.13 TASTy reader</strong> (<code class="language-plaintext highlighter-rouge">-Ytasty-reader</code>) remains useful for migrations and consuming Scala 3 artifacts, but it <strong>will never be able to consume Scala 3.8</strong> and later artifacts. 
Scala 3.7 is the last minor version whose artifacts will remain consumable from Scala 2.</p>

<p>This post summarizes the current state, the compatibility boundaries, and the recommended publishing strategy for teams that still need Scala 2.13 to consume Scala 3 libraries.</p>

<h2 id="what-is-the-tasty-reader">What is the TASTy reader?</h2>

<p><a href="https://docs.scala-lang.org/scala3/guides/tasty-overview.html">TASTy</a> (Typed Abstract Syntax Trees) is the high-level interchange format used by Scala 3.
Every Scala 3 compilation produces <code class="language-plaintext highlighter-rouge">.tasty</code> files alongside <code class="language-plaintext highlighter-rouge">.class</code> files.
While <code class="language-plaintext highlighter-rouge">.class</code> files lose type information due to JVM type erasure (e.g. <code class="language-plaintext highlighter-rouge">List[String]</code> becomes <code class="language-plaintext highlighter-rouge">List[Object]</code>), TASTy files preserve the complete type information — generic types, union types, intersection types, and more.</p>

<p>The <strong>TASTy reader</strong> is a feature built into the Scala 2.13 compiler, enabled with the <code class="language-plaintext highlighter-rouge">-Ytasty-reader</code> flag.
It allows a Scala 2.13 project to depend on libraries only published for Scala 3, by reading their <code class="language-plaintext highlighter-rouge">.tasty</code> files to reconstruct the precise type signatures that <code class="language-plaintext highlighter-rouge">.class</code> files alone cannot convey.</p>

<p>The TASTy reader was designed as a <strong>migration aid</strong> — not a permanent compatibility layer.
It addressed a critical chicken-and-egg problem during the Scala 3 ecosystem bootstrap: library authors wanted to publish for Scala 3 but couldn’t abandon Scala 2.13 users, while application developers on Scala 2.13 couldn’t migrate until their dependencies were available.
The TASTy reader broke this deadlock by enabling a <a href="https://www.scala-lang.org/blog/2020/11/19/scala-3-forward-compat.html#forward-compatibility">forward-compatibility</a> path, allowing Scala 2.13 projects to consume Scala 3-only libraries directly.</p>

<h2 id="technical-limitations-of-the-scala-2-tasty-reader">Technical limitations of the Scala 2 TASTy reader</h2>

<p>Even though both Scala 2 and Scala 3 share most of the language features, some features of each cannot be used by the other.
As an example, macros or existential types produced by Scala 2 cannot be represented or consumed by Scala 3.</p>

<p>The Scala 2 TASTy reader is able to consume the Scala 3 TASTy format, but is not able to correctly represent an increasing amount of language features in a semantically correct way. Some of the unsupported features include:</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">inline</code> (including Scala 3 macros)</li>
  <li>Union types</li>
  <li>Match types</li>
  <li>Context functions</li>
  <li>Polymorphic function types</li>
  <li>Trait parameters</li>
</ul>

<p>The classpath compatibility guide documents which Scala 3 features are or are not representable for Scala 2 consumption:
<a href="https://docs.scala-lang.org/scala3/guides/migration/compatibility-classpath.html">Scala 3 migration guide - classpath compatibility</a>.</p>

<h2 id="what-changed-in-scala-38">What changed in Scala 3.8</h2>

<p>Scala 3.8 introduced a major ecosystem change by publishing <code class="language-plaintext highlighter-rouge">scala-library</code> artifacts compiled using Scala 3, instead of reusing the Scala 2.13 standard library. As a result, it introduced a major new dependency that needs to be covered by the TASTy reader when consuming from Scala 2.</p>

<p>What’s more, Scala 3 contained a mechanism to patch parts of the Standard Library to improve its performance, with its <code class="language-plaintext highlighter-rouge">scala3-library_3</code> replacements. One such example is <a href="https://github.com/scala/scala3/blob/3.8.1/library/src/scala/Predef.scala#L314-L333"><code class="language-plaintext highlighter-rouge">scala.Predef.assert</code></a>, defined as a normal method in Scala 2 but replaced with a <code class="language-plaintext highlighter-rouge">transparent inline</code> variant in Scala 3. Inlines are one of the features that Scala 2 cannot support.
Starting with Scala 3.8, this mechanism was removed and replaced with direct standard library modifications after ensuring both source and binary backward compatibility.</p>

<p>The standard library of Scala 3 is now compiled with enabled support for explicit-nulls and capture-checking. Unless explicitly enabled, this change is not visible to users. However, at the TASTy level, each of these introduces additional features that cannot be represented in a Scala 2 compatible way: union types and capture checking.</p>

<p>As a result, the old “Scala 2.13 ↔ Scala 3 sandwich” is no longer bidirectional at the classpath level.</p>

<h2 id="compile-classpath-incompatibilities">Compile Classpath incompatibilities</h2>

<p>Both Scala 2 and Scala 3 make strict assumptions about what’s available on the classpath and assume usage of <code class="language-plaintext highlighter-rouge">scala-library</code> artifacts matching <strong>exactly</strong> the version of <code class="language-plaintext highlighter-rouge">scala-compiler</code>. This is required for correct emission of references to members of the Scala Standard Library.
Usage of an incompatible version can lead to severe problems resulting in compilation crashes. On the Scala 2 side, it can trigger bytecode optimizer failures, as described in the <a href="https://scala-lang.org/blog/post-mortem-3.8.0.html">Scala 3.8.0 post-mortem</a>. On the Scala 3 side, it can lead to <a href="https://github.com/scala/scala3/issues/22890">references to members not yet introduced</a> or already removed from the actively developed standard library.
This requirement can easily be violated due to eviction rules of third-party dependencies and transitive dependencies on different versions of the Standard Library.</p>

<p>Eviction of the Standard Library can also affect build tools. As an example, <code class="language-plaintext highlighter-rouge">sbt</code> has always ensured the use of matching versions of <code class="language-plaintext highlighter-rouge">scala-compiler</code>, <code class="language-plaintext highlighter-rouge">scala-library</code>, and <code class="language-plaintext highlighter-rouge">scala-reflect</code> to ensure their compatibility. Their perspective was recently documented in the sbt 1.12.3 release notes <a href="https://eed3si9n.com/sbt-1.12.3#sm%C3%B8rrebr%C3%B8d---the-end-of-scala-213-3x-sandwich">“Smørrebrød - the end of Scala 2.13-3.x sandwich”</a>.
In that scenario, a Scala 2.13 project can pull in <code class="language-plaintext highlighter-rouge">scala-library:3.8.x</code> through <code class="language-plaintext highlighter-rouge">for2_13Use3</code>, while sbt still needs aligned Scala 2 compiler artifacts for the 2.13 side.
That leads resolution to attempt <code class="language-plaintext highlighter-rouge">org.scala-lang:scala-reflect:3.8.x</code> (and <code class="language-plaintext highlighter-rouge">scala-compiler:3.8.x</code>), but these artifacts do not exist.
The result is the well-known sbt update error: <code class="language-plaintext highlighter-rouge">Error downloading org.scala-lang:scala-reflect:3.8.x</code>.</p>

<h2 id="hard-compatibility-guarantee">Hard compatibility guarantee</h2>

<p><strong>Scala 3 consuming Scala 2.13 artifacts will remain supported for the foreseeable future.</strong></p>

<p>That direction of compatibility is a hard requirement and remains part of the Scala 3 migration model.
The no-longer-supported direction is specifically Scala 2.13 consuming Scala 3.8+ artifacts through the TASTy reader.</p>

<h2 id="stable-tasty-reader-compatibility-scala-2136">Stable TASTy reader compatibility (Scala 2.13.6+)</h2>

<p>The table below starts at Scala 2.13.6 and covers stable, non-experimental TASTy support milestones.</p>

<table>
  <thead>
    <tr>
      <th>Scala 2.13 release</th>
      <th>Scala 3 minor version supported via <code class="language-plaintext highlighter-rouge">-Ytasty-reader</code></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2.13.6</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>2.13.7</td>
      <td>3.1</td>
    </tr>
    <tr>
      <td>2.13.9</td>
      <td>3.2</td>
    </tr>
    <tr>
      <td>2.13.11</td>
      <td>3.3</td>
    </tr>
    <tr>
      <td>2.13.13</td>
      <td>3.4</td>
    </tr>
    <tr>
      <td>2.13.15</td>
      <td>3.5</td>
    </tr>
    <tr>
      <td>2.13.16</td>
      <td>3.6</td>
    </tr>
    <tr>
      <td>2.13.17</td>
      <td>3.7</td>
    </tr>
  </tbody>
</table>

<p>Each Scala 2.13 release can consume artifacts from all Scala 3 versions up to and including the one listed in its row. For example, Scala 2.13.17 can consume artifacts published with Scala 3.0 through 3.7, but not forward-incompatible 3.8 or later. Intermediate Scala 2.13 releases between these milestones follow the most recent support level above.</p>

<h2 id="recommendations">Recommendations</h2>

<h3 id="for-scala-213-users">For Scala 2.13 users</h3>

<ol>
  <li>Keep TASTy-reader-based dependencies on versions published using Scala 3.7 or below.</li>
  <li>Prefer dependencies published on Scala 3.3 LTS when available.</li>
  <li>Plan migration of remaining Scala 2.13 modules to Scala 3 to remove reliance on the TASTy compatibility layer.</li>
</ol>

<h3 id="for-scala-3-library-authors-supporting-scala-213-users">For Scala 3 library authors supporting Scala 2.13 users</h3>

<p>We recommend cross-compilation of libraries for Scala 2.13 and Scala 3 so that each series can be natively consumed.</p>

<p>However, if that’s not possible we recommend for library authors:</p>
<ol>
  <li>Publish Scala 3 artifacts on <strong>Scala 3.3 LTS</strong> for that compatibility path.</li>
  <li>Scala 3.3 LTS will remain actively supported until at least Q2 2027 (at least one year after Scala 3.9 LTS is released), making it the safest choice for the transition period.</li>
  <li>Prefer publishing library using Scala 3.3 LTS - Scala 3.7 is not expected to receive further releases. Use Scala 3 Next series only if access to their features features is necessary.</li>
  <li>Plan and communicate potential dropping of Scala 2.13 support with the users, before migrating to Scala 3.9 LTS.</li>
</ol>

<h2 id="summary">Summary</h2>

<p>The TASTy reader remains a migration bridge for Scala 2.13 with Scala 3.0 to 3.7 artifacts.
That bridge does not extend to Scala 3.8+ artifacts.</p>

<p>At the same time, one direction is unchanged and guaranteed:
<strong>Scala 3 consuming Scala 2.13 artifacts is always supported.</strong></p>

<p>For mixed ecosystems that still need Scala 2.13 consumers, Scala 3.3 LTS is the recommended publishing baseline, with newer lines used only when post-3.3 features are required.</p>
