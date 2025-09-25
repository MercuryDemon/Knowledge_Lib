25-09-2025 21:55
Tags: #Go #lesson 
# Шаблоны для вывода текста

## ТЕКСТОВЫЙ ТЕМПЛЕЙТ
import "text/template"

правило хорошего тона - шаблонизация в виде отдельного шаблона

пример работы с шаблоном:
```Go
package main

import (

"fmt"

"net/http"

"text/template"

)

type tplParams struct {
URL string
Browser string
}
  
// для доступа к элементам перед их именем стоит ".", если работаем с циклом, то точка это элемент цикла
const EXAMPLE = `
Browser {{.Browser}}

you at {{.URL}}
`

func handle(w http.ResponseWriter, r *http.Request) {
	
	tmpl := template.New(`example`)

	tmpl, _ = tmpl.Parse(EXAMPLE)

  

	params := tplParams {
		URL: r.URL.String(),
		Browser: r.UserAgent(),
	}
	tmpl.Execute(w, params)
}

func main() {
http.HandleFunc("/", handle)

fmt.Println("starting server at :8080")
}
```
 простой текстовый шаблон, где нам ненужно добавлять какие-то параметры от пользователя

## HTTP ТЕМПЛЕЙТ

```Go
package main

import (
	"fmt"
	"net/http"
	"text/template"
)

type User struct {
	ID int
	Name string
	Active bool
}

func main() {
	tmpl := template.Must(template.ParseFiles("templates_testing/users.html")) 

	users := []User{
	User{1, "Nikitos", true},
	User{2, "IvAn", true},
	User{3, "fish", false},
	}

  	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl.Execute(w,	
			struct {	
				Users []User
			}{
				users,
			})
	})
		
	fmt.Println("starting server ar :8080")
	
	http.ListenAndServe(":8080", nil)
}
```
html - файл 
```html
<html>
	<body>
		<h1>Users</h1>
		{{range .Users}}
		<b>{{.Name}}</b>
		{{if .Active}}active{{end}}
		<br />
		{{end}}
	</body>
</html>
```

## Шаблоны позволяют вызывать методы структур

```Go
package main

import (
	"fmt"
	"net/http"
	"text/template"
)

  

type User struct {
	ID int
	Name string
	Active bool
}

  

func (u *User) PrintActive() string {
	if !u.Active {
		return "asleep"
	}
	return "method says user " + u.Name + "Active"
}

  

func main() {
		tmpl := template.Must(template.ParseFiles("templates_testing/methogWithTemplate/method.html"))	
	users := []User{	
		User{1, "Nikitos", true},		
		User{2, "IvAn", true},		
		User{3, "fish", false},	
	}
	
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {	
		tmpl.Execute(w,	
			struct {	
				Users []User	
			}{	
				users,	
			})	
	})	  
	
	fmt.Println("starting server ar :8081")	
	http.ListenAndServe(":8081", nil)
}
```

```html
<html>
	<body>	
		<h1>Users</h1>		
		{{range .Users}}		
		<b>{{.Name}}</b>		
		{{.PrintActive}}		
		<br />		
		{{end}}	
	</body>
</html>
```



### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]
- [[00 network]]


### Links
- https://stepik.org/lesson/1131168/step/1?unit=1142765

