---
title: "Last mile towards sbt 2"
url: "https://www.scala-lang.org/blog/2026/04/14/last-mile-towards-sbt2.html"
date: "2026-04-14T00:00:00+02:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<blockquote>
  <p>This post covers work done under the <a href="https://www.scala-lang.org/blog/2026/01/27/sta-invests-in-scala.html">Sovereign Tech Fund investment</a> umbrella: <a href="https://contributors.scala-lang.org/t/sbt-2-production-ready-roadmap/7351">sbt 2 Stable Release and Maintenance</a>. The work is coordinated by the <a href="https://scala.epfl.ch/">Scala Center</a>.</p>
</blockquote>

<p>At conferences or on social media, the question we get most often is:</p>

<blockquote>
  <p>When is sbt 2 coming out?</p>
</blockquote>

<p>We’ll discuss the plan in this post. But let’s go over the status first.</p>

<h2 id="whats-new-in-sbt-2">What’s new in sbt 2?</h2>

<p>sbt 2 is a new major version of sbt. If you’re familiar with sbt 1.x, hopefully the jump is not too far, but we have pushed sbt to a more modern standard. The headline features are:</p>

<ul>
  <li>sbt 2.x uses Scala 3.x (rather than Scala 2.12) for build definitions and plugins (Both sbt 1.x and 2.x are capable of building Scala 2.x and 3.x)</li>
  <li>Embraces a simpler build.sbt via common settings</li>
  <li><code class="language-plaintext highlighter-rouge">test</code> changed to an incremental test</li>
  <li>Local and remote cache system that is Bazel-compatible</li>
  <li>Uses sbtn (native-image client) for faster startup</li>
  <li>Project matrix that can cross build subprojects in parallel</li>
</ul>

<p><br />
For more details, please check out the Scala Days 2025 talk <a href="https://www.youtube.com/watch?v=GM2ywMb4z7A">sbt 2.0: go big</a> that I gave in August 2025.</p>

<p>To share the progress thus far, we released the <a href="https://eed3si9n.com/sbt-2.0-ideas">sbt 2.0 ideas</a> post in 2023, with sbt 2.0.0-alpha7. After a few more years of development, we released a beta version <a href="https://eed3si9n.com/sbt-2.0.0-RC2">sbt 2.0.0-RC2</a> that’s ready for testing, around Scala Days 2025. Since then, we have been releasing more beta versions with both bug fixes and community-contributed feature enhancements. Perhaps surprising to some, sbt 2.x already started the binary compatibility from the RC series. This gave us a head start on repopulating the plugin ecosystem.</p>

<h2 id="repopulating-the-plugin-ecosystem">Repopulating the plugin ecosystem</h2>

<p>Thanks to the community effort, we already have <a href="https://www.scala-sbt.org/2.x/docs/en/community-plugins.html">60+ plugins</a> ported to sbt 2.x. This is amazing for a build tool that hasn’t been released yet. Special thanks to Kenji Yoshida for pull requests, preparing and porting many plugins to sbt 2.</p>

<p>Under the sbt 2 workstream, Anatolii from Scala Center has created <a href="/blog/2026/03/02/sbt2-compat.html">the sbt2-compat plugin</a>, which bridges the source-level differences between sbt 1.x and 2.x. This allows cross-building of a plugin, aiding the migration process. Also under the STA workstream, Rikito Taniguchi from VirtusLab has created a pull request to <a href="https://github.com/scala-js/scala-js/pull/5314">cross build Scala.JS plugin to sbt 2.x (scala-js#5314)</a>.</p>

<table>
  <thead>
    <tr>
      <th style="text-align: right;">Plugin</th>
      <th style="text-align: right;">Version</th>
      <th style="text-align: center;">Published</th>
      <th style="text-align: right;">Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: right;">Scala Native</td>
      <td style="text-align: right;"><code class="language-plaintext highlighter-rouge">0.5.11</code></td>
      <td style="text-align: center;">✅</td>
      <td style="text-align: right;"> </td>
    </tr>
    <tr>
      <td style="text-align: right;">sbt-assembly</td>
      <td style="text-align: right;"><code class="language-plaintext highlighter-rouge">2.3.1</code></td>
      <td style="text-align: center;">✅</td>
      <td style="text-align: right;"> </td>
    </tr>
    <tr>
      <td style="text-align: right;">Play</td>
      <td style="text-align: right;"><code class="language-plaintext highlighter-rouge">3.1.0-M9</code></td>
      <td style="text-align: center;">⚠️</td>
      <td style="text-align: right;">Pending scripted tests.</td>
    </tr>
    <tr>
      <td style="text-align: right;">Scala.JS</td>
      <td style="text-align: right;">n/a</td>
      <td style="text-align: center;">n/a</td>
      <td style="text-align: right;">Pending <a href="https://github.com/scala-js/scala-js/pull/5314">scala-js#5314</a></td>
    </tr>
  </tbody>
</table>

<p>Independently, the Play and Scala Native projects have been working towards sbt 2.x support as well.</p>

<h2 id="tooling-support-and-documentation">Tooling support and documentation</h2>

<p>Both IntelliJ Scala Plugin and Metals have published sbt plugins to import sbt 2.x projects.</p>

<p>Documentation has been reorganized and partly rewritten as <a href="https://www.scala-sbt.org/2.x/docs/en/">The book of sbt</a>, which has also been translated to <a href="https://www.scala-sbt.org/2.x/docs/ja/">Japanese</a> and <a href="https://www.scala-sbt.org/2.x/docs/zh-cn/">Chinese</a>.</p>

<h2 id="locking-down-to-20x-branch">Locking down to 2.0.x branch</h2>

<p>We have now created the <code class="language-plaintext highlighter-rouge">2.0.x</code> branch, so by default, all pull requests will target sbt 2.1. Only the critical bug fixes will be backported to the <code class="language-plaintext highlighter-rouge">2.0.x</code> branch.</p>

<ul>
  <li>On March 26, we released <a href="https://eed3si9n.com/sbt-2.0.0-RC10">sbt 2.0.0-RC10</a>, kicking off the last mile process.</li>
  <li>On April 7, we released <a href="https://eed3si9n.com/sbt-2.0.0-RC11">sbt 2.0.0-RC11</a>.</li>
  <li>On April 13, we released <a href="https://eed3si9n.com/sbt-2.0.0-RC12">sbt 2.0.0-RC12</a>.</li>
</ul>

<p><strong>Next steps</strong>: Please try using <a href="https://github.com/sbt/sbt/releases">the latest RC</a> on your projects, and check out the newly updated documentation. If you find bugs or missing documentation, please let us know by creating <a href="https://github.com/sbt/sbt/issues">an issue on GitHub</a>.</p>

<p>We will likely release a few more release candidates, but if no critical bugs are found, we will graduate one of them to the final release. So when is sbt 2 coming out? Depending on the bugs we discover, we are hopeful that it can happen in a few weeks to a few months.</p>

<h2 id="participation">Participation</h2>

<p>The Scala Center has been entrusted with coordinating the commissioned Scala work for the Sovereign Tech Fund. The Scala Center is an independent, not-for-profit center sponsored by <a href="/blog/2023/09/11/scala-center-fundraising.html">corporate members and individual backers like you</a> to promote and facilitate Scala. If you would like to participate and/or see more of these types of efforts, please reach out to your manager to see if your company can donate engineering time or membership to the Scala Center.</p>

<p>See <a href="/blog/2023/09/11/scala-center-fundraising.html">The Scala Center Fundraising Campaign</a> for more details.</p>
