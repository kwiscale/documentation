Getting Started
===============

Prerequists
-----------

You have to install ``go`` and have set ``$GOPATH`` to point on a
writable directory.

You need to set ``$PATH`` to append ``$GOPATH/bin``.

An example ``.bashrc`` modification:

.. code-block:: bash

    export GOPATH=~/goproject
    export PATH=$GOPATH/bin:$PATH

After having set those variables, you **must** reset your shell. Restart
your session or call:

.. code-block:: bash

    source ~/.bashrc

It's recommanded to install ``goimports`` command that kwiscale CLI will
try to call:

.. code-block:: bash

    go get -u golang.org/x/tools/cmd/goimports

**Important**: If you don't install ``goimports``, kwiscale CLI may have
problem to generate a working main.go file.

Installation
------------

Kwiscale is a standard Go package, so you may install it with the
``go get`` command.

Please, **don't use github url** but use the
`gopkg.in <http://gopkg.in>`__ url that provides versionning. , the
package is at
`gopkg.in/kwiscale <http://gopkg.in/kwiscale/framework.v1>`__

Installation is made by the following command:

::

    go get gopkg.in/kwiscale/framework.v1

The version ``v1`` is the current version. To use master version, please
use ``v0`` (while it's not recommanded either you need a specific
feature that is not yet in next version).

At this time, kwiscale is installed and you can develop service.

You may install kwiscale cli:

::

    go get gopkg.in/kwiscale/framework.v1/kwiscale

Right now, if you set ``$GOPATH/bin`` in your ``$PATH``, the "kwiscale"
command should work:

.. code-block:: bash

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
       --project "kwiscale-app" project name, will set \
                $GOPATH/src/[projectname] [$KWISCALE_PROJECT]
       --handlers "handlers"    handlers package name \
                [$KWISCALE_HANDLERS]
       --help, -h           show help
       --generate-bash-completion   
       --version, -v        print the version

Basic application
-----------------

You may create and modify application by using the ``kwiscale`` cli or
manually.

With CLI
~~~~~~~~

It's recommanded to use environment variables to not repeat paths in
command. To create an application named "kwiscale-tutorial", please set
this environment variable:

.. code-block:: bash

    export KWISCALE_PROJECT=kwiscale-tutorial

Now, create application:

.. code-block:: bash

    kwiscale new app

This command should create a directory named
``$GOPATH/src/kwiscale-tutorial``.

Create a new handler to respond to the ``/`` route that is the "index":

.. code-block:: bash

    kwiscale new handler index "/"

This command makes changes in ``$GOPATH/src/kwiscale-tutorial``:

-  it appends "/" route in ``config.yml``
-  it creates ``handlers/index.go`` containing ``IndexHandler`` and register call
-  it creates or change ``main.go`` to add route to the "app"

You may now edit ``$GOPATH/src/kwiscale-tutorial/handlers/index.go`` to
add "Get" method

.. code-block:: go

    package handlers

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    func init() {
        kwiscale.Register(&IndexHandler{})
    }

    type IndexHandler struct{ kwiscale.RequestHandler }

    // Add this method to serve
    func (h *IndexHandler) Get() {
        h.WriteString("Hello world")
    }


Manually
~~~~~~~~

With config file
^^^^^^^^^^^^^^^^

Create a project directory

::

    mkdir -p $GOPATH/src/kwiscale-tutorial/handlers
    cd $GOPATH/src/kwiscale-tutorial

Now create ``config.yml``:

.. code-block:: yaml

    listen: :8000
    session:
      name: kwiscale-tutorial
      secret: Change this to a secret passphrase

Edit ``./handlers/index.go``:

.. code-block:: go

    package handlers

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    func init(){
        kwiscale.Register(&IndexHandler{})
    }

    type IndexHandler struct{ kwiscale.RequestHandler }

    // Add this method to serve
    func (h *IndexHandler) Get() {
        h.WriteString("Hello world")
    }

Now, create ``main.go``:

.. code-block:: go

    package main

    import (
        _ "kwiscale-tutorial/handlers"
        "gopkg.in/kwiscale/framework.v1"
    )

    func main(){
        app := kwiscale.NewAppFromConfigFile()
        app.ListenAndServe()
    }

**Note**: ``handlers`` package is imported with an underscore here. As you can see, we don't use the package in ``main.go`` but ``app`` will register handlers itself. If the package is not imported, application will panic.

Without config file
^^^^^^^^^^^^^^^^^^^

Create a project directory

::

    mkdir -p $GOPATH/src/kwiscale-tutorial/handlers
    cd $GOPATH/src/kwiscale-tutorial

Edit ``./handlers/index.go``:

.. code-block:: go

    package handlers

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    func init(){
        // not mandatory but recommanded if you want
        // to use config.yml file later to map routes.
        kwiscale.Register(&IndexHandler{})
    }

    type IndexHandler struct{ kwiscale.RequestHandler }

    // Add this method to serve
    func (h *IndexHandler) Get() {
        h.WriteString("Hello world")
    }

Create a ``main.go`` file:

.. code-block:: go

    package main

    import (
        "kwiscale-tutorial/handlers"
        "gopkg.in/kwiscale/framework.v1"
    )

    func main(){
        // Create a new application (nil for default configuration)
        app := kwiscale.NewApp(nil)

        // Add a new route
        app.AddRoute("/", &handlers.IndexHandler{})

        // start service
        app.ListenAndServe()
    }

Launch application
------------------

Go to the project path and launch:

.. code-block:: bash

    go run main.go

By default, application listens ":8000" port. You may now open a browser
and go to http://127.0.0.1:8000.

The page should display "Hello you", if not please check output on
terminal

Adding routes and handlers
--------------------------

The CLI helps a lot to create handlers and routes.

But you may create handlers and routes yourself inside ``config.yml`` file and appending your handler package file in application.


Create handler with CLI:
~~~~~~~~~~~~~~~~~~~~~~~~~~~

:: 
    
    kwiscale new handler user "/user/{username:.+}"

Create handler without CLI:
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In ``handlers`` directory, append a new file named "user.go" 

In ``config.yml`` you have to set new route if you didn't use CLI:

.. code-block:: yaml

    routes:
      /:
        handler: handlers.IndexHandler
      /user/{username:.+}:
        handler: handlers.UserHandler



Both CLI and manually:
~~~~~~~~~~~~~~~~~~~~~~

Now append a method to respond to GET:

.. code-block:: go

    package handlers

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    func init(){
        // Mandatory if you are using config.yml to 
        // map routes and handlers.
        kwiscale.Register(&UserHandler{})
    }

    // Our new handler
    type UserHandler struct { kwiscale.RequestHandler }

    func (h *UserHandler) Get(){
        // "username" should be present in route definition, 
        // see config.yml later
        name := h.Vars["username"] 

        // write !
        h.WriteString("User name:" + name)
    }


As you can see, the route can take a "username" that should respect
regular expression ".+" (at least one char). The "username" key in the
route definition will set ``handler.Vars["username"]`` in UserHandler.

Right now, routes and handlers are defined, you may relaunch application and open http://127.0.0.1:8000/user/Foo to display "Hello Foo" in you browser.
