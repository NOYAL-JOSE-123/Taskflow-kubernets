# TaskFlow — Docker + Kubernetes Assignment

A responsive task manager served by Nginx. Tasks are stored in browser local storage, so this demo needs no database. Browser storage is per browser/device and is not shared between Kubernetes replicas.

## Requirements (Windows)
- Docker Desktop using Linux containers
- `kubectl`
- Minikube

Official setup guides:
- Docker Desktop: https://docs.docker.com/desktop/setup/install/windows-install/
- Minikube Docker driver: https://minikube.sigs.k8s.io/docs/drivers/docker/
- kubectl install: https://kubernetes.io/docs/tasks/tools/

## 1. Build and test with Docker
Open PowerShell in this project directory:

```powershell
docker build -t taskflow:1.0 .
docker run --name taskflow-test -p 8080:80 -d taskflow:1.0
```

Open http://localhost:8080. Test adding/completing/deleting a task, then stop and remove the test container:

```powershell
docker ps
docker logs taskflow-test
docker stop taskflow-test
docker rm taskflow-test
```

## 2. Start Kubernetes
Make sure Docker Desktop is running, then:

```powershell
minikube start --driver=docker
kubectl get nodes
```

Load the locally built image into Minikube:

```powershell
minikube image load taskflow:1.0
```

## 3. Deploy
From the project root:

```powershell
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl get services
```

Open the app in a browser. Keep the terminal running while using the URL returned:

```powershell
minikube service taskflow --url
```

On some Windows setups, this command prints a URL and keeps the process running; leave that terminal open.

## 4. Demonstrate management
```powershell
kubectl describe deployment taskflow
kubectl logs deployment/taskflow
kubectl get pods
kubectl scale deployment/taskflow --replicas=3
kubectl get pods
kubectl scale deployment/taskflow --replicas=2
kubectl rollout status deployment/taskflow
```

To demonstrate a rolling update after rebuilding a changed version:
```powershell
docker build -t taskflow:1.1 .
minikube image load taskflow:1.1
# Change image: taskflow:1.0 -> taskflow:1.1 in k8s/deployment.yaml
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deployment/taskflow
kubectl rollout history deployment/taskflow
```

To clean up:
```powershell
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
minikube stop
```

## What to demonstrate in a viva
1. `docker build`: creates an image from the Dockerfile.
2. `docker run`: runs a standalone container and maps host port 8080 to container port 80.
3. Kubernetes Deployment: declares two desired replicas and replaces failed Pods.
4. Service: exposes the Pods through a stable endpoint and NodePort.
5. Readiness/liveness probes: Kubernetes checks whether the web server is ready and healthy.
6. Scaling: `kubectl scale` changes the desired replica count.
7. Logs/describe: inspect runtime output and configuration.

## Important limitation
This is a local demonstration, not a public cloud deployment. Task data is stored in each browser's local storage, not in a shared backend/database.
