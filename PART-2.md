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

### Note: 

According to strimzi.io documentation 4GB memory is recommended to run kafka. So this is the correct command to start minikube:

```bash
minikube start --memory=4096
```

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

### Note 2

After the problem face the past day I have found the bug. Running the repo as it is, the kafka deployment (coming from the CRD) does not get up and running. Debugging into and running this command `kubectl -n debezium describe k dbz-kafka`, I found this message:

```
Message: Unsupported Kafka.spec.kafka.version: 3.4.0. Supported versions are: [3.7.0, 3.7.1, 3.8.0]
```

So going to the [line 24 of the 002_kafka-ephemeral.yaml file](https://github.com/debezium/debezium-operator/blob/7f95edaaea3146b361603dccd6fb22aecdcd187e/examples/postgres/002_kafka-ephemeral.yml#L24) and changing that `3.4` with `3.8` solved the problem and now all the deployment, replicas, pods and services are up and running:

```
➜  debezium-operator git:(main) kubectl get all -n debezium
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dbz-kafka-entity-operator-56f9886c78-hs4j2   2/2     Running   0          6m13s
pod/dbz-kafka-kafka-0                            1/1     Running   0          6m39s
pod/dbz-kafka-zookeeper-0                        1/1     Running   0          7m23s
pod/debezium-operator-5f7649b665-k6j7w           1/1     Running   0          10m
pod/my-debezium-559d8cd5f6-r8qrm                 1/1     Running   0          5m21s
pod/postgresql-7745b945fb-75kwf                  1/1     Running   0          6m31s
pod/strimzi-cluster-operator-6f9fbb4c75-lchkg    1/1     Running   0          10m

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                        AGE
service/dbz-kafka-kafka-bootstrap    ClusterIP   10.99.113.156    <none>        9091/TCP,9092/TCP,9093/TCP                     6m40s
service/dbz-kafka-kafka-brokers      ClusterIP   None             <none>        9090/TCP,9091/TCP,8443/TCP,9092/TCP,9093/TCP   6m40s
service/dbz-kafka-zookeeper-client   ClusterIP   10.100.217.231   <none>        2181/TCP                                       7m24s
service/dbz-kafka-zookeeper-nodes    ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP                     7m24s
service/debezium-operator            ClusterIP   10.105.27.114    <none>        80/TCP                                         10m
service/my-debezium-api              ClusterIP   10.100.225.176   <none>        8080/TCP                                       4m30s
service/postgresql                   ClusterIP   10.105.79.161    <none>        5432/TCP                                       6m31s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dbz-kafka-entity-operator   1/1     1            1           6m13s
deployment.apps/debezium-operator           1/1     1            1           10m
deployment.apps/my-debezium                 1/1     1            1           5m21s
deployment.apps/postgresql                  1/1     1            1           6m31s
deployment.apps/strimzi-cluster-operator    1/1     1            1           10m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dbz-kafka-entity-operator-56f9886c78   1         1         1       6m13s
replicaset.apps/debezium-operator-5f7649b665           1         1         1       10m
replicaset.apps/my-debezium-559d8cd5f6                 1         1         1       5m21s
replicaset.apps/postgresql-7745b945fb                  1         1         1       6m31s
replicaset.apps/strimzi-cluster-operator-6f9fbb4c75    1         1         1       10m
```

## Grab Debezium server pod logs

The last step is the most simple one. Just grabbing the logs from the deployment with the following command:

```bash
kubectl logs pod/my-debezium-{replicaId}-{podId} -n debezium | tee debezium-server.logs
```

The given logs can be foud [here](./logs/part2/debezium-server.logs).

### Note 

After solving the problem I have run again the command above and left the logs in [this file](./logs/part2/debezium-server-take-2.logs). 
