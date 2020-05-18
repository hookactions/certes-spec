The Event Gateway is the public facing interface into Certes. When I say "public facing", it may actually be available for the internet to see or it could be internal within a VPC network. This means that you could use Certes for internal events only if you wanted to, but generally you will want to receive events from other third-party sources so for our purposes "public" will mean "available to the internet".

This gateway can be thought of as a custom proxy like [Envoy](https://www.envoyproxy.io/) or [Nginx](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/). The implementation is discussed more in the [Event Gateway](/event-gateway) section.

This component is responsible for receiving incoming HTTP and GRPC requests and forwarding them to the proper components. Currently, there are three cases for forwarding:

1. A new incoming event is received over HTTP, it is forwarded to the [Event Broker](/event-broker)
1. An API request is received over HTTP, it is forwarded to the [Master API](/master-api#http)
1. An API request is received over GRPC, it is forwarded to the [Master API](/master-api#grpc)

Generally, there will be two ports available from this component:

1. `9600`: HTTP
1. `9601`: GRPC

Although, there is a possibility of a single port `9600`, see [this example](https://github.com/philips/grpc-gateway-example/blob/master/cmd/serve.go#L51). More investigation of this usage with `fasthttp` is needed when starting the actual implementation.

```go
// ... snip ...

func grpcHandlerFunc(grpcServer *grpc.Server, otherHandler http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// This is a partial recreation of gRPC's internal checks https://github.com/grpc/grpc-go/pull/514/files#diff-95e9a25b738459a2d3030e1e6fa2a718R61
		if r.ProtoMajor == 2 && strings.Contains(r.Header.Get("Content-Type"), "application/grpc") {
			grpcServer.ServeHTTP(w, r)
		} else {
			otherHandler.ServeHTTP(w, r)
		}
	})
}

// ... snip ...
```
