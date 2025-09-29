22-09-2025 21:04
Tags: #Go #lesson 

# Бенчмарки - тест производительности

бенчмарка - это такакя контрольная задача, необходимая для определения сравнительных характеристик производительности компьютерной системы, или отдельной программы.

В Go есть готовый пакет "Benchmark". Он относится к пакету "test"

пример вызова функции - бенчмарки
```Go
func BenchmarcCodegen(b *testing.b) {
	for i := 0; i <b.N; i++ {
		u := %User{}
		u.Unpack(data)
	}
}
```
тут мы проверяем функцию распаковки данных из структуры реализованную через [[Codegen(Кодогенерация)]]

запускается из консоли
```console
go test -bench .fileName.go
```
точка "." перед именем файла необходима!
```
go test -bench . -benchmem fileName.go
```
дополнительный ключ '- benchmem' если мы хотим посмотреть сколько памяти использует программа

# Еще инструменты/[[pprof]]

При выполнении бенчмарки можно снять профиль процессора/памяти

для процессора ключ 
```
-cpuprofile=cpu.out
```
после равно пишем куда складировать данные

для памяти
```
-memprofile=mem.out -memprofilerate=1
```
рейт = 1 - регистрировать каждую аллокацию памяти

после запуска, снимается профиль и его можно проанализировать
анализируем через инструмент "pprof"
```
go tool pprof main.test.exe cpu.out
```

запускается pprof

команды:
- top - посмотреть самые дорогие(по времени выполнения/памяти) операции. 
- alloc_space затем top - посмореть операции алокаций памяти.
- list funcName - посмотреть по строчкам функции где что и сколько заняло рессурсов.
- web - строит графическую картинку где все функции описаны описаны построчно как дерево алгоритма с затратами рессурсов на каждом шаге.

# sync.Pool

sync.Pool - инструмент для оптимизации работы с памятью. 
Позволяет выделить пул памяти, с уже преаллоцированными данными, и переиспользовать его. Из этого пула данные будут браться и возвращаться обратно, не придется аллоцировать новую память для них.

пример:
```Go
const itemNum = 100

type PublicPage struct {
	ID          int
	Name        string
	Url         string
	OwnerID     int
	imageUrl    string
	tags        []string
	Description string
	Rules       string
}

var CoolGokPublic = PublicPage{
	ID:          42,
	Name:        "CoolGokPage",
	Url:         "https://example.com",
	OwnerID:     100500,
	imageUrl:    "https://example.com/image_9788.png",
	tags:        []string{"Gok", "PipipipidorGay", "Negga"},
	Description: "Page about cock sucking",
	Rules:       "N-word must be said",
}

var Pages = []PublicPage{
	CoolGokPublic,
	CoolGokPublic,
	CoolGokPublic,
}

func BenchmarkAllocNew(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			data := bytes.NewBuffer(make([]byte, 0, 64)) //читаем в буфер
			                       // при этом алоцируем память под буфер
			_, json.NewEncoder(data).Encode(Pages)// записываем в json
		}
	})
}


var dataPool = sync.Pool{
	New: func() interface{}{
		return bytes.NewBuffer(make([]byte, 0, 64)) //инициализирует объект
	},	                                            //в пуле. Алоцируем память
}


func BenchmarkAllocPool(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			data := dataPool.Get().(*bytes.Buffer) //достаём из пула преобразуем
			                                       //в нужный тип - буфер байт
			_, json.NewEncoder(data).Encode(Pages) // записываем в json
			data.Reset() //сбрасываем данные
			dataPool.Put(data) // возвращаем в пул обратно
		}
	})
}
```


# Подсчет покрытия тестами

```console
go test -v -cover
```
посмотреть сколько кода покрыто тестами в процентах

```console
go test -v -coverprofile=cover.out


go tool cover -html=cover.out -o cover.html
```
посмотреть в html какие куски кода покрыты тестами а какие нет


# Поточная обработка данных(парсинг)

- если существует задача парсить не весь объем, а какуюто часть, имеет смысл применить
- быстрее выполняется, меньше аллокаций памяти, так как мы заглядываем внутрь файла преже чем декодировать из него гужную информацию, а не парсим все в лоб, а потом идем циклом for по всем данным, чтобы найти нужный элемент.



### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]
- 



### Links
- https://stepik.org/lesson/1131165/step/1?unit=1142762
- 
Пакет для маршалинга/анмаршалинга JSON реализованный кодогенерацией от mail.ru
https://github.com/mailru/easyjson - используй!
