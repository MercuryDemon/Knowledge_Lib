01-06-2025 19:09
Tags: #z #lesson 
# Определение.

Контекст это объект который позволяет контролить процессы извне:

- Возможность отмены потецниально долгой операции.
- передача информации по флоу программы.

## Когда использовать?

- метод или функция ходит кудато по сети
- горутина может долго висеть

## Советы

- всегда передавать контекст первым аргументом
- передавать контекст только в функции и методы( он одноразовый, не нужно хратить в памяти его переменную)
- использовать `context.WithValue` только в крайних случаях
- `context.Background` должен использоваться только как родитель(это заглушка в нем нет средств контроля)
- не забывать о функции отмены контекста
- передавать только контекст, без функции отмены(контроль за зовершение контекста должен оставаться на вызывающей стороне)
## Отмена(Галя у нас отмена)

способы отмены:
- явный сигнал `context.WithCancel()` 
- по истечении времени `context.WithTimeout()`
- или по дедлайну (фикс дата/время) `context.With Deadline`

Примеры:
- В кафе долго трубку не берут. по истечении какогото времени звонок сбрасывается.
- ищеш порево, сервер порнхаба трещит по швам от количества запросов, по истечении какогото времени попытки получить ответ от сервера прекращаются - "таймаут, превышено время ожидания ответа от сервера"

пример приоритата Таймаута:
```go
func doWork(ctx context.Context) {

	newCtx, cancel := context.WithTimeout(ctx, 30*time.Second)// изменили таймаут, но он всё равно истечет через 10 секунд.
	defer cancel()
	
	log.Println("starting working...")
	
	  
	
	for {
		select {
			case <-newCtx.Done():
				log.Printf("ctx done: %v", ctx.Err())
				return
			default:
				log.Println("working...")
				time.Sleep(1 * time.Second)
		}
	}
}

  

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second) // задаем таймаут 10 секунд 
	defer cancel()

	doWork(ctx) //что-то делаем, у работы есть таймаут

}
```
- `context.WithTimeout` внутри себя вызывает `context.WithDeadline(настоящее время + время таймаута`
- если задавать новые таймауты, то выполнится тот который был с меньшим временем.

Пример применения явного сигнала отмены:
```go 
func requestRide(ctx context.Context, serviceName string, output chan string) {
	time.Sleep(3 * time.Second)

	for {
		select {
		case <-ctx.Done(): // если пришел сигнал отмены выходит из горутины
			log.Printf("stopped the search in %q (%v)", serviceName, ctx.Err())
			return
		default:
			if rand.Float64() > 0.75 { // выбирает сервис
				output <- serviceName
				return
			}
			
			continue
		}
	}
}

func main() {

	var (
		resultCh = make(chan string)
		ctx, cancel = context.WithCancel(context.Background())
		services = []string{"Dupper", "Gufier", "Nigger", "UltraSLUT"}
		wg sync.WaitGroup
		winner string
	)
	defer cancel()
	
	for i := range services { // плодит горутины, какая первая передаст значения в канал тот сервис и будет выбран
		svc := services[i]
		wg.Add(1)
		go func() {
			requestRide(ctx, svc, resultCh)
			wg.Done()
		}()
	}
	
	go func() {
		defer close(resultCh) // планиркем закрыть канал после получения из канала инфы
		winner = <-resultCh // считываение из канала операция которая блочит эту горутину
		cancel() // передаём сигнал отмены
	}()
	
	wg.Wait() // блокируем мейн
	log.Println(winner) // выводим победителя
}
```

## Передача данных через контекст

`context.WithValue(context.Background(), "key", "value")` принимает родительский контекст, ключ и значение напримеп - "name", "Nikita".

- значение может быть любого типа. (любого типа который реализует пустой интерфейс)
- возвращает только контекст без функции отмены

```go
func main() {
	ctx := context.WithValue(context.Backgroung, "skinColor" "nigga")
	
	log.Printf("name = %v", ctx.Value("name")) // ключ есть возвращает значение
	log.Printf("age = %v", ctx.Value("age")) // ключа нет возвращает nil
}

```

тип возвращаемого значения interface{ }, придется привести значение к нужному типу.

### Когда передавать данные через контекст?

- никогда(серьезно лучше ненадо) (пораждает ненадежный, неявный контракт между компанентами приложения)
- HTTP Middleware




### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://www.youtube.com/watch?v=Fjkckov4F38

