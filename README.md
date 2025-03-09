# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

Создадим под

root@DebianNew:~/.kube# kubectl create ns kube999

Создадим deploy

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-app-multitool-busybox
  namespace: kube999
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo "current date = $(date)" >> /busybox_dir/date.log; sleep 5; done']
        volumeMounts:
          - mountPath: "/busybox_dir"
            name: deployment-volume
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: deployment-volume
            mountPath: "/multitool_dir"
      volumes:
        - name: deployment-volume
          emptyDir: {}
```

Запускаем deploy

root@DebianNew:~/.kube# kubectl apply -f dep_busybox_multitool.yaml

Заходим в под

root@DebianNew:~/.kube# kubectl exec -n kube999 netology-app-multitool-busybox-7fd46d4c46-d7flt -c multitool -it -- bash

Проверяем доступность файлов

netology-app-multitool-busybox-7fd46d4c46-d7flt:/# ls -lah

![alt text](https://github.com/MaratKN/kuber-homeworks-06/blob/main/1.png)

netology-app-multitool-busybox-7fd46d4c46-d7flt:/# cat multitool_dir/date.log

![alt text](https://github.com/MaratKN/kuber-homeworks-06/blob/main/2.png)
![alt text](https://github.com/MaratKN/kuber-homeworks-06/blob/main/3.png)
------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

Создадим DaemonSet
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: multitool-logs
  namespace: kube999
spec:
  selector:
    matchLabels:
      app: mt-logs
  template:
    metadata:
      labels:
        app: mt-logs
    spec:
      containers:
        - name: multitool
          image: wbitt/network-multitool
          volumeMounts:
            - name: log-volume
              mountPath: /log_data
      volumes:
        - name: log-volume
          hostPath:
            path: /var/log
```

Запускаем DaemonSet

root@DebianNew:~/.kube# kubectl apply -f daemonSet_multitool.yaml 

Заходим в под

root@DebianNew:~/.kube# kubectl exec -n kube999 multitool-logs-vgrcl -it -- bash

Проверяем доступность файлов

multitool-logs-vgrcl:/# ls -lah

multitool-logs-vgrcl:/# ls -lah log_data

![alt text](https://github.com/MaratKN/kuber-homeworks-06/blob/main/4.png)

multitool-logs-vgrcl:/# tail log_data/syslog

![alt text](https://github.com/MaratKN/kuber-homeworks-06/blob/main/5.png)


------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------