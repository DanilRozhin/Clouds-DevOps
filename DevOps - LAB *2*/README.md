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
1. порты открыты глобально (0.0.0.0)    
2. нет ограничений на ресурсы    
3. секреты прописаны напрямую в файле     

Почему это плохо?  
1. порты открыты глобально (0.0.0.0)    
Порты доступны всем устройствам в сети, что небезопасно    
Мы можем получить уязвимость для атак извне и доступ к сервисам получают неавторизованные пользователи     
``` 
ports:
  - "8000:80"
  - "6379:6379"
```
2. нет ограничений на ресурсы      
Возможна перегрузка CPU и памяти и нестабильность работы системы    
 
3. секреты прописаны напрямую в файле     
Риск утечки секретов, потому что пароли попадают в Git и становятся видимыми    


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
1. теперь порты привязаны к локальному хосту (127.0.0.1)     
```
ports:
  - "127.0.0.1:8000:80"
  - "127.0.0.1:6379:6379"
```
 
2. добавлены ограничения на использование ресурсов    
```
deploy:
  resources:
    limits:
      cpus: "0.5"
      memory: 256M
```
3. секреты вынесены в .env файл   
```
environment:
  - REDIS_PASSWORD=${REDIS_PASSWORD}
  - REDIS_USER=${REDIS_USER}
```
### Ход работы
Теперь поговорим про ход самой работы.    

Сначала переходим в директорию где лежит наш проект и запускаем плохой Docker Compose файл с помощью команды:   
``` docker-compose -f docker-compose-bad.yml up ```
![image](https://github.com/user-attachments/assets/0bc80371-8f19-413f-88a9-cb8d3db994a4)

Запущены два сервиса:     
app с образом Nginx (порт 8000:80)     
redis с образом Redis (порт 6379:6379)     

Теперь проверим работу Redis в плохом файле. Перейдя в браузере по адресу localhost:6379 мы увидим, что он не доступен    
![image](https://github.com/user-attachments/assets/99c5a044-1c3a-4661-abc8-d48396427411)

Теперь оставновим плохой файл с помощью ``` docker-compose -f docker-compose-bad.yml down ```    

![image](https://github.com/user-attachments/assets/fb6b1b27-67cc-4df1-814e-1349a96be546)

Дальше запустим наш хороший файл с помощью команды ``` docker-compose -f docker-compose-good.yml up ```    
Сейчас мы получаем, что у нас запущены сервисы:    
app на 127.0.0.1:8000 (Nginx)    
redis на 127.0.0.1:6379 с настройками окружения из .env    
![image](https://github.com/user-attachments/assets/78242d2a-4bbd-43ca-bdf2-03adbd81e07f)   

Проверим, что наш файл рабоатет и все верно. Перейдя в браузере по адресу 127.0.0.1:8000, увидим стандартную nginx страницу, значит все работает правильно    
![image](https://github.com/user-attachments/assets/d4a1c682-28ea-4846-b935-147de21546c1)    

Теперь нам надо проверить на изоляцию наших сервисов, для этого пропишем    
```
docker exec -it lab-2-app-1 sh 
curl redis:6379
```
![image](https://github.com/user-attachments/assets/ef9a6127-441d-4f1f-a103-2865c0a39c1c)    
Получаем ошибку ``` Could not resolve host: redis``` , это значит, что сервисы изолированы      
Чтобы этого добиться мы сделали так, что в рамках одного docker-compose.yml, каждому сервису назначается своя отдельная сеть     
Это исключит возможность взаимодействия между контейнерами, даже если они запущены в одном проекте     
Конкретно в нашем файле это:   
app_network — для сервиса app   
redis_network — для сервиса redis   

Теперь нам надо оставить наш файл с помощью команды ``` docker-compose -f docker-compose-good.yml down ```
![image](https://github.com/user-attachments/assets/e95337be-7177-49a4-aff4-475e46ced52a) 
(это до команды)   
![image](https://github.com/user-attachments/assets/e7b5e786-e622-4b97-9840-2838214c2c71)
(это после)
Теперь все файлы остановлены, задание выполнено   

