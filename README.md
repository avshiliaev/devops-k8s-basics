### Create yaml

_deployment.yaml_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
        - name: tomcat
          image: tomcat:9.0
          ports:
            - containerPort: 8080
````

### Apply yaml

`kubectl apply -f ./deployment.yaml`

### Expose deployment (create service)

`kubectl expose deployment tomcat-deployment --type=NodePort`

### Get the url

`minikube service tomcat-deployment --url`

