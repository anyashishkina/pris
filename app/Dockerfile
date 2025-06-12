FROM python:3.9-slim
WORKDIR /app
COPY app.py /app
RUN pip install flask
CMD ["python", "app.py"]

eval $(minikube docker-env)
docker build -t my-app .

