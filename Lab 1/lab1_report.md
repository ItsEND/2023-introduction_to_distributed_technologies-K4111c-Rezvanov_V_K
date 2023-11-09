University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2023
Group: K4111c
Author: Rezvanov Vladislav Konstantinovich
Lab: Lab1
Date of create: 30.10.2023
Date of finished:

## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."
### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".
### Ход работы
Запуск minikube cluster
```
minikube start
```
```
kubectl run void --image=hashicorp/vault --port=8200  -o yaml > mymanifest.yml
```
Создание "пода" с использованием файла HashiCorpVault-pod
```
kubectl create -f HashiCorpVault-pod.yml 
```
Содержание манифеста
```
apiVersion: v1
kind: Pod
metadata:
  name: vault
  namespace: default
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: hashicorp/vault

```
Создание сервиса и проброс порта 8200 сервиса на порт 8200 локальной машины
```
minikube kubectl -- expose pod vault --type=NodePort --port=8200
minikube kubectl -- port-forward service/vault 8200:8200
```
![img2](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/23398d80260130f00c836e595c818771f036f5e7/Lab%201/vault_sign.png)
После указанных действий хранилище доступно по адресу [http://localhost:8200](http://localhost:8200), однако для доступа нужно найти токе в логах:
```
minikube kubectl -- logs vault
```
![img3](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/4b892d341d30037325f8497fa6d62698b6a37888/Lab%201/vault_sign.png).
### Ответы на вопросы
1. Что сейчас произошло и что сделали команды указанные ранее?
Команда `kubectl run vault --image=hashicorp/vault --port=8200 -o yaml > mymanifest.yml` создала под с именем `MyPodName` в кластере Kubernetes.
Этот под использует образ `hashicorp/vault`. Команда также прокинула порт 8200 из контейнера в порт на узле. 
Результат этой команды был записан в файл `mymanifest.yml` в формате YAML
Затем, командf `minikube kubectl -- expose pod vault --type=NodePort --port=8200` для создания сервиса, который предоставляет доступ к подам в кластере.
Этот сервис слушает на порту 8200 и перенаправляет все входящие соединения на этот порт в под "vault"
Команда `minikube kubectl -- port-forward service/vault 8200:8200` для проброса порта 8200 с компьютера в контейнер. Это позволило обращаться к сервису Vault, используя URL http://localhost:8200.


2. Где взять токен для входа в Vault?
Токен для входа в Vault обычно генерируется при инициализации Vault. 
Скорее всего, он будет в логах пода. Можно просмотреть логи пода, используя команду `kubectl logs vault`

### Схема
![schema](https://github.com/ItsEND/2023-introduction_to_distributed_technologies-K4111c-Rezvanov_V_K/blob/37f9511d7f89ca594585173879f313b4c7aef1ff/Lab%201/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80.drawio.png)
