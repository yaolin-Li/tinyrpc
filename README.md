# tinyrpc

[![Go Report Card](https://goreportcard.com/badge/github.com/zehuamama/tinyrpc)&nbsp;](https://goreportcard.com/report/github.com/zehuamama/tinyrpc)![GitHub top language](https://img.shields.io/github/languages/top/zehuamama/tinyrpc)&nbsp;![GitHub](https://img.shields.io/github/license/zehuamama/tinyrpc)&nbsp;[![CodeFactor](https://www.codefactor.io/repository/github/zehuamama/tinyrpc/badge)](https://www.codefactor.io/repository/github/zehuamama/tinyrpc)&nbsp;[![codecov](https://codecov.io/gh/zehuamama/tinyrpc/branch/main/graph/badge.svg)](https://codecov.io/gh/zehuamama/tinyrpc)&nbsp; ![go_version](https://img.shields.io/badge/go%20version-1.17-yellow)

tinyrpc is a high-performance RPC framework based on `protocol buffer` encoding. It is based on `net/rpc` and supports multiple compression formats (`gzip`, `snappy`, `zlib`).

Language&nbsp;&nbsp;English

## Install


- install `protoc` at first :http://github.com/google/protobuf/releases

- install `protoc-gen-go` and `protoc-gen-tinyrpc`:

```
go install github.com/golang/protobuf/protoc-gen-go
go install github.com/zehuamama/tinyrpc/protoc-gen-tinyrpc
```

## Quick Start
Fisrt, create a demo and import the tinyrpc package:
```shell
> go mod init demo
> go get github.com/zehuamama/tinyrpc
```
Under the path of the project, create a protobuf file `arith.proto`:
```protobuf
syntax = "proto3";

package message;
option go_package="/message";

// ArithService Defining Computational Digital Services
service ArithService {
  // Add addition
  rpc Add(ArithRequest) returns (ArithResponse);
  // Sub subtraction
  rpc Sub(ArithRequest) returns (ArithResponse);
  // Mul multiplication
  rpc Mul(ArithRequest) returns (ArithResponse);
  // Div division
  rpc Div(ArithRequest) returns (ArithResponse);
}

message ArithRequest {
  int32 a = 1;
  int32 b = 2;
}

message ArithResponse {
  int32 c = 1;
}
```
an arithmetic operation service is defined here, using `protoc` to generate code:
```shell
> protoc --tinyrpc_out=. arith.proto --go_out=. arith.proto
```
at this time, two files will be generated in the directory `message`: `arith.pb.go` and `arith.svr.go`

the code of arith.svr.go is as follows:
```go
// Code generated by protoc-gen-tinyrpc.

package message

// ArithService Defining Computational Digital Services
type ArithService struct{}

// Add addition
func (this *ArithService) Add(args *ArithRequest, reply *ArithResponse) error {
	// define your service ...
	return nil
}

// Sub subtraction
func (this *ArithService) Sub(args *ArithRequest, reply *ArithResponse) error {
	// define your service ...
	return nil
}

// Mul multiplication
func (this *ArithService) Mul(args *ArithRequest, reply *ArithResponse) error {
	// define your service ...
	return nil
}

// Div division
func (this *ArithService) Div(args *ArithRequest, reply *ArithResponse) error {
	// define your service ...
	return nil
}
```
We need define our services. 

Finally, under the path of the project, we create a file named `main.go`, the code is as follows:
```go
package main

import (
	"demo/message"
	"log"
	"net"

	"github.com/zehuamama/tinyrpc"
)

func main() {
	lis, err := net.Listen("tcp", ":8082")
	if err != nil {
		log.Fatal(err)
	}

	server := tinyrpc.NewServer()
	server.RegisterName("ArithService", new(message.ArithService))
	server.Serve(lis)
}
```
A tinyrpc server is completed.

## Client
We can create a tinyrpc client and call it synchronously with the `Add` function:
```go
import (
        "demo/message"
	"github.com/zehuamama/tinyrpc"
...

conn, err := net.Dial("tcp", ":8082")
if err != nil {
	log.Fatal(err)
}
defer conn.Close()
client := tinyrpc.NewClient(conn)
resq := message.ArithRequest{A: 20, B: 5}
resp := message.ArithResponse{}
err = client.Call("ArithService.Add", &resq, &resp)
log.Printf("Arith.Add(%v, %v): %v ,Error: %v", resq.A, resq.B, resp.C, err)
```
you can also call asynchronously, which will return a channel of type *rpc.Call:
```go

result := client.AsyncCall("ArithService.Add", &resq, &resp)
select {
case call := <-result:
	log.Printf("Arith.Add(%v, %v): %v ,Error: %v", resq.A, resq.B, resp.C, call.Error)
case <-time.After(100 * time.Microsecond):
	log.Fatal("time out")
}
```
of course, you can also compress the encoding, tinyrpc currently supports `gzip`, `snappy`, `zlib`:
```go
import "github.com/zehuamama/tinyrpc/compressor"

...
client := tinyrpc.NewClient(conn, tinyrpc.WithCompress(compressor.Gzip))

```
## Contributing

If you are intersted in contributing to tinyrpc, please see here: [CONTRIBUTING](https://github.com/zehuamama/tinyrpc/blob/main/CONTRIBUTING.md)

## License

tinyrpc is licensed under the term of the [BSD 2-Clause License](https://github.com/zehuamama/tinyrpc/blob/main/LICENSE)

