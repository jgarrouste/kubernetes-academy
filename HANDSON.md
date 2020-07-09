# Table of contents

* **[Preparation](#preparation)**
* **[First steps](#first-steps)**
* **[Namespaces](#namespaces)**
  * **[Your own namespace](#important-create-your-own-namespace)**
* **[Persisting Data](#persisting-data)**
* **[Health Checks](#health-checks)**

# Preparation

1. Visit the GCP Console using an **incognito window** by going to
https://console.cloud.google.com/kubernetes/list?folder=&organizationId=&project=wizeline-academy-k8s-36bd66a7
2. Log in using the *Google Account* that you provided on the Wizeline
Academy registration form.
3. Click on the *Connect* button that appears besides the
`gke-academy-1` cluster.
4. Next hit the *Run in Cloud Shell* button to get the command that
connects to the K8s Cluster.
5. Hit enter on the `gcloud` command to execute it.
6. Send the *Cloud Shell* to a new window by hitting the button with
the little arrow.
7. Open the *Cloud Shell Editor* by clicking on the little pencil icon,
we'll use the editor to create the YAML files.

# First steps

## Inspecting the Cluster

```bash
kubectl get componentstatuses

kubectl get nodes
```

# Namespaces

## IMPORTANT: Create your own namespace.

The format suggested to use for the namespace name is
`firstName-lastName`, i.e juan-perez.

```bash
# Create your own namespace
kubectl create namespace <firstName-lastName>

# IMPORTANT: Set the namespace in the current context
kubectl config set-context --current --namespace=<firstName-lastName>
```

## Namespaces explained

```bash
# List the namespaces
kubectl get namespaces
```

The students will see all the namespaces from the other participants.
The resources each participant create will be created
inside their own namespace, collisions will not happen
no matter if their resources are named the same.

One common example about how namespaces are used in real
world scenarios is to separate application environments, like
`development` and `production`.

# Persisting Data

## Create the Pod manifest file

Google Cloud Shell Editor can be used to easily create the file.

```yaml
# Using the Google Cloud Shell Editor
# Create the file nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

```bash
# Create the pod using the manifest file
kubectl create -f nginx-pod.yaml

# List pods
kubectl get pods

# See the pod's detailed information
kubectl describe pod nginx-pod

# Delete the pod
kubectl delete pod nginx-pod
```


# Health Checks

```bash
# Create a health-check namespace
kubectl create namespace health-check
```

## Secrets

```bash
# Create user mysql secret to health check files
kubectl create secret generic user-secret --from-literal=mysql-user='root' -n health-check

# Create password mysql secret to health check files
kubectl create secret generic password-secret --from-literal=mysql-root-password='root' -n health-check

# Delete user mysql secrets
kubectl delete secret user-secret -n health-check

# Delete password mysql secrets
kubectl delete secret password-secret -n health-check
```

## Liveness

[mysql-liveness.yaml](lesson02/mysql-liveness.yaml)

```bash
# Apply liveness yaml file
kubectl apply -f mysql-liveness.yaml

# Get pod name
kubectl get pods -n health-check

# Validate liveness health-check
kubectl describe pod wordpress-mysql-liveness-<id> -n health-check
```

### Events output

No problem with the pod.
![Events Output](lesson02/k8s_pod_event_1.png)

```bash
# Break the liveness probe
kubectl exec -n health-check wordpress-mysql-liveness-<id> -c mysql -- mv /usr/bin/mysqladmin /usr/bin/mysqladmin.off
```

### Events output

Liveness probe failed and restart the mysql container.
![Events Output](lesson02/k8s_pod_event_2.png)

```bash
# Validate health-check
kubectl get pods -n health-check
# Delete deployment
kubectl delete -f mysql-liveness.yaml
```

## Readiness

[mysql-readiness.yaml](lesson02/mysql-readiness.yaml)

```bash
# Apply readiness yaml file
kubectl apply -f mysql-readiness.yaml

# Get pod name
kubectl get pods -n health-check

# Validate liveness health-check
kubectl describe pod wordpress-mysql-readiness-<id> -n health-check
```

### Events output

No problem with the pod.
![Events Output](lesson02/k8s_pod_event_3.png)


```bash
# Break the readiness probe
kubectl exec -n health-check wordpress-mysql-readiness-<id> -c mysql -- mv /usr/bin/mysqladmin /usr/bin/mysqladmin.off
```

### Events output

Readiness probe failed.
![Events Output](lesson02/k8s_pod_event_4.png)

```bash
# Repair the readiness probe
kubectl exec -n health-check <Pod name> -c mysql -- mv /usr/bin/mysqladmin.off /usr/bin/mysqladmin
```

```bash
# Validate health-check
kubectl get pods -n health-check
# Delete deployment
kubectl delete -f mysql-readiness.yaml
```



# Clean up

Type `exit` on your *Cloud Shell* session.

Close your incognito browser window.
