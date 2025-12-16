## Основные команды

```
# запуск контейнера
docker run <image>

# показать список активных контейнеров
docker ps

# показать все контейнеры
docker ps -a

# остановить контейнер
docker stop <container>

# запустить контейнер
docker start <container>

# перезапустить контейнер
docker restart <container>

# удалить контейнер
docker rm <container>

# показать список всех локальных образов
docker images

# загрузить образ
docker pull <image>

# удалить локальный образ
docker rmi <image>

# выполнить команду внутри контейнера
docker exec -it <container> <command>

# просмотреть логи контейнера
docker logs <container>
```

---

## Сборка и взаимодействие с образами

```
# Создание образа
docker build -t <name>:<tag> <path>

# Опции сборки:
# - указать Dockerfile
# - передать аргументы сборки (--build-arg)
# - сборка без кэша (--no-cache)
# - принудительно подтянуть базовые образы (--pull)
```

---

## Запуск и взаимодействие с контейнером

```
# Запуск в фоне (detached)
docker run -d --name <name> <image>

# Запуск в интерактивном режиме
docker run -it --name <name> --rm <image> <command>

# Проброс портов
docker run -p <host_port>:<container_port> <image>

# Примонтировать директорию (bind mount)
docker run -v <host_path>:<container_path> <image>

# Использовать named volume
docker run -v <volume_name>:<container_path> <image>

# Передать переменные окружения
docker run -e "KEY=VALUE" --env-file <file> <image>

# Автоперезапуск при падении
docker run --restart <policy> <image>

# Указать имя контейнера
docker run --name <name> <image>

# Удалять контейнер после выхода
docker run --rm <image>
```

---

## Репозитории: tag / push / pull

```
# Присвоить тег образу
docker tag <source> <target>

# Вход в реестр
docker login

# Публикация образа
docker push <repository>:<tag>

# Скачивание образа
docker pull <repository>:<tag>
```

---

## Работа с образами

```
# Показать историю образа
docker history <image>

# Сохранить образ в tar
docker save -o <file.tar> <image>

# Загрузить образ из tar
docker load -i <file.tar>

# Получить подробную информацию
docker inspect <image_or_container>
```

---

## Тома и примонтированные директории

```
# Создать volume
docker volume create <name>

# Просмотреть тома
docker volume ls

# Удалить том
docker volume rm <name>

# Примонтировать bind-том
docker run -v <host_path>:<container_path> <image>
```

---

## Сети

```
# Список сетей
docker network ls

# Создать сеть
docker network create <name>

# Подключить контейнер к сети
docker network connect <network> <container>

# Инспект сети
docker network inspect <name>
```

---

## Dockerfile минимум
Ключевые инструкции и их назначение (без примеров):

- **FROM** — указание базового образа.
    
- **WORKDIR** — рабочая директория внутри образа.
    
- **COPY / ADD** — копирование файлов в образ.
    
- **RUN** — команды, выполняемые при сборке.
    
- **CMD** — команда по умолчанию при старте контейнера.
    
- **ENTRYPOINT** — фиксирует исполняемую точку входа.
    
- **ARG** — аргументы сборки.
    
- **ENV** — переменные окружения.
    
- **EXPOSE** — документирование портов.
    
- **HEALTHCHECK** — проверка состояния контейнера.
    

Советы по написанию Dockerfile:

- Минимизируйте количество слоёв.
    
- Используйте multi-stage builds для уменьшения размера.
    
- Очищайте временные файлы после установки пакетов.
    

---

## Docker Compose — базовые команды

```
# Запустить сервисы
docker compose up -d

# Пересобрать и запустить
docker compose up --build -d

# Остановить и удалить ресурсы
docker compose down

# Просмотр логов
docker compose logs -f

# Выполнение команды в сервисе
docker compose exec <service> <command>
```

---

## Утилиты и очистка

```
# Удалить все остановленные контейнеры
docker container prune

# Удалить все неиспользуемые образы
docker image prune -a

# Освободить неиспользуемые ресурсы
docker system prune -a --volumes

# Показать использование диска
docker system df
```

---
