# ZapRay - XRay logger that wraps uber/zap

[![GoDoc](http://godoc.org/github.com/yosssi/gohtml?status.png)](http://godoc.org/github.com/joe-davidson1802/zapray)

ZapRay is a logger for [Go](http://golang.org/). To be used in conjunction with [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/security-logging-monitoring.html) and [Zap](https://github.com/uber-go/zap).

## Install

```
go get -u github.com/joe-davidson1802/zapray
```

## Example

``` go
package main

import (
	"github.com/joe-davidson1802/zapray"
	"github.com/aws/aws-xray-sdk-go/xray"
	"go.uber.org/zap"
	)

func main() {
	// Create a zap logger as usual
	z, err := zap.NewProduction()
	if err != nil {
		panic(err)
	}
	// Create a zapray logger
	logger := zapray.NewZaprayLogger(z)

	// zapray requires the context to have been created by an xray segment
	// if this isn't the case then zapray will log without appending trace information
	ctx, seg := xray.BeginSegment(context.Background(), "someSegment")
	defer seg.Close(nil)

	// Trace returns a regular zap logger but will log trace id with any logs chained onto it
	logger.Trace(ctx).Info("my zap log")
}
```

## Web Example

``` go
package main

import (
	"net/http"
	"github.com/joe-davidson1802/zapray"
	"github.com/aws/aws-xray-sdk-go/xray"
	"go.uber.org/zap"
	)
	
var logger *zapray.ZaprayLogger
	
func HandleRequest(w http.ResponseWriter, r *http.Request) {
	logger.TraceRequest(r).Info("some log")
}

func main() {
	// Create a zap logger as usual
	z, err := zap.NewProduction()
	if err != nil {
		panic(err)
	}
	// Create a zapray logger
	logger = zapray.NewZaprayLogger(z)
	handler := http.HandlerFunc(HandleRequest)
	segmentedHandler := xray.Handler(handler)
	// Uncomment to make a subsegment rather than a segment
	// segmentedHandler := zapray.NewMiddleware(handler)
	panic(http.ListenAndServe(":8000", segmentedHandler)
}
```
