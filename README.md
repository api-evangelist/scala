# Scala

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
