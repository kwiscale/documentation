Deploy Kwiscale in Google Appe Engine
=====================================

**Notes**: Kwiscale is not entirely tested for Google App Engine. 

Test application
----------------

Kwiscale can be deployed in `Google App Engine`_. 

.. _`Google App Engine`: https://cloud.google.com/appengine/docs

You should download the go app engine environment and follow instruction to launch or deploy application.

Open your main package file, eg. hello/hello.go - import kwiscale, create a handler and change `init()` function to handle application:

.. code-block:: go

    package hello

    import (
        "net/http"
        "gopkg.in/kwiscale/framework.v1"
    )

    type IndexHandler struct { kwiscale.RequestHandler }

    func (handler *IndexHandler) Get() {
        handler.WriteString("Hello from kwiscale in app engine !!!")
    }

    // Change
    func init() {
        app := kwiscale.NewApp(nil);
        app.AddRoute("/", &IndexHandler{})
        http.Handle("/", app)
    }


You may now launch application::
    
    goapp serve

Open http://localhost:8080 to see your application running.

Behind the scene
----------------

With App Engine, should not give a full application program (no "main" package). You must provide a package that provides "handles". App Engine makes use of "http" package to "Listen" connection. "Http" package provides a "handling" system to map functions and handler to the default "ServerMux".

Kwiscale application respects the `net/http.Handler` interface (providing ServeHTTP method). That's why you can request http to "handle" your application.

By calling:

.. code-block:: go

    http.Handle("/", app)

you ask http package to use "app" for the entire route that matches "/". That means that the entire uri you will call will be sent to your "app". That way, you can register as much route as you want with "AddRoute" method, they will match "/" url. 

