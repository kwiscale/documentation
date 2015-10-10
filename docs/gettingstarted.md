# Prerequists

You have to install `go` and have set `$GOPATH` to point on a writable directory. 

You need to set `$PATH` to append `$GOPATH/bin`.

An example `.bashrc` modification:

```bash
export GOPATH=~/goproject
export PATH=$GOPATH/bin:$PATH
```

After having set those variables, you **must** reset your shell. Restart your session or call:

```bash
source ~/.bashrc
```

It's recommanded to install `goimports` command that kwiscale CLI will try to call:

```bash
go get -u golang.org/x/tools/cmd/goimports
```

**Important**: If you don't install `goimports`, kwiscale CLI may have problem to generate a working main.go file.

# Installation

Kwiscale is a standard Go package, so you may install it with the `go get` command. 

Please, **don't use github url** but use the [gopkg.in](http://gopkg.in) url that provides versionning. , the package is at [gopkg.in/kwiscale](http://gopkg.in/kwiscale/framework.v1)

Installation is made by the following command:

```
go get gopkg.in/kwiscale/framework.v1
```

The version `v1` is the current version. To use master version, please use `v0` (while it's not recommanded either you need a specific feature that is not yet in next version).

At this time, kwiscale is installed and you can develop service.


You may install kwiscale cli:

```
go get gopkg.in/kwiscale/framework.v1/kwiscale
```

Right now, if you set $GOPATH/bin in your $PATH, the "kwiscale" command should work:

```bash
$ kwiscale
NAME:
   kwiscale - tool to manage kwiscale application

USAGE:
   kwiscale [global options] command [command options] [arguments...]
   
VERSION:
   0.0.1
   
COMMANDS:
   new      Generate resources (application, handlers...)
   generate Parse configuration and generate handlers, main file...
   help, h  Shows a list of commands or help for one command
   
GLOBAL OPTIONS:
   --project "kwiscale-app" project name, will set $GOPATH/src/[projectname] [$KWISCALE_PROJECT]
   --handlers "handlers"    handlers package name [$KWISCALE_HANDLERS]
   --help, -h           show help
   --generate-bash-completion   
   --version, -v        print the version

```

# Basic application

You may create and modify application by using the `kwiscale` cli or manually. 

## With CLI

It's recommanded to use environment variables to not repeat paths in command. To create an application named "kwiscale-tutorial", please set this environment variable:

```bash
export KWISCALE_PROJECT=kwiscale-tutorial
```

Now, create application:

```bash
kwiscale new app
```

This command should create a directory named `$GOPATH/src/kwiscale-tutorial`.

Create a new handler to respond to the `/` route that is the "index":

```bash
kwiscale new handler index "/"
```

This command makes changes in `$GOPATH/src/kwiscale-tutorial`:

- it appends "/" route in `config.yml`
- it creates `handlers/index.go` containing `IndexHandler`
- it creates or change `main.go` to add route to the "app"

You may now edit `$GOPATH/src/kwiscale-tutorial/handlers/index.go` to add "Get" method

```go
package handlers

import (
    "gopkg.in/kwiscale/framework.v0"
)

type IndexHandler struct{ kwiscale.RequestHandler }

// Add this method to serve
func (h *IndexHandler) Get() {
    h.WriteString("Hello world")
}
```

## Manually 

### With config file


Create a project directory

```
mkdir -p $GOPATH/src/kwiscale-tutorial/handlers
cd $GOPATH/src/kwiscale-tutorial
```

Now create `config.yml`:

```yaml
listen: :8000
session:
  name: kwiscale-tutorial
  secret: Change this to a secret passphrase
```


Edit `./handlers/index.go`:

```go
package handlers

import (
    "gopkg.in/kwiscale/framework.v1"
)

type IndexHandler struct{ kwiscale.RequestHandler }

// Add this method to serve
func (h *IndexHandler) Get() {
    h.WriteString("Hello world")
}

```


Now, create `main.go`:

```go
package main

import (
    "./handlers"
    "gopkg.in/kwiscale/framework.v1"
)

func main(){
    app := kwiscale.NewAppFromConfigFile()
    app.AddRoute("/", handlers.IndexHandler{})
    app.ListenAndServe()
}
```

### Without config file

Create a project directory

```
mkdir -p $GOPATH/src/kwiscale-tutorial/handlers
cd $GOPATH/src/kwiscale-tutorial
```

Edit `./handlers/index.go`:

```go
package handlers

import (
    "gopkg.in/kwiscale/framework.v1"
)

type IndexHandler struct{ kwiscale.RequestHandler }

// Add this method to serve
func (h *IndexHandler) Get() {
    h.WriteString("Hello world")
}

```

Create a `main.go` file:

```go
package main

import (
    "./hanlders"
    "gopkg.in/kwiscale/framework.v1"
)


func main(){
    // Create a new application (nil for default configuration)
    app := kwiscale.NewApp(nil)
    // Add a new route
    app.AddRoute("/", HomeHandler{})
    // start service
    app.ListenAndServe()
}
```


# Launch application

Go to the project path and launch:

```bash
go run main.go
```

By default, application listens ":8000" port. You may now open a browser and go to [http://127.0.0.1:8000](http://127.0.0.1:8000).

The page should display "Hello you", if not please check output on terminal

# Adding routes and handlers with CLI

The CLI helps a lot to create handlers and routes. There are 2 ways:

- call "kwiscale new handler" to create new handler
- or create handlers and add route in config.yml then call `kwiscale generate` to change `main.go`

The first one was presented earlier when we generate "IndexHandler", let's try to create a new handler manually and regenerate the main file.

If you didn't generate application with the CLI, you should change your main.go file to append special comments:

```go
//...
func main(){
    //@routes@
    //@end routes@
}
```

CLI will rewrite routes inside this comments. So **do not append code between this lines**, you will lose your code !

In `handlers` directory, append a new file named "user.go" and append an new "UserHandler":

```go
package handlers

import (
    "gopkg.in/kwiscale/framework.v1"
)

// Our new handler
type UserHandler struct { kwiscale.RequestHandler }

func (h *UserHandler) Get(){
    // "username" should be present in route definition, 
    // see config.yml later
    name := h.Vars["username"] 

    // write !
    h.WriteString("User name:" + name)
}
```

In `config.yml`:

```yaml
routes:
  /:
    handler: handlers.IndexHandler
  /user/{username:.+}:
    handler: handlers.UserHandler
```

As you can see, the route can take a "username" that should respect regular expression ".+" (at least one char). The "username" key in the route definition will set `handler.Vars["username"]`.

Now, call this command:

```bash
kwiscale generate
```

Let's take a look in main.go:

```go
package main

import (
    "kwtest/handlers"

    "gopkg.in/kwiscale/framework.v0"
)

func main() {

    app := kwiscale.NewAppFromConfigFile()

    //@routes@ -- DO NOT REMOVE THIS COMMENT
    app.AddRoute(`/`, handlers.IndexHandler{})
    app.AddRoute(`/user/{id:.+}`, handlers.UserHandler{})
    //@end routes@ -- DO NOT REMOVE THIS COMMENT

    app.ListenAndServe()
}
```

It's **mandatory** to let the special comments to let CLI append routes. Right now, routes are defined in application and you may relaunch application, then open http://127.0.0.1:8000/user/Foo to display "Hello Foo" in you browser.
