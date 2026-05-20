---
title: "Scala 3.8.3 is now available!"
url: "https://www.scala-lang.org/news/3.8.3/"
date: "2026-03-31T00:00:00+02:00"
author: ""
feed_url: "https://www.scala-lang.org/rss.xml"
---
Scala 3.8.3 is now available! Release highlights Local coverage exclusions with // $COVERAGE-OFF$ blocks ( #24486 ) Coverage-instrumented builds can now disable coverage for a selected region of code, instead of excluding a whole file or class. This is useful for generated code, intentionally defensive branches, or support code that would otherwise distort coverage results. //> using scala 3.8.3 //> using options --coverage-out coverage-data class Parser : def parse ( input: String ) : Int = input . toInt // $COVERAGE-OFF$ def debugFallback ( input : String ) : Int = if input == "zero" then…
