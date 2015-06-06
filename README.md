## Hello World using Go, Statically-linked C library, and CMake

This program prints "Hello World". A Go language program prints the word 
'Hello' and then it calls a statically linked C library to print 
'World'. It uses the CMake build system to take care of many details
so that you can type 'make' to rebuild everything after modifying the source. 

This is a fork of [shadowmint](https://github.com/shadowmint)'s 
[go-static-linking](https://github.com/shadowmint/go-static-linking) 
project

### Usage

To build and run, copy/paste these commands into a terminal:
   
    mkdir build && cd build && cmake ..
    make
    ./bin/hello

After you modify any source code, .c .h or .go, you can rebuild everything
by running
 
    make

This will rebuild any modified C code, then it will call the 'go' 
builder to rebuild any modified go code. This 'hybrid' build system allows
'make' to deal with C dependencies and 'go' to deal with go dependencies.

### Status

This is a basic exploration and learning tool. It is not intended for 
any fancy use. See file LICENSE-2.0 for more info.

### Design

The source tree is layed out as follows:

    src/hello/hello.go       # main go package, prints 'hello' and calls C
    src/bridge/bridge.go.in  # template, used to create bridge.go package
    src/c/world.c            # c code to print 'world'.
    src/c/world.h            # c header. all c code lives under 'c' path
    CMakeLists.txt           # Cmake build file

As you can see, this is a sort of 'hybrid' Go project layout. It 
has subdirs under 'src' for Go packages. But it also has c code and
a Cmake build file.

The process is as follows:

    Cmake creates a bridge.go file from the bridge.go.in template
    Cmake creates a 'Makefile'
    'make' creates a static C library (.a/.lib file) from c source code
    'make' calls 'go install' 
    'go install' builds executable and links with c library using #cgo

The resulting binary tree will hopefully be as follows:

    build/bin/hello          # executable file generated by Go's builder
    build/lib/libworld.a     # statically linkable C library
    build/src/bridge.go      # "bridge" generated by cmake process
    build/cmake*             # usual Cmake generated files (cache, etc)
    build/Makefile           # cmake-generated, builds C lib & runs 'go install'
    build/pkg/machine/bridge.a # bridge library generated by 'go install'

Running ./bin/hello should produce output like this:

    Hello (Invoking c statically linked library...)
    World
    (Done)

### How to get more debug info of the build process

Set your environment variable VERBOSE to 1. This will enable the '-x' 
flag in Go as well as the VERBOSE option to 'make'.

    export VERBOSE=1 # bash shell (typical on linux)
    setenv VERBOSE 1 # csh shell (typical on BSD)
    mkdir build && cd build && cmake ..
    make
    ./bin/hello

### Running raw go commands afterwards (GOBIN, GOPATH, etc)

If you want to do some special 'go' commands, not covered by the 'go'
call setup in the Makefile, you will have to set up the GOBIN and 
GOPATH environment variables. The CMakeLists.txt generates .sh scripts to 
help with this. Run one depending on your shell:

    . ./CMakeFiles/setenv.sh       # for bash shell (typical shell on Linux)
    source ./CMakeFiles/setenv.csh # for csh shell (typical shell on BSD)

Now you should be able to run go commands by themselves:

    go build hello         # build the go program

### On the static C glue of bridge.go.in and bridge.go

The magic static glue of the bridge file works using 'cgo' as follows:

bridge.go.in has two lines like this:

    // #cgo CFLAGS: ${CFLAGS}
    // #cgo LDFLAGS: ${LDFLAGS}

bridge.go, generated by cmake & CmakeLists.txt, has two lines like this:

    // #cgo CFLAGS: -I/tmp/go-hello-static-world/src/c
    // #cgo LDFLAGS: -L/tmp/go-hello-static-world/build/lib -lworld

### How do you know if a file is really statically linked?

ldd and file can help you

    don@serebryanya[build]$ ldd ./bin/hello 
	not a dynamic executable
    don@serebryanya[build]$ file bin/hello 
        bin/hello: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), 
        statically linked, for GNU/Linux 2.6.24, 

### Windows(TM)

Untested in this fork.

### See Also

#<http://golang.org/doc/code.html> How to Write Go Code (golang.org)
#<http://blog.golang.org/c-go-cgo> C-Go (especially for the bridge.go file)
#<http://golang.org/misc/cgo/testso/cgoso.go> setting LDFLAGS on different OSes
#<http://blog.hashbangbash.com/2014/04/linking-golang-statically/> Static Go-linking

### Why do this?

This build style can theoretically help when transforming C programs to Go 
programs, by providing a 'hybrid' system to use during transformation. 

This build style also is a nice way to distribute programs to other 
people without them having to worry about library versions and 
installing extra packages.

Thanks for reading. Thanks shadowmint.


