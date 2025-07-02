<<<<<<< HEAD
28-05-2025 21:30
Tags: #z #lesson #Go 

Напрямую связано с [[Многопоточность в Go]].
## WaitGroup

Такая структура  со счётчиком, которая помогает отследить, что все горутины выполнены.

Она имеет три метода Add, Wait и Done.
Add(например 1) - добавляет к счётчику число "1" - инкрементирует счётчик.

Wait  - блокирует текущую(ту в которой был вызван) горутину. до тех пор пока счётчик != 0

Done - декркментирует счётчик (на единицуесли верить исходникам пакета=) )

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func service(wg *sync.WaitGroup, instance int) {
	time.Sleep(2 * time.Second)
	fmt.Println("Service called on instance", instance)
	wg.Done() // decrement counter
}

func main() {
	fmt.Println("main() started")
	var wg sync.WaitGroup // create waitgroup (empty struct)

	for i := 1; i <= 3; i++ {
		wg.Add(1) // increment counter
		go service(&wg, i)
	}

	wg.Wait() // blocks here
	fmt.Println("main() stopped")
}

```


## Пул воркеров(набор работяг)

когда в горутинах используются передача данных по канала м они становятся воркерами.

Пример нескольких работях выполняющих одну функцию

```go
package main

import (
	"fmt"
	"time"
)

// worker than make squares
func sqrWorker(tasks <-chan int, results chan<- int, instance int) {
	for num := range tasks {
		time.Sleep(time.Millisecond) // simulating blocking task
		fmt.Printf("[worker %v] Sending result by worker %v\n", instance, instance)
		results <- num * num
	}
}

func main() {
	fmt.Println("[main] main() started")

	tasks := make(chan int, 10)
	results := make(chan int, 10)

	// launching 3 worker goroutines
	for i := 0; i < 3; i++ {
		go sqrWorker(tasks, results, i)
	}

	// passing 5 tasks
	for i := 0; i < 5; i++ {
		tasks <- i * 2 // non-blocking as buffer capacity is 10
	}

	fmt.Println("[main] Wrote 5 tasks")

	// closing tasks
	close(tasks)

	// receving results from all workers
	for i := 0; i < 5; i++ {
		result := <-results // blocking because buffer is empty
		fmt.Println("[main] Result", i, ":", result)
	}

	fmt.Println("[main] main() stopped")
}

```

вывод программы: 
```
[main] main() started
[main] Wrote 5 tasks
[worker 0] Sending result by worker 0
[main] Result 0 : 0
[worker 2] Sending result by worker 2
[main] Result 1 : 4
[worker 1] Sending result by worker 1
[main] Result 2 : 16
[worker 2] Sending result by worker 2
[main] Result 3 : 64
[worker 0] Sending result by worker 0
[main] Result 4 : 36
[main] main() stopped
### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]
```

Приведенный пример достаточно большой, но прекрасно объясняет, как несколько горутин могут извлекать данные из канала и выполнять свою работу. Горутины весьма эффективны, когда они могут блокироваться. Если убрать вызов `time.Sleep()`, то только одна горутина будет выполняться, так как другие горутины не будут запланированы, до тех пор пока цикл не закончится и горутина не завершится.

как это можно оптимизировать используя Waitgroup:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// worker than make squares
func sqrWorker(wg *sync.WaitGroup, tasks <-chan int, results chan<- int, instance int) {
	for num := range tasks {
		time.Sleep(time.Millisecond)
		fmt.Printf("[worker %v] Sending result by worker %v\n", instance, instance)
		results <- num * num
	}
	// done with worker
	wg.Done()
}

func main() {
	fmt.Println("[main] main() started")

	var wg sync.WaitGroup

	tasks := make(chan int, 10)
	results := make(chan int, 10)

	// launching 3 worker goroutines
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go sqrWorker(&wg, tasks, results, i)
	}

	// passing 5 tasks
	for i := 0; i < 5; i++ {
		tasks <- i * 2 // non-blocking as buffer capacity is 10
	}

	fmt.Println("[main] Wrote 5 tasks")

	// closing tasks
	close(tasks)

	// wait until all workers done their job
	wg.Wait()

	// receving results from all workers
	for i := 0; i < 5; i++ {
		result := <-results // non-blocking because buffer is non-empty
		fmt.Println("[main] Result", i, ":", result)
	}

	fmt.Println("[main] main() stopped")
}
```

вывод программы:
```
[main] main() started
[main] Wrote 5 tasks
[worker 2] Sending result by worker 2
[worker 0] Sending result by worker 0
[worker 1] Sending result by worker 1
[worker 0] Sending result by worker 0
[worker 2] Sending result by worker 2
[main] Result 0 : 16
[main] Result 1 : 0
[main] Result 2 : 4
[main] Result 3 : 64
[main] Result 4 : 36
[main] main() stopped
```

В приведенном результате мы получили немного другой, более аккуратный вывод, потому что операция чтения из канала `results` в `main` не блокируется, так как канал `results` уже содержит данные из-за вызванного ранее `wg.Wait()`. Используя `WaitGroup`, мы можем предотвратить много (ненужных) переключений контекста (планирование горутин и их запуск), в данном случае 7 против 9 в предыдущем примере. Но при этом вам приходится ожидать завершения всех горутин.



### Race condition

Когда много горутин обрабатывают общие данные и они пытаются взаимодействовать с данными в общей области памяти, что приводит к нерпедсказуемосу результату.

Грубо говоря горутины могут не успевать завершить какието действия с данными в тот момент как другая горутина уже была запланирована. 
И по сути вторая проделывает те же шаги что и первая с теми же занчениями что и первая, таким образом выполняет ненужную работу не влияющую на результат.

Можно проверить программу на `race condition`  запуская с ключём `race`:

`go run -race program.go`

### data race

Когда несколько потоков одновременно считывают и изменяют общий рессурс.

### Мьютекс

Концепция програмирования когда во время работы какого либо потока с переменной, эта переменная блокируется и ее нельзя считать и нельзя в нее что-то записать. 

Мьютекс предоставляет пакет `sync`.
`mutex.Lock()` и `mutex.Unlock()`

```go
package main

import (
	"fmt"
	"sync"
)

var i int // i == 0

// goroutine increment global variable i
func worker(wg *sync.WaitGroup, m *sync.Mutex) {
	m.Lock() // locking
	i = i + 1
	m.Unlock() // unlocking
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	var m sync.Mutex

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go worker(&wg, &m)
	}

	// wait until all 1000 gorutines are done
	wg.Wait()

	// value of i should be 1000
	fmt.Println("value of i after 1000 operations is", i)
}
```

вывод программы: 
```
value of i after 1000 operations is 1000
```


## Паттерны конкурентного програмирования 

### Генератор

На каналах можно достаточно просто организовать генератор, в котором генерация чего либо будет происходить паралелльно в отдельных горутинах.
Это будет на много быстрее.

Пример, генератор ряда фибоначи:

```go
package main

import "fmt"

// fib returns a channel which transports fibonacci numbers

func fib(length int) <-chan int { 
	// make buffered channel   
	c := make(chan int, length)   
	// run generation concurrently  
	go func() {  
		for i, j := 0, 1; i < length; i, j = i+j, i { 
		c <- i        
		}
		close(c)   
	}()   
	// return channel  
	return c
}
func main() {
	// read 10 fibonacci numbers from channel returned by `fib` function    
	for fn := range fib(10) {
	    fmt.Println("Current fibonacci number is", fn)
    }
}
```

### Fan-in и Fan-out

Fun-in - страиегия мультиплексирования, когда входы нескольких каналов обьёдиняются в один выходной канал.

Fun-out - тоже самое но наоборот



### Links
- https://habr.com/en/articles/490336/
- https://go.dev/blog/race-detector
- https://github.com/luk4z7/go-concurrency-guide // Полезные паттерны конкурентности
||||||| (empty tree)
=======
28-05-2025 21:30
Tags: #z #lesson #Go 

Напрямую связано с [[Многопоточность в Go]].
## WaitGroup

Такая структура  со счётчиком, которая помогает отследить, что все горутины выполнены.

Она имеет три метода Add, Wait и Done.
Add(например 1) - добавляет к счётчику число "1" - инкрементирует счётчик.

Wait  - блокирует текущую(ту в которой был вызван) горутину. до тех пор пока счётчик != 0

Done - декркментирует счётчик (на единицуесли верить исходникам пакета=) )

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func service(wg *sync.WaitGroup, instance int) {
	time.Sleep(2 * time.Second)
	fmt.Println("Service called on instance", instance)
	wg.Done() // decrement counter
}

func main() {
	fmt.Println("main() started")
	var wg sync.WaitGroup // create waitgroup (empty struct)

	for i := 1; i <= 3; i++ {
		wg.Add(1) // increment counter
		go service(&wg, i)
	}

	wg.Wait() // blocks here
	fmt.Println("main() stopped")
}

```


## Пул воркеров(набор работяг)

когда в горутинах используются передача данных по канала м они становятся воркерами.

Пример нескольких работях выполняющих одну функцию

```go
package main

import (
	"fmt"
	"time"
)

// worker than make squares
func sqrWorker(tasks <-chan int, results chan<- int, instance int) {
	for num := range tasks {
		time.Sleep(time.Millisecond) // simulating blocking task
		fmt.Printf("[worker %v] Sending result by worker %v\n", instance, instance)
		results <- num * num
	}
}

func main() {
	fmt.Println("[main] main() started")

	tasks := make(chan int, 10)
	results := make(chan int, 10)

	// launching 3 worker goroutines
	for i := 0; i < 3; i++ {
		go sqrWorker(tasks, results, i)
	}

	// passing 5 tasks
	for i := 0; i < 5; i++ {
		tasks <- i * 2 // non-blocking as buffer capacity is 10
	}

	fmt.Println("[main] Wrote 5 tasks")

	// closing tasks
	close(tasks)

	// receving results from all workers
	for i := 0; i < 5; i++ {
		result := <-results // blocking because buffer is empty
		fmt.Println("[main] Result", i, ":", result)
	}

	fmt.Println("[main] main() stopped")
}

```

вывод программы: 
```
[main] main() started
[main] Wrote 5 tasks
[worker 0] Sending result by worker 0
[main] Result 0 : 0
[worker 2] Sending result by worker 2
[main] Result 1 : 4
[worker 1] Sending result by worker 1
[main] Result 2 : 16
[worker 2] Sending result by worker 2
[main] Result 3 : 64
[worker 0] Sending result by worker 0
[main] Result 4 : 36
[main] main() stopped
### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]
```

Приведенный пример достаточно большой, но прекрасно объясняет, как несколько горутин могут извлекать данные из канала и выполнять свою работу. Горутины весьма эффективны, когда они могут блокироваться. Если убрать вызов `time.Sleep()`, то только одна горутина будет выполняться, так как другие горутины не будут запланированы, до тех пор пока цикл не закончится и горутина не завершится.

как это можно оптимизировать используя Waitgroup:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// worker than make squares
func sqrWorker(wg *sync.WaitGroup, tasks <-chan int, results chan<- int, instance int) {
	for num := range tasks {
		time.Sleep(time.Millisecond)
		fmt.Printf("[worker %v] Sending result by worker %v\n", instance, instance)
		results <- num * num
	}
	// done with worker
	wg.Done()
}

func main() {
	fmt.Println("[main] main() started")

	var wg sync.WaitGroup

	tasks := make(chan int, 10)
	results := make(chan int, 10)

	// launching 3 worker goroutines
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go sqrWorker(&wg, tasks, results, i)
	}

	// passing 5 tasks
	for i := 0; i < 5; i++ {
		tasks <- i * 2 // non-blocking as buffer capacity is 10
	}

	fmt.Println("[main] Wrote 5 tasks")

	// closing tasks
	close(tasks)

	// wait until all workers done their job
	wg.Wait()

	// receving results from all workers
	for i := 0; i < 5; i++ {
		result := <-results // non-blocking because buffer is non-empty
		fmt.Println("[main] Result", i, ":", result)
	}

	fmt.Println("[main] main() stopped")
}
```

вывод программы:
```
[main] main() started
[main] Wrote 5 tasks
[worker 2] Sending result by worker 2
[worker 0] Sending result by worker 0
[worker 1] Sending result by worker 1
[worker 0] Sending result by worker 0
[worker 2] Sending result by worker 2
[main] Result 0 : 16
[main] Result 1 : 0
[main] Result 2 : 4
[main] Result 3 : 64
[main] Result 4 : 36
[main] main() stopped
```

В приведенном результате мы получили немного другой, более аккуратный вывод, потому что операция чтения из канала `results` в `main` не блокируется, так как канал `results` уже содержит данные из-за вызванного ранее `wg.Wait()`. Используя `WaitGroup`, мы можем предотвратить много (ненужных) переключений контекста (планирование горутин и их запуск), в данном случае 7 против 9 в предыдущем примере. Но при этом вам приходится ожидать завершения всех горутин.



### Race condition

Когда много горутин обрабатывают общие данные и они пытаются взаимодействовать с данными в общей области памяти, что приводит к нерпедсказуемосу результату.

Грубо говоря горутины могут не успевать завершить какието действия с данными в тот момент как другая горутина уже была запланирована. 
И по сути вторая проделывает те же шаги что и первая с теми же занчениями что и первая, таким образом выполняет ненужную работу не влияющую на результат.

Можно проверить программу на `race condition`  запуская с ключём `race`:

`go run -race program.go`

### data race

Когда несколько потоков одновременно считывают и изменяют общий рессурс.

### Мьютекс

Концепция програмирования когда во время работы какого либо потока с переменной, эта переменная блокируется и ее нельзя считать и нельзя в нее что-то записать. 

Мьютекс предоставляет пакет `sync`.
`mutex.Lock()` и `mutex.Unlock()`

```go
package main

import (
	"fmt"
	"sync"
)

var i int // i == 0

// goroutine increment global variable i
func worker(wg *sync.WaitGroup, m *sync.Mutex) {
	m.Lock() // locking
	i = i + 1
	m.Unlock() // unlocking
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	var m sync.Mutex

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go worker(&wg, &m)
	}

	// wait until all 1000 gorutines are done
	wg.Wait()

	// value of i should be 1000
	fmt.Println("value of i after 1000 operations is", i)
}
```

вывод программы: 
```
value of i after 1000 operations is 1000
```


## Паттерны конкурентного програмирования 

### Генератор

На каналах можно достаточно просто организовать генератор, в котором генерация чего либо будет происходить паралелльно в отдельных горутинах.
Это будет на много быстрее.

Пример, генератор ряда фибоначи:

```go
package main

import "fmt"

// fib returns a channel which transports fibonacci numbers

func fib(length int) <-chan int { 
	// make buffered channel   
	c := make(chan int, length)   
	// run generation concurrently  
	go func() {  
		for i, j := 0, 1; i < length; i, j = i+j, i { 
		c <- i        
		}
		close(c)   
	}()   
	// return channel  
	return c
}
func main() {
	// read 10 fibonacci numbers from channel returned by `fib` function    
	for fn := range fib(10) {
	    fmt.Println("Current fibonacci number is", fn)
    }
}
```

### Fan-in и Fan-out

Fun-in - страиегия мультиплексирования, когда входы нескольких каналов обьёдиняются в один выходной канал.

Fun-out - тоже самое но наоборот



### Links
- https://habr.com/en/articles/490336/
- https://go.dev/blog/race-detector

>>>>>>> 9d44e37 (first commit down there)
