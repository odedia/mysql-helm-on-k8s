# MySQL helm chart on Kubernetes
This README describes how to install MySQL on a kubernetes cluster, based on the Bitnami Helm chart. 

## Install helm
Make sure you have helm installed.

For Mac with Homebrew, run the following:

```bash
brew install kubernetes-helm
```

For Windows with Chocolatey

```bash
choco install kubernetes-helm
```

For linux with snap:

```bash
sudo snap install helm --classic
```

Or simply run a script:

```bash
curl -L https://git.io/get_helm.sh | bash
```

## Setup tiller
Tiller is the server-side component for helm on the Kubernetes cluster. Although it will be removed in the future versions of helm, it is still needed for most Charts out there.

Apply the included RBAC yaml file for tiller:

```bash
kubectl apply -f initialize_helm_rbac.yaml
```

## Install Bitnami MySQL

For the simplest installaton, just run:

```bash
helm install --name todo-mysql stable/mysql
```

helm will output a bunch of information about the installation:

```bash
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
todo-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default todo-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h todo-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/todo-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

If you ever want to see this information again, simply run `helm status`:

```bash
(⎈ |kubecon:default)➜  ~ helm ls
NAME          	REVISION	UPDATED                 	STATUS  	CHART                   	APP VERSION	NAMESPACE
todo-mysql    	1       	Wed Jun 19 10:26:53 2019	DEPLOYED	mysql-0.15.0            	5.7.14     	default  
(⎈ |kubecon:default)➜  ~ helm status todo-mysql
LAST DEPLOYED: Wed Jun 19 10:26:53 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME             DATA  AGE
todo-mysql-test  1     99s

==> v1/PersistentVolumeClaim
NAME        STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGE
todo-mysql  Bound   pvc-9ca7a803-9263-11e9-a736-42010a000c34  8Gi       RWO           standard      99s

==> v1/Pod(related)
NAME                         READY  STATUS   RESTARTS  AGE
todo-mysql-74c985f54b-nqdln  1/1    Running  0         99s

==> v1/Secret
NAME        TYPE    DATA  AGE
todo-mysql  Opaque  2     99s

==> v1/Service
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
todo-mysql  ClusterIP  10.100.200.251  <none>       3306/TCP  99s

==> v1beta1/Deployment
NAME        READY  UP-TO-DATE  AVAILABLE  AGE
todo-mysql  1/1    1           1          99s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
todo-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default todo-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h todo-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/todo-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

A DNS entry was created for this installation under `todo-mysql.default.svc.cluster.local` on port `3306`. This is only accessible from within the kubernetes cluster. Although it's a good security measure, you cannot access this database from other platforms such as Pivotal Cloud Foundry.

## Overriding parameters

There are many parameters that you can override for the installation. Most helm charts describe these parameters in their documentation. Although it is possible to clone the helm chart repository and edit the `values.yaml` file with your new parameters, it is often a lot simpler to just pass these parameters as part of your `helm install` command.

The list of available configuration parametersfor Bitnami MysQL is available here: https://github.com/helm/charts/tree/master/stable/mysql#configuration

Let's override the `imagePullPolicy` from its default `IfNotPresent` to `Always`

First, let's delete the currentl installation:

```bash
helm del --purge todo-mysql
```

Give it around 10 seconds to purge all existing resources. Then, install MySQL again with the overriden parameter:

```bash
helm install --name todo-mysql \
  --set imagePullPolicy=Always \
    stable/mysql
```

You can also create a yaml file with the overriden parameters. The included `values.yaml` overrides the `imagePullPolicy` as well.

```bash
helm install --name todo-mysql -f values.yaml stable/mysql
```
