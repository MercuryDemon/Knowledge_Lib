17-06-2025 18:23
Tags: #z #lesson
# Создание HTTP-сервера и клиента в Go

# HTTP - что такое
![[what is HTTP]]

# package net/http

## HTTP-клиент

Пакет net/http предастовляет структуру http.Client для создания и настройки HTTP-клиента.

```Go
type Client struct {
		Transport RoundTripper
		CheckRedirect func(req *Request, via []*Request)
		Jar CookieJar
		Timeout time.Duration
}
```

- Transport - определяет механизм выполнения запросов (по дефолту `http.DefaultTransport`) - обрабатывает все части запроса. Допускается создавать собственные реализации интерфейса `RoundTripper` для более тонкой настройки поведения клиента.

- CheckRedirect - управляет поведением клиента при перенаправлении(статусы 301 или 302)

- Jar - Управление cookie (по дефолту `http.CookieJar`) - позволяет сохранять данные между запросами, важно при рабоите с сессиями

- Timeout - определяет максимальное время ожидания ответа на запрос( после которого отменяет запрос с ошибкой)


## Запросы GET и POST

Запрос GET через  URL это изи
```go
func main() {
	response, err := http.Get("https://httpbin.org/get?proglib=cool")
	if err != nil {
		log.Fatal(err)
	}
	defer response.Body.Close()

	body, err := io.ReadAll(response.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(response.Status, string(body))
}
```


считываем ответ `io.Reader` и выводим на экран:
```console
200 OK {
  "args": {
    "proglib": "cool"
  },
  "headers": {
    "Accept-Encoding": "gzip",
    "Host": "httpbin.org",
    "User-Agent": "Go-http-client/2.0",
    "X-Amzn-Trace-Id": "Root=1-673b84cf-0d7e53aa14f60ac80478a841"
  },
  "origin": "80.85.241.77",
  "url": "https://httpbin.org/get?proglib=cool"
}

```

Ответ от сервера задаётся структурой `http.Response`

Тело ответа `responce.Body` представляет собой структуру io.ReadCloser. С ней можно взаимодействовать припомощи пакета `io.`


## Отправка запросов

Поле Transport в структуре `http.client` позволяет настроить механизм, используемый для выполнения HTTP-запросов.

Является интерфейсом `RoundTripper`, который содержит метод `RoundTrip(*http.Recuest)(*http.Response, error)` - отвечаюзий за выполнение запроса, установку соединений, обработку прокси, настройка времени ожидания и т.д.

Соответсвенно для изменения http-запросов надо реализовать интерфейс RoundTripper

пример:
```Go
// CustomTransport - наша собственная реализация RoundTripper
type CustomTransport struct {
	Transport http.RoundTripper
}

func (c *CustomTransport) RoundTrip(req *http.Request) (*http.Response, error) {
	// Логирование каждого запроса
	fmt.Println("Отправляется запрос на URL:", req.URL)

	// Дополнительная логика (например, добавление заголовков)
	req.Header.Add("X-Custom-Header", "CustomValue")

	return c.Transport.RoundTrip(req)
}

func main() {
	client := &http.Client{
		Transport: &CustomTransport{Transport: http.DefaultTransport}, // Используем кастомный транспорт
	}

	// Отправляем GET запрос
	resp, err := client.Get("https://httpbin.org/get")
	if err != nil {
		fmt.Println("Ошибка:", err)
		return
	}
	defer resp.Body.Close()

	fmt.Println("Статус ответа:", resp.Status)
}
```


### Прокси-сервер

Для настройки прокси-сервера используем структуру `http.ProxyURL`

```Go
func main() {
	// Вместо адреса ниже стоит указать адрес существующего прокси-сервера
	proxyURL, err := url.Parse("http://proxyserver:8080")
	if err != nil {
		fmt.Println("Ошибка при парсинге прокси:", err)
		return
	}

	// Структура клиента
	client := &http.Client{
		Transport: &http.Transport{
			Proxy: http.ProxyURL(proxyURL), // Настройка прокси
		},
	}

	// Отправляем запрос через прокси
	resp, err := client.Get("https://httpbin.org/get")
	if err != nil {
		fmt.Println("Ошибка:", err)
		return
	}
	defer resp.Body.Close()

	fmt.Println("Статус ответа:", resp.Status)
}
```

## Перенаправление запросов

Поле `CheckRedirect` в структуре `http.Client` определяет поведение клиента при получении ответа на запрос с перенаправлением.

Предаставляет собой функцию которая принимает в качестве аргументов: 
1. Запрос для выполнения
2. Слайс ответов, полученных в процессе перенаправлений.

Самый простой сценарий использования поля `CheckRedirect` — это ограничение числа перенаправлений для избежания зацикливания или бесконечного выполнения запросов:

```GO
client := http.Client{
    CheckRedirect: func(req *http.Request, via []*http.Request) error {
        if len(via) >= 5 {
            return http.ErrUseLastResponse // Вернуть последний запрос, не интерпретируя его как ошибку
        }
        return nil // Выполнить перенаправление
    },
}
```

Через 5 перенаправлений клиент вернет последний полученный ответ.


## Таймауты

Сколлько ждать ответа от сервера.

пример:
```go
func main() {
	// Настройка клиента с таймаутом
	client := &http.Client{
		Timeout: 5 * time.Second, // Таймаут на 5 секунд
	}

	// Отправляем GET-запрос
	resp, err := client.Get("https://httpbin.org/get")
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	// Читаем тело ответа
	fmt.Println("Статус ответа:", resp.Status)
}
```

Если значение поля `Timeout` превысит заданное значение, то запрос будет прерван, и вернется ошибка `context.DeadlineExceeded`.


# HTTP-Сервер

## Схема работы сервера

1. Клиент посылает запросы на сервер

2. Сервер распределяет (мультиплексирует) запросы на обработчики

3. Запрос проходит промежуточный слой `middleware`

4. Запрос попадает к конкретному обработчику, который отсылает ответ клиенту

## Основные компанениы обработки запросов

1. Обработчики(handlers) - отвечаю за логикку приложения и формирование ответа клиенту.Могут быть Функциями или структурами реализующими интерфейс `http.Handler`

2. Серверные мультиплексоры (servemaxes) - копоненты сопоставляющие URL-пути с соответстыующими обработчиками.

Пример простого HTTP-сервера с одним мультиплексором и одним обработчиком:

```go
func main() {
	// Создаем и возвращаем новый экземпляр структуры ServeMux
	mux := http.NewServeMux()

	// Создаем обработчик, перенаправляющий запросы по указанному URL с HTTP-кодом 308
	redirectHandler := http.RedirectHandler("http://google.com", http.StatusPermanentRedirect)

	// Регистрируем ранее созданный servemux по указанному URL
	mux.Handle("/redirect", redirectHandler)

	log.Print("Сервер запущен...")

	// Запуск сервера на порту 8080
	http.ListenAndServe(":8080", mux)
}
```

Чтобы протестировать работу сервера, достаточно запустить код и отправить запрос по адресу `http://localhost:8080/redirect` Сделать это можно с помощью популярной программы Postman или утилиты curl с дополнительными флагами:
```bush
curl -i http://localhost:8080/redirect
```

Получим подробный вывод, содержащий HTTP-заголовки, информацию о подключении и др. диагностическую инфу.

вывод:
```
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /redirect HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/8.1.2
> Accept: */*
>
< HTTP/1.1 308 Permanent Redirect
< Content-Type: text/html; charset=utf-8
< Location: http://google.com
< Date: Wed, 20 Nov 2024 17:46:38 GMT
< Content-Length: 53
<
<a href="http://google.com">Permanent Redirect</a>.

* Connection #0 to host localhost left intact
```

## Пользловательские обработчики

Любой пользовательский обработчик должен реализовывать интерфейс `http.Handler` с единственным методом `ServeHTTP`
```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

Пример пользовательского обработчика:

```go
// Структура, содержащая данные о пользователе
type userHandler struct {
	name  string
	age   int
}

// Реализация ServeHTTP для userHandler
func (uh userHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// Формируем строку с данными о пользователе
	message := fmt.Sprintf("User: %s, Age: %d", uh.name, uh.age)

	// Один из способов записи данных в соединение
	w.Write([]byte(message))
}

func main() {
	mux := http.NewServeMux()

	// Инициализация userHandler с данными о пользователе
	uh := userHandler{name: "Гоша", age: 30}

	// Регистрация обработчика для маршрута "/user"
	mux.Handle("/user", uh)

	log.Print("Сервер запущен...")
	http.ListenAndServe(":8080", mux)
}
```

Обработка запроса на стороне сервера происходит следующим образом: сервер получает HTTP-запрос и передает его мультиплексору, который указан вторым аргументом функции `http.ListenAndServe`. Следующим этапом мультиплексор ищет подходящий обработчик по указанному маршруту, в данном случае – "/user". Далее мультиплексор вызывает у подобранного обработчика метод `ServeHTTP`, который записывает данные в HTTP-соединение и отсылает ответ клиенту.


### Функция как обработчик

В качестве обработчиков могут выступать обычные функции с сигнатурой `func(http.ResponseWriter, *http.Request)`, обернутые в специальный тип `http.HandlerFunc`, который реализует интерфейс Handler:

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

Для демонстрации создадим обработчик `infoHandler`, конвертируем его в тип `HandlerFunc` и передадим в мультиплексор:

```go
func infoHandler(w http.ResponseWriter, r *http.Request) {
	log.Printf("Информация о запросе: %s %s от %s", r.Method, r.URL.Path, r.RemoteAddr)
	fmt.Fprintf(w, "Запрос обработан\\\\n")
}

func main() {
	mux := http.NewServeMux()

	ih := http.HandlerFunc(infoHandler)

	mux.Handle("/info", ih)

	log.Print("Сервер запущен...")
	http.ListenAndServe(":8080", mux)
}
```

Так как конвертация функции в тип http.HandlerFunc и добавление её в мультиплексор является довольно частой задачей, для нее придумали отдельный метод mux.HandleFunc. Поэтому код:

```go
ih := http.HandlerFunc(infoHandler)
mux.Handle("/info", ih)
```

можно заменить на 

```go
mux.HandleFunc("/info", infoHandler)
```

# Middleware 

![[HTTP middleware]]


# 


### Zero-Links
- [[00 Golang]]
-  [[00 Обучение]]


### Links
- https://proglib.io/p/samouchitel-po-go-dlya-nachinayushchih-chast-18-protokol-http-sozdanie-http-servera-i-klienta-paket-net-http-2024-12-13
- https://pkg.go.dev/net/http


