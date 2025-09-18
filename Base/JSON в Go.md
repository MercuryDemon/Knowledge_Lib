17-09-2025 20:01
Tags: #Go #lesson 
# Что такое json

Это стандартизированный формат кодировки и передачи данных в современной WEB-разработке.
JSON - JavaScript Object Notation

# Обработка JSON в Go

Существует стандартный пакет для работы с JSON форматом.
```Go
import "encoding/json"
```
В го упаковка и распакоука данных называется маршалингом и анмаршалингом.

пример работы с JSON
```Go 
package main

import (
	"fmt"
	"encoding/json"
)

type User struct {
	ID          int
	UserName    string
	phone       string
	}
	
var jsonStr = '{"id": 42, "username": "ZhNikita", "phone": "123"}'

func main() {
	data := []byte(jsonStr)
	
	u := &User{}
	json.Unmarshal(data, u)
	fmt.Printf("struct:\n\t%#v\n\n", u)
	
	u.phone = "99912345"
	result, err := json.Marshal(u)
	if err != nil {
		panic(err)
	}
	fmt.Printf("json string:\n\t%s\n", string(result))
}
```

При запуске этой программы , мы не причитаем и не запишем в поле "phone" ничего, потому что это поле структуры не доступно из других пакетов. Доступ к этому полю могут иметь толко те методы/ф-ции которые обьявлены в том же пакете где и структура "User".

# Теги структур в JSON

В формате JSON можно записывать некую метаинформацию напротив каждого поля
она оформляется в бэктиках 

Пример
```Go
type User struct {
	ID          int `json:"user_id,string"` //`к чему относится:"имя куда      распаковывать или откуда запаковывать,тип данных"`
	UserName    string 
	Addres      string `json:",omitempty"` //
	Company     string `json:"-"` 
	}
```
где для ID 
- `json:` - то к чему относится метаинформация
- user_id - имя переменной, куда распаковывать и откуда запаковывать
- string - тип данных, в который он должен записать
- 
для Addres
- ` ,` - пустота перед запятой, значит что не хотим переопределять имя
- omitempty - если поле пустое, его н нужно писать в результат при запаковке

для Company
- "-" - значит полевообще не нужно сериализовывать или десериализовывать.

пример нормальной работы

```Go

package main

import (
	"fmt"
	"encoding/json"
)

type User struct {
	ID          int `json:"user_id,string"` //`к чему относится:"имя куда      распаковывать или откуда запаковывать,тип данных"`
	UserName    string 
	Addres      string `json:",omitempty"` //
	Company     string `json:"-"` 
	}
	
func main() {
	u := &User{
		ID:       42,
		Username: "Nikitosik",
		Address:  "test", // если поле будет пустым, ничего не выведется в result
		Company:  "Laser Center",
	}
	
	result, _ := json.Marshal(u)
	fmt.Printf("json string: %s\n", string(result))
}
```

# Как работать с JSON, в котором не определена структура

Напомощь приходит пустой interface{}

пример
```Go
package main

import (
	"fmt"
	"encoding/json"
)

var jsonStr = `[
	{"id": 17, "username": "IvAn", "phone":0},
	{"id": "17", "addres": "chpok str.", "company": "The Blue Oyster"}
]`

func main() {
	data := []byte(jsonStr)
	
	var user1 interface{}
	json.Unmarshal(data, &user1)
	fmt.Printf("unpacked in empty interface:\n%#v\n\n", user1)
	
	user2 := map[string]interface{}{
		"id": 42,
		"username": "Nikita",
	}
	var user2i interfce{} = user2
	result, _ := json.Marshal(user2i)
	fmt.Printf("json string from map:\n%s\n", string(result))
}
```
При использовании интерфейса для парсинга JSON, его поля преобразуются в мапу.
Придется преобразовывать значения из типа interface{} в нужный тип данных, для работы с ними.

Чтобы упаковать какие-то поля в JSON можно в обратную сторону применить `map[string]interfase{}`



### Zero-Links
- [[00 network]]
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://stepik.org/lesson/1131163/step/1?unit=1142760

