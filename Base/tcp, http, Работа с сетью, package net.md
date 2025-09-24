24-09-2025 18:38
Tags: #Go #lesson
# Слушаем tcp сокет

пример:
```Go
package main


func handleConnection(conn net.Conn) {
	name := conn.RemoteAddr().String()
	
	fmt.Printf("%+v connected\n", name)
	conn.Write([]byte("wassaaaaa nigga, "+name+"\n\r"))
	
	defer conn.Close()
	
	scaner := bufio.NewScaner(conn)
	for scaner.Scan() {
		text := scaner.Text()
		if text == "Exit" {
			conn.Write([]byte("see ya\n\r"))
			fmt.Println(name, "disconnected")
			break
		} else if text != ""{
			fmt.Println(name, "enters", text)
			conn.Write([]byte("Nigga said - "+ text + "\n\r"))
		}
	}
}




func main() {
	listner, err := net.Listen("tcp", ":8080") //создаем объект слушающий сокет
	if err != nil {
		panic(err)
	}
	for {
		conn, err := listner.Accept()// принимаем  вход подключ.
		if err != nil {
			panic(err)
		}
		go handleConnection(conn)//обрабатываем соединение
	}
}
```

# http сервер

у сервера обязательно должны быть хэндлеры(обработчик)

```Go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponceWriter, r *http.Request) {
	fmt.Fprintln(w, "hello world")
	w.Write([]byte("!!!"))
}

func main() {
	
	http.HandleFunc("/page", 
		func(w http.ResponseWriter, r *http.Request) {//обрабатывает 1 страницу
			fmt.Fprintln(w, "Single page", r.URL.String())
		})
	
	http.HandleFunc("/page/", 
		func(w http.ResponseWriter, r *http.Request) {//обрабатывает страницу "page" и все что после нее, например "page/gay"
			fmt.Fprintln(w, "Multiple page", r.URL.String())
		})

	http.HandleFunc("/", handler) //обрабатывает всё, что не попало в URL
	
	fmt.Println("starting server ar :8080")
	http.ListenAndServe(":8080", nil)
}
```
минус такой реализации в том, что мы не можем внедрять зависимости и отправлять какието данные.

чтобы чтото прокидывать на сервер, и пользоваться какимито его функциями мы можем вешать обработчик на структуру

пример
```Go
type Handler struct {
	Name string
}

func(h *Handler) ServeHTTP(w http.ResponseWtiter, r http.Request)
{
	fmt.Fprintln(w,"Name", h.Name, "URL:", r.URL.String())
}

func main() {
	testHandler := &Handler{Name: "test"}
	http.Handle("/test/", testHandler)
	
	rootHandler := &Handler{Name: "root"}
	http.Handle("/", rootHandler)
	
	fmt.Println("starting server at :8080")
	http.ListenAndServe(":8080", nil)
}
```
удобно - можно делать свитч по полям структуры и распределять их по хэндлерам

# Работа с входящими от пользователя параметрами.

## get парам

пример получения get параметров
```Go
func handler(w http.ResponseWtiter, r http.Request) {
	myParam := r.URL.Query().Get("param")
	if myParam != "" {
		fmt.Fprintln(w, "`myParam` is", myParam)// 1 способ
	}
	
	key := r.FormValue("key")
	if key != "" {
		fmt.Fprintln(w, "`key` is ", key )// 2 способ
	}
}

func main() {
	http.HandleFunc("/", handler)
	
	fmt.Println("starting server at :8080")
	http.ListenAndServe(":8080", nil)
}

```

## Post парам

```Go
package main

import (
	"fmt"
	"net/http"
)

var loginFormTmpl = []byte(` //форма авторизации
<html>
	<body>
	<form action="/" method="post">
		Login: <input type="text" name="login">
		Password: <input type="password" name="password">
		<input type="submit" value="Login">
	</form>
	</body>
</html>
`)

func mainPage(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost { //если метод с которым пришли не пост
		w.Write(loginFormTmpl)       //выдать форму логина
		return
	}

	// r.ParseForm()//при необходимости парсим форму,
	// inputLogin := r.Form["login"][0]  // чтобы достать наши данные

	inputLogin := r.FormValue("login")
	fmt.Fprintln(w, "you enter: ", inputLogin)
}

func main() {
	http.HandleFunc("/", mainPage)

	fmt.Println("starting server at :8080")
	http.ListenAndServe(":8080", nil)
}
```


## coockies

куки в рамках http сесии позволяет запоминать дейсвия и какието данные пользователя( например чтобы не логиниться каждый раз при смене страницы)
несекьюрное значение.

запоминает идентификатор сесии, чтобы уже на сервере работать с секьюрными данными.


## headers

хедеры могут быть как выставлены так и получены с сервера



### Zero-Links
- [[00 Обучение]]
- [[00 Golang]]
- [[00 network]]


### Links
- https://stepik.org/lesson/1131166/step/1?unit=1142763


