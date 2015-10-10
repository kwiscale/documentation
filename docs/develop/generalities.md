# Behind the scene

kwiscale is a web framework that uses [GorillaToolkit](http://www.gorillatoolkit.org/). The main purpose is to allow developers to create handlers that serve reponses.

There are two Handlers types:

- RequestHandler  to respond to HTTP requests (Get, Post, Put, Delete, Patch, Trace, Head)
- WebSocketHandler to serve websocket connection to client

Kwiscale proposes addon system to be able to plug template engines and session engines. By default you may be able to use the standard html/template package provided by Go and session by encrypted cookies provided by GorillaToolkit.

# What is provided

## Handler story

When an user calls a route, Kwiscale will find the corresponding handler in a stack. When a route matches, kwiscale app detect handler type and call a serie of methods:



![Handler story](http://plant.metal3d.org/png/c2tpbnBhcmFtIGFjdGl2aXR5IHsKICAgIEJvcmRlckNvbG9yICMwMDAwMDAKICAgIEJhY2tncm91bmRDb2xvciAjRkFGQUZBCiAgICBBcnJvd0NvbG9yICMyMjIyMjIKICAgIFNoYWRvdyAjQUFBQUFBCn0KCnNraW5wYXJhbSBub3RlIHsKICAgIEJhY2tncm91bmRDb2xvciAjQUE4ODg4CiAgICBCb3JkZXJDb2xvciAjOTk1NTU1Cn0KCnN0YXJ0CjpGaW5kICoqYmVzdCoqIHJvdXRlIGZvciB0aGUgcmVxdWVzdGVkIFVSTDsKaWYgKFJvdXRlIGZvdW5kKSB0aGVuIChubykKICAgIDo9SFRUUCA0MDQgRVJST1I7CiAgICBlbmQKZWxzZSAoeWVzKQogICAgOj1TdGFydCB0aGUgc2VydmUgcHJvY2VzczsKCiAgICBpZiAoaGFuZGxlci5Jbml0KCkgPT4gc3RhdHVzLCBlcnJvcikgdGhlbiAoZXJyb3IgIT0gbmlsKQogICAgICAgIGlmIChzdGF0dXMgPiAtMSkgdGhlbiAoeWVzKQogICAgICAgICAgICA6SFRUUCBFcnJvciAoc3RhdHVzKTsKICAgICAgICBlbmRpZgogICAgICAgIGVuZAogICAgZW5kaWYKICAgIAogICAgZm9yawogICAgICAgIDpoYW5kbGVyIGlzIEhUVFBSZXF1ZXN0SGFuZGxlcnwKICAgICAgICBpZiAoSFRUUCB2ZXJiIGlzIGNvcnJlY3QpIHRoZW4gKHllcykKICAgICAgICAgICAgOkNhbGwgdGhlIHJlcXVlc3QgSFRUUCB2ZXJiOgogICAgICAgICAgICAKICAgICAgICAgICAgaGFuZGxlci5HZXQoKSAKICAgICAgICAgICAgaGFuZGxlci5Qb3N0KCksIAogICAgICAgICAgICAuLi47CiAgICAgICAgZWxzZSAobm8pCiAgICAgICAgICAgIDo9SFRUUCBCYWQgbWV0aG9kOwogICAgICAgICAgICBlbmQKICAgICAgICBlbmRpZgogICAgICAgIAogICAgZm9yayBhZ2FpbgogICAgICAgIDpoYW5kbGVyIGlzIFdTSGFuZGxlcnwKICAgICAgICBpZihoYW5kbGVyIGltcGxlbWVudHMgd2Vic29ja2V0KSB0aGVuICh5ZXMpCiAgICAgICAgICAgIDpoYW5kbGVyLk9uQ29ubmVjdCgpOwogICAgICAgICAgICA6U2VydmUgV1NIYW5kbGVyCiAgICAgICAgICAgIAogICAgICAgICAgICBoYW5kbGVyLk9uSnNvbigpCiAgICAgICAgICAgIGhhbmRsZXIuT25NZXNzYWdlKCkKICAgICAgICAgICAgaGFuZGxlci5TZXJ2ZXIoKS4uLjsKICAgICAgICBlbHNlIChubykKICAgICAgICAgICAgOj1IVFRQIEJhZCBNZXRob2Q7CiAgICAgICAgICAgIGVuZAogICAgICAgIGVuZGlmCiAgICBlbmRmb3JrCiAgICAKICAgIDpEZXN0cm95KCk8CiAgICBlbmQKZW5kaWY=)

## Serve static files

**Important** The static handler provided by kwiscale is provided for development and **not for the production**. It's not recommanded to let Kwiscale serve directoy web application, you'd rather use HTTP Server as nginx or Apache as reverse proxy. That way, the HTTP server will serve static files instead of using static handler provided by Kwiscale. 

To serve static files  (css, js, images, and so on) you may configure Kwiscale.App like this:

```
cfg := kwiscale.Config{
    StaticDir: "./statics",
}
app := kwiscale.NewApp(&cfg)
```

Kwiscale uses the directory name to serve files that resides inside. You can now hit URL http://127.0.0.1:8000/statics/...

Note that static handler doesn't make directory index. Hitting the static route without any filename will result on 404 Error.


## URL Routing

Kwiscale make use of GorillaToolkit route system. This routing implementation allows you to set url parameters and to reverse an url from a handler name.

Example:

```go
type MyHandler struct { kwiscale.RequestHandler }

func (h *UserHandler) Get(){
    userid := h.Vars["userid"]
}

func main(){
    //...
 
    // Add a route that need an user id named "userid".
    // Route parameters are regular expression.
    app.AddRoute("/user/{userid:\d+}", UserHandler{})

    //...
}

```

The corresponding route could be "/user/123456", then in `Get()`, `userid` contains a string value: "123456".


To reverse an url, you need the name of the handler. The "kwiscale.App" can provide the named route and you may use `URL` to return the corresponding URL. Here is an example:

```go

// Route /user/{userid:\d+}
url := myhandler.GetApp().GetRoute("main.UserHandler").URL("userid", "123456")


// If myhandler is the wanted handler
url := myhandler.GetURL("userid", "123456")

```

## Named route

If you want to not use handler name based on reflected value, you may use `AddNamedRoute()` instead:

```go
app.AddNamedRoute("/user/{userid:\d+}", UserHandler{}, "users")
```


So, to reverse URL:

```go
// Route /user/{userid:\d+}
url := myhandler.GetApp().GetRoute("users").URL("userid", "123456")
```



