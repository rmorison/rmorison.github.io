+++
title = "A Protocol Buffers Build Toolchain"
author = ["Rod Morison"]
date = 2023-12-17T12:49:00-08:00
draft = false
topics = ["Protocol Buffers", "Protobuf", "gRPC", "Makefile"]
description = "Master Protocol Buffers with Ease: A Step-by-Step Guide to Building a Robust Protobuf Toolchain"
+++

{{< figure src="/ox-hugo/scopio-48ea3889-59b4-4eb8-939b-278f2faf947f.jpg" >}}


## Introduction {#introduction}

[Protocol Buffers](https://protobuf.dev/), or Protobuf, developed by Google, is widely used for serializing structured data, offering a more efficient alternative to formats like XML or JSON. However, getting started with Protobuf is a "batteries not included" proposition. The best place to start is the [Protocol Buffer Basics: Go](https://protobuf.dev/getting-started/gotutorial/) doc which provides plenty of background on using Protobuf, but is hardly a step by step introduction.

The [hands on](https://protobuf.dev/getting-started/gotutorial/#compiling-protocol-buffers) section of that doc simply states

> If you haven’t installed the compiler, download the package and follow the instructions in the README.

The link in that step sends you to the [protobuf repo releases page](https://github.com/protocolbuffers/protobuf/releases/) where you're faced with a list of zip and tar files to guess^H^H^H^H^Hchoose from. Nowhere is the issue of build system or toolchain discussed head on. The simplistic "install it somewhere and here's the command" approach is not adequate for a production Protobuf based service.

In particular, since the Protocol Buffers compiler and plugins are outside whatever package managing you're using (pip, poetry, npm, yarn, ...), how do we lock that part of the build, too?

That, then, is the point of this article. We'll walk through a simple Makefile based Protobuf build that versions the `protoc` compiler and any plugins or Protobuf extensions in use.

The reference repository and the app in it include a working client/server example based on [gRPC](https://grpc.io/). Though gRPC is a common pairing for Protobuf, keep in mind Protobuf is a serialization format, not a protocol, web stack, or network server by itself. Protobuf is utilized by other (non-gRPC) system such as [twirp](https://github.com/twitchtv/twirp), [envoy](https://www.envoyproxy.io/), etc., and can also be used by itself, much like the json library in your favorite language SDK.


## What's a Toolchain? {#what-s-a-toolchain}

ChatGPT (after some redirect prompting) says:

> A "toolchain" in software development is a suite of tools used in a coordinated manner to build and maintain applications, especially those involving complex processes or specialized components. When developing an application that employs Protocol Buffers, the toolchain is central to the workflow. It typically encompasses the protocol compiler, its various plugins, the supporting libraries, and the build scripts—like Makefiles—that orchestrate the setup and execution of these tools. This integrated set of tools is essential for transforming high-level design into a functional software product, ensuring consistency, efficiency, and reliability throughout the development process.


## Just Do It {#just-do-it}

Before diving into the build code itself, let's "just build it", and give it a quick "go". The client-server application is the [gRPC Hello World](https://github.com/grpc/grpc-go/tree/master/examples/helloworld) from the [grpc-go](https://github.com/grpc/grpc-go) repo.

The commands below assume a recent Go install, either with a version manager like [g](https://github.com/stefanmaric/g), from [go.dev](https://go.dev/doc/install), [Homebrew](https://brew.sh/), etc.

**OSX Note**: change `make` to `make PROTOC_ARCH=osx-aarch` or `make PROTOC_ARCH=osx-x86_64`, depending on your platform.

```shell
git clone git@github.com:rmorison/protobuf-toolchain-template.git
make
./greeter_server/main &
./greeter_client/main
./greeter_client/main --name everybody
```


## Build System Walkthrough {#build-system-walkthrough}

...And what you can expect to change when you use this template for your project.


### Toolchain Build {#toolchain-build}

Usually the `toolchain` build is done once at project startup,
committed with artifacts not excluded in `toolchain/.gitignore` and
touched lightly after that, e.g., when adding a plugin or updated
plugins or the protocol compiler itself.


#### [toolchain/Makefile](https://github.com/rmorison/protobuf-toolchain-template/blob/main/toolchain/Makefile) {#toolchain-makefile}

-   Go modules path for the toolchain package. Note that `GO_PACKAGE` is typically set by an invoking Makefile.

<!--listend-->

```make
GO_TOOLCHAIN_PACKAGE := $(GO_PACKAGE)/toolchain
```

-   Protocol compiler version. See the [Protobuf release page](https://github.com/protocolbuffers/protobuf/releases) for
    available versions and architectures.

<!--listend-->

```make
PROTOC_VERSION := 25.1
```

-   Protocol compiler architecture, defaulting to Linux on x86_64. The
    top level `Makefile` will set this when invoked. (Todo: script
    auto-detect for `PROTOC_ARCH`.)

<!--listend-->

```make
PROTOC_ARCH := $(if $(PROTOC_ARCH),$(PROTOC_ARCH),linux-x86_64)
```

-   Protocol compiler plugin builds. Fyi, `protoc` plugins transform the
    Protobuf file into artifacts, usually, but not always, code
    files. [protoc-gen-doc](https://github.com/pseudomuto/protoc-gen-doc) is a widely used example of a documentation
    producing `protoc` plugin.

<!--listend-->

```make
bin/protoc-gen-go: go.sum | bin
      export GOBIN=$$(pwd)/bin && go install google.golang.org/protobuf/cmd/protoc-gen-go
```


#### [toolchain/tools.go](https://github.com/rmorison/protobuf-toolchain-template/blob/main/toolchain/tools.go) {#toolchain-tools-dot-go}

-   Go module plugins for `protoc`. Each line should have an entry in
    the `toolchain/Makefile` as cited above. Note that the build process
    will create a `go.sum` that will "lock" the toolchain versions of these.

<!--listend-->

```go
import (
	_ "google.golang.org/grpc/cmd/protoc-gen-go-grpc"
	_ "google.golang.org/protobuf/cmd/protoc-gen-go"
)
```


### Application Build {#application-build}


#### [Makefile](https://github.com/rmorison/protobuf-toolchain-template/blob/main/Makefile) {#makefile}

-   Set `GO_PACKAGE` to your project Go Modules path. This Makefile with
    run the `go mod init` with it.

<!--listend-->

```make
GO_PACKAGE := github.com/rmorison/protobuf-toolchain-template
```

-   The source `.proto` files to compile.

<!--listend-->

```make
PROTO_FILES := proto/helloworld/helloworld.proto
```

-   Protocol compiler architecture, defaulting to Linux on
    x86_64. Change to `osx-aarch_64` or `osx-x86_64` for OSX.

<!--listend-->

```make
PROTOC_ARCH := $(if $(PROTOC_ARCH),$(PROTOC_ARCH),linux-x86_64)
```

-   The application build. Many apps will have a single "main" target, though this example has two. You will need to change the `all` target and its dependencies. You may need or want to update the wildcard build rule for `%/main`, e.g., move it to top level of the project, add more specific `go build` options, or replace this section entirely. Note that for most projects "main" can depend on all of the `protoc` build artifacts. As project complexity and maturity grows, expect this section to be revised.

<!--listend-->

```make
# Build go program from main.go
%/main: %/main.go go.mod $(PROTOC_GO_FILES)
      go mod tidy
      go build -o $@ $<

all: greeter_client/main greeter_server/main

greeter_client/main: greeter_client/main.go
greeter_server/main: greeter_server/main.go
```

-   Typically you should commit all build artifacts not excluded in the top level and toolchain `.gitignore` files. Specifically, the `go.mod`, `go.sum` and source files in each proto directory should be committed. The former "lock" the build and the latter provider importable Protobuf client stubs.


#### Protobuf Source Files {#protobuf-source-files}

-   Create your project `.proto` files in `proto/{name}/{name}.proto` and update the `Makefile`. Be sure your `option go_package` is set correctly and corresponds to `GO_PACKAGE` and your proto module name. E.g., for the helloworld example:

<!--listend-->

```go
option go_package = "github.com/rmorison/protobuf-toolchain-template/proto/helloworld";
```


### PS {#ps}

Questions or comments to repo [Discussions](https://github.com/rmorison/protobuf-toolchain-template/discussions). Bugs, feature requests to [Issues](https://github.com/rmorison/protobuf-toolchain-template/issues).
