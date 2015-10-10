# RequestHandler
## Usage

RequestHandler handles HTTP verbs (Get, Post, Put, Delete, Head, Pathch, Trace, Option) as structure method.

It implements IBaseHandler, each HTTP verb is already implemented but returns a 404 Error by default. That way, you only have to create your own RequestHandler based type to implement the needed method.


## Call story

When a client enter an URL, the framework finds the right handler to use. Then your own request handler is spawned (as a new instance) and a list of methods are called:

- `Init()` - you can override this method to initialize the response or reject client (usefull for authentification and authorisation check). This method should return an integer and a nil error to let handler continue. If error is not nil, the integer is used as status returne to the clien
- Http method - `Get()` or `Post()`, and so on
- `Destroy()` - Called after response is sent to client

You may override this methods. Note that `Init()` method must return integer status and an `error` (that should be `nil`) if you want to continue to serve with HTTP verb method.

Example:

```go
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
```

This `PrivateHandler` can be used as a "parent" handler to privatize other handlers:

```go

type AdminHandler { PrivateHandler }

// only if user is authenticated
func (ah *AdminHandler) Get(){
    //..
}

```
