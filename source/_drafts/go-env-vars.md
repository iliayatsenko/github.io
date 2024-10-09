GOPATH:
environment variable lists places to look for Go code
when using modules, only serves as storage for downloaded code, not used for resolving imports
colon-separated string
default $HOME/go

	should contain subdirs
		src (source code, all the path below is import path)
		pkg (compiled files, for concrete OS)
		bin (executable binaries)

	https://pkg.go.dev/cmd/go#hdr-GOPATH_environment_variable

	GOPATH=/home/user/go

	/home/user/go/
	    src/
	        foo/
	            bar/               (go code in package bar)
	                x.go
	            quux/              (go code in package main)
	                y.go
	    bin/
	        quux                   (installed command)
	    pkg/
	        linux_amd64/
	            foo/
	                bar.a          (installed package object)

go get:
manages deps of module
caches modules code in $GOPATH/pkg/mod

go install
compiles given module
installs its executables in the bin dir ($GOBIN or $GOPATH/bin)
