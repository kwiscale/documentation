TL;DR - Basic Usage
===================

This chapter is a "TL;DR" [#]_  to briefly describe how to build a kwiscale application. 

.. [#] Too Long; Don't Read

Install kwiscale and CLI::

    go get -u gopkg.in/kwiscale/framework.v1
    go get -u gopkg.in/kwiscale/framework.v1/kwiscale

Set project environment variable to ease development, then enerate application and handler:

.. code-block::
    
    export KWISCALE_PROJECT="mywebsite"
    kwiscale new app 
    kwiscale new handler index "/" home
    cd $GOPATH/src/$KWISCALE_PROJECT

Create `templates/index.tmpl`:

.. code-block:: html
    
    <!doctype html>
    <html>
    <head>
        <title>{{ .Title }}</title>
    </head>
    <body>
        <main>
            {{ template "CONTENT" .}}
        </main>
    </body>
    </html>

Create `template/home.tmpl`:

.. code-block:: html
    
    {{/* override "main.tmpl" */}}
    {{define "CONTENT" }}
        <p>This is the home page</p>
    {{end}}

Open `handlers/index.go` and add a "Get" method for IndexHandler

.. code-block:: go
    
    func (handler *IndexHandler) Get(){
        handler.Render("home.tmpl", map[string]interface{
            "Title": "Home",
        })
    }


Launch application::

    go run main.go


Open http://localhost:8000 - you should see "This is the home page" in the browser. If not, please check logs



