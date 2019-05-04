# Kubernetes Spring Boot Example in Google Kubernetes Engine (GKE)

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

