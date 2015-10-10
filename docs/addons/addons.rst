Addons creation
===============

Kwiscale provides extensibility for database, session and template
engines.

Database Addons
---------------

Goal
~~~~

Kwiscale aims to give a "database engine agnostic" database system to
allows usage of a lot of database. To provide an ORM there are a lot of
complexity to manage.

Kwiscale project refuses to reinvent the wheel and provides a simple
interface to implement in addons.

Addons can:

-  simply map some well known packages to the interface
-  manage database itself

Build a database addon
~~~~~~~~~~~~~~~~~~~~~~

Create a directory where you want to develop a driver. Then create the
package development file(s).

The name of your package is not important. The standard form is
"kwiscaledb[name]". The package name will not be seen by developpers.

The addon should import "kwiscale" and call ``RegisterDatabase()``
function. Commonly, you have to call this function in the ``init()``
function of you package.

.. code:: go

    package kwiscaledbexample

    import (
        "gopkg.in/kwiscale/framework.v1"
    )

    func init(){
        kwiscale.RegisterDatabase("example", MyDB{})   
    }

    // Should implement kwisclae.DB interface
    type MyDB struct {
        //...
    }

Interface to implement
~~~~~~~~~~~~~~~~~~~~~~

The interface to implement:

.. code:: go

    type DB interface {
        SetOptions(DBOptions)
        Init()
        Insert(what interface{}) error
        Get(id, result interface{})
        Find(query Q) DBQuery
        Update(where map[string]interface{}, what interface{}) error
        Delete(what interface{}) error
        Close()
    }

-  SetOptions take a ``map[string]interface{}`` (kwiscale.DBOptions
   type) that are set in ``kwiscale.Config.DBOptions`` or db.options in
   yaml file. They are commonly used to set database uri, user and
   password to use, and so on
-  Init() is the method called once by kwiscale to initialise database
   package. It's called when the app begins to serve
-  Insert() takes a object to insert in database
-  Get(id, res) takes an id to fetch and set the result on "res"
-  Find(query Q) set the search query. Note that kwiscale.Q is a
   map[string]interface{} that you have to parse
-  Update(where, what) does a query to update data. The "where" is used
   to find elements in database, and "what" is the values structure to
   set
-  Delete() removes data that matches "what" interface
-  Close() close connection

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

.. code:: go

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

.. code:: go

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

Session engine
--------------
