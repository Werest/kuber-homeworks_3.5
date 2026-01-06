# kuber-homeworks_3.5

# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

-------------------------------------------
<img width="1711" height="270" alt="image" src="https://github.com/user-attachments/assets/388ef2a8-8ce0-4d88-b359-376f5fe75e43" />

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-consumer
  namespace: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-consumer
  template:
    metadata:
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do curl auth-db; sleep 5; done
        image: radial/busyboxplus:curl
        name: busybox
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-db
  namespace: data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-db
  template:
    metadata:
      labels:
        app: auth-db
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: data
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: auth-db
```

Основная проблема: приложения web-consumer и auth-db находятся в разных Namespace (пространствах имен) Kubernetes. Из-за этого они не могут обнаружить друг друга по короткому DNS-имени.

**Выявленная проблема**
- web-consumer запущен в Namespace web.
- auth-db (Deployment и Service) запущен в Namespace data.
Команда curl auth-db внутри контейнера web-consumer ищет сервис только в своем собственном Namespace (web), но не находит его, так как сервис находится в Namespace data.

**Кардинальное решение**
Чтобы исправить проблему и создать конфигурацию, которая будет работать у любого другого человека без дополнительных шагов, нужно разместить все компоненты приложения в одном Namespace.

Исправлю и сделаю так: все ресурсы будут находиться в пространстве имен default (можно заменить на любое другое единое имя).

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-consumer
  namespace: default # замена
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-consumer
  template:
    metadata:
      labels:
        app: web-consumer
    spec:
      containers:
      - command:
        - sh
        - -c
        # Теперь сервис auth-db будет найден, т.к. находится в том же namespace
        - while true; do curl -s http://auth-db; sleep 5; done
        image: curlimages/curl:latest
        name: curl-container
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-db
  namespace: default # замена
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-db
  template:
    metadata:
      labels:
        app: auth-db
    spec:
      containers:
      - image: nginx:1.19.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: auth-db
  namespace: default # замена
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: auth-db
```

```
kubectl get pods -n default --watch
kubectl get pods -n default -l app=web-consumer -o name
kubectl logs -n default web-consumer-7cbbd4dffd-abcde
```
<img width="891" height="288" alt="image" src="https://github.com/user-attachments/assets/8ecbc405-b15f-44da-b7c8-1d83c59a49f3" />

<img width="1826" height="242" alt="image" src="https://github.com/user-attachments/assets/c34b3ade-cf05-4585-a8a5-b95f913b71b7" />

Ошибка - образ radial/busyboxplus:curl использует устаревший формат манифеста (schema 1), который больше не поддерживается Docker и container runtime. Это распространенная проблема со старыми образами.

После исправлений:
<img width="1528" height="859" alt="image" src="https://github.com/user-attachments/assets/2a6056a8-9dca-4ae8-82f4-4f12de175bf6" />

Второй под:
<img width="947" height="485" alt="image" src="https://github.com/user-attachments/assets/61c35f8a-645d-4ad6-8dfe-1eb6b1c90c01" />


-------------------------------------------


### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
