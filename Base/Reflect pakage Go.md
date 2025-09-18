17-09-2025 21:49
Tags: #Go #lesson 
# Что такое рефлексия и зачем

Мы не можем пройти по полям структуры циклом for, чтобы узнать какие у нас поля и какие  в них данные.
Одним из подходов работы со структурами это рефлексия.
Рефлексия позволяет оперировать со структурами и другимиобьектами(типами) программы, прямо во время ее выполнения

Пример ф-ции основанной на пакете reflect
```Go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	ID       int
	RealName string `unpack:"-"`
	Login    string
	Flags    int
}

func PrintReflect(u intrface{}) error{
	val := reflect.ValueOf(u).Elem()
	
	fmt.Printf("%T have %d fields:\n", u, val.NumField())
	for i := 0; i < val.NumField(); i++ {
		valueField := val.Field(i)
		typeField := val.Type().Field(i)
		
		fmt.Printf("\tname=%v, type=%v, value=%v, tag=`%v`\n",
			typeField.Name
			typeField.Type.Kind()
			valueField,
			typeField.Tag,
			)
	}
	return nil
}

func main() {
	u := &User{
		ID: 42,
		RealName: "Nikitos",
		Flags: 32,
	}
	err := PrintReflect(u)
	if err != nil {
		panic(err)
	}
}


```


Далее пример программы, как при помощи рефлексии можно заносить данные в поля структуры
```Go
package main

import (
	"fmt"
	"binary"
	"encoding/binary"
	"reflect"
)

type User struct {
	ID       int
	RealName string `unpack:"-"`
	Login    string
	Flags    int
}

func UnpackReflect(u interface{}, data []byte) error {
	r := bytes.NewReader(data)  //читаем слайс байт
	
	val := reflect.ValueOf(u).Elem() //получаем внутреннюю структуру из рефлексии
	
	for i := 0; i < val.NumField(); i++ { //идем по полям
		valueField := val.Field(i)
		typeField := val.Type().Field(i)
		
		if typeField.Tag.Get("unpack") == "-" { //обрабатываем тег(пропустить)
			continue
		}
		
		switch typeField.Type.Kind() { //делаем свич по типу данных
		case reflect.Int:
			var value uint32
			binary.Read(r, binary.LittleEndian, &value)
			valueField.Set(reflect.ValueOf(int(value))\
		case reflect.String:
			var lenRow uint32
			binary.Read(r, binary.LittleEndian, &lenRow)
			
			dataRow := make([]byte, lenRow)
			binary.Read(r, binary.LittleEndian, & dataRow)
			
			valueField.SetString(string(dataRow))
		default:
			return fmt.Errorf("bad type: %v for field %v",
			typeField.Type.Kind(), typeField.Name)
		}	
	}
	return nil
}

func main() {
	data := []byte{
		128, 36, 17, 0, //Есть слайс байт с запакованной, пакетом perl, инфой 
		
		9, 0, 0, 0,
		118, 46, 114, 111, 109, 97, 110, 111, 118,
		
		16, 0, 0, 0, 
	}
	
	u := new(User)
	err := UnpackReflect(u, data) // в какую структуру распаковываем, что распак.
	if err != nil {
		panic(err)
	}
	
	fmt.Printf("%#v\n", u)
}

```


### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://stepik.org/lesson/1131164/step/1?unit=1142761

