# Лабораторная работа 2*
Для начала напишем плохой Docker Compose файл, и поймем где в нем ошибки
## Плохой Docker Compose

```
version: "3"

services:
  app:
    image: nginx:latest
    ports:
      - "8000:80"

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    environment:
      - REDIS_PASSWORD="qwerty"
      - REDIS_USER="admin"
```

В нашем плохом файле есть три ключевые ошибки это:   
порты открыты глобально (0.0.0.0)   
нет ограничений на ресурсы   
секреты прописаны напрямую в композе    

Теперь исправим эти ошибки в хорошем файле

## Хороший Docker Compose
```
version: "3.8"

services:
  app:
    image: nginx:latest
    ports:
      - "127.0.0.1:8000:80"
    networks:
      - app_network
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M

  redis:
    image: redis:alpine
    ports:
      - "127.0.0.1:6379:6379"
    networks:
      - redis_network
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}
      - REDIS_USER=${REDIS_USER}
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M

networks:
  app_network:
    driver: bridge
  redis_network:
    driver: bridge
```
Исправив ошибки мы получили, то что:
теперь порты привязаны к локальному хосту (127.0.0.1)   
добавлены ограничения на использование ресурсов   
секреты вынесены в .env файл

### Ход работы
Теперь поговорим про ход саомй работы.   

Сначала переходим в директорию где лежит наш проект и запускаем плохой Docker Compose файл с помощью команды:  
``` docker-compose -f docker-compose-bad.yml up ```
