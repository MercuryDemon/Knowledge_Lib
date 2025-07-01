27-05-2025 22:32
Tags: #z #lesson #Go 

Разделение программы на потоки, для того чтобы разные ыункции выполнялись одновременно.

## Горутины(goroutines)
В Го это это виртуальные потоки. Как потоки операционной системы, только пизже, потому что потребляет меньше ресурсов.

горутина вызывается ключевым словом "go" перед вызовом функции.

- Функция main() не ждёт окончания выполнения горутин(может завершиться до того как они закончат работать)
- main() представляет собой виртуальный поток, но он выполняется последовательно, равно как и другие фенкции в Го
- Горутины имеют независимый стек, следовательно нет необходимости в обмене данными между ними

### Отличие потоков ОС от горутин

![[Отличие потоков ОС от горутин.png]]

### Анонимные горутины(инкогнито срать незаметность)


```go
package main

import "fmt"

func main() {
	fmt.Println("main() started")
	c := make(chan string)

	// launch anonymous goroutine
	go func(c chan string) {
		fmt.Println("Hello " + <-c + "!")
	}(c)

	c <- "John"
	fmt.Println("main() stopped")
}
```


## Каналы

Это такая труба по которой можно передавать и получать данные от ГОРУТИН.
Канал обеспечивает взаимодействие и синхронизацию горутин.

канал создаётся с помощью функции "make()" :
```go
ch := make(chan int)
```
chan - ключевое слово для создания канала, далее тип данных который будет передаваться по каналу.

В Go каналы являются указателями. Поэтому при передаче их в качестве аргумента в функцию или метод их не нужно разименовывать.

отправляем данные в канал)))
```go
ch <- 8
```

получаем данные с канала 
```go
value := <- ch
```
данные идут в направлении стрелки

Эти операции являются блокируемыми. Например пока значение не вернется в `main()` из канала, программа не завершится.

- Это значит не нужно писать блокеры для синхронизации горутин.
- Операции канала говорят планировщику о планировании другой горутины.

Можно созданные каналы передавать в качесве аргументов в функцию которая вызывается горутиной, для того чтобы вернуть в него результат действия функции

```go
func evenSum(from, to int, ch chan int) { // не забываем правильно 
  result := 0                             // написать, что принимаем  
  for i:=from; i<=to; i++ {               // каналы, а не переменные
    if i%2 == 0 {
      result += i
    }    
  }
  ch <- result
}
func squareSum(from, to int, ch chan int) {
  result := 0
  for i:=from; i<=to; i++ {
    if i%2 == 0 {
      result += i*i
    }    
  }
  ch <- result
} 
func main() {
	evenCh := make(chan int) // создаём каналы
	sqCh := make(chan int)

	go evenSum(0, 100, evenCh)  // передаем их вместе с аргументами
	go squareSum(0, 100, sqCh)

	fmt.Println(<-evenCh + <- sqCh)
}
```

### deadlock
Возникает при попытке считать данные из канала в котором будут отсутствовать данные, или если мы отправляем данные в канал который никто (ни одна горутина) не планирует считывать.

Например:
```Go
package main

import "fmt"

func main() {
	fmt.Println("main() started")
	c := make(chan string)
	c <- "John"
	
	fmt.Println("main() stopped")}
```

Может возникнуть при чтении данных из канала бесконечным циклом `for` внутри горутины.  В таком случае когда данные в канале заканчиваются, а цикл продолжает пытаться считывать даные. 


для этого мы закрываем канал.
![[close chanel example.png]]
Можно закрывать канал при помощи:
```
 close(c) // close channel
 ```

Канал всегда возвращает 2 значения `value` - то что передается по каналу и `state` - булевое состояние true - пердает данные, false - не будет передавать данные.

Вме5сто бесконечного цыкла, для проверки закрытия канала можно использовать `range` :

```go
for msg := range massage {        
	fmt.Println(msg)
    }
```

### Размер буфера канала

Изначально размер буфера нравен 0 - небуферизированный канал.
Горутина не блокируется каналом ёпока буфер не заполнится.

Буферизированный канал инициируется дополнительным аргументом при его создании:
```go
chanelTest := make(chan int, 2) // примет два значение типа integer
```
вторым значением указывается размер буфера.

#### Особенности.
- не падает в дедлок если мы отпарвляем и считываем из канала в одной и тойже горутине. Потому что канал буферизирован и не блокирует выполнение при записи.
- падает в дедлок если отправить в канал элементов больше чем размер буфера.
- при считывании из канала считываются все значения пока буфер не опустеет.
- Из закрытых буферизированных каналов можно считать элементы так как они остаются в буфере.

#### Длина и ёмкость канала

- Длина(len) канала - количество несчитаных элементов отправленных в канал. Вызывается: `len(chanel)`
- Ёмкость(cap) канала - размер буфера. Вызывается `cap(chanel)`

### Однонаправленные каналы.

создаются при помощи `make`, но со стрелочным синтаксисом.

```go
roc := make(<-chan int) // канал только для чтения
soc := make(chan<- int) // канал только для записи
```


### Канал с типом данных канала

Да мы можем передавать саналы в качестве элементов в другой канал.

```go
package main

import "fmt"

// gets a channel and prints the greeting by reading from channel
func greet(c chan string) {
	fmt.Println("Hello " + <-c + "!")
}

// gets a channels and writes a channel to it
func greeter(cc chan chan string) {
	c := make(chan string)
	cc <- c
}

func main() {
	fmt.Println("main() started")

	// make a channel `cc` of data type channel of string data type
	cc := make(chan chan string)

	go greeter(cc) // start `greeter` goroutine using `cc` channel

	// receive a channel `c` from `greeter` goroutine
	c := <-cc

	go greet(c) // start `greet` goroutine using `c` channel

	// send data to `c` channel
	c <- "John"

	fmt.Println("main() stopped")
}
```

### Выбор(Select)

используется для ожидания нескольких операций канала.

Синтаксис похоэж на "switch()"
```go
evenCh := make(chan int)
sqCh := make(chan int)

go evenSum(0, 100, evenCh)
go squareSum(0, 100, sqCh)

select {
  case x := <- evenCh:
     fmt.Println(x)
  case y := <- sqCh:
     fmt.Println(y)
}
```

В данном случае оператор "select" ожидает пока канал получит данные.
в case отражены условия - какой канал первым отправит данные, тот и будет выполнен.
- если каналы получают данные одновременно case выбирается случайным образом.

default выполняется если ни один из каналов не готов
Наприер у нас есть бесконечный цикл  "for", ожидающий пока один из каналов получит данные.

```go
for {
    select {
        case x := <- evenCh:
            fmt.Println(x)
            return
        case y := <- sqCh:
            fmt.Println(y)
            return
        default:
            fmt.Println("Ничего не доступно")
            time.Sleep(50 * time.Millisecond)
    }
}
```

Цикл **for** использует **select** для проверки того, какой канал получил данные. Если ни один из них не готов, будет выполнен случай **default**, который будет ждать 50 мс.  
Как только канал получит данные, оператор **return** завершит выполнение программы

default неблокируемый - всегда делает неблокируемым блок селект.

!!! При наличии default без настроек задержек, или синхронизации, он всегда сработает первым.

### Добавляем Timeout

Можно использовать пакет `"time"`, в нем есть функция `After`, реализующая канальную операцию.

как это выглядит:

```go
package main

import (
	"fmt"
	"time"
)

var start time.Time

func init() { // не понял что делает эта функция
	start = time.Now() 
}

func service1(c chan string) {
	time.Sleep(3 * time.Second)
	c <- "Hello from service 1"
}

func service2(c chan string) {
	time.Sleep(5 * time.Second)
	c <- "Hello from service 2"
}

func main() {
	fmt.Println("main() started", time.Since(start))

	chan1 := make(chan string)
	chan2 := make(chan string)

	go service1(chan1)
	go service2(chan2)

	select {
	case res := <-chan1:
		fmt.Println("Response from service 1", res, time.Since(start))
	case res := <-chan2:
		fmt.Println("Response from service 2", res, time.Since(start))
	case <-time.After(2 * time.Second):// из-за такой конструкции                                             горутина [main] будет                                                 разблокирована через 2 секунды
		fmt.Println("No response received", time.Since(start))
	}

	fmt.Println("main() stopped", time.Since(start))
}

```

Это полезно если мы не хотим ждать ответа от сервера целый день)))


### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://stepik.org/course/100208/syllabus
- https://habr.com/en/articles/490336/

