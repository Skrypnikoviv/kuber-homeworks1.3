# Домашнее задание к занятию «Запуск приложений в K8S» - Скрыпников Илья

## Задание 1. Создание Deployment и обеспечение доступа к репликам приложения из другого Pod

### 1. Создание Deployment с двумя контейнерами (nginx и multitool)

Создан файл `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        command: ["sleep", "infinity"]
        ports:
        - containerPort: 8080
```

Применён манифест:

```bash
kubectl apply -f deployment.yaml
```
![image](https://github.com/user-attachments/assets/c2ac6711-bb53-47a0-aa8b-cfb8274eb286)

### 2. Увеличение количества реплик до 2

Изменено количество реплик в `deployment.yaml`:

```yaml
replicas: 2
```

Применены изменения:

```bash
kubectl apply -f deployment.yaml
```
![image](https://github.com/user-attachments/assets/5c080af4-4c1b-4523-906a-cf64f7ddcb4a)

Проверено количество подов до и после масштабирования:

```bash
kubectl get pods
```
![image](https://github.com/user-attachments/assets/9112451b-e468-474c-9726-eb97d4c4297a)

**Результат:**
- До масштабирования: 1 под.
- После масштабирования: 2 пода.

### 3. Создание Service для доступа к репликам

Создан файл `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-service
spec:
  selector:
    app: nginx-multitool
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Применён манифест:

```bash
kubectl apply -f service.yaml
```
![image](https://github.com/user-attachments/assets/f2674105-6b52-4ce3-82cc-0ce8b589d0bd)

### 4. Создание отдельного Pod с multitool и проверка доступа

Создан файл `multitool-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool-pod
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool:latest
```

Применён манифест:

```bash
kubectl apply -f multitool-pod.yaml
```
![image](https://github.com/user-attachments/assets/95e46218-f3cb-4e2f-b0bf-936a52495389)

Проверен доступ к сервису из пода:

```bash
kubectl exec -it multitool-pod -- curl http://nginx-multitool-service
```
![image](https://github.com/user-attachments/assets/b7b34373-9fd2-4852-8367-bc1bf49a34a6)

**Результат:**
- Успешный доступ к сервису `nginx-multitool-service` из пода `multitool-pod`.

---

## Задание 2. Создание Deployment с Init-контейнером

### 1. Создание Deployment с Init-контейнером

Создан файл `init-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-init-deploy
spec:
  selector:
    matchLabels:
      app: nginx-init
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-init
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
      initContainers:
      - name: delay
        image: busybox
        command: ['sh', '-c', "until nslookup nginx-init-svc.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for nginx-init-svc; sleep 2; done"]
```

Применён манифест:

```bash
kubectl apply -f init-deployment.yaml
```
![image](https://github.com/user-attachments/assets/5d5ca959-4dde-4c99-8309-4a30d4fba475)

### 2. Создание и запуск Service

Создан файл `init-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-init-svc
spec:
  ports:
    - name: nginx-init
      port: 80
  selector:
    app: nginx-init
```

Применён манифест:

```bash
kubectl apply -f init-service.yaml
```

### 3. Проверка состояния пода до и после запуска сервиса

Проверено состояние пода до запуска сервиса:

```bash
kubectl get pods
```

**Результат:**
- Пода находился в состоянии `Init:0/1`, так как сервис ещё не был создан.
- 
![image](https://github.com/user-attachments/assets/eabd8ea7-bd25-4dee-ae81-48239463ecc2)

После запуска сервиса проверено состояние пода:

```bash
kubectl get pods
```
![image](https://github.com/user-attachments/assets/dadab5ea-d04e-430a-84f8-1783f79313d3)

**Результат:**
- Init-контейнер завершился успешно, и основной контейнер `nginx` запустился.

---

## Заключение

В ходе выполнения задания:
1. Успешно создан Deployment с двумя контейнерами (nginx и multitool), масштабирован до 2 реплик и обеспечен доступ к репликам через Service.
2. Создан Deployment с Init-контейнером, который обеспечивает запуск основного контейнера только после старта сервиса.

