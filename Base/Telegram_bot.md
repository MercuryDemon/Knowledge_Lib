29-09-2025 20:19
Tags: #Go #telegram #lesson 
# Есть готовая библа
``` Go
import "gopkg.in/telegram-bot-api.v4"
```

1. Находим в телеге бота "BotFather"
2. создаем нового бота /newbot
3. пришем имя
4. пишем логин -> получаем токен (HTTP API)
5. програмируем чего нам надо, например:
```Go
package main

import (
	"encoding/xml"
	"io"
	"net/http"
	"gopkg.in/telegram-bot-api.v4"
	)

const (
	BotToken = "" //токен полученый от BotFather
	WebhookURL = "https://XXXXXX.ngrok.io" //лучше не использовать но для примера пойдет
)

var rss = map[string]string{
"Habr": "https://habrhabr.ru/rss/best/",
}

type RSS struct {
	Items []Item `xml:"channel>item"`
}

type Item struct {
	URL string `xml:"guid"`
	Title string `xml:"title"`
}
func getNews(url string) (*RSS, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}

	defer resp.Body.Close()
	body, _ := io.ReadAll(resp.Body)  
	
	rss := new(RSS)
	err = xml.Unmarshal(body, rss)
	if err != nil {
		return nil, err
	}
	return rss, nil
}
  
func main() {
	bot, err := tgbotapi.NewBotAPI(BotToken)
	if err != nil {
		panic(err)
	}
  
	fmt.Printf("Authorized on acc %s\n", bot.Self.UserName)
	_, err = bot.SetWebhook(tgbotapi.NewWebhook(WebhookURL))
	if err != nil {
		panic(err)
	}
	updates := bot.ListenForWebhook("/")
	for update := range updates {
		if url, ok := rss[update.Message.Text]; ok {
			rss, err :=getNews(url)
			if err != nil {
				bot.Send(tgbotapi.NewMessage(
				update.Message.Chat.ID,
				"sorry, error happend",
				))
			}
			for _, item :+ range rss.Items {
				bot.Send(tgbotapi.NewMessage(
					update.Message.Chat.ID,
					item.URL+"\n"+item.Title,
				))	
			}
		} else {
			bot.Send(tgbotapi.NewMessage(
				update.Message.Chat.ID,
				`there is only Habr feed avalible`,
			))
		}
	}
}
```

6.  go run
7. Находим бота в телеге
8. инициализируем /start
9. Бот запущен
### Zero-Links
-[[00 Golang]]
[[00 Обучение]]
[[00 network]]


### Links
- https://stepik.org/lesson/1131170/step/1?unit=1142767

