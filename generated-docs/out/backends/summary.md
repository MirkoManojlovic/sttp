# Supported backends

sttp supports a number of synchronous and asynchronous backends. It's the backends that take care of managing connections, sending requests and receiving responses: sttp defines only the API to describe the requests to be send and handle the response data. Backends do all the heavy-lifting.

Choosing the right backend depends on a number of factors: whether you are using sttp to explore some data, or is it a production system; are you using a synchronous, blocking architecture or an asynchronous one; do you work mostly with Scala's `Future`, or maybe you use some form of a `Task` abstraction; finally, if you want to stream requests/responses, or not.

Which one to choose?

* for simple exploratory requests, use the [synchronous](synchronous.md) `HttpURLConnectionBackend`, or `HttpClientSyncBackend` if you are on Java11.
* if you have Akka in your stack, use [Akka backend](akka.md)
* otherwise, if you are using `Future`, use the `AsyncHttpClientFutureBackend` [Future](future.md) backend
* finally, if you are using a functional effect wrapper, use one of the "functional" backends, for [ZIO](zio.md), [Monix](monix.md), [Scalaz](scalaz.md), [cats-effect](catseffect.md) or [fs2](fs2.md). 

Each backend has two type parameters:

* `F[_]`, the effects wrapper for responses. That is, when you invoke `send(backend)` on a request description, do you get a `Response[_]` directly, or is it wrapped in a `Future` or a `Task`?
* `P`, the capabilities supported by the backend, in addition to `Effect[F]`. If `Any`, no additional capabilities are provided. Might include `Streams` (the ability to send and receive streaming bodies) and `WebSockets` (the ability to handle websocket requests).

Below is a summary of all the JVM backends; see the sections on individual backend implementations for more information:

```eval_rst
==================================== ============================ ================================================= ==========================
Class                                Effect type                  Supported stream type                             Supports websockets
==================================== ============================ ================================================= ==========================
``HttpURLConnectionBackend``         None (``Identity``)          n/a                                               no
``TryHttpURLConnectionBackend``      ``scala.util.Try``           n/a                                               no
``AkkaHttpBackend``                  ``scala.concurrent.Future``  ``akka.stream.scaladsl.Source[ByteString, Any]``  yes (regular & streaming)
``AsyncHttpClientFutureBackend``     ``scala.concurrent.Future``  n/a                                               yes (regular)
``AsyncHttpClientScalazBackend``     ``scalaz.concurrent.Task``   n/a                                               yes (regular)
``AsyncHttpClientZioBackend``        ``zio.Task``                 ``zio.stream.Stream[Throwable, Byte]``            yes (regular & streaming)
``AsyncHttpClientMonixBackend``      ``monix.eval.Task``          ``monix.reactive.Observable[ByteBuffer]``         yes (regular & streaming)
``AsyncHttpClientCatsBackend``       ``F[_]: cats.effect.Async``  n/a                                               no
``AsyncHttpClientFs2Backend``        ``F[_]: cats.effect.Async``  ``fs2.Stream[F, Byte]``                           yes (regular & streaming)
``OkHttpSyncBackend``                None (``Identity``)          n/a                                               yes (regular)
``OkHttpFutureBackend``              ``scala.concurrent.Future``  n/a                                               yes (regular)
``OkHttpMonixBackend``               ``monix.eval.Task``          ``monix.reactive.Observable[ByteBuffer]``         yes (regular & streaming)
``Http4sBackend``                    ``F[_]: cats.effect.Effect`` ``fs2.Stream[F, Byte]``                           no
``HttpClientSyncBackend``            None (``Identity``)          n/a                                               no
``HttpClientFutureBackend``          ``scala.concurrent.Future``  n/a                                               yes (regular)
``HttpClientMonixBackend``           ``monix.eval.Task``          ``monix.reactive.Observable[ByteBuffer]``         yes (regular & streaming)
``HttpClientZioBackend``             ``zio.RIO[Blocking, *]``     ``zio.stream.ZStream[Blocking, Throwable, Byte]`` yes (regular & streaming)
``FinagleBackend``                   ``com.twitter.util.Future``  n/a                                               no
==================================== ============================ ================================================= ==========================
```

The backends work with Scala 2.11, 2.12 and 2.13 (with some exceptions for 2.11). Moreover, `HttpURLConnectionBackend`, `AsyncHttpClientFutureBackend`, `AsyncHttpClientZioBackend`, `HttpClientSyncBackend`, `HttpClientFutureBackend` and `HttpClientZioBackend` are additionally built with Dotty (Scala 3).

There are also backends which wrap other backends to provide additional functionality. These include:

* `TryBackend`, which safely wraps any exceptions thrown by a synchronous backend in `scala.util.Try`
* `OpenTracingBackend`, for OpenTracing-compatible distributed tracing. See the [dedicated section](wrappers/opentracing.md).
* `PrometheusBackend`, for gathering Prometheus-format metrics. See the [dedicated section](wrappers/prometheus.md).
* slf4j backends, for logging. See the [dedicated section](wrappers/slf4j.md).

In addition, there are also backends for Scala.JS:

```eval_rst
================================ ============================ ========================================= ===================
Class                            Effect type                  Supported stream type                     Supports websockets
================================ ============================ ========================================= ===================
``FetchBackend``                 ``scala.concurrent.Future``  n/a                                       no
``FetchMonixBackend``            ``monix.eval.Task``          ``monix.reactive.Observable[ByteBuffer]`` no
================================ ============================ ========================================= ===================
```

And a backend for scala-native:

```eval_rst
================================ ============================ ========================================= ===================
Class                            Effect type                  Supported stream type                     Supports websockets
================================ ============================ ========================================= ===================
``CurlBackend``                  None (``Identity``)          n/a                                       no
================================ ============================ ========================================= ===================
```

Finally, there are third-party backends:

* [sttp-play-ws](https://github.com/ragb/sttp-play-ws) for "standard" play-ws (not standalone).
* [akkaMonixSttpBackend](https://github.com/fullfacing/akkaMonixSttpBackend), an Akka-based backend, but using Monix's `Task` & `Observable`.