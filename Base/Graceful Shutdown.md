18-06-2025 22:35
Tags: #z #lesson
# Что такое Изящное завершение.

По сути мы перехватыфвает сигнал отмены `^C` ОС в коде и останавливаем все внутренние процессы и очищаем все занятые программой рессурсы.

пример программы, которая изящно завершается находится тут
```go
package main

  

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os/signal"
	"syscall"
	"time"
)

const (
	listenAddr = "127.0.0.1:8080"
	shutdownTimeout = 5 * time.Second
)
	
type handlerIndex struct {
		handlerIndex uint
}

  func (hi handlerIndex) ServeHTTP(w http.ResponseWriter, r *http.Request) {

	messege := fmt.Sprintf("index is: %v", hi.handlerIndex)
	w.Write([]byte(messege))
}

func main() {

	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()
		
	if err := runServer(ctx); err != nil {
		log.Fatal(err)
	}
}

  

func runServer(ctx context.Context) error {

	var (
	mux = http.NewServeMux()	
	srv = &http.Server{	
		Addr: listenAddr,	
		Handler: mux,	
		}	
	)
	
	  
	
	handlerIndex := handlerIndex{handlerIndex: 1}
	
	  
	
	mux.Handle("/", handlerIndex)
	
	  

	go func() {	
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		log.Fatal("listen and serve: %v", err)
		}
	}()
		
	log.Printf("listening on %v", listenAddr)
	<-ctx.Done()

	log.Println("shutting down server gracefully")

	shutdownCtx, cancel := context.WithTimeout(context.Background(), shutdownTimeout)
	defer cancel()
	
	if err := srv.Shutdown(shutdownCtx); err != nil {
	return fmt.Errorf("shutdown: %w", err)

}
	longShutdown := make(chan struct{}, 1)
	
	go func() {
		time.Sleep(2 * time.Second)
		longShutdown <- struct{}{}
	}()
		select {
		case <-shutdownCtx.Done():	
			return fmt.Errorf("server shutdown: %w", ctx.Err())
		case <-longShutdown:
			log.Println("finished")
	}
	return fmt.Errorf("server was shutted downed by system call")
}

  

// press ctrl+c and see what heppend))
```


Там еще можно создать структуру Closer которая будет регистрировать обработчик и ввызывать все операции по завершению программы.

подробне в видосе
### Zero-Links
- [[00 Golang]]
- [[00 Обучение]]


### Links
- https://youtu.be/JKZuKkquoqk?si=jwJVlr_NI4ZXwcIY

