29-09-2025 19:02
Tags: #Go #lesson 
# Профилирование веба

подключаем профилевщик: 
```Go
import _ "net/http/pprof" // "_" предед названием пакета т.к. в программе не вызываем ни одну из ф-ций, но используем в терминале
```
- импорт пишем в ту прогу которую будем тестить

далее запускаем программу под нагрузкой:
```
ab -t 20 -n 100000000000000 -c 5 http://127.0.0.1:8080
```

и снимаем дамп HEAP(памяти) и дамп CPU(процессора) при помощи команд:
```bash
curl http://127.0.0.1:8080/debug/pprof/heap -o mem_out.txt
curl http://127.0.0.1:8080/debug/pprof/profile?seconds=5 -o cpu_out.txt

go tool pprof -svg -alloc_objects pprof_1.exe mem_out.txt > mem_ao.svg
go tool pprof -svg pprof_1.exe cpu_out.txt > cpu.svg
```

- отключаем нагрузку (в консоли где включали нажми Ctrl+c)


# Можно найти утечку горутин
как снять стектрейс горутин
```
curl http://localhost:8080/debug/pprof/goroutine?debug=2 -o goroutines.txt
```

затем в тексте смотрим утекли ли горутины 
если горутина в бесконечном ожидании (например в ожидании считывания из канала) - она течет)




### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://stepik.org/lesson/1131169/step/1?unit=1142766

