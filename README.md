# Helm

![Logo](docs/src/img/logo.png)

[![Build Status](https://travis-ci.org/Verizon/helm.svg?branch=master)](https://travis-ci.org/Verizon/helm)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.verizon.helm/core_2.11/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.verizon.helm/core_2.11)
[![codecov](https://codecov.io/gh/Verizon/helm/branch/master/graph/badge.svg)](https://codecov.io/gh/Verizon/helm)

A native [Scala](http://scala-lang.org) client for interacting with [Consul](https://www.consul.io/). There is currently one supported client, which uses [http4s](http://http4s.org) to make HTTP calls to Consul. Alternative implementations could be added with relative ease by providing an additional free interpreter for the `ConsulOp` algebra.

## Getting Started

Add the following to your `build.sbt`:

    libraryDependencies += "io.verizon.helm" %% "core"   % "4.0.0"
    libraryDependencies += "io.verizon.helm" %% "http4s" % "4.0.0"

The *Helm* binaries are located on maven central, so no additional resolvers are needed.

### Algebra

Consul operations are specified by the `ConsulOp` algebra.  Two
examples are `get` and `set`:

```
import ConsulOp.ConsulOpF

val setAlgebra: ConsulOpF[Unit] = ConsulOp.kvSet(key = "testKey", value = "testValue".getBytes)

val getAlgebra: ConsulOpF[QueryResponse[List[KVGetResult]]] = ConsulOp.kvGet(key = "testKey",
                                                                             recurse = Some(false),
                                                                             datacenter = None,
                                                                             separator = None,
                                                                             index = None,
                                                                             maxWait = None)
```

These are however just descriptions of what operations the program might perform in the future, just creating these operations does not
actually execute the operations. In order to perform the gets and sets, we need to use the [http4s](http://http4s.org) interpreter.

### The http4s interpreter

First we create an interpreter, which requires an http4s client and
a base url for consul:

```
import cats.effect.IO
import helm.http4s._
import org.http4s.Uri.uri
import org.http4s.client.blaze.Http1Client

val client = Http1Client[IO]().unsafeRunSync
val baseUrl = uri("http://127.0.0.1:8500")

val interpreter = new Http4sConsulClient(baseUrl, client)
```

Now we can apply commands to our http4s client to get back IOs
which actually interact with consul.

```
import helm._
import cats.effect.IO


val set: IO[Unit]                             = helm.run(interpreter, setAlgebra)
val get: IO[QueryResponse[List[KVGetResult]]] = helm.run(interpreter, getAlgebra)

// actually execute the calls
val setResult: Unit =                             set.unsafeRunSync()
val getResult: QueryResponse[List[KVGetResult]] = get.unsafeRunSync()

```

Typically, the *Helm* algebra would be a part of a `Coproduct` with other algebras in a larger program, so running the `IO` immediately after `helm.run` is not typical.

## Contributing

Contributions are welcome; particularly to expand the algebra with additional operations that are supported by Consul but not yet supported by *Helm*.

