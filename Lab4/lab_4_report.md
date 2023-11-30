University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023
Group: K4111c
Author: Rezvanov Vladislav Konstantinovich
Lab: Lab4
Date of create: 30.11.2023
Date of finished:30/11/2023

### Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

## Описание
Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают underlay и overlay сети, а управление может быть организованно различными CNI.

## Цель работы
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

## Ход работы
В процессе выполнения работы были выполнены следующие шаги:

Запуск Minikube с плагином CNI Calico и в режиме Multi-Node Clusters:

```
minikube start --cni=calico --nodes=2
```
![img1](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/bb5492c96f446decc821b23612de15556a62e896/Lab4/Images/1.png)

Проверка наличия подов calico и количество запущенных нод
```
kubectl get nodes
kubectl get pods -n kube-system -l k8s-app=calico-node
```
![img2](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/bb5492c96f446decc821b23612de15556a62e896/Lab4/Images/2.png)

Присвоим запущенных нодам метки на основании географического положения:
```
kubectl label nodes minikube location=east
kubectl label nodes minikube-m02 location=west
```
![img3](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/563541c67abf2aa7eb35ff38ba8e08a24ad805b2/Lab4/Images/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-11-30%2017-32-08.png)

Создадим манифест для Calico, определяющий IP-пулы в соответствии с метками узлов:
```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: east-pool
spec:
  cidr: 10.0.0.0/24
  nodeSelector: location == 'east'
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: west-pool
spec:
  cidr: 10.0.1.0/24
  nodeSelector: location == 'west'
```
Создадим манифест с необходимым окружением:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: "Rezvanov"
        - name: REACT_APP_COMPANY_NAME
          value: "ITMO"
```
![img4](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/c0feacfb8892dffe114b8a8368d190590d459c16/Lab4/Images/3.png)
Применим манифесты
```
kubectl-calico create -f calicoctl.yaml --allow-version-mismatch
kubectl apply -f dep.yml
```
Выводим информацию о IP пулах
```
kubectl-calico get ippool -o wide --allow-version-mismatch
```
Пишем сервис и применяем его
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 9090
      targetPort: 3000
```
Открываем доступ к сервису Minikube: 
```
minikube service frontend-service
```
![img5](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/c0feacfb8892dffe114b8a8368d190590d459c16/Lab4/Images/6.1.png)
Получим IP одного из контейнеров и пропингуем из другого:
```
kubectl get pods -o wide
kubectl exec -it frontend-5bd66466-f5dpp -- /bin/sh
ping 10.244.205.193
```
![img7](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/c0feacfb8892dffe114b8a8368d190590d459c16/Lab4/Images/7.png)

### Схема
![sc1](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/5160f71c95a8c3d430f736409b1dbe4a1a5fe00b/Lab4/Images/%D0%9A%D1%83%D0%B11.drawio.png)
