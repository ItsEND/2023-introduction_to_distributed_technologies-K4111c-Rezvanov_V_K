University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023
Group: K4111c
Author: Rezvanov Vladislav Konstantinovich
Lab: Lab1
Date of create: 30.10.2023
Date of finished:16/11/2023

## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."
### Цель работы
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.
### Ход работы
Запуск minikube cluster
```
minikube start
```
Создаем деплоймент
```
vim deployment.yaml

```
Содержание манифеста

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secondlab-deployment
  labels:
    app: secondlab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secondlab
  template:
    metadata:
      labels:
        app: secondlab
    spec:
      containers:
      - name: secondlab
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: "Rezvanov"
        - name: REACT_APP_COMPANY_NAME
          value: "ITMO_Univer"
        ports:
        - containerPort: 3000
```
```
kubectl apply -f deployment.yaml

```
Создаём сервис для доступа к контейнеру:
```
minikube kubectl -- expose deployment.apps secondlab-deployment --type=NodePort --port=3000

```

Проверяем список объектов:
```
minikube kubectl get all

minikube kubectl -- port-forward service/vault 8200:8200
```
Запускаем
```
minikube kubectl -- port-forward service/secondlab-deployment 3000:3000
```
![img1](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/7455a5296ab2af39acaf1038d636bc33c6c13425/Lab%202/Images/%D0%97%D0%B0%D0%BF%D1%83%D1%81%D0%BA.png)

Переходим на http://localhost:3000/

![img2](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/7455a5296ab2af39acaf1038d636bc33c6c13425/Lab%202/Images/Second.png)

Смотрим логи контейнеров

```
minikube kubectl logs secondlab-deployment-5d94784d4-8pv75

```
```
minikube kubectl logs secondlab-deployment-5d94784d4-h7swg 

```
K8s сразу балансирует запросы между репликами подов. При переходе на страницу, запрос падает на одну из двух реплик, чтобы понять на какую именнно развернутое приложение сообщает на странице информацию о названии контейнера, в котором оно развернуто и его IP внутри кластера. Эти значения можно сравнить со скришотом запущенных pod'ов - они должны совпадать с одним из них. Повторный запрос на страницу может попасть на тот же самый pod, а может на другой, следует попробовать несколько раз обновить страницу и следить за информацией о контейнере.
![img3](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/7455a5296ab2af39acaf1038d636bc33c6c13425/Lab%202/Images/%D0%9B%D0%BE%D0%B3%D0%B8.png).

### Схема
![img4](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/76d5cc72d90c1d7f4beae55da69aa4757bd7b9f1/Lab%202/Images/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%202.drawio.png)
