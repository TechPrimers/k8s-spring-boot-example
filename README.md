# Kubernetes Spring Boot Example in Google Kubernetes Engine (GKE)

Table of Contents
=================

   * [Kubernetes Spring Boot Example in Google Kubernetes Engine (GKE)](#kubernetes-spring-boot-example-in-google-kubernetes-engine-gke)
   * [Table of Contents](#table-of-contents)
      * [Create Spring Boot app](#create-spring-boot-app)
      * [Create Docker image](#create-docker-image)
      * [Run the Docker image](#run-the-docker-image)
      * [Login to the K8s Cluster](#login-to-the-k8s-cluster)
      * [Kubernetes Commands](#kubernetes-commands)
         * [List Pods](#list-pods)
         * [List Deployments](#list-deployments)
         * [List Services](#list-services)
         * [Deploy an image](#deploy-an-image)
         * [Expose Load Balancer](#expose-load-balancer)
         * [Scale deployments](#scale-deployments)
      * [K8s YAML Creator](#k8s-yaml-creator)
         * [Deployment YML used](#deployment-yml-used)
         * [Service YML used](#service-yml-used)
         * [Commands to Create/Update](#commands-to-createupdate)
         * [Command to retrieve logs](#command-to-retrieve-logs)
      * [Deployment Strategies](#deployment-strategies)
         * [Recreate Strategy](#recreate-strategy)
         * [Rolling Update Strategy](#rolling-update-strategy)
         * [Blue Green Deployment](#blue-green-deployment)
            * [Commands](#commands)
      * [Canary Deployments](#canary-deployments)
         * [Type 1](#type-1)
            * [Commands](#commands-1)
         * [Type 2](#type-2)
            * [Commands](#commands-2)
         
## Create Spring Boot app
You can use the sample project which I have in here.

`git clone https://github.com/TechPrimers/spring-boot-lazy-init-example.git`

## Create Docker image
Command to create docker image using Google JIB plugin

`./mvnw com.google.cloud.tools:jib-maven-plugin:build -Dimage=gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1`

## Run the Docker image
Command to run the docker image which we created in the previous step

`docker run -ti --rm -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1`

## Login to the K8s Cluster
Command to login to the K8s cluster from Cloud Shell

`gcloud container clusters get-credentials techprimer-cluster-1 --zone  us-central1-a`

## Kubernetes Commands
### List Pods
`kubectl get pods`

### List Deployments
`kubectl get deployments`

### List Services
`kubectl get services`

### Deploy an image
`kubectl run spring-boot-example --image=gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-example:v1 --port=8080`

### Expose Load Balancer
`kubectl expose deployment spring-boot-example --type=LoadBalancer`

### Scale deployments
`kubectl scale deployment spring-boot-example --replicas=3`

## K8s YAML Creator
Link to Brandon Potter's YML builder - [https://static.brandonpotter.com/kubernetes/DeploymentBuilder.html](https://static.brandonpotter.com/kubernetes/DeploymentBuilder.html)

### Deployment YML used
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: spring-boot-example
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v1'
          ports:
            - containerPort: 8080
```

### Service YML used
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
  type: LoadBalancer
```

### Commands to Create/Update
- `kubectl apply -f deployment.yml`
- `kubectl apply -f service.yml`

### Command to retrieve logs
`kubectl logs <POD_NAME>`
- Pod Name can be retrived using `kubectl get pods`

## Deployment Strategies
- Recreate
- RollingUpdate
- Blue/Green
- Canary

### Recreate Strategy
kube.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example
spec:
  replicas: 3
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: spring-boot-example
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v1'
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
  type: LoadBalancer
```

### Rolling Update Strategy
kube.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v1'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
  type: LoadBalancer
```

### Blue Green Deployment
- deployment-blue-v1.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v1"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v1'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```
- service-blue-v1.yml
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
    version: "v1"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v1"
  type: LoadBalancer
```

- deployment-green-v2.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v2
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v2"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v2'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```
- service-green-v2.yml
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example-green
  labels:
    name: spring-boot-example-green
    version: "v2"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v2"
  type: LoadBalancer
```
- deployment-blue-v2.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v2
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v2"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v2'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```
- service-blue-v2.yml
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
    version: "v2"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v2"
  type: LoadBalancer
```

#### Commands
- `kubectl apply -f deployment-blue-v1.yml`
- `kubectl apply -f service-blue-v1.yml`
- `kubectl apply -f deployment-green-v2.yml`
- `kubectl apply -f service-green-v2.yml`
- `kubectl apply -f deployment-blue-v2.yml`
- `kubectl apply -f service-blue-v2.yml`
- `kubectl delete deployment.apps/spring-boot-example-v1 service/spring-boot-example-green`

## Canary Deployments
### Type 1 
Using Tags in Kubernetes
- kube-v1.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v1"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v1'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
    version: "v1"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v1"
  type: LoadBalancer
```

- deployment-v2.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v2
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v2"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v2'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```
- Update Service to remove "version" tag.
service-v1.yml
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
  type: LoadBalancer
```
- add "v2" for version in the Service object
service-v2.yml
```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
    version: "v2"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v2"
  type: LoadBalancer
```
#### Commands
- `kubectl apply -f kube-v1.yml`
- `kubectl apply -f deployment-v2.yml`
- `kubectl apply -f service-v1.yml
- `kubectl apply -f service-v2.yml

### Type 2
- Create V3 version along with Service object
kube-v3.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: spring-boot-example-v3
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app: spring-boot-example
        version: "v1"
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/fleet-resolver-237016/spring-boot-example:v3'
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /v3/lazy
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example-v3
  labels:
    name: spring-boot-example
    version: "v3"
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
    version: "v3"
  type: LoadBalancer
```
- Ingress config
ingress.yml
```
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: sb-ingress
spec:
  rules:
  - http:
      paths:
      - path: /lazy/*
        backend:
          serviceName: spring-boot-example
          servicePort: 8080
      - path: /v3/lazy
        backend:
          serviceName: spring-boot-example-v3
          servicePort: 8080
```
- Remove and make default backend rule in ingress
ingress-default.yml
```
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: sb-ingress
spec:
  backend:
    serviceName: spring-boot-example-v3
    servicePort: 8080
```
#### Commands
- `kubectl apply -f kube-v3.yml`
- `kubectl apply -f ingress.yml`
- `kubectl apply -f ingress-default.yml`
