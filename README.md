## Install minikube

To install minikube, follow the instructions [here](https://kubernetes.io/docs/tasks/tools/install-minikube/).

Additionally, we will need to activate the metrics-server.

`minikube addons enable metrics-server`

## Initialize a watch mode

In a separate terminal window apply the following command:

`watch kubectl get svc,deployment,pods`

Now you should be able to controll the current status of your deployment.

## Deploy a basic app

Take a look at a basic manifest in the ./01_basic folder of this project. The basic type of resource, the deployment, 
is defined there. Open a new terminal window. We will run several commands to get us up and running.  

### Apply yaml

You can spin up a cluster by running the following command:

`kubectl apply -f ./01_basic/deployment.yaml`

### Expose deployment (create service)

Now we can create a service through which our nginx instance will be available for the world 
(for now without load balancer).

`kubectl expose deployment nginx-deployment --type=NodePort`

### Get the url

To get the actual url, run the following:

`minikube service nginx-deployment --url`

Now you can open our nginx server in your browser.

## Scaling

### Scaling manually

The simpliest way to scale up is to increase the replica set manually as following:

`kubectl scale --replicas=4 deployment/nginx-deployment`

Now we need to define a Load Balancer so we could access our replica set as a service:

`kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80 --target-port=80 --name nginx-load-balancer`
`kubectl describe service nginx-load-balancer`

Services of type LoadBalancer can be exposed via the `minikube tunnel` command on your local machine.
It will run in a separate terminal until Ctrl-C is hit.

### Autoscaling

The scaling is closely related to the resource limit. Let's set one as following:

`kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=100m,memory=64Mi`

As the limits are set, we can now define a autoscaling policy:

`kubectl autoscale deployment.v1.apps/nginx-deployment --min=2 --max=15 --cpu-percent=60`

Now let's make the same load test to see how many requests we're giong to process, 
and how the k8s will handle the increased load with autoscaling.

With Apache Bench we now make:
* 1.000.000 requests
* with 100 concurent threads

`ab -n 1000000 -c 100 http://<service-ip>/`

You can further inspect your autoscaler with the following commands:

`kubectl get hpa`
`kubectl delete hpa nginx-deployment`

You can also allpy custom limits on resources. Take a look at the manifest `./02_autoscaling/hpa-v2.yaml` for how
you can set limits on the ammount of incomming requests per pod. 

### Cleaning up resources

Cleaning up all the resources we've created so far:

`kubectl delete --all deployment,svc,pods,hpa`

Cleaning up orphaned routes:

If the minikube tunnel shuts down in an abrupt manner, it may leave orphaned network 
routes on your system. If this happens, the ~/.minikube/tunnels.json file will 
contain an entry for that tunnel. To remove orphaned routes, run:

`minikube tunnel --cleanup`

## Rolling updates

Run the basic deployment again: 

`kubectl apply -f ./01_Basic/deployment.yaml`

Set the number of replicas to 10:

`kubectl scale --replicas=10 deployment/nginx-deployment`

And axpose a service:

`kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80 --target-port=80 --name nginx-load-balancer`

You should see the following status in your watch mode:

[img](images/10_repl.png)

Let's change the image to `nginx:latest` while the cluster is running:

`kubectl set image deployment/nginx-deployment nginx=nginx:latest`

The rolling update spin up, but without any downtime on the cluster:

[img](images/rolling_update.png)

Now you can access the rollout history:

`kubectl rollout history deployment/nginx-deployment`,

iterate over rollout revisions:

`kubectl rollout history deployment/nginx-deployment --revision=2`,

and, most importantly, roll back:

`kubectl rollout undo deployment.v1.apps/nginx-deployment`.
