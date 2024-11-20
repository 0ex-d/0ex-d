---
title: "Using IPFS with Golang"
summary: "How I integrated IPFS into a project using Golang."
date: 2022-10-20
layout: default
---

## Introduction

InterPlanetary File System (IPFS) is a distributed system for storing and accessing files, websites, applications, and data. Recently, I used IPFS in a project to decentralize file storage and ensure high availability.

In this article, I'll share how I integrated IPFS into a Go application, including code snippets and explanations.

---

## Setting Up the Environment

To work with IPFS in Go, you'll need the [go-ipfs-api](https://github.com/ipfs/go-ipfs-api) package. Start by installing it:

```bash
go get github.com/ipfs/go-ipfs-api
```

## Connecting to an IPFS Node

First, establish a connection to an IPFS node. The example below demonstrates how to connect to a local IPFS node running on `localhost:5001`.

```go
package main

import (
	"log"

	shell "github.com/ipfs/go-ipfs-api"
)

func main() {
	sh := shell.NewShell("localhost:5001")

	// test connection
	if _, err := sh.ID(); err != nil {
		log.Fatalf("Failed to connect to IPFS: %v", err)
	}

	log.Println("Connected to IPFS node!")
}

```

## Adding a File to IPFS

To store a file on IPFS, you can use the `Add` method. Here's an example of adding a text file:

```go
package main

import (
	"bytes"
	"fmt"
	"log"

	shell "github.com/ipfs/go-ipfs-api"
)

func main() {
	sh := shell.NewShell("localhost:5001")

	data := bytes.NewBufferString("Hello, IPFS!")
    // upload
	cid, err := sh.Add(data)
	if err != nil {
		log.Fatalf("Failed to add file to IPFS: %v", err)
	}

	fmt.Printf("File added to IPFS with CID: %s\n", cid)
}

```

## Retrieving a File from IPFS

Once a file is added, it can be retrieved using its Content Identifier (CID):

```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"log"

	shell "github.com/ipfs/go-ipfs-api"
)

func main() {
	sh := shell.NewShell("localhost:5001")
	cid := "YourFileCIDHere"

	reader, err := sh.Cat(cid)
	if err != nil {
		log.Fatalf("Failed to retrieve file from IPFS: %v", err)
	}

	buf := new(bytes.Buffer)
	_, err = io.Copy(buf, reader)
	if err != nil {
		log.Fatalf("Error reading file content: %v", err)
	}

	fmt.Printf("File content: %s\n", buf.String())
}

```

## Conclusion

Integrating IPFS into your Go projects can decentralize your file storage and improve resilience. The `go-ipfs-api` package makes it easy to connect to IPFS nodes, add files, and retrieve them programmatically.

Feel free to adapt the code snippets for your needs and explore the other capabilities of IPFS, such as pinning, managing directories, and more!
