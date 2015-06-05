# Hello World using Go and Statically-linked C library

This program prints "Hello World". A go language program prints the word 
'Hello' and then it calls a statically linked C library to print 
'World'.

This is a fork of shadowmint's go-static-linking project

## Design

The source tree is layed out as follows:

    src/hello/hello.go
    src/bridge/bridge.go.in
    src/c/world.c
    src/c/world.h

As you can see, this is a sort of 'hybrid' go project layout. The main 
package is under 'hello'. There is a special subdir for c code. The 
'bridge' package contains a template file, that will be able to form
a static glue between C and go during the cmake/make process.

To build, run the following command
   
    mkdir build && cd build && cmake .. && VERBOSE=1 make

Cmake & build will go through the following steps:

    Create bridge .go file from the bridge.go.in template
    Create c library (.a/.lib) file from c source code
    Call 'go install' to create main executable file

The resulting binary tree will hopefully be as follows:

    build/bin/hello          # executable file generated by Go's builder
    build/lib/libworld.a     # statically linked C library
    build/src/bridge.go      # "bridge" generated by cmake process
    build/cmake*             # usual Cmake generated files (cache, etc)
    build/Makefile           # cmake-generated, builds C lib & runs 'go install'
    build/pkg/machine/bridge.a # bridge library generated by 'go install'

Running ./bin/hello should produce output like this:

    Hello (Invoking c statically linked library...)
    World
    (Done)

## GOBIN, GOPATH, What do to do after 'make'

To re-run the go build under the build tree, you should set GOBIN and GOPATH
environment variables in your command shell. 

The values for GOBIN and GOPATH are printed during the cmake process. Here
is an example:

    GOPATH=/tmp/go-static-linking/build:/tmp/go-static-linking
    GOBIN=/tmp/go-static-linking/build/bin

If you dont know how to set env variables in your shell, do a quick google
search on it and then you will be able to properly copy/paste those values.
Or fork/submit a patch to make this easier somehow.

After this you can type 'go build hello' or 'go install hello'

## Static glue of bridge.go.in and bridge.go

The magic static glue of the bridge file works as follows:

bridge.go.in has two lines like this:

    // #cgo CFLAGS: -I${C_INCLUDE_PATH}
    // #cgo LDFLAGS: -L${C_LIBRARY_PATH} -l${C_LIBRARY_NAME}

bridge.go, generated by CmakeLists.txt, has two lines like this:

    // #cgo CFLAGS: -I/tmp/go-static-linking/src/c
    // #cgo LDFLAGS: -L/tmp/go-static-linking/build/lib -lworld

## Windows(TM)

Untested in this fork.

## Differences from shadowmint's original code

    1. remove usage of separate cmake BindConfig.txt file
    2. put all generated files under 'build' directory (bridge.go)
    3. move call of 'go build' into cmake process (go install)
    4. rearrange and simplify directory structure

## See Also

    http://golang.org/doc/code.html How to Write Go Code

Thanks for reading. Thanks shadowmint.
