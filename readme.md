# Welcome!

This is a very compact k3s repository (not for a project but for training) for myself to understand core and some advanced concepts of Kubernetes. I have completed several achievements of mine and wanted to publish them so that I can reach easily, and maybe it could help someone. Please contact me if you go through my steps and had an issue.

# K3S MASTER!

This repository includes following applications.

[Independently running Node and PostgreSQL DB](/independentApps.md)

[Running a local Docker image in K3S cluster](/runLocalImage.md)

[Connection of Node and PostgreSQL](postgresNodeConnection.md)

[COREDNS CUSTOM DOMAIN](coreDnsCustomDomain.md) (Which is my favourite)

### Useful Links

[https://github.com/alperen-selcuk/kubectl-cheat-sheet](https://github.com/alperen-selcuk/kubectl-cheat-sheet)

https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

https://learn.microsoft.com/en-us/azure/aks/coredns-custom

# Prerequisites

K3S, DOCKER and Kubectl must be installed

> `curl -sfL https://get.k3s.io | sh -`

# Ana Komutlar

To Apply Manifest File and Start the Postgre

```bash
kubectl apply -f manifests/postgresql.yaml
```

To check pod is running

```bash
kubectl get pods
```

To Forward the port in your local machine

```bash
kubectl port-forward service/postgres 5432:5432
```

To delete the postgre pods and manifes

```bash
kubectl delete -f manifests/postgresql.yaml
```

If you want to scale it you can do it in manifest file or following command works.

```yaml
kubectl scale deployment nodeapp4 --replicas=3
```

Also you can describe pods to get more information about the pod

```yaml
kubectl describe pod <pod-name>
```

If you need logs of a pod

```yaml
kubectl logs <pod-name>
```

If you need to instantly run a curl command, you can use following pod

```yaml
kubectl run mycurlpod --image=curlimages/curl -i --tty -- sh

or if pod is already running

kubectl attach mycurlpod -c mycurlpod -i -t
```

Export KubeConfig

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

To Curl a ip address in a loop

```yaml
while true; do curl http://172.16.10.88/index.html && sleep 1; done;
```

To nslookup a service url (do apt get update and apt get dns-utils in corresponding pod) (use service name)

```yaml
kubectl exec -it nginx-deployment-7c5ddbdf54-6rgf2 -- nslookup nginx-lb.default.svc.cluster.local
```

To get kube-system pods

```bash
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS         AGE
coredns-84c4b5f474-858b8                  1/1     Running   0                5m57s
```

### DNS DEBUGGING BASH

```bash
$ kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
If you don't see a command prompt, try pressing enter.
/ # host foo
foo.default.svc.cluster.local has address 10.0.0.72
/ # host foo.example.com
foo.example.com has address 10.0.0.72
/ # host bar.example.com
Host bar.example.com not found: 3(NXDOMAIN)
/ #
```
