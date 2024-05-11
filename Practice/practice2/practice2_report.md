```
University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Application containerization and orchestration](https://github.com/itmo-ict-faculty/application-containerization-and-orchestration)
Year: 2023/2024
Group: K4113c
Author: Karaulov Andrei Olegovich
Practice: practice2
Date of create: 11.05.2024
Date of finished: 11.05.2024
```

1. Создадим namespace
```bash
➜  ~ kubectl create ns postgresql
namespace/postgresql created
```
2. Создадим креды в виде secret
```bash
kubectl -n postgresql create secret generic postgresql --from-literal POSTGRES_USER="admin" --from-literal POSTGRES_PASSWORD="admin" --from-literal POSTGRES_DB="postgres"
```
```bash
➜  ~ kubectl get secret postgresql -n postgresql -o json | jq -r '.data | map_values(@base64d)'

{
  "POSTGRES_DB": "postgres",
  "POSTGRES_PASSWORD": "admin",
  "POSTGRES_USER": "admin"
}
```

3. Создадим Stateful Set
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: postgresql
spec:
  serviceName: "postgres"
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
          image: "postgres:16"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgresql
                  key: POSTGRES_USER
                  optional: false
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgresql
                  key: POSTGRES_PASSWORD
                  optional: false
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: postgresql
                  key: POSTGRES_DB
                  optional: false
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgresdata
  volumeClaimTemplates:
    - metadata:
        name: postgresdata
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi

```
4. Запустим
```bash
➜  practice2 git:(main) ✗ kubectl apply -f psql-statefulset.yml 
statefulset.apps/postgres created
```
```bash
➜  practice2 git:(main) ✗ kubectl get pods -n postgresql
NAME         READY   STATUS    RESTARTS   AGE
postgres-0   1/1     Running   0          78s
```
```bash
➜  practice2 git:(main) ✗ kubectl -n postgresql logs postgres-0

The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.
....

PostgreSQL init process complete; ready for start up.

2024-05-11 12:22:13.612 UTC [1] LOG:  starting PostgreSQL 16.3 (Debian 16.3-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
2024-05-11 12:22:13.612 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2024-05-11 12:22:13.612 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2024-05-11 12:22:13.615 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2024-05-11 12:22:13.619 UTC [63] LOG:  database system was shut down at 2024-05-11 12:22:13 UTC
2024-05-11 12:22:13.624 UTC [1] LOG:  database system is ready to accept connections
```

5. Зайдем в psql
```bash
➜  practice2 git:(main) ✗ kubectl -n postgresql exec -it postgres-0 -- psql --username=admin postgres
psql (16.3 (Debian 16.3-1.pgdg120+1))
Type "help" for help.

postgres=# 
```

6. Создадим таблицу
```bash
postgres=# CREATE TABLE users (id serial, name text);
CREATE TABLE
postgres=# \dt
       List of relations
 Schema | Name  | Type  | Owner 
--------+-------+-------+-------
 public | users | table | admin
(1 row)
```

7. Выйдем и удалим наш под
```bash
➜  practice2 git:(main) ✗ kubectl -n postgresql delete po postgres-0
pod "postgres-0" deleted
```
8. Зайдем во вновь автоматически созданный под и проверим созданную таблицу
```bash
➜  practice2 git:(main) ✗ kubectl get pods -n postgresql            
NAME         READY   STATUS    RESTARTS   AGE
postgres-0   1/1     Running   0          13s
➜  practice2 git:(main) ✗ kubectl -n postgresql exec -it postgres-0 -- psql --username=admin postgres
psql (16.3 (Debian 16.3-1.pgdg120+1))
Type "help" for help.

postgres=# \dt
       List of relations
 Schema | Name  | Type  | Owner 
--------+-------+-------+-------
 public | users | table | admin
(1 row)
```