# Pongo2 template

Pongo2 is a template engine that is quasi compatible with Jinja2 (python) or Twig (PHP). Syntax is powerfull and designed to be easy to learn.

To use Pongo2 template, install addon:

```
go get gopkg.in/kwiscale/template-pongo2.v1
```

Create templates directory and set `templates/index.html`:

```html
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
```

Then, in `main.go`:

```go
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
```
