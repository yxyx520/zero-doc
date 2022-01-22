---
sidebar_position: 1
---

# rest

### Overview

From daily development experience, a good web framework needs to meet the following features in general.

* route matching/multi-route support
* support for custom middleware
* complete decoupling of framework and business development to facilitate rapid development
* parameter validation / matching
* monitoring / logging / metrics and other service self-checking features
* Service self-protection 

### rest overview

rest has the following characteristics:

* initialize resources with `context` (different from `context` of `gin`) → save them in `serviveCtx` and share them in `handler` (as for resource pooling, leave it to the resources themselves, `serviveCtx` is just the entry point and sharing point)
* independent router declaration file, and add the concept of router group, convenient for developers to organize the code structure
* Built-in middleware: monitoring/fusing/forensics, etc.
* Use goctl codegen + option design pattern, convenient for developers to control part of the middleware access

The following diagram depicts the pattern and most of the processing paths for rest to handle requests.

* The framework's built-in middleware already helps developers to solve most of the self-processing logic of the service
* Also go-zero gives developers out-of-the-box components at the business logic (dq, fx, etc.)
* from the development model to help developers only need to focus on their business logic and the required resources to prepare

![rest](/img/rest.png)

### Startup process

The following diagram depicts the modules and the general flow of the overall server startup. Prepare to analyze the rest implementation according to the following flow.

* Based on http.server encapsulation and modification: separating engine (web framework core) and option
* radix-tree construction for multi-route matching
* middleware using the onion model → []Middleware
* http parse parsing and match-checking → httpx.Parse()
* Metrics (createMetrics()) and monitoring buried sites (prometheus) are collected during the request process

![rest_start](/img/rest_start.png)

#### server engine

The engine is used throughout the server life cycle.

* router will carry a developer-defined path/handler that will be executed at the end of router.handle()
* Registered custom middleware + framework middleware, executed before the router handler logic

Here: go-zero processing granularity is on route, wrapping and processing is performed at route level

![server_engine](/img/server_engine.jpeg)

### Routing Matching

So when the request arrives, how does it get to the routing layer in the first place?

First of all, in the development of the most primitive http server, there is a piece of code like this.

![basic_server](/img/basic_server.png)

`http.ListenAndServe()`  Internally it will execute to：`server.ListenAndServe()`

Let's see how this works in the rest.

![rest_route](/img/rest_route.png)

The handler passed in is actually the router generated by router.NewRouter(), which carries the entire set of handler functions for the server.

At the same time, the http.Server structure is initialized with the handler injected into it.

![rest_route](/img/rest_handle.png)

After the http.Server receives the req, the final execution is also：`handler.ServeHTTP(rw, req)`

![rest_route](/img/servehttp.png)

So the built-in `router` Also need to achieve `ServeHTTP` . As for `router` How to achieve it yourself `ServeHTTP` :It's just a matter of finding a matching route and then executing the route's corresponding `handle logic`.

### Parameter analysis

Parsing arguments is a basic capability that the http framework needs to provide. In the code generated by goctl code gen, the req argument parse function is already integrated in the handler layer.

![rest_route](/img/rest_parse.png)

Go to `httpx.Parse()` , The main analysis of the following pieces：

```go title="https://github.com/zeromicro/go-zero/blob/master/rest/httpx/requests.go#L32:6"
```

* Parsing path
* Parsing form forms
* Parsing http header
* parsing json

:::info

The function of the parameter checks in Parse() is described in:

The tag modifier in https://go-zero.dev/cn/api-grammar.html

:::

### Usage examples

[Usage examples](https://github.com/zeromicro/zero-examples/tree/main/http)
