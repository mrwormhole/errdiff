# Errdiff

[![Version](https://img.shields.io/github/tag/mrwormhole/errdiff.svg)](https://github.com/mrwormhole/errdiff/tags)
[![CI Build](https://github.com/mrwormhole/errdiff/actions/workflows/test.yaml/badge.svg)](https://github.com/mrwormhole/errdiff/actions/workflows/test.yaml)
[![GoDoc](https://godoc.org/github.com/mrwormhole/errdiff?status.svg)](https://godoc.org/github.com/mrwormhole/errdiff)
[![Report Card](https://goreportcard.com/badge/github.com/mrwormhole/errdiff)](https://goreportcard.com/report/github.com/mrwormhole/errdiff)
[![License](https://img.shields.io/github/license/mrwormhole/errdiff)](https://github.com/mrwormhole/errdiff/blob/master/LICENSE)
[![Coverage Status](https://coveralls.io/repos/github/mrwormhole/errdiff/badge.svg?branch=master)](https://coveralls.io/github/mrwormhole/errdiff?branch=master)

This is a fork of h-fam/errdiff, this is created in order to achieve type-safety and better Check() method that can understand wrapped/contained errors.
In the process of doing so, I have removed interface{} argument that is passed to Check() method. Also cleaned up deprecated grpc status code pkgs and made diffs more consistent.

# Moral

- Avoid using `interface{} or any` for test tables which creates havoc on test readability and maintenance. Use statically typed test methods with statically typed table tests to reduce the mental burden.
- Consistent error diff reporting with `=` and `:` and `,` distinguished.
- Single purpose for a single function, no generic test funcs

# Usage

### errdiff.Check

The most common option that is used to compare against error

```go
tests := []struct {
  ...
  wantErr error
}{
  // Success
  {...},
  // Failures
  {..., wantErr: errors.New("something failed: EOF")}, // an explicit full error
  {..., wantErr: io.EOF}, // a contained/wrapped error
}
for _, tt := range tests {
  t.Run(tt.name, func(t *testing.T) {
    got, err := fn(...)
    if diff := errdiff.Check(err, tt.wantErr); diff != "" {
      t.Errorf("fn() %s", diff)
    }
  })
}
```

### errdiff.Text

It is used for exact strings to compare against error

```go
tests := []struct {
  ...
  wantErr string
}{
  // Success
  {...},
  // Failures
  {..., wantErr: "something failed: EOF"}, // full text case-sensitive
}
for _, tt := range tests {
  t.Run(tt.name, func(t *testing.T) {
    got, err := fn(...)
    if diff := errdiff.Text(err, tt.wantErr); diff != "" {
      t.Errorf("fn() %s", diff)
    }
  })
}
```

### errdiff.Code

It is used for grpc status codes to compare against error

```go
tests := []struct {
  ...
  wantCode codes.Code
}{
  // Success
  {...},
  // Failures
  {..., wantCode: codes.InvalidArgument}, // grpc status code
}
for _, tt := range tests {
  t.Run(tt.name, func(t *testing.T) {
    got, err := fn(...)
    if diff := errdiff.Code(err, tt.wantCode); diff != "" {
      t.Errorf("fn() %s", diff)
    }
  })
}
```
