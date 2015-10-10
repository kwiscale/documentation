# Introduction

kwiscale make use of [html/template](http://golang.org/pkg/html/template/) provided by Golang. This template engine is fast and can give enough of functionalities to create web pages or generate some other formats.

Kwiscale appends an override system based on a simple template comment that will allow you to reuse bases structure.

# Example

Create a template directory named "templates". Create a file named "templates/index.html" and append this content:

```html
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
```

Then, in `main.go`:

```go
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
```

# Override templates

TODO


