02-06-2025 20:31
Tags: #z #lesson 
# КАРАУУУУЛ

паника это пиздец - крашит всё нахуй.
Паника это критическая ошибка и руина всей программы.

recovery - перехват паники в какойлибо горутине(чтобы не крашилось всё)

панику нельзя перехватывать из другой горутины, потому что у каждой горутины свой изолированный стек.

## NeverExit пттерн

```go 
func task() {
	time.Sleep(200*time.Millisecond)
	panic("ахуеть что произошло?")
}

func NeverExit (name string, action func()) {
	defer func() {
		if v := recover(); v != nil {
			log.Println(name, "is crashed restarting")
			go NeverExit(name, action) // нет рекурсии, есть свой стек
		}
	}()
	if action != nil {
		action()
	}
}

func main() {
	go NeverExit("first_goroutine", task)
	go NeverExit("second_goroutine", task)

	time.Sleep(5 * time.Second)

}

```

### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://youtu.be/p2JixdG950Y?si=mwa1cO7Nune-T3O8

