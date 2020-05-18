# Event Gateway

[summary](_media/event-gateway-summary.md ':include')

## Input & Output

1. Event received (HTTP) &rarr; Call Event Broker (GRPC) &rarr; Return HTTP response to requester
1. Master API call (HTTP) &rarr; Call Master API (HTTP) &rarr; Return HTTP response to requester
1. Master API call (GRPC) &rarr; Call Master API (GRPC) &rarr; GRPC response

## HTTP

For the HTTP proxy, there will be two endpoints:

1. `/events` &rarr; Event Broker
1. `/api` &rarr; Master API

It will function similar to a reverse proxy, [there's a good article that describes this](https://www.integralist.co.uk/posts/golang-reverse-proxy/).

```go
package main

import (
	"log"
	"net"
	"net/http"
	"net/http/httputil"
	"time"

	"github.com/gorilla/mux"
)

type config struct {
	Path     string
	Host     string
}

func generateProxy(conf config) http.Handler {
	proxy := &httputil.ReverseProxy{Director: func(req *http.Request) {
		originHost := conf.Host
		req.Header.Add("X-Forwarded-Host", req.Host)
		req.Header.Add("X-Origin-Host", originHost)
		req.Host = originHost
		req.URL.Host = originHost
		req.URL.Scheme = "https"
	}, Transport: &http.Transport{
		Dial: (&net.Dialer{
			Timeout: 5 * time.Second,
		}).Dial,
	}}

	return proxy
}

func main() {
	r := mux.NewRouter()

	configuration := []config{
		config{
			Path: "/events.*",
			Host: "event-broker-grpc-gateway.svc.local",  // see the "HTTP to GRPC" section
		},
		config{
			Path: "/api.*",
			Host: "master-api.svc.local",
		},
	}

	for _, conf := range configuration {
		proxy := generateProxy(conf)

		r.HandleFunc(conf.Path, func(w http.ResponseWriter, r *http.Request) {
			proxy.ServeHTTP(w, r)
		})
	}

	log.Fatal(http.ListenAndServe(":9001", r))
}
```

!> We would likely want to use the [`fasthttp`](https://github.com/valyala/fasthttp) package instead of `gorilla/mux` for extra speed. Additionally, we may need to rewrite [`httputil.ReverseProxy`](https://golang.org/src/net/http/httputil/reverseproxy.go) to work with `fasthttp`.

## GRPC

The GRPC protocol is actually just on top of [HTTP/2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) but we can't just run a normal HTTP/2 proxy. For this proxy we can use the [`grpc-proxy`](https://github.com/vgough/grpc-proxy) package which also allows us to run custom logic for filtering or directing as necessary.

```go
func (d *ExampleDirector) Connect(ctx context.Context, method string) (context.Context, *grpc.ClientConn, error) {
  // Disable forwarding for all services prefixed with com.example.internal.
  if strings.HasPrefix(method, "/com.example.internal.") {
    return nil, nil, grpc.Errorf(codes.Unimplemented, "Unknown method")
  }
  md, ok := metadata.FromIncomingContext(ctx)
  if ok {
    // Decide on which backend to dial
    if val, exists := md[":authority"]; exists && val[0] == "staging.api.example.com" {
      // Make sure we use DialContext so the dialing can be cancelled/time out together with the context.
      conn, err := grpc.DialContext(ctx, "api-service.staging.svc.local", grpc.WithCodec(proxy.Codec()))
      return ctx, conn, err
    } else if val, exists := md[":authority"]; exists && val[0] == "api.example.com" {
      conn, err := grpc.DialContext(ctx, "api-service.prod.svc.local", grpc.WithCodec(proxy.Codec()))
      return ctx, conn, err
    }
  }
  return nil, nil, grpc.Errorf(codes.Unimplemented, "Unknown method")
}
```

This example changes the server depending on the host name the request is sent with. We would use a transparent proxy based on the method name to forward requests to the Master API. We want to filter because we may allow Event Broker requests via GRPC in the future.

## HTTP to GRPC

We can use the open source package called [`grpc-gateway`](https://github.com/grpc-ecosystem/grpc-gateway) to help solve this problem.

![GRPC proxy](/_media/grpc-proxy.png)

In order to make this work with our HTTP proxy mentioned above, we have a few options:

1. Create a new component specifically for the HTTP to GRPC proxy
1. Run the HTTP to GRPC proxy next to the HTTP proxy

I think running the HTTP to GRPC proxy next to the HTTP proxy will work for now and quite frankly may not even require the same proxy configuration as the regular HTTP proxy for the Master API.


## Other Implementation Options

We could use an out of the box open source solution that fits our needs. Here are some possible candidates:

1. [Database-less Kong gateway](https://docs.konghq.com/2.0.x/db-less-and-declarative-config/)
1. [Envoy Proxy](https://www.envoyproxy.io/)

The biggest requirement is to not have a database. The gateway should be deployable by itself without any other requirements.

### Kong

- Can be extended using Go plugins

### Envoy

- Can be extended using C++ plugins
