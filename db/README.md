```sh
#kubectl apply -f secret.yaml
kubectl apply -f configmap.yaml
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

```sh
k get po
```

```sh
k logs -f postgres-7c6b976c95-wjls2
```

```
PostgreSQL Database directory appears to contain a database; Skipping initialization

2023-05-22 02:45:53.640 UTC [1] LOG:  starting PostgreSQL 15.1 (Debian 15.1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2023-05-22 02:45:53.641 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-05-22 02:45:53.641 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2023-05-22 02:45:53.646 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-05-22 02:45:53.653 UTC [27] LOG:  database system was shut down at 2023-05-22 02:40:23 UTC
2023-05-22 02:45:53.659 UTC [1] LOG:  database system is ready to accept connections
```

Connect

```sh
export POSTGRES_PASSWORD=$(kubectl get cm --namespace default db-credentials -o jsonpath="{.data.POSTGRES_PASSWORD}")

kubectl run psql-client --rm --tty -i --restart='Never' --namespace default --image postgres:15.3 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres -U app1 -d app_db -p 5432
```

Port Forward

```sh
kubectl port-forward --namespace default svc/postgres 45432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U app1 -d app_db -p 5432
```


# Migrate

```sh
export DB_HOST=$(kubectl get secret --namespace default postgres-secret-config -o jsonpath="{.data.host}" | base64 -d)
export DB_PORT=$( kubectl get secret --namespace default postgres-secret-config -o jsonpath="{.data.port}" | base64 -d)
export DB_NAME=$( kubectl get secret --namespace default postgres-secret-config -o jsonpath="{.data.name}" | base64 -d)
export DB_USER=$( kubectl get secret --namespace default postgres-secret-config -o jsonpath="{.data.user}" | base64 -d)
export DB_PASS=$( kubectl get secret --namespace default postgres-secret-config -o jsonpath="{.data.password}" | base64 -d)


k apply -f migrate.yaml
k logs -f migrate
k delete po migrate
```

Delete (in order)

```sh
k delete svc postgres
k delete deploy postgres
k delete pvc postgres-pv-claim
k delete pv postgres-pv-volume
k delete cm db-credentials
```