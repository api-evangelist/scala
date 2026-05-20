---
title: "Scala 3.8.2 is now available!"
url: "https://www.scala-lang.org/news/3.8.2/"
date: "2026-02-24T00:00:00+01:00"
author: ""
feed_url: "https://www.scala-lang.org/rss.xml"
---
Scala 3.8.2 is now available! Release highlights Warning for for with many val s and overloaded map ( #25090 ) Scala 3.8’s betterFors (available since 3.7 under -preview ) changes for-comprehension desugaring and removes an intermediate map used for consecutive val bindings. The following code snippet behaves differently at runtime depending on Scala version used for compilation.
