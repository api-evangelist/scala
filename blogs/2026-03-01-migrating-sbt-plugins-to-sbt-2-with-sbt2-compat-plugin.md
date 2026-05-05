---
title: "Migrating sbt plugins to sbt 2 with sbt2-compat plugin"
url: "https://www.scala-lang.org/blog/2026/03/02/sbt2-compat.html"
date: "2026-03-02T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/feed/index.xml"
---
<blockquote>
  <p>This post covers work done under the <a href="https://www.scala-lang.org/blog/2026/01/27/sta-invests-in-scala.html">Sovereign Tech Fund investment</a> umbrella: <a href="https://contributors.scala-lang.org/t/sbt-2-production-ready-roadmap/7351">sbt 2 Stable Release and Maintenance</a>. The work is coordinated by the <a href="https://scala.epfl.ch/">Scala Center</a>.</p>
</blockquote>

<p>There’s an ongoing, community-driven effort to repopulate the sbt plugin ecosystem in preparation for the sbt 2 release. From sbt 1.x, plugin authors can cross publish against sbt 2.0 release candidates. To facilitate the plugin migration, we’ve created the <a href="https://github.com/sbt/sbt2-compat">sbt2-compat plugin</a>.</p>

<!-- more -->

<h2 id="the-plugincompat-pattern">The PluginCompat Pattern</h2>
<p>sbt 2 is a major upgrade from sbt 1 and makes breaking changes on the API level. As a plugin maintainer, you want to preserve compatibility with sbt 1 when migrating - so you want to cross-publish your plugin for sbt 1 and sbt 2. Ideally, you want to have the same codebase that compiles for both sbt 1 and sbt 2. However, due to the breaking changes to the API, this is not always possible. You end up with a situation where the same concept is expressed in a different way in sbt 1 and sbt 2.</p>

<p>One such example is how files are represented in sbt 1 and sbt 2. In sbt 1, everything is a <code class="language-plaintext highlighter-rouge">java.io.File</code>, whereas in sbt 2 the types are more granular depending on the context. For example, <code class="language-plaintext highlighter-rouge">HashedVirtualFileRef</code> is used for classpath entries, <code class="language-plaintext highlighter-rouge">VirtualFile</code> - for task outputs and caching artifacts, <code class="language-plaintext highlighter-rouge">VirtualFileRef</code> - for path-like references. These types are defined in the <code class="language-plaintext highlighter-rouge">xsbti</code> package.</p>

<p>The current approach to handling these discrepancies is to have a single codebase that compiles for both sbt 1 and sbt 2, and use the <a href="https://www.scala-sbt.org/2.x/docs/en/changes/migrating-from-sbt-1.x.html#the-plugincompat-technique">PluginCompat pattern</a> to provide a compatibility layer. For example, suppose you want to define a new task key that builds a JAR file and returns it.</p>

<p>In sbt 1, you would define it as:</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">lazy</span> <span class="k">val</span> <span class="nv">assembly</span> <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">java.io.File</span><span class="o">](</span><span class="s">"Builds a deployable JAR file"</span><span class="o">)</span>
</code></pre></div></div>

<p>In sbt 2, you would define it as:</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">lazy</span> <span class="k">val</span> <span class="nv">assembly</span> <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">xsbti.file.HashedVirtualFileRef</span><span class="o">](</span><span class="s">"Builds a deployable JAR file"</span><span class="o">)</span>
</code></pre></div></div>

<p>Since you want to use the same codebase for both sbt 1 and sbt 2, you would need to abstract over the file type and define it separately for each sbt version. So:</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c1">// src/main/scala-2.12/PluginCompat.scala</span>
<span class="k">type</span> <span class="kt">FileRef</span> <span class="o">=</span> <span class="nv">java</span><span class="o">.</span><span class="py">io</span><span class="o">.</span><span class="py">File</span>

<span class="c1">// src/main/scala-3/PluginCompat.scala</span>
<span class="k">type</span> <span class="kt">FileRef</span> <span class="o">=</span> <span class="nv">xsbti</span><span class="o">.</span><span class="py">file</span><span class="o">.</span><span class="py">HashedVirtualFileRef</span>
</code></pre></div></div>

<p>You would then be able to define the task using this unified API:</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">lazy</span> <span class="k">val</span> <span class="nv">assembly</span> <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">FileRef</span><span class="o">](</span><span class="s">"Builds a deployable JAR file"</span><span class="o">)</span>
</code></pre></div></div>

<p>This approach is a recurring pattern that you will encounter from plugin to plugin when attempting to cross-compile for sbt 1 and sbt 2. This is exactly the motivation behind the sbt2-compat plugin.</p>

<h2 id="cross-building-for-sbt-1-and-sbt-2-using-the-sbt2-compat-plugin">Cross-building for sbt 1 and sbt 2 using the sbt2-compat plugin</h2>
<p>The above encoding of the differences in the file API between sbt 1 and sbt 2 is already abstracted and ready to be reused in the <code class="language-plaintext highlighter-rouge">sbt2-compat</code> plugin. This plugin follows a similar pattern to the <a href="https://github.com/dwijnand/sbt-compat">sbt-compat</a> plugin that handled cross-builds between sbt 0.x and 1.x. As a plugin maintainer, instead of manually defining the <code class="language-plaintext highlighter-rouge">PluginCompat.scala</code> shim as described above, you would instead add the <code class="language-plaintext highlighter-rouge">sbt2-compat</code> plugin to your build and use the <code class="language-plaintext highlighter-rouge">PluginCompat</code> pattern as before, relying on the unified implementation provided by <code class="language-plaintext highlighter-rouge">sbt2-compat</code>.</p>

<h3 id="1-add-sbt2-compat-to-your-plugin">1. Add sbt2-compat to your plugin</h3>

<p>Add the plugin in your plugin’s <code class="language-plaintext highlighter-rouge">build.sbt</code> (<strong>not</strong> <code class="language-plaintext highlighter-rouge">project/plugins.sbt</code>):</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nf">addSbtPlugin</span><span class="o">(</span><span class="s">"com.github.sbt"</span> <span class="o">%</span> <span class="s">"sbt2-compat"</span> <span class="o">%</span> <span class="s">"&lt;version&gt;"</span><span class="o">)</span>
</code></pre></div></div>

<p>To cross-build for sbt 1 and sbt 2, see <a href="https://www.scala-sbt.org/2.x/docs/en/changes/migrating-from-sbt-1.x.html#cross-building-sbt-plugins">Cross building sbt plugins</a> in the official migration guide.</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nc">ThisBuild</span> <span class="o">/</span> <span class="n">crossScalaVersions</span> <span class="o">:=</span> <span class="nc">Seq</span><span class="o">(</span><span class="s">"3.8.1"</span><span class="o">,</span> <span class="s">"2.12.21"</span><span class="o">)</span>
<span class="nc">ThisBuild</span> <span class="o">/</span> <span class="n">scalaVersion</span> <span class="o">:=</span> <span class="nv">crossScalaVersions</span><span class="o">.</span><span class="py">head</span>

<span class="o">(</span><span class="n">pluginCrossBuild</span> <span class="o">/</span> <span class="n">sbtVersion</span><span class="o">)</span> <span class="o">:=</span> <span class="o">{</span>
  <span class="nv">scalaBinaryVersion</span><span class="o">.</span><span class="py">value</span> <span class="k">match</span> <span class="o">{</span>
    <span class="k">case</span> <span class="s">"2.12"</span> <span class="k">=&gt;</span> <span class="s">"1.12.3"</span>
    <span class="k">case</span> <span class="k">_</span>      <span class="k">=&gt;</span> <span class="s">"2.0.0-RC9"</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<p>For a concrete example, see <a href="https://github.com/sbt/sbt-assembly/blob/1c2f9c5eb38f86abc9516d93ed1f1ccd5a76e374/build.sbt">sbt-assembly</a>.</p>

<h3 id="2-use-sbt2-compat-to-handle-the-api-differences-between-sbt-1-and-sbt-2">2. Use sbt2-compat to handle the API differences between sbt 1 and sbt 2</h3>

<p>Let’s take a look at how <a href="https://github.com/sbt/sbt-assembly">sbt-assembly</a> uses sbt2-compat to cross-build for sbt 1 and sbt 2. Here is how it is applied:</p>

<p><strong>1. Task keys</strong> — Import <code class="language-plaintext highlighter-rouge">FileRef</code> and use it for task return types. Notice how the type comes from the <code class="language-plaintext highlighter-rouge">sbt2-compat</code> plugin and does not need to be defined manually. (<a href="https://github.com/sbt/sbt-assembly/blob/1c2f9c5eb38f86abc9516d93ed1f1ccd5a76e374/src/main/scala/sbtassembly/AssemblyKeys.scala#L10">AssemblyKeys.scala L6–10</a>):</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">import</span> <span class="nn">sbtcompat.PluginCompat.FileRef</span>

<span class="k">lazy</span> <span class="k">val</span> <span class="nv">assembly</span>                  <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">FileRef</span><span class="o">](</span><span class="s">"Builds a deployable über JAR"</span><span class="o">)</span>
<span class="k">lazy</span> <span class="k">val</span> <span class="nv">assemblyPackageScala</span>      <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">FileRef</span><span class="o">](</span><span class="s">"Produces the Scala artifact"</span><span class="o">)</span>
<span class="k">lazy</span> <span class="k">val</span> <span class="nv">assemblyPackageDependency</span> <span class="k">=</span> <span class="n">taskKey</span><span class="o">[</span><span class="kt">FileRef</span><span class="o">](</span><span class="s">"Produces the dependency artifact"</span><span class="o">)</span>
</code></pre></div></div>

<p><strong>2. Shared logic</strong> — Import compat helpers and use them for classpath handling, file conversion, and module metadata (<a href="https://github.com/sbt/sbt-assembly/blob/1c2f9c5eb38f86abc9516d93ed1f1ccd5a76e374/src/main/scala/sbtassembly/Assembly.scala#L225-L294">Assembly.scala L225–294</a>):</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">import</span> <span class="nn">sbtcompat.PluginCompat.</span><span class="o">{</span><span class="nc">FileRef</span><span class="o">,</span> <span class="nc">Out</span><span class="o">,</span> <span class="n">toNioPath</span><span class="o">,</span> <span class="n">toFile</span><span class="o">,</span> <span class="n">toOutput</span><span class="o">,</span> <span class="n">toNioPaths</span><span class="o">,</span> <span class="n">toFiles</span><span class="o">,</span> <span class="n">moduleIDStr</span><span class="o">,</span> <span class="n">parseModuleIDStrAttribute</span><span class="o">}</span>

<span class="c1">// Classpath iteration</span>
<span class="nf">val</span> <span class="o">(</span><span class="n">jars</span><span class="o">,</span> <span class="n">dirs</span><span class="o">)</span> <span class="k">=</span> <span class="nv">classpath</span><span class="o">.</span><span class="py">toVector</span>
  <span class="o">.</span><span class="py">sortBy</span><span class="o">(</span><span class="n">x</span> <span class="k">=&gt;</span> <span class="nf">toNioPath</span><span class="o">(</span><span class="n">x</span><span class="o">).</span><span class="py">toAbsolutePath</span><span class="o">().</span><span class="py">toString</span><span class="o">())</span>
  <span class="o">.</span><span class="py">partition</span><span class="o">(</span><span class="n">x</span> <span class="k">=&gt;</span> <span class="nv">ClasspathUtil</span><span class="o">.</span><span class="py">isArchive</span><span class="o">(</span><span class="nf">toNioPath</span><span class="o">(</span><span class="n">x</span><span class="o">)))</span>

<span class="c1">// Convert classpath entry to File for JarFile</span>
<span class="k">val</span> <span class="nv">jarFile</span> <span class="k">=</span> <span class="k">new</span> <span class="nc">JarFile</span><span class="o">(</span><span class="nf">toFile</span><span class="o">(</span><span class="n">jar</span><span class="o">))</span>

<span class="c1">// ModuleID from metadata</span>
<span class="k">val</span> <span class="nv">module</span> <span class="k">=</span> <span class="nv">jar</span><span class="o">.</span><span class="py">metadata</span><span class="o">.</span><span class="py">get</span><span class="o">(</span><span class="n">moduleIDStr</span><span class="o">)</span>
  <span class="o">.</span><span class="py">map</span><span class="o">(</span><span class="n">parseModuleIDStrAttribute</span><span class="o">)</span>
  <span class="o">.</span><span class="py">map</span><span class="o">(</span><span class="n">m</span> <span class="k">=&gt;</span> <span class="nc">ModuleCoordinate</span><span class="o">(</span><span class="nv">m</span><span class="o">.</span><span class="py">organization</span><span class="o">,</span> <span class="nv">m</span><span class="o">.</span><span class="py">name</span><span class="o">,</span> <span class="nv">m</span><span class="o">.</span><span class="py">revision</span><span class="o">))</span>
  <span class="o">.</span><span class="py">getOrElse</span><span class="o">(</span><span class="nc">ModuleCoordinate</span><span class="o">(</span><span class="s">""</span><span class="o">,</span> <span class="nv">jar</span><span class="o">.</span><span class="py">data</span><span class="o">.</span><span class="py">name</span><span class="o">.</span><span class="py">replaceAll</span><span class="o">(</span><span class="s">".jar"</span><span class="o">,</span> <span class="s">""</span><span class="o">),</span> <span class="s">""</span><span class="o">))</span>

<span class="c1">// Return task output</span>
<span class="nf">toOutput</span><span class="o">(</span><span class="n">builtAssemblyJar</span><span class="o">)</span>
</code></pre></div></div>

<p><strong>3. Disabling task caching</strong> — Use <code class="language-plaintext highlighter-rouge">Def.uncached</code> so classpath tasks read fresh values on sbt 2 (<a href="https://github.com/sbt/sbt-assembly/blob/1c2f9c5eb38f86abc9516d93ed1f1ccd5a76e374/src/main/scala/sbtassembly/AssemblyPlugin.scala#L86-L87">AssemblyPlugin.scala L86–87</a>):</p>

<div class="language-scala highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">import</span> <span class="nn">sbtcompat.PluginCompat._</span>

<span class="n">assembly</span> <span class="o">/</span> <span class="n">fullClasspath</span> <span class="o">:=</span> <span class="nv">Def</span><span class="o">.</span><span class="py">uncached</span><span class="o">((</span><span class="n">fullClasspath</span> <span class="nf">or</span> <span class="o">(</span><span class="nc">Runtime</span> <span class="o">/</span> <span class="n">fullClasspath</span><span class="o">)).</span><span class="py">value</span><span class="o">),</span>
<span class="n">assembly</span> <span class="o">/</span> <span class="n">externalDependencyClasspath</span> <span class="o">:=</span> <span class="nv">Def</span><span class="o">.</span><span class="py">uncached</span><span class="o">((</span><span class="n">externalDependencyClasspath</span> <span class="nf">or</span> <span class="o">(</span><span class="nc">Runtime</span> <span class="o">/</span> <span class="n">externalDependencyClasspath</span><span class="o">)).</span><span class="py">value</span><span class="o">),</span>
</code></pre></div></div>

<h2 id="project-status">Project status</h2>

<p>sbt2-compat currently covers the core API surface needed for cross-building plugins that work with files, classpaths, and packaging:</p>

<table>
  <thead>
    <tr>
      <th>sbt2-compat provides</th>
      <th>Use for</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">FileRef</code>, <code class="language-plaintext highlighter-rouge">Out</code>, <code class="language-plaintext highlighter-rouge">ArtifactPath</code></td>
      <td>Task key types, return types</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">toNioPath</code>, <code class="language-plaintext highlighter-rouge">toNioPaths</code>, <code class="language-plaintext highlighter-rouge">toFile</code>, <code class="language-plaintext highlighter-rouge">toOutput</code>, <code class="language-plaintext highlighter-rouge">toFileRefsMapping</code>, <code class="language-plaintext highlighter-rouge">toAttributedFiles</code></td>
      <td>Classpath and file conversion</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">moduleIDStr</code>, <code class="language-plaintext highlighter-rouge">artifactStr</code>, <code class="language-plaintext highlighter-rouge">parseModuleIDStrAttribute</code>, <code class="language-plaintext highlighter-rouge">parseArtifactStrAttribute</code></td>
      <td>Module metadata from classpath</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">Def.uncached</code></td>
      <td>Opt out of sbt 2 task caching</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">.name()</code> on <code class="language-plaintext highlighter-rouge">FileRef</code></td>
      <td>File name (via <code class="language-plaintext highlighter-rouge">FileRefOps</code> on sbt 1)</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">toDirectCredentials</code>, <code class="language-plaintext highlighter-rouge">credentialForHost</code></td>
      <td>Credentials handling</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">createScopedKey</code>, <code class="language-plaintext highlighter-rouge">setSetting</code></td>
      <td>ScopedKey / settings API</td>
    </tr>
    <tr>
      <td><code class="language-plaintext highlighter-rouge">attributedPutFile</code>, <code class="language-plaintext highlighter-rouge">attributedGetFile</code>, etc.</td>
      <td>Attributed metadata (typed vs string keys)</td>
    </tr>
  </tbody>
</table>

<p>Plugin-specific compat (e.g. custom disk caching, test API differences, Scala stdlib bridges like <code class="language-plaintext highlighter-rouge">Streamable</code>) stays in each plugin’s own <code class="language-plaintext highlighter-rouge">PluginCompat.scala</code>. sbt-assembly, for instance, still maintains local shims for its cache, test settings, and <code class="language-plaintext highlighter-rouge">PackageOption</code> types. The sbt2-compat <a href="https://github.com/sbt/sbt2-compat">README</a> documents this design and lists known caveats.</p>

<p><strong>Development model.</strong> sbt2-compat evolves iteratively by porting real-world plugins. Each port validates the existing API and may reveal missing compat methods. The idea is that as you port your plugins to sbt 2, you will discover gaps in the API that you can contribute back to sbt2-compat. For the exact development model, see the <a href="https://github.com/sbt/sbt2-compat">README</a>.</p>

<p><strong>Contributions are welcome!</strong> If you’re cross-building an sbt plugin for sbt 1 and sbt 2, the sbt 2 team would be happy to hear about your experiences! Share what worked, what didn’t, and what you wish sbt2-compat had. Pull requests with compat helpers that might help other plugins are welcome! You can share your experiences and participate in the discussion in the <a href="https://contributors.scala-lang.org/t/sbt-2-production-ready-roadmap/7351">sbt 2 production-ready roadmap</a> thread at the Scala Contributors forum.</p>

<h2 id="participation">Participation</h2>

<p>The Scala Center has been entrusted with coordinating the commissioned Scala work for the Sovereign Tech Fund. The Scala Center is an independent, not-for-profit center sponsored by <a href="/blog/2023/09/11/scala-center-fundraising.html">corporate members and individual backers like you</a> to promote and facilitate Scala. If you would like to participate and/or see more of these types of efforts, please reach out to your manager to see if your company can donate engineering time or membership to the Scala Center.</p>

<p>See <a href="/blog/2023/09/11/scala-center-fundraising.html">The Scala Center Fundraising Campaign</a> for more details.</p>
