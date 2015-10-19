Addons creation
===============

Kwiscale provides extensibility for session and template
engines. Soon, an ORM will be provided and you will be able to 
create database drivers.

Template addons
---------------

Goal
~~~~

Built-in template is based on "html/template" built-in package and
doesn't need any dependency. But you may prefer to use other templates
(eg. `Pango2 </templates/pongo2>`__)

Kwiscale implements a template addons system to allows usage of other
templates.

Build a template addon
~~~~~~~~~~~~~~~~~~~~~~

Create a directory where you'll develop template addon. The package file
should call ``kwiscale.RegisterTemplateEngine()`` function.

The package name is not important and will not be visible by
developpers. But a common way to name the package is
``kwiscaletemplate[name]``.

Commonly, you have to call this function in the ``init()`` function of
your package.

.. code-block:: go

    package kwiscaletemplateexample

    import(
        "gopkg.in/kwiscale/framework.v1"
    )

    func init(){
        kwiscale.RegisterTemplateEngine("example", MyTemplateEngine{})
    }

    // should implement kwiscale.Template interface
    type MyTemplateEngine struct {
        //...
    }

Interface
~~~~~~~~~

The interface to implement:

.. code-block:: go

    type Template interface {
     // Render method to implement to compile and run template
     // then write to RequestHandler "w" that is a io.Writer.
     Render(w io.Writer, template string, ctx interface{}) error

     // SetTemplateDir should set the template base directory
     SetTemplateDir(string)

     // SetOptions pass TplOptions to template engine
     SetTemplateOptions(TplOptions)
    }

-  Render(w, template, ctx) should write content in the writer "w".
   "template" is the template filename to use and "ctx" contains values
   to set in the template
-  SetTemplateDir() should register where templates reside. The path
   comes from template.dir yaml value or Config.TemplateDir
-  SetTemplateOptions() receive other configuration that comes from
   Config.TemplateOptions or templates.options yaml configuration. Some
   template engine may need some special configuration and they are
   provided that way

