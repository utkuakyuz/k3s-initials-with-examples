# Bağımsız Çalışan Node ve PostgreSql Uygulamaları

# STEPS TO REPRODUCE

Create Project Directory

```bash
mkdir -p my-k3s-postgresql
cd my-k3s-postgresql
mkdir -p manifests
```

In Manifests folder, create postgresql.yaml file

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_DB
              value: mydatabase
            - name: POSTGRES_USER
              value: myuser
            - name: POSTGRES_PASSWORD
              value: mypassword
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          emptyDir: {}
```

To Apply Manifest File and Start the Postgre

```bash
kubectl apply -f manifests/postgresql.yaml
```

Check the pods

```bash
kubectl get pods
```
