# Задание 6. Настройка Rate Limiting

Количество запросов от одного из партнёров всё ещё снижает производительность приложения. 
Это влияет на опыт использования API других партнёров. Этой проблемой тоже нужно заняться.
Одно из возможных решений — динамическое масштабирование приложения, которое вы уже внедрили. 
Однако ресурсы системы не бесконечные. Поэтому необходимо предусмотреть дополнительную защиту на случай, 
если количество запросов от партнёра снова превысит оговорённое количество.

С этим поможет паттерн Rate Limiting.

**Что нужно сделать**

Перед вами конфигурационный файл Nginx. Доработайте его таким образом, чтобы обеспечить ограничение количества отправляемых запросов. 
Должно быть не более 10 запросов в минуту.

```
http {
   # Настройка upstream для балансировки нагрузки
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }

   server {
       listen 80;

       location / {
           proxy_pass http://backend_servers;
       }

   }
}
```

Сделайте так, чтобы при превышении лимита запросов клиент получал HTTP-ошибку с кодом 429.

# Предлагаемое решение

```
http {
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }

   # Определение зоны для ограничения запросов. Ограничиваем по IP адресу
   limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/m;

   server {
       listen 80;

       location / {
           proxy_pass http://backend_servers;
           limit_req zone=api_limit nodelay;
           limit_req_status 429;
       }
   }
}
```
