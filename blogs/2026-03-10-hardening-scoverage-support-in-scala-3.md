---
title: "Hardening Scoverage Support in Scala 3"
url: "https://www.scala-lang.org/blog/2026/03/11/scoverage.html"
date: "2026-03-11T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<blockquote>
  <p>This post covers work done under the <a href="https://www.scala-lang.org/blog/2026/01/27/sta-invests-in-scala.html">Sovereign Tech Fund investment</a> umbrella: <a href="https://contributors.scala-lang.org/t/scoverage-hardening/7352">Scoverage hardening</a>. The work is coordinated by the <a href="https://scala.epfl.ch/">Scala Center</a>.</p>
</blockquote>

<h2 id="summary">Summary</h2>

<p>Code coverage is a key part of maintaining high-quality Scala projects. We’ve recently made progress on making Scoverage more robust for Scala 3, expanding the way we test it and discovering and fixing new issues.</p>

<h2 id="background">Background</h2>

<p>Measuring how much of your Scala code is exercised by tests sounds simple until you try to do it yourself. That’s where code coverage tools come in. Code coverage is not optional for many organizations. As one of the QA metrics, coverage helps ensure reliability of the solution which is especially important in regulated sectors. Scoverage is the standard coverage tool for Scala, built directly into the Scala 3 compiler as a dedicated phase.</p>

<p>A crucial requirement for Scoverage’s wide industry adoption is that Scoverage itself is reliable and well-tested. The tool is already tested via its own dedicated test suite - however, that is not enough. Most tricky bugs happen not in isolation but at intersections of language features. Therefore, Scoverage needs to be tested in interaction with all of the language features to be truly considered reliable. Furthermore, all future changes to the compiler should be tested in interaction with Scoverage to ensure the tool remains compatible with the compiler.</p>

<p>To guarantee such a level of reliability, we have recently started a systematic rework of the testing strategy for Scoverage. This article reports on the progress we’ve made in 2026 so far.</p>

<h2 id="enabling-coverage-on-the-compiler-test-suite">Enabling coverage on the compiler test suite</h2>

<p>The first step was mapping out the current failures of Scoverage in interaction with other language features. The strategy for that was to enable Scoverage on the existing compiler test suite. This means, in addition to the usual CI run of the tests that we do for each new PR, now we run the same tests again, but with Scoverage enabled.</p>

<p>Under this mode, for each test we verify three things:</p>

<ul>
  <li>The compiler doesn’t crash</li>
  <li>The coverage output file is produced</li>
  <li>The coverage output file is valid and deserializable</li>
</ul>

<p>As a result, we have discovered <em>97 failing tests</em> when coverage instrumentation was switched on. Those tests are being addressed in the order of impact. They were disabled for the time being to allow the validation logic to be merged early, thus future-proofing Scoverage.</p>

<p>As for the rest of the tests, they are now running with coverage instrumentation for each PR, and passing them is a requirement for a PR to be merged to the compiler codebase. Furthermore, every newly added test to the compiler test suite is tested against Scoverage by default, thus preventing future PRs from unintentionally introducing Scoverage breakages.</p>

<p>Details of this work are in <a href="https://github.com/scala/scala3/pull/25009">PR #25009</a>.</p>

<h2 id="addressing-breakages-erased-values-and-the-purity-constraint">Addressing breakages: erased values and the purity constraint</h2>

<p>We have also taken the first steps to address discovered issues. A pattern that emerged is that failing tests were clustered around a few common root causes.</p>

<p>One of the root causes behind a significant cluster of failures was the interaction between coverage instrumentation and Scala 3’s capability system. Capabilities in Scala 3 are represented as erased values - values that exist at compile time but are eliminated at runtime. Because they are erased, they must be pure: they cannot have side effects, and the compiler enforces this constraint, failing compilation if that is not the case.</p>

<p>Coverage instrumentation works by injecting calls into the compiled code to record which expressions were executed. These calls introduce side effects. The erasure phase would then reject the result with an error.</p>

<p>The fix is conceptually simple: the coverage instrumentation phase now checks whether a value is erased and, if so, skips it. <em>23 tests</em> that previously failed under coverage are now passing as a result of this fix, opening the door to using Scoverage together with capabilities.</p>

<p>Details in <a href="https://github.com/scala/scala3/pull/25298">PR #25298</a>.</p>

<h2 id="expanding-the-testing-surface">Expanding the testing surface</h2>

<p>The Scala 3 compiler’s test suite is extensive. The strategy has been to enable coverage testing incrementally: start with the core tests, exclude the currently failing ones so regressions are caught from day one, then gradually expand the testing surface and fix discovered issues.</p>

<p>The latest expansion of the testing surface was done after fixing the first cluster of issues. Coverage instrumentation is now also exercised on additional test suites such as <code class="language-plaintext highlighter-rouge">rewrites</code>, <code class="language-plaintext highlighter-rouge">warn</code>, <code class="language-plaintext highlighter-rouge">explicit-nulls/pos</code>, <code class="language-plaintext highlighter-rouge">explicit-nulls/warn</code>, and <code class="language-plaintext highlighter-rouge">init</code> test suites. As a part of this expansion, we have discovered <em>13 new breakages</em> that are being addressed in the same manner.</p>

<p>Details in <a href="https://github.com/scala/scala3/pull/25385">PR #25385</a>.</p>

<h2 id="conclusion">Conclusion</h2>

<p>To ensure the reliability of Scoverage in the industry, we are continuing to expand the testing surface of the tool and fix discovered issues. We will be sharing more updates as the work continues. If you are actively using Scoverage, your feedback is welcome! You can join the discussion on <a href="https://contributors.scala-lang.org/t/scoverage-hardening/7352">contributors.scala-lang.org</a> or contribute Scoverage-related issues to the Scala 3 <a href="https://github.com/scala/scala3/issues">issue tracker</a>.</p>

<h2 id="participation">Participation</h2>

<p>The Scala Center has been entrusted with coordinating the commissioned Scala work for the Sovereign Tech Fund. The Scala Center is an independent, not-for-profit center sponsored by <a href="/blog/2023/09/11/scala-center-fundraising.html">corporate members and individual backers like you</a> to promote and facilitate Scala. If you would like to participate and/or see more of these types of efforts, please reach out to your manager to see if your company can donate engineering time or membership to the Scala Center.</p>

<p>See <a href="/blog/2023/09/11/scala-center-fundraising.html">The Scala Center Fundraising Campaign</a> for more details.</p>
