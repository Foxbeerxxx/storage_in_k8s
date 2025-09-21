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

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
  labels:
    pv: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""        
  hostPath:
    path: /var/k8s/pv-local   # каталог на ноде
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      pv: local-pv            # привязка к PV по метке
  storageClassName: ""        # должен совпадать
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pv-pvc-demo
  labels:
    app: pv-pvc-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pv-pvc-demo
  template:
    metadata:
      labels:
        app: pv-pvc-demo
    spec:
      containers:
        - name: busybox-writer
          image: busybox:1.36
          volumeMounts:
            - name: data
              mountPath: /data
          command: ["/bin/sh","-c"]
          args:
            - |
              echo "writer started"
              while true; do
                date +"%F %T: heartbeat" >> /data/exchange.log
                sleep 5
              done
        - name: multitool-reader
          image: wbitt/network-multitool:alpine-minimal
          volumeMounts:
            - name: data
              mountPath: /data
          command: ["/bin/sh","-c"]
          args: ["sleep infinity"]
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: local-pvc

```



2. `Подготовка каталога на ноде`

```
sudo mkdir -p /var/k8s/pv-local
sudo chmod 777 /var/k8s/pv-local

```
![4](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img4.png)

3. `Применение и проверка`

```
kubectl apply -f pv-pvc.yaml

kubectl get pv,pvc
kubectl get pods -l app=pv-pvc-demo
kubectl describe pv local-pv
kubectl describe pvc local-pvc
kubectl describe pod -l app=pv-pvc-demo
```
![5](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img5.png)

![6](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img6.png)

![7](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img7.png)

4. `Чтение файла из multitool`
```
POD=$(kubectl get pod -l app=pv-pvc-demo -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it "$POD" -c multitool-reader -- /bin/sh -c 'tail -f /data/exchange.log'

```
![8](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img8.png)

5. `Удаление Deployment и PVC, наблюдение за PV`

```
kubectl delete deploy pv-pvc-demo
kubectl delete pvc local-pvc
kubectl get pv
kubectl describe pv local-pv


PV перейдёт в Released. Причина - PVC удалён, но persistentVolumeReclaimPolicy: Retain, поэтому том не очищается и не возвращается в пул. 
Связанный путь и данные остаются, и PV помечен как занятый.

```
![9](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img9.png)

6. `Проверка файла на локдиске ноды` 

```
sudo ls -l /var/k8s/pv-local
sudo tail -n 5 /var/k8s/pv-local/exchange.log


Файл есть, потому что это обычный каталог хоста, а не объект в кластере
```
![10](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img10.png)


7. `Удаление PV и проверка данных` 

```
kubectl delete pv local-pv
sudo ls -l /var/k8s/pv-local
sudo tail -n 5 /var/k8s/pv-local/exchange.log

Файл остаётся. Удаление PV удаляет только объект Kubernetes. Для hostPath Kubernetes не управляет жизненным циклом данных на хосте.
```
![11](https://github.com/Foxbeerxxx/storage_in_k8s/blob/main/img/img11.png)



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
