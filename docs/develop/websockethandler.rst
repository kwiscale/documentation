Websocket Handler
=================

Usage
-----

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
-----

The most common way to use websocket is to listen JSON message or text
message. Then answer to the client.

To use JSON, you must implement WSJsonHandler, that means you should
impement :

``OnJSON (interface{}, error)``

Example:

.. code:: go


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

.. code:: go


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
-----------------

You may implement your own server loop implementing ``WSServer``
interface, that means you may implement the method:

``Serve()``

The method should make a loop to read messages from client.

Example:

.. code:: go


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
-----

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
