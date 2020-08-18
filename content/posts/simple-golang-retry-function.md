+++
date = "2017-05-29T09:54:15-04:00"
draft = false
title = "Simple Golang Retry Function"
weight = 9998
page = "blog"

author = "Nick Stogner"
description = "If at first you don't succeed, try, try again. In go code, that translates to..."

tags = [
    "golang",
]

categories = [
    "Development",
]


+++

Adding retry policies in your software is an easy way to increase resiliency. This is especially useful when making HTTP requests or doing anything else that has to reach out across the network.

If at first you don’t succeed, try, try again. In go code, that translates to:

```go
func retry(attempts int, sleep time.Duration, fn func() error) error {
	if err := fn(); err != nil {
		if s, ok := err.(stop); ok {
			// Return the original error for later checking
			return s.error
		}
 
		if attempts--; attempts > 0 {
			time.Sleep(sleep)
			return retry(attempts, 2*sleep, fn)
		}
		return err
	}
	return nil
}
 
type stop struct {
	error
}
```

The `retry` function recursively calls itself, counting down attempts and sleeping for twice as long each time (i.e. exponential backoff). This technique works well until the situation arises where a good number of clients start their retry loops at roughly the same time. This could happen if a lot of connections get dropped at once. The retry attempts would then be in sync with each other, creating what is known as the [Thundering Herd](https://en.wikipedia.org/wiki/Thundering_herd_problem) problem. To prevent this, we can add some randomness by inserting the following lines before we call `time.Sleep`:

```go
jitter := time.Duration(rand.Int63n(int64(sleep)))
sleep = sleep + jitter/2
```

The improved, jittery version:

```go
func init() {
	rand.Seed(time.Now().UnixNano())
}

func retry(attempts int, sleep time.Duration, f func() error) error {
	if err := f(); err != nil {
		if s, ok := err.(stop); ok {
			// Return the original error for later checking
			return s.error
		}

		if attempts--; attempts > 0 {
			// Add some randomness to prevent creating a Thundering Herd
			jitter := time.Duration(rand.Int63n(int64(sleep)))
			sleep = sleep + jitter/2

			time.Sleep(sleep)
			return retry(attempts, 2*sleep, f)
		}
		return err
	}

	return nil
}

type stop struct {
	error
}
```

There are two options for stopping the retry loop before all the attempts are made:

1. Return `nil`
2. Return a wrapped error: `stop{err}`

Choose option #2 when an error occurs where retrying would be futile. Consider most `4XX` HTTP status codes. They indicate that the client has done something wrong and subsequent retries, without any modification to the request will result in the same response. In this case we still want to return an error so we wrap the error in the `stop` type. The actual error that is returned by the retry function will be the original non-wrapped error. This allows for later checks like `err == ErrUnauthorized`.

Take a look at the following implementation for retrying a HTTP request. Note: In this case, there are only 2 additional lines needed for adding in the retry policy to the existing `DeleteThing` function (lines 14 and 34).

```go
// DeleteThing attempts to delete a thing. It will try a maximum of three times.
func DeleteThing(id string) error {
	// Build the request
	req, err := http.NewRequest(
		"DELETE",
		fmt.Sprintf("https://unreliable-api/things/%s", id),
		nil,
	)
	if err != nil {
		return fmt.Errorf("unable to make request: %s", err)
	}

	// Execute the request
	return retry(3, time.Second, func() error {
		resp, err := http.DefaultClient.Do(req)
		if err != nil {
			// This error will result in a retry
			return err
		}
		defer resp.Body.Close()

		s := resp.StatusCode
		switch {
		case s >= 500:
			// Retry
			return fmt.Errorf("server error: %v", s)
		case s >= 400:
			// Don't retry, it was client's fault
			return stop{fmt.Errorf("client error: %v", s)}
		default:
			// Happy
			return nil
		}
	})
}
```

**Updates**

[June 1, 2017] Edited to add the jitter example
