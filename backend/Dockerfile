# Этап 1: Сборка приложения
FROM golang:1.22.7-alpine AS builder

# Установка необходимых инструментов для сборки
RUN apk add --no-cache git

# Рабочая директория внутри контейнера
WORKDIR /app

# Копирование файлов зависимостей
COPY go.mod go.sum ./

# Загрузка зависимостей (будет кэшироваться, если go.mod/go.sum не изменились)
RUN go mod download

# Копирование исходного кода
COPY . .

# Сборка приложения с отключенным CGO и статической линковкой
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o /go/bin/app ./cmd/api

# Этап 2: Создание финального образа
FROM alpine:3.18

# Установка сертификатов CA для HTTPS запросов
RUN apk --no-cache add ca-certificates && \
    update-ca-certificates

# Создаем непривилегированного пользователя в финальном образе
RUN adduser -D -g '' appuser

# Создаем директорию для приложения
WORKDIR /app

# Копирование скомпилированного бинарника из builder
COPY --from=builder /go/bin/app /app/

# Настройка базовых прав доступа
RUN chmod 500 /app/app && \
    chown -R appuser:appuser /app

# Переключение на непривилегированного пользователя
USER appuser

# Открытие порта, который слушает приложение
EXPOSE 8081

# Запуск приложения
CMD ["/app/app"]
