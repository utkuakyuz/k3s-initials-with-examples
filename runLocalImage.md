# Local Ortamdaki Image K3S ile Nasıl Çalıştırılır?

In terminal, save your docker image to your machine

```bash
docker save nodeapp4:latest -o nodeapp4.tar
```

And then load the docker image into k3s containerd (2. line is to check image loaded correctly)

```bash
sudo k3s ctr images import nodeapp4.tar
sudo k3s ctr images list | grep nodeapp4
```

If there is no error, create following yaml file in the manifests folder. (nodeapp4.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeapp4
spec:
  selector:
    app: nodeapp4
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeapp4
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodeapp4
  template:
    metadata:
      labels:
        app: nodeapp4
    spec:
      containers:
        - name: nodeapp4
          image: nodeapp4:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

Apply the manifest file and check pods

```yaml
kubectl apply -f manifests/nodeapp4.yaml
kubectl get pods
```
