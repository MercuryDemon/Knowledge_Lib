27-05-2025 22:30
Tags: #z #lesson #Go 
map - хэш-таблица в Go. [[Хэш-таблица (HashMap)]]
- хранит пары ключ-значение
- запись и чтение элементов имеет временную константную сложность O(1)

```go
// создание пустой мапы
var m map[int]string

// сокращенное создание пустой мапы
m := map[int]string{}

// рекомендуемое создание с обозначением размера
m := make(map[int]string, 10)

// создание мапы с элементами
m := map[int]string{1: "hello", 2: "world"}

// добавление элемента
m[3] = "!" // map[1:hello, 2:world, 3:!]

// чтение элемента
word := m[1] // "hello"
```

Если ключа нет в мапе позваращается ноль заданного типа.

```go
elements := map[int64]bool{1: true, 2: false}

element, elementExists := elements[1] // true, true // значение, наличие элемента(тоже значение по сути)
element, elementExists := elements[2] // false, true
element, elementExists := elements[225] // false, false
```


проверка наличия ключа может быть 

Удаление элементов через встроенную функцию 

```go 
delete(m map[Type]Type1, key Type
```
       ^     ^      ^       ^
имя мапы   тип      тип     ключ который удаляем
          ключа   значений        и его тип
          в мапе   в мапе

```go
engToRus := map[string]string{"hello":"привет", "world":"мир"}

delete(engToRus, "world")
// функция принимает навход имя мапы и "ключ" в кавычках, который удаляем
fmt.Println(engToRus) // map[hello:привет]
```

Мапы всегда передаются по ссылке. Если мы вносим изменения в мапу внутри области видимости функции, мапа меняется в main().





### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://ru.hexlet.io/courses/go-basics/lessons/map/theory_unit

