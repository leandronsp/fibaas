# Fibonacci as a Service

Just a simple language benchmark with a naive implementation of Fibonacci served via HTTP.

```haskell
fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
```

# Results

Running on a Retina Macbook Pro i5@2.6GHz and 8GB RAM using [wrk](https://github.com/wg/wrk):

    $ wrk -c 64 -d 30s http://localhost:4000/10

| Framework      | Requests/sec | Total requests | Avg Latency | Memory footprint |
|----------------|--------------|----------------|-------------|------------------|
| Scotty         | 50048        | 1502738        | 2.90ms      | **26MB**         |
| Spock          | **57416**    | **1724062**    | **2.57ms**  | 28MB             |
| Snap           | 23756        | 715166         | 7.11ms      | 33MB             |
| Happstack Lite | 7888         | 237474         | 8.11ms      | 34MB             |

# Building

    $ cabal update
    $ cabal install --only-dependencies
    $ cabal configure
    $ cabal build

# Running

Following are the WEB frameworks we are benchmarking:

* [Scotty](https://github.com/scotty-web/scotty)
* [Spock](https://github.com/agrafix/Spock)
* [Snap](https://github.com/snapframework/snap)
* [Happstack Lite](https://github.com/Happstack/happstack-lite)

All the exectuables are built with maximum optimization, concurrency support, and a increased GC allocation size, as the GHC default's (512 Kb) is pretty unrealistic for production servers. Consult the `fibaas.cabal` file for details.

### Scotty

```haskell
import Fibonacci
import Web.Scotty
import Data.Text.Lazy (pack)

main :: IO ()
main = scotty 4000 $
  get "/:number" $ do
    number <- param "number"
    text . pack . show . fib $ number
```

    $ ./dist/build/fibaas-scotty/fibaas-scotty

### Spock

```haskell
import Fibonacci
import Web.Spock.Simple
import Data.Text (pack)

main :: IO ()
main = runSpock 4000 $ spockT id $
  get "/:number" $ do
    Just number <- param "number"
    text . pack . show . fib $ number
```

    $ ./dist/build/fibaas-spock/fibaas-spock

### Snap

```haskell
import Fibonacci
import Snap.Core
import Snap.Http.Server
import Data.ByteString.Char8 (pack, unpack)

fibHandler :: Snap ()
fibHandler = do
  Just param <- getParam "number"
  let number = read $ unpack param :: Int
      result = fib number in
    writeBS . pack . show $ result

main :: IO ()
main = do
  let config = setPort 4000 $
               setAccessLog ConfigNoLog $
               setErrorLog ConfigNoLog
               defaultConfig

  httpServe config $
    route [("/:number", fibHandler)]
```

    $ ./dist/build/fibaas-snap/fibaas-snap

### Happstack Lite

```haskell
import Fibonacci
import Happstack.Lite

main :: IO ()
main = do
  let config = defaultServerConfig { port = 4000 }
  serve (Just config) $
    path $ \(number) ->
      ok . toResponse . show . fib $ number
```

    $ ./dist/build/fibaas-happstack/fibaas-happstack
