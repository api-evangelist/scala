# Scala

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

A topic collection covering the Scala programming language ecosystem, including its standard library, key frameworks, and widely-used libraries. Scala is a strongly-typed, JVM-based language blending object-oriented and functional programming, widely used in big data engineering, distributed systems, fintech, and backend development. Scala 3.8 is the current major version (January 2026).

**Type:** Topic Collection
**Tags:** Big Data, Distributed Systems, Functional Programming, JVM, Programming Language, Scala, Scala 3, Type Safety

## APIs and Libraries

### Scala Standard Library API
Core Scala language API providing data structures, collections, concurrent primitives, and runtime utilities. Runs on JVM, JavaScript (Scala.js), and Native (Scala Native) runtimes.

- **Documentation:** [https://www.scala-lang.org/api/current/](https://www.scala-lang.org/api/current/)
- **GitHub:** [https://github.com/scala/scala](https://github.com/scala/scala)

### Akka API
Toolkit for building highly concurrent, distributed, and fault-tolerant applications using the Actor model. Includes Akka Actors, Akka HTTP, Akka Streams, and Akka Cluster.

- **Documentation:** [https://doc.akka.io/docs/akka/current/](https://doc.akka.io/docs/akka/current/)
- **GitHub:** [https://github.com/akka/akka](https://github.com/akka/akka)

### Akka HTTP API
Full server- and client-side HTTP stack built on Akka Streams. High-throughput, non-blocking HTTP handling with a powerful Scala DSL for routing and marshalling.

- **Documentation:** [https://doc.akka.io/docs/akka-http/current/](https://doc.akka.io/docs/akka-http/current/)
- **GitHub:** [https://github.com/akka/akka-http](https://github.com/akka/akka-http)

### Play Framework API
Reactive web framework for Scala (and Java) built on Akka. Provides MVC routing, template engine, WS client, and reactive database integrations for web applications and REST APIs.

- **Documentation:** [https://www.playframework.com/documentation/latest/Home](https://www.playframework.com/documentation/latest/Home)
- **GitHub:** [https://github.com/playframework/playframework](https://github.com/playframework/playframework)

### ZIO API
Type-safe, composable library for asynchronous and concurrent programming in Scala. Purely functional effect system with structured concurrency and a rich ecosystem (ZIO HTTP, ZIO Kafka, ZIO Schema).

- **Documentation:** [https://zio.dev/overview/](https://zio.dev/overview/)
- **GitHub:** [https://github.com/zio/zio](https://github.com/zio/zio)

### Cats API
Lightweight, modular library for functional programming in Scala. Provides type class abstractions (Functor, Monad, Applicative) for standard library types. Most widely used FP library in Scala (56% adoption).

- **Documentation:** [https://typelevel.org/cats/](https://typelevel.org/cats/)
- **GitHub:** [https://github.com/typelevel/cats](https://github.com/typelevel/cats)

### http4s API
Typeful, functional, streaming HTTP library built on cats-effect and fs2. Server and client abstractions with Blaze, Ember, Jetty, and Tomcat backends. Second most popular HTTP library (45% adoption).

- **Documentation:** [https://http4s.org/v1/docs/](https://http4s.org/v1/docs/)
- **GitHub:** [https://github.com/http4s/http4s](https://github.com/http4s/http4s)

### Slick API
Functional Relational Mapping (FRM) for Scala providing type-safe, composable database access. Supports PostgreSQL, MySQL, H2, SQLite, and more.

- **Documentation:** [https://scala-slick.org/doc/stable/](https://scala-slick.org/doc/stable/)
- **GitHub:** [https://github.com/slick/slick](https://github.com/slick/slick)

### Circe API
Most widely used JSON library for Scala, built on Cats. Provides encoding, decoding, traversal, and transformation of JSON values with automatic derivation for case classes.

- **Documentation:** [https://circe.github.io/circe/](https://circe.github.io/circe/)
- **GitHub:** [https://github.com/circe/circe](https://github.com/circe/circe)

### Apache Spark API
Dominant big data processing framework in the Scala ecosystem. Enables large-scale data processing, SQL analytics, streaming, and machine learning across distributed clusters.

- **Documentation:** [https://spark.apache.org/docs/latest/api/scala/](https://spark.apache.org/docs/latest/api/scala/)
- **GitHub:** [https://github.com/apache/spark](https://github.com/apache/spark)

### sbt Build Tool
Dominant build tool in the Scala ecosystem (90% adoption). sbt 2.0 release candidates show up to 41% faster startup. Supports incremental compilation, test frameworks, and a rich plugin ecosystem.

- **Documentation:** [https://www.scala-sbt.org/1.x/docs/](https://www.scala-sbt.org/1.x/docs/)
- **GitHub:** [https://github.com/sbt/sbt](https://github.com/sbt/sbt)

## Artifacts

### JSON Schema
- [Scala Library Schema](json-schema/scala-library-schema.json) — Schema for a Scala library catalog entry including Maven coordinates, Scala version compatibility, runtime support, and effect system integration.

### JSON Structure
- [Scala Library Structure](json-structure/scala-library-structure.json) — Structural documentation for Scala library catalog entries.

### JSON-LD Context
- [Scala Context](json-ld/scala-context.jsonld) — Linked data context mapping Scala ecosystem vocabulary.

### Vocabulary
- [Scala Vocabulary](vocabulary/scala-vocabulary.yml) — Domain vocabulary covering Actor Model, Case Class, Effect System, Fiber, For Comprehension, Monad, Pattern Matching, Sealed Trait, Type Class, and more.

### Examples
- [ZIO HTTP Server Example](examples/scala-zio-http-example.json) — Example of a simple ZIO HTTP server implementing a REST API in Scala 3.
- [http4s REST API with cats-effect](examples/scala-cats-effect-http4s-example.json) — Complete http4s HTTP server implementing a JSON REST API using cats-effect IO, circe, and Scala 3 syntax.

## Common Resources

- [Scala Language Website](https://www.scala-lang.org/)
- [Scala Documentation](https://docs.scala-lang.org/)
- [Scala Blog](https://www.scala-lang.org/blog/)
- [Scala Users Forum](https://users.scala-lang.org/)
- [Scala GitHub Organization](https://github.com/scala)
- [Scala Times Newsletter](https://scalatimes.com/)
- [Scala Discord](https://discord.gg/scala)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
**URL:** https://apievangelist.com
