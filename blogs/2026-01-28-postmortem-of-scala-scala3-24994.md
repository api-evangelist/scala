---
title: "Postmortem of scala/scala3#24994"
url: "https://www.scala-lang.org/blog/post-mortem-3.8.0.html"
date: "2026-01-29T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/rss.xml"
---
Incident Date: January 13th, 2026 Nature of the incident: The Scala 3.8.0 artifacts contain invalid references to private fields The TL;DR The Scala 3.8.0 artifacts were released with invalid references to private fields in its standard library. The bug, while severe, only affects a very limited group of users. A hotfix was implemented and included in a subsequent 3.8.1 release.
