# Этап сборки
FROM node:18-alpine AS builder

# Создаем пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Установка рабочей директории
WORKDIR /app

# Копирование файлов зависимостей для кэширования слоев
COPY package.json package-lock.json ./

# Установка зависимостей
RUN npm ci --no-audit --no-fund

# Копирование остальных файлов проекта
COPY . .

# Сборка приложения
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm run build && \
    chown -R appuser:appgroup /app/dist

# Этап запуска приложения
FROM nginx:1.25-alpine

RUN apk update && apk upgrade

# Создаем пользователя
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Создаем директорию для публикации приложения с учетом префикса
RUN mkdir -p /usr/share/nginx/html/momo-store

# Копирование собранного приложения из этапа сборки в директорию с префиксом
COPY --from=builder --chown=appuser:appgroup /app/dist/ /usr/share/nginx/html/momo-store/

# Копирование кастомного конфига nginx
RUN mkdir -p /etc/nginx/conf.d
COPY --chown=appuser:appgroup nginx.conf /etc/nginx/conf.d/default.conf

# Установка правильных прав доступа для директорий
RUN chown -R appuser:appgroup /var/cache/nginx && \
    chown -R appuser:appgroup /var/log/nginx && \
    chown -R appuser:appgroup /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R appuser:appgroup /var/run/nginx.pid && \
    chmod -R 755 /usr/share/nginx/html

# Переключение на непривилегированного пользователя
USER appuser

# Открытие порта
EXPOSE 80

# Запуск nginx в режиме переднего плана
CMD ["nginx", "-g", "daemon off;"]
