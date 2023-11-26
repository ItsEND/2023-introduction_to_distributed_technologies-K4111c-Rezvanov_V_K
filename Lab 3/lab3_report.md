University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023
Group: K4111c
Author: Rezvanov Vladislav Konstantinovich
Lab: Lab3
Date of create: 30.11.2023
Date of finished:30/11/2023

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных"
### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
### Ход работы
Запуск minikube cluster
```
minikube start
```
Подключаем ingress
```
minikube addons enable ingress
```
Создаем манифест
```
vim lab3_manifest.yml

```
Содержание манифеста

```
apiVersion: v1
data:
  REACT_APP_USERNAME: "Rezvanov"
  REACT_APP_COMPANY_NAME: "ITMO"
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: config

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab3
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab3
  template:
    metadata:
      labels:
        app: lab3
    spec:
      containers:
      - name: itdt-contained-frontend
        image: ifilyaninitmo/itdt-contained-frontend:master
        ports:
        - containerPort: 3000
        envFrom:
          - configMapRef:
              name: config
---

apiVersion: v1
kind: Service
metadata:
  name: lab3
spec:
  type: ClusterIP
  ports:
    - port: 3000
      protocol: TCP
      targetPort: 3000
  selector:
    app: lab3


---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab3
spec:
  tls:
    - hosts:
        - rezvanov-lab3.app
      secretName: rezvanov-lab3-tls
  rules:
    - host: rezvanov-lab3.app
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: lab3
                port:
                  number: 3000
```
#Генерируем TLS сертификат

```
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out cert.crt -keyout cert.key
```
![img0](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/062ade7089dc5011324021b9fa876062f7aabafd/Lab%203/Images/%D0%A1%D0%B5%D1%80%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82.png)
Далее сертификат подключается к миникубу

```
kubectl create secret tls rezvanov-lab3-tls --key="cert.key" --cert="cert.crt"
```
![img1](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/d953ec0cfa93a94c2b39e3c93c750dbf4014017a/Lab%203/Images/%D0%A1%D0%B5%D1%80%D1%82.png)

Подключаем манифест к кубу
```
kubectl apply -f manifest.yaml

```
##Связываем доменное имя и IP

```
echo "127.0.0.1 rezvanov-lab3.app" | sudo tee -a /etc/hosts
```

![img2](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/4b57902ebcc8ad93edff43f034a385895c1fb590/Lab%203/Images/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-11-26%2021-16-55.png)

Создаем тунель
```
minikube tunnel
```
![img3](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/d953ec0cfa93a94c2b39e3c93c750dbf4014017a/Lab%203/Images/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-11-26%2019-59-36.png)

Открываем страницу адресса https://rezvanov-lab3.app и проверяем наличие сертификата 
![img4](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/d953ec0cfa93a94c2b39e3c93c750dbf4014017a/Lab%203/Images/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-11-26%2019-57-37.png)


###Схема
![SCH1](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/6f07619f57948e3be0c409e4fc802184eabd7ec6/Lab%203/Images/%D0%9A%D1%83%D0%B1.drawio.png) 

