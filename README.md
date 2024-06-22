# local-deploy-clearml
## Introduction
As a backend the clearml server service infrastructure for ClearML.  
It allows multiple users to manage  and collaborate experiments. 
In order to host your own server, you will need to install clearml-server and point ClearML agent to it.

## Clearml-server contains the following components:
## Web-App
The ClearML Web-App, is a single-page UI for experiment management and browsing
RESTful API for:
- Querying experiments history, logs and results.
- Documenting and logging experiment information, statistics and results.
- Locally-hosted file server for storing images and models making them accessible using the Web-App.


# Deploy ClearML on Minikube
To deploy ClearML on Minikube and run a pipeline example, we need the following steps. 
These steps include setting up Minikube, deploying ClearML using Helm charts, and then deploying a ClearML pipeline example.

Step 1: Setting up Minikube
Install Minikube: Follow the official Minikube installation guide.

Start Minikube:
``` 
minikube start
```
Set Docker environment to use Minikube’s Docker daemon:
```
eval $(minikube -p minikube docker-env)
```

Step 2: Deploying ClearML on Minikube
ClearML provides a Helm chart for easy deployment. First, ensure you have Helm installed. Follow the official Helm installation guide.

1. Add the ClearML Helm repository:
   ```
   helm repo add allegroai https://allegroai.github.io/clearml-helm-charts/
   helm repo update
   ```
2. Create a namespace for ClearML:
```
kubectl create namespace clearml
```
3.  Install ClearML using Helm:
```
helm install clearml allegroai/clearml --namespace clearml
```
Step 3: Accessing ClearML Dashboard:
1. Get the ClearML web service URL:
```
minikube service -n clearml clearml-web --url
```
2. Access the ClearML Dashboard: Open the provided URL in your web browser.

Step 4: Running ClearML Agent in Kubernetes
1. Create a Kubernetes Deployment for ClearML Agent:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: clearml-agent
  namespace: clearml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: clearml-agent
  template:
    metadata:
      labels:
        app: clearml-agent
    spec:
      containers:
      - name: clearml-agent
        image: allegroai/clearml-agent:latest
        env:
        - name: CLEARML_API_ACCESS_KEY
          value: "<Your ClearML API Access Key>"
        - name: CLEARML_API_SECRET_KEY
          value: "<Your ClearML API Secret Key>"
        - name: CLEARML_API_HOST
          value: "http://<ClearML Web Service URL>"
        command: ["clearml-agent", "daemon"]

```
2. Deploy the ClearML Agent
 ```
      kubectl apply -f clearml-agent-deployment.yaml
 ```
# If ClearML Web Service is Missing
Check svc:
```
kubectl get services -n clearml
```
# Manual Service Creation (If Necessary)
 1. Create a Service YAML File:
```
apiVersion: v1
kind: Service
metadata:
  name: clearml-web
  namespace: clearml
spec:
  type: NodePort
  selector:
    app: clearml-web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001

 ```
2. Apply the Service YAML:
```
kubectl apply -f clearml-web-service.yaml
```

3. Access ClearML Web Service:
```
minikube service -n clearml clearml-web --url
```
