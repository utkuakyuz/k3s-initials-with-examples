# Postgres ve Node Bağlantısı Kurmak

Postgres ve Node uygulamalarının ENV dosyalarını okuduğundan emin olunur.

Gerekli Local Docker Image’i kaydedilir

```bash
docker save nodeapp4:latest -o nodeapp4.tar
```

Kaydedilen Docker Image’i k3s containerd içine yüklenir.

```bash
sudo k3s ctr images import nodeapp4.tar
sudo k3s ctr images list | grep nodeapp4
```

# postgres.yaml

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
              value: postgres
            - name: POSTGRES_USER
              value: postgres
            - name: POSTGRES_PASSWORD
              value: "123456"
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          emptyDir: {}
```

# nodeapp4.yaml

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
  replicas: 1 # Adjust replicas as needed
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
          env:
            - name: DATABASE_URL
              value: "postgres://postgres:123456@postgres:5432/postgres" # Update with your actual connection string # "postgres://myuser:mypassword@postgres:5432/mydatabase"
```

## Node uygulaması dosyası içerisinde DB bağlantısı korunur

```
const pool = new Pool({
  user: "postgres",
  host: "postgres",
  database: "postgres",
  password: "123456",
  port: "5432",
});
```

# BAĞLANTI KONTROLLERİ SAĞLANIR

POD’un çalıştığını gözlemlemek için

```bash
kubectl get pods
```

Log kontrolünde Tablonun oluştuğu görülür

```yaml
kubectl logs <pod-name>
```
