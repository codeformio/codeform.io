+++
date = "2017-06-06T10:37:14-04:00"
draft = false
title = "Go: Wrapping http.ResponseWriter with Middleware"
subtitle = "Utilizing Interfaces and Composition"
weight = 9999
page = "blog"

author = "Nick Stogner"
description = "How to capture and log status codes written by nested handler functions in Golang middleware (utilizing composition and interfaces)."

tags = [
    "golang",
]

categories = [
    "Development",
]

+++

Using middleware provides a clean way to reduce code duplication when handling HTTP requests in Go. By utilizing the standard handler signature, `func(w http.ResponseWriter, r *http.Request)` we can write functions that are easily dropped into any Go web service. One of the most common uses of middleware is to provide request/response logging. In order to log responses we will need to capture the status code that was written by a nested handler.

Since we don't have direct access to this status code, we will wrap the `ResponseWriter` that is passed down:

```go
type statusRecorder struct {
	http.ResponseWriter
	status int
}

func (rec *statusRecorder) WriteHeader(code int) {
	rec.status = code
	rec.ResponseWriter.WriteHeader(code)
}

func logware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Initialize the status to 200 in case WriteHeader is not called
		rec := statusRecorder{w, 200}

		next.ServeHTTP(&rec, r)

		log.Printf("response status: %v\n", rec.status)
	})
}
```

Because the [`http.ResponseWriter`](https://godoc.org/net/http#ResponseWriter) type is an interface we can pass a custom type to subsequent handlers as long as it implements all of the defined methods:

```go
type ResponseWriter interface {
    Header() Header
    Write([]byte) (int, error)
    WriteHeader(int)
}
```
By using composition we only need to explicitly implement the `WriteHeader` method that we want to modify. This is because we have embedded the `ResponseWriter` type in our struct so all of its methods get promoted and can be called on `statusRecorder`.

Note: We pass a pointer into `ServeHTTP` so that nested handlers can change the `status` field that we later reference.

Using the middleware is simple:

```go
func main() {
	http.ListenAndServe(":8080",
		logware(
			http.HandlerFunc(handle),
		),
	)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(201)
	w.Write([]byte("Accepted"))
}
```

The above `handle` function calls two methods:

1. `WriteHeader`: This calls our `statusRecorder.WriteHeader` method which records the `201` status for logging and then calls the original `ResponseWriter.WriteHeader` method.
2. `Write`: This calls the original `ResponseWriter.Write` method directly.

Why do we choose to store the status in our struct rather than having the `WriteHeader` method do the logging? Calling `WriteHeader` inside of a handler is optional. An implicit 200-OK status is sent to the client if the status code is not set. In this situation the logging call would be missed.

Happy coding, and be on the lookout for other places to use composition and interfaces to simplify your Go code.
