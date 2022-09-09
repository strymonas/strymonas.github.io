## Stream Fusion, to Completeness

__strymonas__ is the codename of a streaming library for OCaml and Scala that offers support for fast, bulk, in-memory processing. It is developed using the state of the art facilities of Multi-Stage Programming (MSP) for each language. The utmost goal of the library is to offer a streaming API that achieves stream fusion at the highest level without altering the compiler backend. It covers the combination of many interesting (and challenging) combinators. The OCaml flavor, depends on [BER MetaOCaml](http://okmij.org/ftp/ML/MetaOCaml.html), OCaml's dialect for MSP. The Scala one depends on [scala-lms](https://scala-lms.github.io/).

The approach is described in detail in the paper _Stream Fusion, to Completeness_, that will be presented at the 44th ACM SIGPLAN Symposium on Principles of Programming Languages ([POPL'17](http://conf.researchr.org/home/POPL-2017)) in Paris.

## Getting Started

The project is organized in three separate projects.

```bash
git clone https://github.com/strymonas/staged-streams.ocaml
git clone https://github.com/strymonas/staged-streams.scala
git clone https://github.com/strymonas/java8-benchmarks
```

### Prerequisites

To run each one of the three benchmark and test suites, please install the following:

1. Java 8 (JVM version of at least 1.8.0 65): from your system's package manager
1. Install `maven` the Apache build manager for Java projects
1. OCaml: from your system's package manager
1. OPAM: after you [install](https://opam.ocaml.org/doc/Install.html) OPAM you will need to initialize it  with ```opam init``` and resolve any dependencies with:
	- either the ```opam depext``` command
	- or your system's package manager (e.g., OML's dependencies)
1. sbt: install following [sbt's documentation](http://www.scala-sbt.org/0.13/docs/Setup.html)

### How to compile and run the benchmarks

#### MetaOCaml

The following will enable MetaOCaml, install dependencies, make the project and run unit tests.

```bash
$ opam update
$ opam switch 4.02.1+BER
$ eval "opam config env"
$ cd staging-streams.ocaml
$ opam install oasis batteries.2.3.0 ounit.2.0.0 oml.0.0.5
$ ./configure --enable-tests --enable-benchmarks
$ make all
```

To run the benchmarks via command line use the commands below:

```bash
$ ./benchmark_batteries.byte
$ ./benchmark_stream.byte
```

#### LMS
The following will fire-up the sbt console, compile and run the test suite.

```bash
$ cd staging-streams.scala
$ sbt
$ test # to run the property tests or
$ testOnly <name of test> # to run the property tests (tab completion works)
```

To run the benchmarks via the sbt command line:

```bash
$ cd staging-streams.scala
$ sbt
$ jmh:run -i 10 -wi 10 -f1 .* # to run all benchmarks
```

#### Java 8
To run the benchmarks via command line:

```bash
$ cd java8-benchmarks
$ mvn clean install
$ java -Xms6g -Xmx6g -XX:-TieredCompilation -jar target/benchmarks.jar -i 10 -wi 10 -f1 .* # to run all benchmarks
```

## Hello World

Developers can use our library as any other streaming/collection library. We assume the reader is familiar with: i) streaming APIs and ii) Multi-Stage Programming.

Firstly, the API is similar with F#'s ```Seq``` type, Java 8's ```Stream``` API, collection APIs from OCaml and OCaml Batteries and many more. Our library supports all the core combinators for pipeline construction. Creation of streams is realized with ```of_arr``` (from arrays) and ```unfold``` (for infinite streams). We support: ```fold```, ```map```, ```filter```, ```take```, ```flat_map``` and ```zip_with```.

Secondly, Multi-Stage Programming (MSP) is the core technique that this library follows. MSP is a meta-programming technique that allows a disciplined and safe form of run-time code generation. Staging, refers to the distinction of stages between compile- and run-time with more stages in-between. When more information about run-time values is available, staging can make a program profit in performance, by partially evaluating.

In staged-streams, this kind of (dynamic) information consists of the structure of the pipeline, the in-between combinators and the bodies of the lambdas used.

We prompt the user to read about MSP in the paper [A Gentle Introduction to Multi-stage Programming](https://www.cs.rice.edu/~taha/publications/journal/dspg04a.pdf), on [BER MetaOCaml](http://okmij.org/ftp/ML/MetaOCaml.html) and on [Scala-Lightweight Modular Staging (LMS)](https://scala-lms.github.io//index.html) which we use for our Scala implementation. The details of our technique and the benefits on the high level of stream fusion that is achieved, are included in the POPL17 paper.

### Hello World in OCaml!

In the following example we create a simple stream from an array of six elements in OCaml using MetaOCaml's staging annotations.

```ocaml
open Stream_combinators

let example =
	of_arr .<[| 0;1;2;3 |]>.
      |> filter (fun x   -> .<.~x mod 2 = 0>.)
      |> fold   (fun z a -> .<.~a :: .~z>.) .<[]>.

Runcode.run example;;
```

When the execution reaches the ```run``` method, an optimized version of the stream will be compiled (emitted) and executed (for optimal results, native code compilation must be used using ```ocamlopt```). The resulted code will consist of a tight, for-loop.

### Hello World in Scala!

With Scala LMS, the same example is demonstrated in the snippet below.

```scala
def example (xs : Rep[Array[Int]]) : Rep[Int] = Stream[Int](xs)
      .filter(d => d % 2 == 0)
      .fold(0, ((a : Rep[Int]) => (b : Rep[Int]) => a + b))

val example_s = compile(example)

example_s(Array(0, 1, 2, 3))
```

For more examples see the unit directories with the unit tests for [OCaml](https://github.com/strymonas/staged-streams.ocaml/test/?at=master) and [Scala](https://github.com/strymonas/staged-streams.scala/src/test/scala/StagedStreamSpec.scala?at=master&fileviewer=file-view-default).

## Bugs and Feedback

To discuss bugs, improvements and post questions please use our Github Issues.

## Team

- Oleg Kiselyov, [site](http://okmij.org/)
- Aggelos Biboudis, [@biboudis](https://twitter.com/biboudis), [site](https://biboudis.github.io/)
- Nick Palladinos, [@nickpalladinos](https://twitter.com/nickpalladinos)
- Yannis Smaragdakis, [site](https://yanniss.github.io/)
