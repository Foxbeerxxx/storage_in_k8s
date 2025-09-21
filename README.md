# Домашнее задание к занятию "`Хранение в K8s`" - `Татаринцев Алексей`



### Задание 1

1. `Пишу манифест containers-data-exchange.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange
  labels:
    app: data-exchange
spec:
  replicas: 1
  selector:
    matchLabels:
      app: data-exchange
  template:
    metadata:
      labels:
        app: data-exchange
    spec:
      volumes:
        - name: shared-data
          emptyDir: {}
      containers:
        - name: busybox-writer
          image: busybox:1.36
          volumeMounts:
            - name: shared-data
              mountPath: /shared
          command: ["/bin/sh","-c"]
          args:
            - |
              echo "writer started"
              while true; do
                date +"%F %T: heartbeat" >> /shared/exchange.log
                sleep 5
              done
        - name: multitool-reader
          image: wbitt/network-multitool:alpine-minimal
          volumeMounts:
            - name: shared-data
              mountPath: /shared
          command: ["/bin/sh","-c"]
          args: ["sleep infinity"]

```
2. `Запускаю манифест и проверяю`

```
kubectl apply -f containers-data-exchange.yaml
kubectl get pods -l app=data-exchange
kubectl describe pods -l app=data-exchange

```
![1](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img1.png)

![2](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img2.png)

3. `Чтение общего файла из контейнера multitool-reader:`

```
POD=$(kubectl get pod -l app=data-exchange -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it "$POD" -c multitool-reader -- /bin/sh -c 'tail -f /shared/exchange.log'

```
![3](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img3.png)


### Задание 2



1. `Пишу манифест pv-pvc.yaml`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота 2](ссылка на скриншот 2)`


---

### Задание 3

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`

### Задание 4

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`
