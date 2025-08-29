27-05-2025 22:28
Tags: #z #lesson #Go

## Определение

Интерфейс - набор сигнатур методов, которые надо реализовать, чтобы удовлетворить контракту.

- Одному интерфейсу могут соответствовать много типов
- один Тип может реализовывать несколько интерфейсов

Абстракция чтобы абстрагироваться от реализации

Описывают методы, которые обязательны для абстрактного обЪекта или класса.

можно сказать, что это кастомный тип данных.

## Типы
Переменная типа интерфейс может содержать значения типа, реализующего этот интерфейс.
```go
var s Stringer //статический тип
s = time.Time() //динамический тип
```

- Значение переменной типа интерфейс хранит динамического тип и значение этого типа.
- У самой переменной типа интерфейс статический тип
- В пакете "fmt" можно смотреть глаголами %v и %T


## Инициализируется 
```go
type name interface { // mane - название интерфейса(любое)
	doSomething() // сугнатура метода реализуемого интерфейсом
}
```

- Пример: 
```go
type animal interface { 
	makeSound() 
}

type cat struct{}
type dog struct{}

func (c *cat) makeSound() {
	fmt.Println("meow!")
}

func (d *dog) makeSound() {
	fmt.Println("woof!")
}

func main() {
	var c, d animal = &cat{}, &dog{}
	c.makeSound()
	d.makeSound()
}
```

- Функции могут принимать инетрфейсы в качесве переменных.
```go
type greeter intrface{
	greet(string) string
}

type russian struct {}
type nigger struct {}

func (r *russian) greet(name string) string{
	fmt.Scanln(&name)
	return fmt.Printf("Здарова, %s !", name)
}
func (n *nigger) greet(name string) string{
	fmt.Scanln(&name)
	return fmt.Printf("Bomboclat, %s !", name)
}

func sayHello(g greeter, name string) {
	fmt.Print(g.greet(name))
}

func main() {
	sayHello(&russian{}, "Васян")
	sayHello(&nigger{}, "RICH-MILLIONIRE")
}
```


## Композитные интерфейсы

Можно составлять композитные интерфейсы. Инетрфес содержит другие интерфейсы поменьше.

- Необходимо определять интерфейсы с минимальной функциональностью.
(болт прикручивает доску и ничего более, не стучит по гвоздям. ни защищает от влаги)

- нельзя именовать методы внутри одного интерфеса одинаково- у каждого метода уникальное имя. Если один интерфес включает другой и в другом есть метод с именем, которое есть в основном интерфейсе они будут конфликтовать - не скомпилируется.

- Главный интерфейс будет требовать методы всех вложенных интерфейсов.
```go
type animal intrface {
	walker
	runner
}
type bird interface {
	walker
	flyer
}

type walker intrface {
	walk()
}

type runner interface {
	run()
}

type flyer interface {
	fly()
}

type cat struct{}
type owl struct{}

func (c *cat) walk() {
	fmt.Println("Cat is walking")
}

func (c *cat) run() {
	fmt.Println("Cat is running")
}

func (o *owl) walk() {
	fmt.Println("Owl is walking")
}
func (o *owl) fly() {
	fmt.Println("Owl is flying")
}

func walk(w walker) {
	w.walk()
}

func main() {
	var c animal = &cat
	var o bird = &owl
	walk(c)
	walk(o)
}
```

Композитные интерфейсы позволяют придерживаться принципа Interface segregation(разделение интерфейсов)

Значит ходить может и животное и птица, по этому мы создаем функцию "walk" которая принимает 
в качестве аргумента интерфейс "walker". То есть мы можем передать в нее любой обьект который 
реализует только один метод "walker".

### Пустые интерфейсы

нет методов обусловливающих этот интерфейс по этому ему соответствуют любые типы данных.

- Пустой интерфейс не говорит ничего.

ЗОЧЕМ:
- неопределенность в типах данных с которыи предстоит работать.

- парсим json, читаем и структурируем инфу из json формата, там могут быть разные значения с разными типами данных, по этому мы используем пустой интерфейс т. к. все типы данных под него подходят.

Пример для пустого интерфейса:

```go
package main

import ("fmt"
	"reflect"
)
func main() {
	m := map[string]interface{}{} //первые фигурные скобки это пустой интерфейс вторые это мы создаем пустую мапу.
	m["one"] = 1
	m["two"] = 2.0
	m["three"] = "MIBOMBOC"
	m["four"] = "true"

	for k, v := range m {
		switch v.(type){
			case int:
				fmt.Printf("%s is an integer\n", k)
			case float64:
				fmt.Printf("%s is a float64\n", k)
			default:
				fmt.Printf("%s is %v\n", k, reflect.TypeOf(v))
		}
	}
}
```

Самый яркий пример использования пустого интерфейса в качесве принимаемого функцией аргумента - функция fmt.Println
ее сигнатура: `func Println(a ...interface{}) (n int, err error)`


## Устройство интерфейсов

Интерфейс, как и все в го, есть структура
![[Pasted image 20250829185248.png]]

как это примерно выглядит
![[Pasted image 20250829190354.png]]
![[Pasted image 20250829190817.png]]


## Zero value

Пустое значение интерфейса = `nil`

`interface = nil` только когда его динамичесский тип равен `nil` и значение динамического типа равно `nil`








### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://www.youtube.com/watch?si=elDpmLk0g2URvt69&v=tCN8ac6C1tA&feature=youtu.be
- https://youtu.be/nOlkEqftxWg?si=lpA9A04IvEOSoHOf

