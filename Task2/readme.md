## Задание 2. Динамическое масштабирование контейнеров

Сейчас сервисы InsureTech развёрнуты в Kubernetes. Каждый из них развёрнут в определённом количестве экземпляров.

Обычно этих экземпляров достаточно для успешной обработки всех запросов. 
Но в периоды пиковой нагрузки система не справляется: она демонстрирует нестабильное поведение и постоянно перезагружает 
поды из-за нехватки памяти. Как следствие, пользователи получают негативный опыт работы с приложением. Бизнес видит, что NPS снижается.

Можно, конечно, держать больше реплик постоянно активными, чтобы система могла справиться с пиковыми нагрузками. 
Но это экономически невыгодно и приведёт к низким показателям утилизации ресурсов. 
Таким образом, вам необходимо решить проблему с помощью конфигурации динамического масштабирования для сервисов компании.

Вы будете тестировать динамическое масштабирование на примере простого приложения. Оно предоставляет два ресурса:

 - `GET /` — получение идентификатора пода;
 - `GET /metrics` — получение метрик в формате Prometheus.

Оба метода приложения доступны по порту `8080`.

Метрика `http_requests_total` возвращает количество запросов для метода получения идентификатора пода.

Образ тестового приложения:
```
docker pull shestera/scaletestapp
```

**Обратите внимание**: в этом задании две части — одна обязательная, другая дополнительная.
Мы рекомендуем выполнить вторую часть, если вы хотите глубже разобраться в теме и получить опыт конфигурации кастомных метрик 
и горизонтального масштабирования на их основе. 

## Обязательная часть задания: динамическая маршрутизация на основании показателей утилизации памяти

### Что нужно сделать

1. Поднимите локальный кластер Kubernetes в Minikube.
2. Активируйте metrics-server.
3. Напишите манифест развёртывания (Deployment) Kubernetes для запуска тестового приложения. 
   Для начального количества реплик установите значение, равное единице. Лимит памяти установите равный “30Mi”. 
   Примените написанную конфигурацию в вашем кластере:
   ```
   kubectl apply -f deployment.yml
   ```
   В рамках пул-реквеста добавьте файл в директорию **Task2**.

4. Напишите и примените манифест сервиса (Service) для доступа к приложению, которое вы установили на прошлом шаге. 
   ```
   kubectl apply -f service.yml
   ```
   В рамках пул-реквеста файл с этим манифестом тоже загрузите в директорию **Task2**.

5. Настройте динамическую маршрутизацию на основании показателей утилизации оперативной памяти с помощью Horizontal Pod Autoscaler (HPA). 
   Для нашего тестового приложения оптимальный уровень утилизации памяти равен 80%. 
   В качестве максимального количества реплик рекомендуем установить 10. Примените манифест в вашем кластере. 
   ```
   kubectl apply -f hpa.yml
   ```
   В рамках пул-реквеста загрузите готовый манифест в директорию **Task2**.

   ```
   minikube service scaletestapp-service --url
   ```


6. Настройте динамическую маршрутизацию на основании показателей утилизации оперативной памяти с помощью Horizontal Pod Autoscaler (HPA). 
   Для этого нужно активировать поддержку метрик в вашем кластере. Самый простой способ это сделать — воспользоваться командой:
   ```
   minikube addons enable metrics-server
   ```

7. Теперь создайте манифест для `Horizontal Pod Autoscaler`. Этот манифест будет автоматически масштабировать количество реплик 
   вашего приложения в зависимости от роста потребления оперативной памяти (memory). 
   Для нашего тестового приложения оптимальный уровень утилизации памяти равен 80%. 
   В качестве максимального количества реплик рекомендуем установить 10. Примените манифест в вашем кластере. 
   Загрузите готовый манифест в директорию **Task2** в рамках пул-реквеста.

8. Теперь надо убедиться, что всё работает как задумано. Для этого необходимо сгенерировать нагрузку на приложение. 
   Воспользуйтесь инструментом нагрузочного тестирования locust:
   a. Создайте `Locustfile`. Это сценарий на Python, где вы определяете поведение пользователей. 
   Создайте файл с именем `locustfile.py` в удобной для вас директории. Скопируйте туда код:
   ```
     from locust import HttpUser, between, task
      class WebsiteUser(HttpUser):
         wait_time = between(1, 5)
         @task
         def index(self):
             self.client.get("/") 
   ```
   Этот пример создаёт класс пользователя, который переходит на главную страницу ("/") с интервалом между запросами от 1 до 5 секунд.
   b. Откройте терминал и перейдите в директорию, где находится ваш locustfile.py. Выполните команду:
   ```
   locust
   ```
   c. После запуска Locust откройте веб-браузер и введите адрес `http://localhost:8089`. 
   Вы увидите веб-интерфейс Locust, где можно настроить параметры теста
   d. Запустите тест и проанализируйте результаты. Проще всего посмотреть результаты в дашборде Kubernetes. 
   В локальном кластере Minikube его можно открыть с помощью команды:
   ```
   minikube dashboard
   ```
Сделайте скриншоты дашборда или выгрузите логи, которые покажут, что количество реплик базы данных поменялось в ответ на сгенерированную нагрузку. 
Загрузите их в директорию **Task2** в рамках пул-реквеста.


## Дополнительная часть задания: динамическая маршрутизация на основании показателей количества запросов в секунду

**Это задание выполнять необязательно**. Если сдадите работу без него, это не отразится на ревью.
Kubernetes предоставляет возможность управлять масштабированием на основании CPU и memory. 
Однако на реальных проектах зачастую требуется бóльшая гибкость для управления масштабированием. 
Для этого нужно использовать внешние метрики из системы мониторинга, которая может предоставить их в Kubernetes. Например, Prometheus.
В нашем проекте нужно настроить динамическое масштабирование на основании количества запросов в секунду (RPS) на один под приложения.

### Что нужно сделать

1. Установите Prometheus в вашем кластере. Рекомендуем установить Prometheus в Kubernetes с помощью Prometheus Operator через Helm. 
   Лучше всего воспользоваться Prometheus Community Helm charts. Для этого используйте команды:
```
helm repo add prometheus-community <https://prometheus-community.github.io/helm-charts>
helm repo update
helm install prometheus-operator prometheus-community/kube-prometheus-stack 
```

2. Теперь необходимо обеспечить экспорт метрик из приложения в Prometheus. Для этого стоит воспользоваться Service Monitor. 
   Он использует Prometheus Operator для автоматического обнаружения сервисов в Kubernetes посредством Service Discovery. 
   Вот пример манифеста ServiceMonitor:
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: scaletestapp-app-sm
  namespace: default
  labels:
    serviceMonitorSelector: prometheus
spec:
  endpoints:
    - interval: 10s
      targetPort: 8080
      path: /metrics
  namespaceSelector:
    matchNames:
      - default
  selector:
    matchLabels:
      prometheus-monitored: "true" 
```

Требуемый сервис можно обнаружить посредством лейбла `app` или кастомного лейбла (например, `prometheus-monitored`), 
который вы можете отразить в манифесте Service.

3. ServiceMonitor можно применить как отдельный манифест или воспользоваться специальной секцией `additionalServiceMonitors`
   при настройке Prometheus Operator через Helm. \n Вот пример конфигурации Prometheus Operator:
```
defaultRules:
  create: false
alertmanager:
  enabled: false
grafana:
  enabled: false
kubeApiServer:
  enabled: false
kubelet:
  enabled: false
kubeControllerManager:
  enabled: false
coreDns:
  enabled: false
kubeEtcd:
  enabled: false
kubeScheduler:
  enabled: false
kubeStateMetrics:
  enabled: false
nodeExporter:
  enabled: false
prometheus:
  enabled: true
  additionalServiceMonitors:
    - name: app-sm
      namespace: default
      labels:
        serviceMonitorSelector: prometheus
      endpoints:
        - interval: 10s
          targetPort: 8080
          path: /metrics
      namespaceSelector:
        matchNames:
          - default
      selector:
        matchLabels:
          prometheus-monitored: "true" 
```

4. Когда примените конфигурацию, проверьте, что метрики из вашего приложения поступают в Prometheus. 
   Зайдите в Prometheus Web UI, откройте раздел Graph или Targets, чтобы убедиться, что ваше приложение отображается и метрики доступны. 
   Чтобы получить доступ к интерфейсу, нужно открыть доступ к серверу Prometheus с локальной машины. Например, с помощью команды:
```
minikube service <имя сервиса> --url 
```

Сделайте скриншоты интерфейса с метриками и загрузите их в директорию **Task2**.

5. Теперь необходимо настроить Prometheus Adapter для использования метрик Prometheus в Horizontal Pod Autoscaler (HPA). 
   Prometheus Adapter служит мостом между Kubernetes и Prometheus. Он позволяет Kubernetes использовать метрики Prometheus для масштабирования подов. 
   Чтобы настроить Prometheus Adapter:
     a. Когда используете Helm для установки или обновления Prometheus Adapter, предоставьте значения, которые переопределяют базовую конфигурацию. 
        Создайте файл `values.yaml` и включите туда определение для prometheus-adapter:
```
   prometheus:
     url: "http://<адрес_prometheus>"
   rules:
     default: false
     custom:
       - seriesQuery: 'http_requests_total{namespace!="",pod!=""}'
         resources:
           overrides:
             namespace: {resource: "namespace"}
             pod: {resource: "pod"}
         name:
           matches: "^http_requests_total"
           as: "http_requests_per_second"
         metricsQuery: 'sum(rate(http_requests_total{<<.LabelMatchers>>}[30s])) by (<<.GroupBy>>)'
```
     
     b. Установите Prometheus Adapter при помощи команды:
```
   helm intall prometheus-adapter prometheus-community/prometheus-adapter -f values.yaml
```
     c. Чтобы убедиться, что кастомная метрика `http_requests_per_second` стала доступной, воспользуйтесь командой:
```
   kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```
6. Обновите манифест Horizontal Pod Autoscaler. Укажите там, что масштабирование нужно производить на базе новой метрики — `http_requests_per_second`. 
   Вот пример манифеста:
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: scaletestapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: scaletestapp
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: 50 
```

Когда будете сдавать задание, загрузите новую версию манифеста в директорию **Task2**.

7. Теперь надо убедиться, что всё работает как задумано. Для этого сгенерируйте нагрузку на приложение. 
   Действуйте по аналогии с восьмым шагом в обязательной части задания.
  
   Сделайте скриншоты дашборда или выгрузите логи, которые покажут, что количество реплик базы данных поменялось в ответ на сгенерированную нагрузку. 
   Загрузите их в директорию **Task2** в рамках пул-реквеста.
