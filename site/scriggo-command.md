{% extends "/layouts/doc.html" %}
{% macro Title string %}Scriggo command{% end %}
{% Article %}

# Scriggo command

Scriggo has a command line interface, the `scriggo` command, that allows to:

* [serve templates with support for Markdown](#serve-a-template)
* [initialize an interpreter for Go programs](#initialize-an-interpreter)
* [generate the code for package importers](#generate-a-package-importer)

### Install `scriggo` command

Before installing the Scriggo command, <a href="https://golang.org/dl/">download and install Go</a>.

When Go is installed, open a terminal and execute:

```
$ go install github.com/open2b/scriggo/cmd/scriggo@latest
```

then test if `scriggo` can be executed:

```
$ scriggo version
scriggo version 0.50.1 (go1.17)
```

If the `scriggo` command is not found, you should add the directory where the command has been installed to your `PATH`.

`go` installs the `scriggo` command in the directory named by the `GOBIN` environment variable. If it is not set it
defaults to `$GOPATH/bin` or, if the `GOPATH` environment variable is not set, to `$HOME/go/bin`.


### Get help from command line

To get help from the command line run the following command:

```
$ scriggo help
```

## Serve a template

The Scriggo Serve command runs a web server and serves the template rooted at the current directory.
All Scriggo builtins are available in template files. It is useful to learn [Scriggo templates](/templates).

The basic Serve command takes this form:

```
$ scriggo serve
```

It renders HTML and Markdown files based on file extension.

For example, serving this request:

    http://localhost:8080/article

it renders the file "article.html" as HTML if exists, otherwise renders the file "article.md" as Markdown.

Serving a URL terminating with a slash:

    http://localhost:8080/blog/

it renders "blog/index.html" or "blog/index.md". 

Markdown is converted to HTML with the [Goldmark](https://github.com/yuin/goldmark) parser with the options
`html.WithUnsafe`, `parser.WithAutoHeadingID` and `extension.GFM`.

Templates are automatically rebuilt when a file changes.

### Complete syntax

The complete `scriggo serve` command takes this form:

```
$ scriggo serve [-S n] [--metrics] 
```

The `-S` flag prints the assembly code of the served file and n determines the maximum length, in runes, of
disassembled `Text` instructions

    n > 0: at most n runes; leading and trailing white space are removed
    n == 0: no text
    n < 0: all text

The `--metrics` flags prints metrics about execution time.

## Initialize an interpreter

The Scriggo Init command initializes an interpreter for Go programs. It creates the following files:

* go.mod
* go.sum
* main.go
* packages.go
* Scriggofile

The syntax is: 

```
$ scriggo init [dir]
```

where `dir` is the directory in which to create the files. If no argument is given, `scriggo init` uses the current directory.
If the directory already contains ".go" files or a "vendor" directory, the command fails. 

The command creates a Scriggofile with the instruction to create an importer for the Go standard library.

If the directory does not contain a `go.mod` file, the command creates it and as module path uses the directory name.

## Generate a package importer

The Scriggo Import command generate the code for a package importer. An importer is used by Scriggo to import a package
when an "import" declaration is executed.

The code for the importer is generated from the instructions in a Scriggofile. The Scriggofile should be in a Go module.

The basic Import command takes this form:

```
$ scriggo import [-o output]
```

For example:

```
$ scriggo import -o packages.go
```

generates the code for an importer, with instructions in a Scriggofile called "Scriggofile" in the current directory
and writes it into the file "packages.go".

### Complete syntax

The complete `scriggo import` command takes this form:

```
$ scriggo import [-f Scriggofile] [-v] [-x] [-o output] [module]
```

If an argument is given, it must be a local rooted path or must begin with a `.` or `..` element and it must be a module
root directory. Import looks for a Scriggofile named "Scriggofile" in that directory.

If no argument is given, the action applies to the current directory.

The `-f` flag forces import to read the given Scriggofile instead of the Scriggofile of the module.

The importer in the generated Go file have type `native.Importer` and it is assigned to a variable named "packages".
The variable can be used as an argument to the `Build` and `BuildTemplate` functions in the `scriggo` package.

To give a different name to the variable use the instruction `SET VARIABLE` in the Scriggofile:

    SET VARIABLE foo

The package name in the generated Go file is by default "main", to give a different name to the package use the
instruction `SET PACKAGE` in the Scriggofile:

    SET PACKAGE boo

The `-v` flag prints the imported packages as defined in the Scriggofile.

The `-x` flag prints the executed commands.

The `-o` flag writes the generated Go file to the named output file, instead to the standard output.

