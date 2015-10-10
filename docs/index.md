# Kwiscale Framework

Welcome to the kwiscale documentation. 

## What is Kwiscale

Kwiscale is a framework built following common model. It provides methods to create handlers holding HTTP verbs that are called following the client request.

Kwiscale can be used to create website, API or Websocket server. 

Basically, Kwiscale offers a way to create website built on MVC. 

## What is not Kwiscale

Kwiscale is not a CMS, not a blog engine... It's a framework that may be used to create CMS or blog engine, or REST API server. If you need comparaison, Kwiscale is more like Symfony or Zend for PHP. But Kwiscale is made in Go.

# Framework design

Kwiscale is a HTTP Handler framework. As [WebApp2](https://webapp-improved.appspot.com/) for Python, the request process is working in this order:

- User call a route with HTTP Verb (GET, POST, HEAD...)
- Application fetch a *handler* that matches this route
- If the handler exists, application instanciate this handler and call the given HTTP verb as a method (Get(), Post()...)
- If route doesn't match, a HTTP 404 ERROR is sent to client

# CLI

A Command Line Interface is provided to help application managment. The next documentation section delivers command to use along the devlopment process.

# About Go conventions

You will notice that Kwiscale doesn't use the largely used handler function design that takes *http.ResponseWriter* and *http.Request*. Also, Kwiscale use a *complex* structure composition to *simulate* class/methods purpose. It's important to understand that choice.

The main goal of Kwiscale is to make web application development as easy as possible. Even if recommandation is to not follow classic "OOP" design, it's not prohibited to use some of interessing concepts comming from "OOP".

That's why we decided to implement methods that deals with ResponseWriter and Request internaly, letting developpers to use 

```go
h.WriteString("Hello")
//or 
h.Render("mytemplate.html", context)
```

So, you will not find the *standard* and largely used:

```go
func Get (w http.ResponseWriter, r *http.Request)
```

But you will be able to get this values if you really need them:

```go
func (h *Handler) Get(){
    w := h.GetResponse()
    r := h.GetRequest()
}
```

## Extensible

Kwiscale provides functions to plug some addons. For example you may use Pongo2 template engine or build your own Session Handler for "memcache".

## What a strange name

kwiscale is a transformation of a french word: [Quiscale](https://fr.wikipedia.org/wiki/Quiscale) that is a bird classification. The word is rarely used in french. So why that name ? That's simple. I was searching a name for the framework and, because I didn't find any idea, I used the "random page" link on Wikipedia website. After 10 clicks, I saw this name that I decided to keep.

