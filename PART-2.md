# PART 2

This is th README regarding the solution to the second part of the challenge for the Red Hat Senior Quality Engineer position. 

## Debezium operator

In the same way as in the first part, to clone the repository we need to run one of the following commands:

```bash
git clone https://github.com/debezium/debezium-operator.git
```

```bash
gh repo clone debezium/debezium-operator
```

```bash
git clone git@github.com:debezium/debezium-operator.git
```

## Run local Openshit/Kubernetes cluster

In my case, I did not have any local cluster installation. In order to install all the needed dependencies (MacOs environment), this are the steps to be done (brew needs to be installed):

```bash
brew install docker minikube kubectl
```

Once this finishes, we have everything installed but not configured. One last step is needed to make it run:

```bash
minikube start
```

This will create a docker container with minikube and will configure your kubectl tool to point to it. When this is ready, we can continue to the next step. 

## PostgreSQL example deployment

Going to the `examples/postgres` directory we will find all the resources involved in the example (postgres, kafka and debezium-server). But before we can deploy those files, we need to load the custom resources that can be found in `k8` folder and kafka deployment. To achieve that, the following command needs to be run: 

```bash
kubectl create ns debezium
kubectl create -f k8/ -n debezium
kubectl create -f "https://strimzi.io/install/latest?namespace=debezium" -n debezium
```

In my case I am working in the debezium namespace, but this can work within any desired namespace. In order to check that everything is running fine run `kubectl get all -n debezium` and you should see something like this:

```
➜  debezium-operator git:(main) kubectl get all -n debezium
NAME                                            READY   STATUS    RESTARTS   AGE
pod/debezium-operator-5f7649b665-c9s4w          1/1     Running   0          79s
pod/strimzi-cluster-operator-6f9fbb4c75-b5cxm   1/1     Running   0          58s

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/debezium-operator   ClusterIP   10.100.82.41   <none>        80/TCP    79s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/debezium-operator          1/1     1            1           79s
deployment.apps/strimzi-cluster-operator   1/1     1            1           58s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/debezium-operator-5f7649b665          1         1         1       79s
replicaset.apps/strimzi-cluster-operator-6f9fbb4c75   1         1         1       58s

```

If your output looks similar to it, you can now try to deploy the desired components. To achieve that, you have the run this command:

```bash
kubectl create -f examples/postgres/ -n debezium
```

If the command output is successfully, this will complete all the deployment process.

### Note

At this point I have everything deployed and running except the debezium pod which is getting a `CrashLoopBackOff`. Checking the logs you can see that Quarkus is failling to initialize because the pod cannot connect to the kafka deployment. Seems to be a problem related on how i did create the cluster and/or kafka deployment/service. 

At this point I have invested more time than recommended (the two hours that Jiri said). Since I've already exceeded that limit a bit, I don't want to exceed it too much. 

To investigate this problem I looked at the description and events of the deployment and pod to see what was happening. To solve the problem, I would start with changing the form of the current kafka deployment: instead of using the strimzi CRD I would use a deployment based on the official kafka image. And in case of need, I would also add a service to make it available.

## Grab Debezium server pod logs

The last step is the most simple one. Just grabbing the logs from the deployment with the following command:

```bash
kubectl logs pod/my-debezium-{replicaId}-{podId} -n debezium | tee debezium-server.logs
```

The given logs can be foud [here](./logs/part2/debezium-server.logs).
