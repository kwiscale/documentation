Developping with Kwiscale
=========================

Project Structure
-----------------

Recommandation is not obligation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The common structure we give here is not mandatory. You can prefer other
file structure and project managment.

The standard Kwiscale structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In a common usage, the following file structure is recommanded:

::

    [projectpath]/
        main.go
        handlers/
            index.go
            [other name].go
            ...
        templates/
            index.html
            - common/
                footer.html
                header.html
                menu.html
            - home/
                main.go
        statics/
            - js/
                ...
            - css/
                ...

Note that "handlers" directory may contains subpackages. The goal is to
classify HTTP handlers in the same directory. An example:

::

    handlers/
        index.go
        user/
            auth.go
            register.go
            profile-edition.go
        cms/
            page.go
            edit.go
        blog/
            index.go
            ticket.go


Serve static files
------------------

**Important** The static handler provided by kwiscale is provided for
development and **not for the production**. It's not recommanded to let
Kwiscale serve directoy web application, you'd rather use HTTP Server as
nginx or Apache as reverse proxy. That way, the HTTP server will serve
static files instead of using static handler provided by Kwiscale.

To serve static files (css, js, images, and so on) you may configure
Kwiscale.App like this:

.. code-block:: go

    cfg := kwiscale.Config{
        StaticDir: "./statics",
    }
    app := kwiscale.NewApp(&cfg)

Kwiscale uses the directory name to serve files that resides inside. You
can now hit URL http://127.0.0.1:8000/statics/...

Note that static handler doesn't make directory index. Hitting the
static route without any filename will result on 404 Error.

URL Routing
-----------

Kwiscale make use of GorillaToolkit route system. This routing
implementation allows you to set url parameters and to reverse an url
from a handler name.

Example:

.. code-block:: go

    type MyHandler struct { kwiscale.RequestHandler }

    func (h *UserHandler) Get(){
        userid := h.Vars["userid"]
    }

    func main(){
        //...
     
        // Add a route that need an user id named "userid".
        // Route parameters are regular expression.
        app.AddRoute("/user/{userid:\d+}", &UserHandler{})

        //...
    }

The corresponding route could be "/user/123456", then in ``Get()``,
``userid`` contains a string value: "123456".

To reverse an url, you need the name of the handler. The "kwiscale.App"
can provide the named route and you may use ``URL`` to return the
corresponding URL. Here is an example:

.. code-block:: go


    // Route /user/{userid:\d+}
    url := myhandler.App().Route("main.UserHandler").URL("userid", "123456")


    // If myhandler is the wanted handler
    url := myhandler.GetURL("userid", "123456")


If you want to not use handler name based on reflected value, you may
use ``AddNamedRoute()`` instead:

.. code-block:: go

    app.AddNamedRoute("/user/{userid:\d+}", &UserHandler{}, "users")

So, to reverse URL:

.. code-block:: go

    // Route /user/{userid:\d+}
    url := myhandler.App().Route("users").URL("userid", "123456")

Behind the scene
----------------

kwiscale is a web framework that uses
`GorillaToolkit <http://www.gorillatoolkit.org/>`__. The main purpose is
to allow developers to create handlers that serve reponses.

There are two Handlers types:

-  RequestHandler to respond to HTTP requests (Get, Post, Put, Delete,
   Patch, Trace, Head)
-  WebSocketHandler to serve websocket connection to client

Kwiscale proposes addon system to be able to plug template engines and
session engines. By default you may be able to use the standard
html/template package provided by Go and session by encrypted cookies
provided by GorillaToolkit.

When a user calls a route, Kwiscale will find the corresponding handler
in a stack. When a route matches, kwiscale app detect handler type and
call a serie of methods (see :ref:`handler-process`)

.. _handler-process:

.. figure:: ../images/handler-process.png
   :alt: Handler story

   Handler story diagram


RequestHandler
--------------

Usage
~~~~~

RequestHandler handles HTTP verbs (Get, Post, Put, Delete, Head, Pathch,
Trace, Option) as structure method.

It implements BaseHandler, each HTTP verb is already implemented but
returns a 404 Error by default. That way, you only have to create your
own RequestHandler based type to implement the needed method.

Call story
~~~~~~~~~~

When a client enter an URL, the framework finds the right handler to
use. Then your own request handler is spawned (as a new instance) and a
list of methods are called:

-  ``Init()`` - you can override this method to initialize the response
   or reject client (usefull for authentification and authorisation
   check). This method should return an integer and a nil error to let
   handler continue. If error is not nil, the integer is used as status
   returne to the clien
-  Http method - ``Get()`` or ``Post()``, and so on
-  ``Destroy()`` - Called after response is sent to client

Response to HTTP Method
~~~~~~~~~~~~~~~~~~~~~~~

There are two possibilities:

- Create a method with no parameters so you have to fetch url vars yourself
- Create a method with parameters so you should respect url params order

For example, to respond to the route "``/page/{heading:.+}/{title:.+}``" you may 

- create ``Get()`` method and get "heading" and "title" with "handler.Vars[]"
- create ``Get(heading, title string)`` 

.. code-block:: go

    func (handler *PageHandler) Get(){
        heading := handler.Vars["heading"]
        title := handler.Vars["title"]
        //...
    }


To use url mapping:


.. code-block:: go

    func (handler *PageHandler) Get(heading, title string){
        // heading and title are set
    }


Note that if you are using url mapping, the parameters are typed and kwiscale will try to cast url vars. For example:

Route: "``/user/{id:\d+}``":


.. code-block:: go

    // we set "id" as integer
    func (handler *UserHandler) Get(id int) {
        log.Println(i)
    }

This is way simpler than:

.. code-block:: go

    // we set "id" as integer
    func (handler *UserHandler) Get() {
        id := strconv.Atoi(handler.Vars["id"])
    }

Note that url mapping uses reflection and could be slower that ``using handler.Vars``.

Init and Destroy
~~~~~~~~~~~~~~~~

You may override this methods. Note that ``Init()`` method must return
integer status and an ``error`` that should be ``nil`` if you want to
continue to serve with HTTP verb method.

Example:

.. code-block:: go

    type PrivateHandler struct { kwiscale.RequestHandler}

    // Initialize - test is client is authenticated 
    func (h *PrivateHandler) Init(){
        isauth, ok := h.GetSession("auth")
        if !ok || !isauth.(bool) {
            return http.StatusForbidden, errors.New("Unauthaurized")
        }

        // authenticated user, we can continue
        return -1, nil
    }

    // When GET method happends.
    func (h *HomeHandler) Get() {
        //...
    }

    // After reponse sent to the client.
    func (h *HomeHandler) Destroy(){

    }

This ``PrivateHandler`` can be used as a "parent" handler to privatize
other handlers:

.. code-block:: go


    type AdminHandler { PrivateHandler }

    // only if user is authenticated
    func (ah *AdminHandler) Get(){
        //..
    }

Websocket Handler
-----------------

Usage
~~~~~

WebSocketHandler will accept websocket connection and react on events.
There are 3 ways to intercept client messages:

-  on json message
-  on text message
-  serve in a loop

Using the URL path, WebSocketHandler provides way to send message in
several form to :

-  the connected client only
-  the "room" clients
-  the entire clients list connected to the server

**Important** Only one of ``Serve()``, ``OnJSON()`` or ``OnMessage()``
method should be declared. If you declared more that one of this method,
only one of those methods will be use. The priority order is:

1. Serve
2. OnJSON
3. OnMessage

Basic
~~~~~

The most common way to use websocket is to listen JSON message or text
message. Then answer to the client.

To use JSON, you must implement WSJsonHandler, that means you should
impement :

``OnJSON (interface{}, error)``

Example:

.. code-block:: go


    // A standard type to communicate
    type Message struct {
        From string
        Message string
    }

    type MyWS struct { kwiscale.WebSocketHanlder}

    func (w *MyWS) OnJSON(i interface{}, err error) {

        if err != nil {
            // an error occured
            return
        }

        // i is an interface{} type, you may cast type
        if i, ok := i.(Message); ok {
            //... work with message

            // Send response
            w.SendJSON(Message{
                From: "server",
                Message: "Hello",
            })
        }
    }

If the ``error`` given as argument is not ``nil``, that means that a
problem occured with client connection. So the connection is probably
closed. After the method returns, the connection will be removed. Client
should reconnect itself to be able to communicate with the server.

To work with text message instead of JSON, you must implement
WSStringHandler interface. That means you must implement

``OnMessage(string, err)``

Example:

.. code-block:: go


    type MyWS struct { kwiscale.WebSocketHanlder}

    func (w *MyWS) OnMessage(s string, err error) {

        if err != nil {
            // an error occured
            return
        }

        // Send response as text
        w.SendText("Hello")
    }

Serving WebSocket
~~~~~~~~~~~~~~~~~

You may implement your own server loop implementing ``WSServer``
interface, that means you may implement the method:

``Serve()``

The method should make a loop to read messages from client.

Example:

.. code-block:: go


    type MyWS struct {kwiscale.WebSocketHandler}

    func (ws *MyWS) Serve() {
        conn := ws.GetConn();
        for {
            var i interface{}
            err := conn.ReadJSON(&i)
            if err != nil {
                break
            }

            // works with interface...

            // send message
            ws.SendJSON(map[string]string{
                "message" : "Hello !",
            })
        }
    }

Using ``Serve()`` can be very usefull to make specific manipulation on
connection or to customize some behaviours.

Rooms
~~~~~

In the following explanation, ``XXX`` shoud be replace by ``JSON`` or
``Text``, respectivally to send JSON or string message. The complete
list follows explanations.

Each websocket connection is kept in a named "room". A room is a
compartimented list where resides connections. Each room is created
using the websocket path given in url.

That could be very usefull if you want to create a chatroom with several
channels.

For example, your website allows 2 routes to connect with websocket:

-  '/chat/general'
-  '/chat/administrators'

Then, in the handler, if you call one of the
``SendXXXToThisRoom``\ method, each clients connected to the the route
named "/chat/administrators" will receive the message, but **not** those
that are only connected to "/chat/general".

To send message to the entire connected clients list, you may use one of
the ``SendXXXToAll()``.

Connected to another room, there is a way to send client to a specific
room: ``SendXXXToRoom(name string)``.

For JSON:

-  ``SendJSONToThisRoom(interface{})`` to send json to this room
-  ``SendJSONToRoom(string, interface{})`` to send json to a specific
   room
-  ``SendJSONToAll(interface{})`` to send json to the entire clients
   list

For text:

-  ``SendTextToThisRoom(interface{})`` to send text message to this room
-  ``SendTextToRoom(string, interface{})`` to send text message to a
   specific room
-  ``SendTextToAll(interface{})`` to send text message to the entire
   clients list
