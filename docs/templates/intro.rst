Templates
=========

kwiscale uses html/template from the built-in package of Go. You may use
`Pongo2 <https://github.com/flosch/pongo2>`__ template engine using the
`kwiscale addon <https://github.com/kwiscale/template-pongo2>`__.

Kwiscale appends an override system based on a simple template comment
that will allow you to reuse bases structure.

Built-in template engine
------------------------

Create a template directory named "templates". Create a file named
"templates/index.html" and append this content:

.. code:: html

    <!doctype html>
    <html>
    <head>
        <title>{{ .Title }}</title>
    </head>
    <body>
    <div>
        {{ .Content }}
    </div>
    </body>
    </html>

Then, in ``main.go``:

.. code:: go

    package main

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    type HomeHandler struct { kwiscale.RequestHandler }

    func (h *HomeHandler) Get(){
        h.Render("index.html", map[string]string{
            "Title": "The title of the page",
            "Content" : "This is the content",
        })
    }

    func main(){
        app := kwiscale.NewApp(&kwiscale.Config{
            TemplateDir : "./templates",
        })
        app.AddRoute("/", HomeHandler{})
        app.ListenAndServe()
    }

Pongo2 template
---------------

Pongo2 is a template engine that is quasi compatible with Jinja2
(python) or Twig (PHP). Syntax is powerfull and designed to be easy to
learn.

To use Pongo2 template, install addon:

::

    go get gopkg.in/kwiscale/template-pongo2.v1

Create templates directory and set ``templates/index.html``:

.. code:: html

    <!doctype html>
    <html>
    <head>
        <title>{% Title %}</title>
    </head>
    <body>
    <div>
        {% Content %}
    </div>
    </body>
    </html>

Then, in ``main.go``:

.. code:: go

    package main

    import (
        "gopkg.in/kwiscale/framework.v1"
        _ "gopkg.in/kwiscale/template-pongo2.v1"
    )

    type HomeHandler struct { kwiscale.RequestHandler }

    func (h *HomeHandler) Get(){
        h.Render("index.html", map[string]string{
            "Title": "The title of the page",
            "Content" : "This is the content",
        })
    }

    func main(){
        app := kwiscale.NewApp(&kwiscale.Config{
            TemplateDir:    "./templates",
            TemplateEngine: "pongo2"
        })
        app.AddRoute("/", HomeHandler{})
        app.ListenAndServe()  
    }
