# pris

# Задание №7 — Kubernetes (k8s)

## Цель

Ознакомиться с Kubernetes и развернуть сервис с использованием Minikube. Настроить мониторинг нагрузки с помощью Prometheus и Grafana.

[Видео с демонстрацией работы](https://disk.yandex.ru/i/huqWbwcaBDvEgw)

---

## Ход работы

### 1. Развёртывание Minikube

Установка через Homebrew:

```
brew install minikube
```

Запуск:

```
minikube start --driver=docker
```

---

### 2. Создание Docker-образа приложения

**app.py**:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Minikube!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**Dockerfile**:

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app.py /app
RUN pip install flask
CMD ["python", "app.py"]
```

Сборка образа:

```bash
eval $(minikube docker-env)
docker build -t my-app .
docker images
```

---

### 3. Создание Deployment

**deployment.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app
        imagePullPolicy: Never
        ports:
        - containerPort: 5000
```

Применение:

```bash
kubectl apply -f deployment.yaml
kubectl get pods -w
```

Создание сервиса:

```bash
kubectl expose deployment my-app --type=NodePort --port=5000
minikube service my-app
```

---

### 4. Установка Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Дополнительный файл:

**metrics-server-service.yaml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    k8s-app: metrics-server
  ports:
    - port: 443
      targetPort: 4443
```

```bash
kubectl apply -f metrics-server-service.yaml
```

---

### 5. Horizontal Pod Autoscaler (HPA)

```bash
kubectl autoscale deployment my-app --cpu-percent=50 --min=2 --max=5
kubectl get hpa
```

**load-generator.yaml**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-generator
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do wget -q -O- http://my-app:5000; done"]
```

```bash
kubectl apply -f load-generator.yaml
```

---

### 6. Prometheus + Grafana через Helm

```bash
brew install helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

**values.yaml**:

```yaml
prometheus:
  prometheusSpec:
    maximumStartupDurationSeconds: 300
```

```bash
helm install prometheus prometheus-community/kube-prometheus-stack -f values.yaml
```

Доступ к Grafana:

```bash
minikube service prometheus-grafana
```

---

### 7. Grafana Dashboard

- Вход:

  - Логин: `admin`
  - Пароль: `prom-operator`

- Открыть дашборд:

  - Dashboards → Kubernetes / Compute Resources / Workloads

- Выбрать workload: `my-app`

---

## Вывод

Удалось развернуть сервис в Kubernetes с использованием Minikube, настроить HPA и визуализировать нагрузку через Grafana.

**Получены навыки:**

- работа с `kubectl`, `helm`, `autoscale`
- управление подами и деплойментами
- визуализация метрик и нагрузки

