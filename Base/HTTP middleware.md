18-06-2025 21:56
Tags: #z #lesson
#  Middleware

В контексте веб-разработки **middleware** выполняет роль промежуточного звена между клиентом и сервером, обрабатывая HTTP-запросы до того, как они будут переданы в конечные обработчики. Middleware широко используется в Go-фреймворках Gin, Echo и Chi.

Middleware может выполнять разные задачи, например:

1. Авторизация и аутентификация
2. Логирование
3. Валидация входных данных
4. Ограничение времени медленных запросов
5. Сокрытие конфиденциальных данных

Middleware работает по принципу цепочки: когда HTTP-запрос поступает в сервер, он сначала проходит через серию middleware-компонентов, и только потом передается в основной обработчик. Каждый middleware может либо завершить обработку запроса, отправив ответ клиенту, либо передать управление следующему middleware в цепочке.

В языке Go создание middleware происходит в два этапа:

1. Написать функцию middleware, чтобы она удовлетворяла интерфейсу http.Handler
2. Выстроить цепочку из middleware и обработчиков

Шаблон middleware:

```go
func sampleMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// ...
		next.ServeHTTP(w, r)
	})
}
```

Функция sampleMiddleware принимает в качестве аргумента обработчик next типа `http.Handler` и возвращает новый обработчик того же типа. Внутри функции создаётся анонимный обработчик с использованием конструктора `http.HandlerFunc`, который преобразует функцию в объект, соответствующий интерфейсу `http.Handler`. Этот анонимный обработчик выполняет основную логику, после чего вызывается метод ServeHTTP у next, который передаёт запрос и ответ следующему обработчику в цепочке.

Пример создания цепочки middleware и регистрация их в мультиплексоре:
```go
mux := http.NewServeMux()
mux.Handle("/", middlewareOne(middlewareTwo(mainHandler)))
```

Пример HTTP-сервера предающего запрос по цепочке из двух middleware:

```go
package main

import (
	"log"
	"net/http"
	"time"
)

// middlewareOne измеряет время выполнения запроса и логирует информацию о начале и конце обработки
func middlewareOne(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		log.Printf("middlewareOne: начало обработки запроса в %s", start.Format("15:04:05"))
v
		// Передаём управление следующему обработчику
		next.ServeHTTP(w, r)

		// Рассчитываем и выводим время выполнения в наносекундах
		duration := time.Since(start).Nanoseconds()
		log.Printf("middlewareOne: запрос обработан за %d наносекунд", duration)
	})
}

// middlewareTwo логирует информацию о маршруте и подтверждает успешную обработку
func middlewareTwo(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Printf("middlewareTwo: обработка маршрута %s", r.URL.Path)

		// Передаём управление следующему обработчику
		next.ServeHTTP(w, r)

		log.Print("middlewareTwo: запрос успешно обработан")
	})
}

// mainHandler — основной обработчик
func mainHandler(w http.ResponseWriter, r *http.Request) {
	log.Print("mainHandler: обработка запроса")
	w.Write([]byte("Запрос выполнен успешно\\\\n"))
}

func main() {
	mux := http.NewServeMux()

	mh := http.HandlerFunc(mainHandler)

	// Формируем цепочку из middleware с основным запросом в конце
	mux.Handle("/", middlewareOne(middlewareTwo(mh)))

	log.Print("Сервер запущен...")
	err := http.ListenAndServe(":8080", mux)
	log.Fatal(err)
}
```

После запуска приведенного кода и отправки запроса 
`curl -i http://localhost:8080` получим следующий вывод:

```curl
HTTP/1.1 200 OK
Date: Wed, 20 Nov 2024 19:13:25 GMT
Content-Length: 45
Content-Type: text/plain; charset=utf-8

Запрос выполнен успешно
```

вывод логов в терминале, где был запущен код как передавались запросы между хэндлерами

```terminal
2024/11/20 22:13:23 Сервер запущен...
2024/11/20 22:13:25 middlewareOne: начало обработки запроса в 22:13:25
2024/11/20 22:13:25 middlewareTwo: обработка маршрута /
2024/11/20 22:13:25 mainHandler: обработка запроса
2024/11/20 22:13:25 middlewareTwo: запрос успешно обработан
2024/11/20 22:13:25 middlewareOne: запрос обработан за 85632 наносекунд
```






### Zero-Links
- [[00 network]]
- [[00 Обучение]]


### Links
- https://proglib.io/p/samouchitel-po-go-dlya-nachinayushchih-chast-18-protokol-http-sozdanie-http-servera-i-klienta-paket-net-http-2024-12-13
- 

