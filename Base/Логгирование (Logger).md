06-10-2025 20:41
Tags: #Go #lesson 
# Зачем логировать

- логи это признаки жизни программы
- позвяют регетрировать действия с программой или внутри программы.
- позволяют искать ошибки

## Standart go Logger

import "log"
- обертка над пакетом fmt
- можно выбрать куда выводить лог (не только в stdOut), например в error bkb syslog
```Go
log.Printf("STD starting server at %s:%d", addr, port)
```
## Zap

- нет логгера по умолчанию, надо создавать
- структурированные логгеры, сами определяем что в него выводим и с каким значениеми типом данных
```Go
zapLogger, _ := zap.NewProduction()
defer zapLogger.Sync()
zapLogger.Info("starting server",
zap.String("logger", "ZAP"),
zap.String("host", addr),
zap.Int("port", port),
)
``` 
- нет аллокации памяти как в Sprintf
- быстрый логгер, не очень много куда может писать логи
## logrus
- структурированные логи - map[string]interface
- в целом те же яйца, только в профиль
```Go
logrus.SetFormatter(&logrus.TextFormatter{disableCollors: true})
logrus.WithFields(logrus.Fields{
	"logger": "LOGRUS",
	"host": addr,
	"port": port,
}).Info("starting server")
```
- медленный но может много куда отправлять логи
### Zero-Links
- [[00 Golang]]
- [[00 network]]
- [[00 Обучение]]


### Links
- https://stepik.org/lesson/1133343/step/1?auth=login&unit=1144962

