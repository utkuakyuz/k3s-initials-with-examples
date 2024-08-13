# COREDNS CUSTOM DOMAIN

This page shows how to use custom domains in a k3s Kubernetes Cluster

https://learn.microsoft.com/en-us/azure/aks/coredns-custom

Check coredns is running

```yaml
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS         AGE
coredns-84c4b5f474-858b8                  1/1     Running   0                5m57s
```

First create a coredns-custom.yaml, insert any host as you wish.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  default.server: |
    ngnix-lb.ekinoks.biz {
        hosts {
              172.16.10.88 ngnix-lb.ekinoks.biz
              fallthrough
        }
    }
```

Inserted IP address is external IP of the service we created (service file can be found at the end)

```bash
$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      10.43.0.1      <none>         443/TCP        18d
nginx-lb     LoadBalancer   10.43.173.55   172.16.10.88   80:31343/TCP   24h
```

Apply created yaml file and restart coredns in k3s cluster

```bash
$ kubectl apply -f coredns-custom.yaml
$ kubectl -n kube-system rollout restart deployment coredns
```

This will extend coredns default domain names and enable k3s network to resolve custom domains

Create a deployment and deploy some applications. For this example, nginx deployments are used, which you can find at the end of the document.

Install dns-utils into the pod

```bash
$ kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- apt-get update
$ kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- apt-get install dns-utils -y
```

Lastly, just execute nslookup command in the installed pod

```bash
$ kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- nslookup nginx-lb.ekinoks.biz
Server:		10.43.0.10
Address:	10.43.0.10#53

Name:	ngnix-lb.ekinoks.biz
Address: 172.16.10.88
```

Also you can use curl if you wish

```bash
$ kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- curl http:// ngnix-lb.ekinoks.biz
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

Also you can sh into a pod, then start a curl loop to see several pod outputs.

```bash
$ kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- bin/sh
-> @nginx-deploymentPod/$ while true; do curl http://nginx-lb.ekinoks.biz && sleep 1; done;
```

## ADDITIONALS

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  ports:
    - port: 80
  type: LoadBalancer
  selector:
    app: nginx
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec: # specification for deployment
  replicas: 2
  selector:
    matchLabels:
      app: nginx
      # Following lines are blueprint for pods
  template:
    metadata:
      labels:
        app: nginx
    spec: #-> specification for pods
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80 #port 80 of container
```
