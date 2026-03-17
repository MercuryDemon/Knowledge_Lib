


go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

curl -o .golangci.yml https://raw.githubusercontent.com/golangci/golangci-lint/master/.golangci.example.yml
Написать 3 Makefile target'а - make go-tools для скачивания golangci-lint и зависимостей; make lint для прогона линтеров; и make lint-fix для автофиксов.


```
GOLANGCI_LINT_VERSION ?= v1.55.2
GOLANGCI_LINT_BIN ?= $(shell go env GOPATH)/bin/golangci-lint
GOLANGCI_LINT_CONFIG ?= .golangci.yml
GO_CMD = go
GO_MOD = $(GO_CMD) mod

# Цвета
GREEN := \033[0;32m
YELLOW := \033[1;33m
RED := \033[0;31m
BLUE := \033[0;34m
NC := \033[0m

.PHONY: go-tools
go-tools:
	@printf "$(BLUE)╔════════════════════════════════════╗$(NC)\n"
	@printf "$(BLUE)║    Установка инструментов Go      ║$(NC)\n"
	@printf "$(BLUE)╚════════════════════════════════════╝$(NC)\n"
	
	@printf "$(YELLOW)📦 Проверка golangci-lint...$(NC)\n"
	@if ! command -v $(GOLANGCI_LINT_BIN) >/dev/null 2>&1; then \
		printf "   Установка golangci-lint $(GOLANGCI_LINT_VERSION)...\n"; \
		curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $$(go env GOPATH)/bin $(GOLANGCI_LINT_VERSION); \
		printf "$(GREEN)   ✓ golangci-lint установлен$(NC)\n"; \
	else \
		printf "$(GREEN)   ✓ golangci-lint уже установлен$(NC)\n"; \
	fi
	
	@printf "$(YELLOW)📥 Управление зависимостями...$(NC)\n"
	@$(GO_MOD) download && \
	 printf "$(GREEN)   ✓ Зависимости скачаны$(NC)\n" || \
	 printf "$(RED)   ✗ Ошибка при скачивании зависимостей$(NC)\n"
	
	@$(GO_MOD) verify && \
	 printf "$(GREEN)   ✓ Зависимости проверены$(NC)\n" || \
	 printf "$(RED)   ✗ Ошибка при проверке зависимостей$(NC)\n"
	
	@printf "$(GREEN)✅ Инструменты готовы к работе$(NC)\n"

.PHONY: lint
lint: go-tools
	@printf "$(BLUE)╔════════════════════════════════════╗$(NC)\n"
	@printf "$(BLUE)║         Запуск линтеров           ║$(NC)\n"
	@printf "$(BLUE)╚════════════════════════════════════╝$(NC)\n"
	
	@if [ -f "$(GOLANGCI_LINT_CONFIG)" ]; then \
		printf "$(YELLOW)🔍 Использую конфиг: $(GOLANGCI_LINT_CONFIG)$(NC)\n"; \
		$(GOLANGCI_LINT_BIN) run --config=$(GOLANGCI_LINT_CONFIG) --timeout=5m ./...; \
	else \
		printf "$(YELLOW)🔍 Конфиг не найден, использую стандартные настройки$(NC)\n"; \
		$(GOLANGCI_LINT_BIN) run --timeout=5m ./...; \
	fi
	
	@if [ $$? -eq 0 ]; then \
		printf "$(GREEN)✅ Линтинг прошел успешно!$(NC)\n"; \
	else \
		printf "$(RED)❌ Найдены проблемы в коде$(NC)\n"; \
		printf "$(YELLOW)   Попробуйте 'make lint-fix' для автоисправления$(NC)\n"; \
		exit 1; \
	fi

.PHONY: lint-fix
lint-fix: go-tools
	@printf "$(BLUE)╔════════════════════════════════════╗$(NC)\n"
	@printf "$(BLUE)║     Автоисправление линтеров      ║$(NC)\n"
	@printf "$(BLUE)╚════════════════════════════════════╝$(NC)\n"
	
	@printf "$(YELLOW)🔧 Запуск автоисправления...$(NC)\n"
	@$(GOLANGCI_LINT_BIN) run --fix --timeout=5m ./...
	
	@if [ $$? -eq 0 ]; then \
		printf "$(GREEN)✅ Автоисправление завершено$(NC)\n"; \
	else \
		printf "$(RED)❌ Ошибка при автоисправлении$(NC)\n"; \
		exit 1; \
	fi
	
	@printf "$(YELLOW)📋 Проверка оставшихся проблем...$(NC)\n"
	-@$(GOLANGCI_LINT_BIN) run --timeout=5m ./... || \
		printf "$(RED)   Остались проблемы, требующие ручного исправления$(NC)\n"

.PHONY: help
help:
	@printf "$(BLUE)╔════════════════════════════════════╗$(NC)\n"
	@printf "$(BLUE)║    Go проект - доступные команды  ║$(NC)\n"
	@printf "$(BLUE)╚════════════════════════════════════╝$(NC)\n"
	@awk '/^[a-zA-Z\-_]+:/ { \
		helpMessage = match(lastLine, /^## (.*)/); \
		if (helpMessage) { \
			helpCommand = substr($$1, 0, index($$1, ":")-1); \
			helpMessage = substr(lastLine, RSTART+3, RLENGTH); \
			printf "  $(GREEN)%-15s$(NC) %s\n", helpCommand, helpMessage; \
		} \
	} { lastLine = $$0 }' $(MAKEFILE_LIST)

# Версия
.PHONY: version
version:
	@printf "$(YELLOW)Инструменты:$(NC)\n"
	@printf "  Go: $$(go version)\n"
	@printf "  golangci-lint: $$($(GOLANGCI_LINT_BIN) version 2>/dev/null || echo 'не установлен')\n"
```

/bin/sh: 10:    : not found

```
/bin/sh: 8: [1: not found
❌ Найдены проблемы в коде
   Попробуйте 'make lint-fix' для автоисправления
make: *** [Makefile:86: lint] Ошибка 1
```


```
DELETE FROM dictionary_shedule_batch

USING dictionaries

WHERE dictionaries.id = dictionary_shedule_batch.id AND dictionaries.title = $1 AND dictionaries.author = $2;

  

DELETE FROM dictionaries

WHERE title = $1 AND author = $2;
```